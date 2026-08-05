# WebAI2API — Teil 3: Server-, Config- und Top-Level-Utils-Ebene

Alle Pfade relativ zu `repos/WebAI2API/`. Belegquellen sind die in diesem Dokument zitierten Evidence-Blöcke (Quelle: Dateiinhalt der unter `Commit: content-copy` eingefrorenen Quellen; Hashes aus `docs/inventory/source-checksum.md`). Umfang: 22 Dateien (12 Server, 3 Config, 7 Utils), 5085 Zeilen, alle vollständig gelesen.

Legende für `Typ` der Evidence: Import / Call / Test / Config / API / Event / Schema / Persistenz. Negative Befunde sind als Negativnachweis dokumentiert.

---

## src/server/api/admin/routes.js

- Zweck: Admin-API-Router (HTTP-Handler-Fabrik) für die Verwaltungsoberfläche: Systemstatus, Neustart/Stop, VNC-Status, Cache-Bereinigung, Logs, Datenordner-Verwaltung, Konfiguration (Server/Browser/Instances/Adapters/Pool), Adapter-Metadaten, Statistik, Queue-Status, Request-Historie und statische Medienauslieferung (belegt durch Dateiinhalt, 626 Zeilen).
- Verantwortlichkeit: Bündelt sämtliche `/admin/*`-Endpunkte in einem einzigen asynchronen Request-Handler, der nach Methode und Pfad dispatcht; liest/schreibt Konfiguration ausschließlich über die Config-Manager- und Validator-Schicht und greift für Zustandsdaten auf die Utils-Module (Systeminfo, Statistik, Historie, IPC) und den Queue-Manager zu.
- Eingaben: HTTP-Requests (GET/POST/DELETE) mit JSON-Bodies und Query-Parametern; Konfigurationsdaten; Worker-/Instance-Listen aus der aktuellen Konfiguration; Queue-Zustand; Dateisystem-Zustand.
- Ausgaben: JSON-Antworten (200/207/404/405/500), statische Mediendateien (Bild/Video) mit MIME-Typ und Cache-Header; fehlgeschlagene Admin-Operationen als strukturierte API-Fehler.
- Datenfluss: HTTP-Request → Pfad-/Methoden-Dispatch → Delegation an Config-Manager/Validator, Systeminfo, Statistik, Historie, IPC, Queue-Manager → JSON-/Medien-Response. Neustart-Fluss: `POST /admin/restart` → (a) IPC-Restart-Signal an Supervisor oder (b) Self-Restart über losgelöstes Kindprozess-Spawn → Prozessende nach 500 ms.
- Persistenz: Indirekt über den Config-Manager (schreibt `data/config.yaml`) und die Historien-Utils (SQLite + Medienordner); selbst keine Schreiboperationen. Keine eigene Storage-Abhängigkeit in dieser Datei.
- Zustände: Zustandsloser Request-Handler; einzige zustandsbehaftete Aktivität sind die verzögerten Neustart-/Stop-Timer (setTimeout).
- APIs: `GET /admin/status`, `POST /admin/restart`, `POST /admin/stop`, `GET /admin/vnc/status`, `POST /admin/cache/clear`, `GET|DELETE /admin/logs`, `GET /admin/data-folders`, `POST /admin/data-folders/delete`, `GET|POST /admin/config/server|browser|instances|workers|adapters|pool`, `GET /admin/adapters`, `GET /admin/stats`, `GET|DELETE /admin/stats/range`, `GET /admin/queue`, `GET /admin/history`, `GET /admin/history/stats`, `GET /admin/history/models`, `GET /admin/history/media/*`, `GET /admin/history/:id`, `POST /admin/history/:id/retry-media`, `DELETE /admin/history` (belegt durch Handler-Zweige).
- Ereignisse: Restart-Request (deferierter Neustart nach 500 ms); Stop-Request (Prozessende nach 1000 ms); IPC-Restart-Signal; WebUI-seitige Konfigurations-Änderungen (validiert und persistiert).
- Nebenwirkungen: Kann Prozess-Neustart bzw. -Stop auslösen; kann Dateisystem-Verzeichnisse rekursiv löschen (nur `camoufoxUserData*`-Präfix, nur nicht aktive Ordner); löscht Temp-Dateien, Logs und Statistikdateien; schreibt Konfigurationsdateien; lädt/liest Log- und Medien-Dateien.
- Fehlerfälle: Ungültige JSON-Bodies (Fallback auf Defaults bzw. `INVALID_REQUEST_BODY`), fehlende Parameter (`start`/`end`, `folders`-Array, `ids`), Pfad-Traversal-Versuch bei Medienpfaden (`..` wird abgelehnt), nicht existente Historien-Records/Medien (404), Pool nicht initialisiert (Fallback auf HTTP-Download), generische Fehler als `INTERNAL_ERROR` (500).
- Sicherheitsrelevanz: Der Router selbst führt keine Authentifizierung durch; er setzt voraus, dass die Auth-Middleware (in `api/index.js`) vorgeschaltet ist. Expliziter Schutz gegen Pfad-Traversal beim Medien-Datei-Serving. Löschoperationen sind durch Präfix- und In-Use-Prüfungen abgesichert. Config-Antworten geben Auth-Token als `authToken`-Feld zurück (verwaltet über Manager), was die Notwendigkeit einer Zugriffskontrolle auf `/admin` betont.
- Geschäftslogik: Restart-Modus-Bestimmung (`loginMode` + `workerName` → `-login`/`-login=NAME`-Argumente); Supervisor-Erkennung mit Fallback auf Self-Restart; Config-Trennung von Server- und Queue-Anteilen beim Speichern unter einem Endpunkt; Zusammenführung von Registry-Adapter-Metadaten mit Adapter-spezifischen Config-Filtern; Media-Download-Retry mit Browser-Kontext (Pool-First, sonst HTTP).
- Algorithmen: Medien-MIME-Bestimmung über Dateiendungs-Mapping; Media-Retry: Auswahl eines freien Pool-Pages und Wiederverwendung der Browser-Download-Funktion mit konfigurierbaren Retries.
- verwendete Datenmodelle: Admin-Konfigurationsdatenmodelle (Server/Browser/Queue/Instances/Workers/Adapters/Pool — Form und Defaults über Manager/Validator), Historien-Record-Modell, Statistik-Zähler, Queue-Status, VNC-Info (enabled/port/display/xvfbMode), Adapter-Metadaten (id/displayName/description/models/modelFilter/configSchema).
- Abhängigkeiten: `respond`, `errors`, Logger, Systeminfo, Config-Manager, Config-Validator, Backend-Registry, IPC-Utils, Statistik-Utils, Historien-Utils, `path`, `fs/promises`, Backend-Download-Utils (belegt durch Importliste Zeilen 6–51).
- Rust-Relevanz: Erfordert eine echte HTTP-Router-Schicht (z. B. axum) mit Pfad-/Methoden-Routing, Body-Parsing, Query-Parsing und JSON-Serialisierung; Konfigurations-Get/Set-Endpunkte benötigen im Rust-Rewrite atomare Read-Modify-Write-Operationen auf der Config-Datei; Sicherheitskritisch: Pfad-Normalisierung vor Dateizugriff (kein `..`-Durchstich), Auth-Gate vor allen Admin-Operationen.

#### Evidence
- Evidence-ID: EV-WEB2API-000201 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/routes.js | Zeilenbereich: 75–96 | Beziehung: Erzeugt den Admin-Request-Handler; wird von `src/server/api/index.js` (Zeile 46) mit Konfiguration/Queue/TempDir/SafeMode instanziiert | Typ: Call | Aussage: Die Datei exportiert eine Router-Fabrik, die einen zustandslosen asynchronen Admin-Handler für `/admin/*` bereitstellt; `GET /admin/status` aggregiert Systemstatus und SafeMode-Flag.
- Evidence-ID: EV-WEB2API-000202 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/routes.js | Zeilenbereich: 98–165 | Beziehung: Nutzt IPC-Utils und Kindprozess-Spawn | Typ: Call | Aussage: `POST /admin/restart` parst Login-Modus und Worker-Name, sendet unter Supervisor ein IPC-Restart-Signal mit Argumenten und fällt sonst auf einen losgelösten Selbst-Neustart mit späterem Prozessende zurück; `POST /admin/stop` beendet den Prozess nach 1 s.
- Evidence-ID: EV-WEB2API-000203 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/routes.js | Zeilenbereich: 167–238 | Beziehung: Delegiert an VNC-/System-/Logger-Utils | Typ: Call | Aussage: VNC-Status (mit Default-Antwort bei Nicht-Supervisor), Temp-Verzeichnis-Bereinigung, Log-Lesen der letzten N Zeilen, Log-Löschen sowie Auflisten und Löschen von Datenordnern (mit 207-Multi-Status bei Teilfehlern) sind implementiert.
- Evidence-ID: EV-WEB2API-000204 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/routes.js | Zeilenbereich: 242–380 | Beziehung: Nutzt Config-Manager und Config-Validator | Typ: Call | Aussage: GET/POST für fünf Konfigurationsbereiche (Server, Browser, Instances, Adapters, Pool) mit Validierung vor dem Persistieren; `/config/instances` und `/config/workers` werden als Alias behandelt; Server-Endpunkt spaltet Queue-Anteile ab.
- Evidence-ID: EV-WEB2API-000205 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/routes.js | Zeilenbereich: 382–512 | Beziehung: Nutzt Registry, Statistik- und Historien-Utils sowie Queue-Manager | Typ: Call | Aussage: Adapter-Metadaten mit ConfigSchema/Modellfilter, Tagesstatistik, Statistik im Datumsbereich (GET/DELETE), Queue-Status, Historienliste mit Filter/Paging, Historien-Statistik und Modellliste sind implementiert.
- Evidence-ID: EV-WEB2API-000206 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/routes.js | Zeilenbereich: 514–625 | Beziehung: Nutzt fs/promises und Backend-Download-Utils | Typ: API | Aussage: Statisches Media-Serving mit MIME-Mapping und Pfad-Traversal-Schutz, Historien-Detail (GET), Media-Download-Retry (POST) mit Pool-Browser-First-Strategie sowie Historien-Batch-Löschung (nach IDs oder Datumsbereich) sind implementiert; unbekannte Pfade enden mit 404.
- Evidence-ID: EV-WEB2API-000207 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/routes.js | Zeilenbereich: 58–65, 115–155 | Beziehung: Interne Hilfsfunktion; Restart-Ausführung | Typ: Import | Aussage: Request-Body wird chunkweise eingelesen und als JSON geparst; der Restart-Mechanismus erzeugt einen losgelösten Kindprozess aus den aktuellen CLI-Argumenten ohne `-login`-Filterung und fügt Login-Argumente hinzu.
- Negativnachweis: Keine Persistenz direkt in dieser Datei (keine eigenen File-/DB-Schreiboperationen; alles über Manager/Utils), keine Thread-/Worker-Verwaltung, keine eigenen Algorithmen zur Bildverarbeitung. Belegt durch vollständige Lektüre der 626 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | bcf3f116d87ad11a79d6fc4464d4b7805dacab54432f730fafec81e52255099d |
| Byte Size | 26447 |
| Line Count | 626 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/api/admin/vncProxy.js

- Zweck: VNC-WebSocket-Proxy: akzeptiert WebSocket-Upgrades auf `/admin/vnc` und brückt den Browsersockel bidirektional zu einem lokalen VNC-TCP-Server (belegt durch Dateiinhalt, 195 Zeilen).
- Verantwortlichkeit: Führt die WebSocket-Handshake-Erweiterung des HTTP-Servers für VNC aus, validiert das Token, ermittelt VNC-Informationen über IPC und realisiert die Frame-Kodierung/-Dekodierung samt Richtungs-Umsetzung.
- Eingaben: Upgrade-HTTP-Request (mit `token`-Query, `sec-websocket-key`), der rohe TCP-Sockel, anfängliche Upgrade-Payload (`head`), gültiges Auth-Token, VNC-Server-Info (Port) über IPC.
- Ausgaben: WebSocket-Handshake-Antworten (101/400/401/503) und bidirektionaler Binär-/Text-Datenstrom zwischen Browser und VNC-Server.
- Datenfluss: Browser-WebSocket ↔ WebSocket-Frame-Dekodierung (maske-auf) ↔ VNC-TCP-Sockel ↔ VNC-Daten als WebSocket-Frame (ohne Maske) zurück zum Browser; Close-Frame (Opcode 0x08) terminiert beide Verbindungen.
- Persistenz: Keine.
- Zustände: Verbindungszustand pro Upgrade: handshake-pending → verbunden (bidirektional) → geschlossen; fragmentierte eingehende Frames werden in einem Akkumulations-Puffer gehalten, bis sie vollständig sind.
- APIs: WebSocket-Endpunkt `/admin/vnc` (Upgrade); Token-Parameter `token`; internes VNC-TCP-Protokoll (127.0.0.1:<port>).
- Ereignisse: TCP-Connect, TCP-Daten, TCP-Fehler, TCP-Close, Socket-Close, Socket-Fehler.
- Nebenwirkungen: Eröffnet ein lokales TCP-Socket zum VNC-Server; beendet Verbindungen bei Fehlern; beim Verbindungsende wird die Gegenrichtung zerstört.
- Fehlerfälle: Token-Mismatch (401), VNC nicht aktiv (503), fehlender WebSocket-Key (400), Frame-Dekodierungsfehler bzw. Oversize-Frames (Verbindungsabbruch), VNC-Verbindungsfehler (Abbruch).
- Sicherheitsrelevanz: Token-Vergleich vor Verbindungsaufbau; Handshake wird manuell durchgeführt (SHA-1 + GUID gemäß WebSocket-Standard); VNC wird ausschließlich an den Loopback-Host geroutet; keine Maske beim Senden vom Server (serverseitige Frames bleiben unmaskiert, eingehende Client-Frames werden demaskiert).
- Geschäftslogik: Nur auf dem Pfad `/admin/vnc` aktiviert (im Server aktiviert der `upgrade`-Handler ausschließlich diesen Pfad); Kontrolle über die VNC-Verfügbarkeit aus der Supervisor-Sicht.
- Algorithmen: WebSocket-Frame-Encoding/-Decoding: FIN+BIN-Header, Längenklassen (≤125 / ≤65535 / 64-bit), Payload-Längen-Erweiterungsfelder, Demaskierung per XOR mit 4-Byte-MaskKey; Close-Erkennung über Opcode.
- verwendete Datenmodelle: VNC-Info (enabled, port, display, xvfbMode); WebSocket-Frame (opcode, payload, bytesConsumed).
- Abhängigkeiten: Node-`net`, Node-`crypto`, IPC-Utils (VNC-Info) (belegt durch Importliste Zeilen 6–7).
- Rust-Relevanz: Im Rust-Rewrite komplett durch eine WebSocket-Crate (z. B. tokio-tungstenite) mit Standard-Handshake, Ping/Pong und Frame-Optimierung ersetzbar; Frame-Manipulation sollte nicht selbst gebaut werden (Sicherheits- und Korrektheitsrisiko). Verbindungs-Lebenszyklus als Task-Paar (bidirektionales Piping) mit Graceful-Shutdown.

#### Evidence
- Evidence-ID: EV-WEB2API-000208 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/vncProxy.js | Zeilenbereich: 16–62 | Beziehung: Wird vom HTTP-Server-Upgrade-Handler in `src/server/server.js` (Zeile 167–177) nur für `/admin/vnc` aufgerufen | Typ: API | Aussage: Der Upgrade-Handler validiert das Token, prüft VNC-Verfügbarkeit, berechnet den WebSocket-Accept-Key (SHA-1 + GUID) und verbindet mit dem lokalen VNC-Port.
- Evidence-ID: EV-WEB2API-000209 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/admin/vncProxy.js | Zeilenbereich: 64–195 | Beziehung: Interne Frame-Kodierung/-Dekodierung | Typ: Call | Aussage: Bidirektionales Forwarding mit Frame-Akkumulation, Close-Frame-Handling und Fehler-induziertem Verbindungsabbruch; Server-Frames werden unmaskiert erzeugt, Client-Frames demaskiert (XOR).
- Negativnachweis: Keine Persistenz, keine Authentifizierung über Cookies/Sessions (nur Token-Vergleich), keine Compression/Extensions des WebSocket-Protokolls. Belegt durch vollständige Lektüre der 195 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 216a706a8d4c4500a6ab91c082b17ba4f28f6488feef4f62cca3c3ed1048e546 |
| Byte Size | 5412 |
| Line Count | 195 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/api/index.js

- Zweck: API-Router-Gesamtanordnung: statisches WebUI-Serving, SPA-Fallback, Authentifizierungs-Gate und Dispatch auf `/admin`- und `/v1`-Router (belegt durch Dateiinhalt, 132 Zeilen).
- Verantwortlichkeit: Verkabelt die Router-Fabriken (OpenAI/Admin) und die Auth-Middleware; setzt die statische Dateiauslieferung mit MIME-Mapping und Pfad-Sicherheitsprüfung um; entscheidet über Verfügbarkeit der OpenAI-API in Safe-/Login-Modus.
- Eingaben: HTTP-Requests aller Pfade; Kontext (authToken, config, queueManager, tempDir, loginMode, getSafeMode).
- Ausgaben: Statische Dateien (HTML/CSS/JS/Assets), SPA-Fallback (`index.html`), API-Antworten von Admin/V1-Routern, 403/404/503-Fehler.
- Datenfluss: Request → (a) statisches Serving für GET ohne `/v1`/`/admin`-Präfix, (b) Auth-Gate, (c) `/admin`-Dispatch mit Pfad-Rest, (d) `/v1`-Dispatch mit Safe-/Login-Modus-Prüfung und Pfad-Rest → Antwort.
- Persistenz: Keine (nur statisches Lesen des WebUI-Builds).
- Zustände: Keine eigenen; SafeMode-Signal wird über die Kontext-Funktion erfragt.
- APIs: Gesamte HTTP-Oberfläche; deklariert `/v1` und `/admin` als API-Namespaces.
- Ereignisse: Keine.
- Nebenwirkungen: Liest statische Dateien synchron vom Dateisystem pro Request; SPA-Fallback liefert die Index-Datei für alle unbekannten Frontend-Pfade.
- Fehlerfälle: Pfad-Ausstieg (403), nicht vorhandene Datei (Fallback), unbekannte Namespaces (404), Safe-/Login-Modus-Sperre für `/v1` (503 mit `service_unavailable`).
- Sicherheitsrelevanz: Pfad-Normalisierung: verweigert Zugriffe, die aus dem WebUI-Verzeichnis herauslaufen; Auth-Gate liegt vor allen API-Pfaden, aber nicht vor dem statischen Serving (WebUI offen, API geschützt — abhängig vom konfigurierten Token).
- Geschäftslogik: Safe-Mode-Steuerung: Wenn der Pool nicht initialisiert werden konnte, bleibt das Admin- und WebUI-Angebot erreichbar, während `/v1` mit einer 503-Antwort und Begründung abgelehnt wird; Login-Modus deaktiviert die OpenAI-API vollständig.
- Algorithmen: MIME-Typ-Bestimmung über Endung; Routing-Pfad-Verkürzung per Präfix-Entfernung.
- verwendete Datenmodelle: MIME-Mapping; Safe-Mode-Signal (enabled, reason); Router-Kontext.
- Abhängigkeiten: `fs`, `path`, OpenAI-Router, Admin-Router, Auth-Middleware (belegt durch Importliste Zeilen 6–10).
- Rust-Relevanz: Im Rust-Rewrite übernimmt die Web-Route-Config (z. B. axum Router) das Routing; statische Dateien über axum-`ServeDir` mit vorangestellter Pfad-Validierung; Auth-als-Middleware-Layer; Safe-/Login-Modus als Zustand im App-State.

#### Evidence
- Evidence-ID: EV-WEB2API-000210 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/index.js | Zeilenbereich: 12–85 | Beziehung: Wird von `src/server/server.js` (Zeile 115) als globaler Handler verwendet | Typ: API | Aussage: Implementiert statisches WebUI-Serving mit MIME-Mapping, Pfad-Sicherheitsprüfung (kein Verlassen des WebUI-Verzeichnisses) und SPA-Fallback auf die Index-Datei.
- Evidence-ID: EV-WEB2API-000211 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/index.js | Zeilenbereich: 87–130 | Beziehung: Nutzt Auth-Middleware und die beiden Sub-Router | Typ: Call | Aussage: Auth-Gate läuft vor allen API-Pfaden; `/admin`- und `/v1`-Requests werden mit entferntem Präfix an die jeweiligen Handler weitergegeben; unbekannte Pfade enden mit 404.
- Evidence-ID: EV-WEB2API-000212 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/index.js | Zeilenbereich: 44–46, 101–125 | Beziehung: Konsumiert SafeMode-Signal und Login-Modus-Flag | Typ: Config | Aussage: Bei aktivem Safe-Mode oder Login-Modus wird `/v1` mit einer 503-Antwort `service_unavailable` abgelehnt; im Login-Modus wird der OpenAI-Router gar nicht erst erzeugt.
- Negativnachweis: Keine Persistenz, keine Queue-/Generate-Logik (nur Routing), kein Request-Body-Parsing. Belegt durch vollständige Lektüre der 132 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 595634b436f5e51812ab03ad96637f72326076f3a7cfeab79809b2cc5b111065 |
| Byte Size | 4759 |
| Line Count | 132 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/api/openai/parse.js

- Zweck: Anfrage-Parser für die OpenAI-kompatible Schnittstelle: zerlegt Chat-Nachrichten, validiert Modell und Bilder, extrahiert Prompts und materialisiert Bilddaten in Temp-Dateien (belegt durch Dateiinhalt, 344 Zeilen).
- Verantwortlichkeit: Konvertiert OpenAI-Chat-Request-Bodies in das interne Auftragsmodell (Prompt + Bildpfade + Modell + Streaming-Flag); unterscheidet Text- und Bild-Modelle und setzt bildpolitische Regeln durch.
- Eingaben: Request-Body (messages-Array, model, stream, reasoning), Konfiguration (tempDir, imageLimit, backendName), Rückruffunktionen (Modellliste, Bildpolitik, Modelltyp), requestId, Logger.
- Ausgaben: Parsing-Ergebnis mit `success`-Flag; bei Erfolg das interne Auftragsmodell, bei Fehler einen strukturierten Fehler (Code + Message).
- Datenfluss: Request-Body → Modell-Auflösung gegen die unterstützte Modellliste → Zweig Textmodell (virtuelle Kontext-Konstruktion) oder Bildmodell (letzte User-Nachricht) → Bild-Materialisierung (Base64 → JPEG via Bildverarbeitung) → Ergebnis.
- Persistenz: Schreibt temporäre Bilddateien (komprimierte JPEGs) in `tempDir`; Verantwortung für die Bereinigung liegt beim Queue-Manager (Nebenwirkung, kein Langzeit-Speicher).
- Zustände: Zustandslos pro Request.
- APIs: Keine eigenen HTTP-Endpunkte; konsumiert den Chat-Request-Datentyp (messages/content-Array mit Text- und image_url-Parts).
- Ereignisse: Keine.
- Nebenwirkungen: Anlegen von Temp-Dateien je Base64-Bild; Logging der Parse-Entscheidungen (Modell, Modus).
- Fehlerfälle: fehlende messages (`NO_MESSAGES`), keine User-Nachricht (`NO_USER_MESSAGES`), nicht unterstütztes Modell (`INVALID_MODEL`), zu viele Bilder (`TOO_MANY_IMAGES`), Bildpflicht/-verbot (`IMAGE_REQUIRED`/`IMAGE_FORBIDDEN`), fehlerhafte Base64- oder Bilddaten (Stillschweigendes Ignorieren bzw. Platzhalter im Textmodus).
- Sicherheitsrelevanz: Bild-URLs werden nicht geladen (nur `data:`-URIs akzeptiert, externe URLs werden als Platzhalter markiert); keine Ausführung von Inhalten.
- Geschäftslogik: Textmodus: System-Nachricht wird als permanente Anweisung extrahiert, Historie (alle Nachrichten vor der letzten User-Nachricht) wird als Dialog-Protokoll gerendert, die letzte User-Nachricht wird als aktueller Input separiert — ergibt einen einzigen Prompt-String. Bildmodus: nur die letzte User-Nachricht wird verarbeitet, Text- und Bildanteile extrahiert; Bildanzahl-Limit (≤10 strikt, darüber hinaus Browser-Hard-Limit von 10). Bildpolitik (REQUIRED/FORBIDDEN/OPTIONAL) wird je Modell angewendet.
- Algorithmen: Base64-Dekodierung + JPEG-Rekodierung (Qualität 90) über eine Bildverarbeitungs-Bibliothek; Bild-Limit-Zählung mit Platzhalter-Ersetzung.
- verwendete Datenmodelle: OpenAI-Message (role, content als String oder Content-Part-Array), Bildpolitik-Enum (REQUIRED/FORBIDDEN/OPTIONAL), internes Auftragsmodell (prompt, imagePaths, modelId, modelName, isStreaming).
- Abhängigkeiten: `fs`, `path`, Bildverarbeitungs-Bibliothek, Backend-Registry (Bildpolitik), Fehlermodul (belegt durch Importliste Zeilen 6–10).
- Rust-Relevanz: Im Rust-Rewrite: Chat-Schema-Deserialisierung (serde), Validierung des Modells gegen eine Modell-Registry, Base64- und Bild-Rekodierung über eine Bild-Crate (z. B. image); Temp-Datei-Lebenszyklus über ein RAII-/Tempdir-Konzept statt manueller Bereinigung; die Prompt-Komposition (System-Historie-Aktuell) als eigene, testbare Builder-Funktion.

#### Evidence
- Evidence-ID: EV-WEB2API-000213 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/openai/parse.js | Zeilenbereich: 64–125 | Beziehung: Wird vom OpenAI-Router (`src/server/api/openai/routes.js`, Zeile 105) aufgerufen | Typ: Call | Aussage: Einstiegspunkt der Anfrage-Parsing: messages-Pflichtprüfung, Modellvalidierung gegen die unterstützte Modellliste, Modus-Ableitung (Text- vs. Bildmodell) und Verzweigung.
- Evidence-ID: EV-WEB2API-000214 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/openai/parse.js | Zeilenbereich: 130–240 | Beziehung: Interne Textmodus-Verarbeitung | Typ: Call | Aussage: Baut eine virtuelle Kontext-Struktur: System-Anweisung, gerenderte Dialog-Historie und separater aktueller Input; Bilder in der Historie werden als Base64-Temp-Dateien materialisiert oder als Platzhalter markiert; ohne User-Nachricht Fehler.
- Evidence-ID: EV-WEB2API-000215 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/openai/parse.js | Zeilenbereich: 245–316 | Beziehung: Nutzt die Bildpolitik der Registry | Typ: Call | Aussage: Bildmodus verarbeitet nur die letzte User-Nachricht, erzwingt Bildlimits und wendet die Modell-Bildpolitik (Pflicht/verboten/optional) an.
- Evidence-ID: EV-WEB2API-000216 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/openai/parse.js | Zeilenbereich: 318–344 | Beziehung: Bild-Materialisierung | Typ: Persistenz | Aussage: Dekodiert Base64-Data-URIs, rekodiert sie als JPEG (Qualität 90) und schreibt sie mit zufälligem Dateinamen in das Temp-Verzeichnis; Fehler führen zu null.
- Negativnachweis: Kein Netzwerkzugriff (Bild-URLs werden nicht gefetcht), keine Persistent-Speicherung über den Request hinaus, keine Bilderkennung/-analyse. Belegt durch vollständige Lektüre der 344 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | b8603c4f0e11394336dfbbfff44726bffe0c182e8171b20669d580534cb26f97 |
| Byte Size | 11481 |
| Line Count | 344 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/api/openai/routes.js

- Zweck: OpenAI-kompatible API-Router für `/v1/models`, `/v1/cookies` und `/v1/chat/completions`; steuert Limitation, SSE-Aktivierung und Übergabe an den Queue-Manager (belegt durch Dateiinhalt, 175 Zeilen).
- Verantwortlichkeit: Dispatchen der `/v1`-Anfragen nach Methode/Pfad, Durchsetzen des Nicht-Streaming-Queue-Limits, Vorbereitung von SSE-Antworten und Erstellung der Queue-Aufträge.
- Eingaben: HTTP-Requests mit JSON-Body (chat/completions), Query-Parameter (cookies: name, domain), Kontext (Modellliste, Bildpolitik, Modelltyp, tempDir, imageLimit, queueManager).
- Ausgaben: Modellliste (JSON), Worker-Cookies (JSON), Chat-Completion-Responses bzw. SSE-Streams, strukturierte Fehler (403/429/500).
- Datenfluss: Request → requestId (UUID-Präfix) → Modell-Liste/Cookies/Chat-Dispatch → (Chat) Body-Parsing → Kapazitätsprüfung → ggf. SSE-Header → Parse → Queue-Einreihung; Fehlerpfad → `sendApiError`.
- Persistenz: Keine.
- Zustände: Zustandslos; Queue-Kapazität wird vom Queue-Manager erfragt.
- APIs: `GET /v1/models`, `GET /v1/cookies?name=&domain=`, `POST /v1/chat/completions` (OpenAI-kompatibel, `stream`-Flag).
- Ereignisse: Queue-Einreihung eines Auftrags (Prompt, Bildpfade, Modell, Streaming, Reasoning-Flag).
- Nebenwirkungen: Anlegen von Queue-Aufträgen; bei Streaming sofortige Belegung der Response mit SSE-Headern; Logging.
- Fehlerfälle: Browser-Pool nicht initialisiert (`BROWSER_NOT_INITIALIZED`), unbekannter Worker (als `INVALID_MODEL` gemappt), volle Queue bei Nicht-Streaming (`SERVER_BUSY` 429 mit Hinweis auf Streaming), Parse-Fehler (durchgereicht), generische Fehler (500).
- Sicherheitsrelevanz: Fehlerantworten verwenden das OpenAI-Format; die eigentliche Authentifizierung liegt außerhalb (Middleware). Cookies-Endpunkt gibt gespeicherte Browser-Cookies preis — ist nur über Auth erreichbar und setzt einen initialisierten Pool voraus.
- Geschäftslogik: Kapazitäts-Logik: Nicht-Streaming-Requests werden abgelehnt, wenn die effektive Queuegröße erreicht ist; Streaming wird uneingeschränkt angenommen; Reasoning-Flag wird in den Auftrag übernommen.
- Algorithmen: requestId-Erzeugung (UUID-Präfix, 8 Zeichen); Fehlerklassifizierung anhand Fehlertext-Teilzeichenketten.
- verwendete Datenmodelle: Chat-Completion-Request (messages, model, stream, reasoning); interner Auftrag (prompt, imagePaths, modelId, modelName, id, isStreaming, reasoning).
- Abhängigkeiten: `crypto`, Logger, Fehlermodul, Respond-Modul, Parse-Modul (belegt durch Importliste Zeilen 6–10).
- Rust-Relevanz: axum-Handler mit JSON-Extractor, State-basiertem Queue-Zugriff und Query-Extractor; SSE über einen Stream statt schreibendem Response; Kapazitäts-Check als Rate-Limiting-/Admission-Control-Element.

#### Evidence
- Evidence-ID: EV-WEB2API-000217 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/openai/routes.js | Zeilenbereich: 159–174 | Beziehung: Wird von `src/server/api/index.js` (Zeile 45) erzeugt, wenn nicht im Login-Modus | Typ: API | Aussage: Router-Dispatch: GET Modellliste, GET Worker-Cookies (mit Pool-Pflichtprüfung und Fehlerklassifikation) und POST Chat-Completions; sonst 404.
- Evidence-ID: EV-WEB2API-000218 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/api/openai/routes.js | Zeilenbereich: 73–150 | Beziehung: Nutzt Queue-Manager und Parse-Modul | Typ: Call | Aussage: Chat-Completions: Body-Parsing, Kapazitätsprüfung für Nicht-Streaming (429 bei Vollauslastung), SSE-Header für Streaming, Parse-Validierung und anschließende Queue-Einreihung mit Reasoning-Flag.
- Negativnachweis: Keine Persistenz, keine direkte Bildverarbeitung (Delegation an Parse), kein direktes Generate (Delegation an Queue). Belegt durch vollständige Lektüre der 175 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | c8808e87ddf660c4fe271dd637a4760eb1d14166f64e5d34a0903a9f3fc699bb |
| Byte Size | 5772 |
| Line Count | 175 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/errors.js

- Zweck: Zentrale Fehlerkodierung der Serverebene: Typen, Codes, Message/Status/Type-Mapping und Adapter-Fehlerkodes (belegt durch Dateiinhalt, 185 Zeilen).
- Verantwortlichkeit: Einzige Quelle für OpenAI-standardisierte Fehlerantworten (invalid_request_error / server_error / rate_limit_error) und für die Fehlerklassifikation der Adapterebene.
- Eingaben: Fehlercode (String), optional eigene Nachricht/Status in Aufrufern.
- Ausgaben: Fehlerdetails (Message, HTTP-Status, Fehlertyp) bzw. Fallback-Werte.
- Datenfluss: Fehlercode → Lookup in Detailtabelle → Message/Status/Type.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Fehler-Nachschlag (Message/Status/Details); OpenAI-Fehlertypen; Adapter-Fehlercodes.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Nicht registrierter Code → Fallback-Message "未知错误" und Status 500 (bewusst toleranter Fallback).
- Sicherheitsrelevanz: Härtet die Fehlerantworten auf ein einheitliches Format; verhindert Leaking interner Fehlertexte über die Message-Zuordnung (Codierung statt Rohtext).
- Geschäftslogik: Fehlerklassifikation als vertrauenswürdige Mapping-Tabelle: 401 (Auth), 429 (Queue/Rate), 400 (Validierung), 403 (CAPTCHA), 500 (intern), 502 (Generierung), 503 (Browser nicht bereit).
- Algorithmen: Keine (reine Nachschlag-Tabellen).
- verwendete Datenmodelle: Fehlerkode-Enum, Fehlertyp-Enum, Detail-Tupel (message, status, type); Adapter-Fehlerkode-Enum (PAGE_CRASHED, NETWORK_ERROR, TIMEOUT_ERROR, RATE_LIMITED, CAPTCHA_REQUIRED, AUTH_REQUIRED, CONTENT_BLOCKED u. a.).
- Abhängigkeiten: Keine (Beziehung: wird von Respond, Parse, Routen, Admin-Router, Queue importiert).
- Rust-Relevanz: Im Rust-Rewrite: eine Fehler-Enum mit `IntoResponse`-Implementierung (Status + OpenAI-Fehlertyp + Code) und From-Konversionen für interne Fehler; Muster, das 1:1 als eigener Typ übernommen werden kann (Konzept, nicht Code).

#### Evidence
- Evidence-ID: EV-WEB2API-000219 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/errors.js | Zeilenbereich: 11–117 | Beziehung: Wird von Respond-/Router-/Parse-Modulen importiert | Typ: Schema | Aussage: Definiert OpenAI-standardisierte Fehlertypen (invalid_request_error, server_error, rate_limit_error) und eine vollständige Code→Detail-Tabelle (Message, HTTP-Status, Typ) inkl. 401/429/403/500/502/503-Fällen.
- Evidence-ID: EV-WEB2API-000220 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/errors.js | Zeilenbereich: 119–184 | Beziehung: Exportiert Nachschlagfunktionen und Adapter-Fehlercodes | Typ: Import | Aussage: Fehler-Nachschlagfunktionen liefern Message, Status bzw. Detailtupel mit sicheren Fallbacks; Adapterebene nutzt eine eigene Fehlerkode-Klassifikation (u. a. Netzwerk, Timeout, CAPTCHA, Auth, Content-Blockade).
- Negativnachweis: Keine Persistenz, keine Logik außer Lookup; keine zyklischen Abhängigkeiten (Datei importiert nichts). Belegt durch vollständige Lektüre der 185 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | f8afcf2668c809cc5acf9b8c103b438794ee658f165d1b3bb86b34171572caef |
| Byte Size | 5026 |
| Line Count | 185 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/index.js

- Zweck: Barrel-/Fassadenmodul der Server-Ebene: re-exportiert Fehler-, Respond-, Queue-, Parse-, Router- und Auth-Fähigkeiten (belegt durch Dateiinhalt, 21 Zeilen).
- Verantwortlichkeit: Einheitliche, stabile Importschnittstelle für die Serverebene; verdeckt die interne Verzeichnisstruktur.
- Eingaben: Keine.
- Ausgaben: Öffentliche Exporte (Fehler, Response-Helfer, Queue-Fabrik, Parse, Global-Router, Auth-Middleware).
- Datenfluss: Kein Laufzeit-Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Fassaden-API der Serverebene.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine eigene; bündelt die Sicherheits-bezogenen Exporte (Auth, Fehler).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine eigenen.
- Abhängigkeiten: `errors`, `respond`, `queue`, `parse`, `api/index`, `middlewares/auth` (belegt durch Exportliste Zeilen 6–19).
- Rust-Relevanz: Reine Modulorganisation; im Rust-Rewrite entfällt die Fassade zugunsten von Modul-Pfaden mit `pub use`.

#### Evidence
- Evidence-ID: EV-WEB2API-000221 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/index.js | Zeilenbereich: 6–19 | Beziehung: Wird von `src/server/server.js` (Zeile 26) dynamisch importiert | Typ: Import | Aussage: Re-exportiert die öffentliche Server-API: Fehler-Nachschlag, JSON/SSE/Fehler-Response-Helfer, Queue-Manager-Fabrik, Request-Parser, Global-Router und Auth-Middleware.
- Negativnachweis: Keine Logik, keine Persistenz, keine Zustände (nur Re-Exporte). Belegt durch vollständige Lektüre der 21 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 01642ffb53e6a9786f6b9bd507f3c145c0f86d0aa8d045255a2ec99c95189eca |
| Byte Size | 574 |
| Line Count | 21 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/middlewares/auth.js

- Zweck: Authentifizierungs-Middleware: prüft den `Authorization`-Header auf das konfigurierte Bearer-Token (belegt durch Dateiinhalt, 43 Zeilen).
- Verantwortlichkeit: Einheitlicher Auth-Gate vor allen API-Pfaden; Entkopplung der Auth-Logik aus dem Router.
- Eingaben: HTTP-Request, konfiguriertes Auth-Token.
- Ausgaben: Boolean-Entscheidung (durchgelassen/abgelehnt); bei Ablehnung eine OpenAI-formatierte 401-Antwort über das Fehler-/Respond-Modul.
- Datenfluss: Header → Token-Vergleich → Zulassen bzw. 401-Antwort.
- Persistenz: Keine.
- Zustände: Keine (Token wird bei Router-Erstellung fixiert).
- APIs: Keine öffentliche API.
- Ereignisse: Keine.
- Nebenwirkungen: Sendet bei Ablehnung die 401-Antwort und schließt den Request.
- Fehlerfälle: Fehlender oder falscher Header → 401 `UNAUTHORIZED`.
- Sicherheitsrelevanz: Zentraler Auth-Kontrollpunkt; leeres/fehlendes Token deaktiviert die Prüfung (bewusst konfigurierbar, im Startpfad wird bei fehlendem `server.auth` eine Warnung ausgegeben); String-Vergleich (kein konstanter Zeitebenen-Vergleich implementiert — im Rust-Rewrite mit Konstantzeit-Vergleich absichern).
- Geschäftslogik: Token-Empty-Fall: wenn kein Token konfiguriert, werden alle Requests zugelassen.
- Algorithmen: Exakter String-Vergleich des Authorization-Headers gegen `Bearer <token>`.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Respond-Modul, Fehlermodul (belegt durch Importliste Zeilen 6–7).
- Rust-Relevanz: axum-Middleware oder Extractors; Konstantzeit-Vergleich des Tokens; Token aus Config in App-State statt Modul-Konstante.

#### Evidence
- Evidence-ID: EV-WEB2API-000222 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/middlewares/auth.js | Zeilenbereich: 15–42 | Beziehung: Wird in `src/server/api/index.js` (Zeile 42) vor beiden Routern installiert | Typ: Call | Aussage: Vergleicht den Authorization-Header exakt mit dem Bearer-Token; ohne konfiguriertes Token werden Requests ungeprüft durchgelassen; Ablehnung erzeugt eine standardisierte 401-Antwort.
- Negativnachweis: Keine Persistenz, keine Session-/Cookie-Verwaltung, keine Rollen (nur ein globales Token). Belegt durch vollständige Lektüre der 43 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | b957f2d7b43e1c1d8f40de5fdbaaa000a34b9e40dc04bd18d97997d858d09a55 |
| Byte Size | 1221 |
| Line Count | 43 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/preflight.js

- Zweck: Start-Preflight: prüft vor Serverstart kritische Laufzeit-Artefakte und beendet den Prozess mit einem definierten Exit-Code bei Mängeln (belegt durch Dateiinhalt, 122 Zeilen).
- Verantwortlichkeit: Nachweisführung, dass die nativen SQLite-Bindings, die Camoufox-Patches, das Camoufox-Executable, die Versionsdatei und die Geo-IP-Datenbank vorhanden sind, bevor der Server hochfährt.
- Eingaben: Dateisystem-Pfade, Plattform, Patchliste aus dem Postinstall-Skript.
- Ausgaben: Ergebnisobjekt `{ok, errors[]}`; bei negativem Ergebnis Prozessbeendigung mit Exit-Code 78.
- Datenfluss: Dateisystem-Checks → Fehlersammlung → Entscheidung ok/fehlerhaft → optionaler Prozess-Exit.
- Persistenz: Keine (nur lesende Checks).
- Zustände: Keine.
- APIs: Preflight-Funktion + ausführende Funktion.
- Ereignisse: Prozess-Exit (78) bei fehlenden Artefakten.
- Nebenwirkungen: Beendet den Prozess bei fehlenden Abhängigkeiten (Exit-Code 78 = Konfigurations-/Abhängigkeitsfehler; Supervisor startet bewusst nicht neu); Logging der Fehlerliste mit Hinweis auf die Init-Schritte.
- Fehlerfälle: Fehlende native SQLite-Datei, nicht angewendete Patches (MD5-Abgleich gegen die Patch-Quellen), fehlendes Camoufox-Executable (plattformabhängiger Pfad), fehlende `version.json`, fehlende GeoLite2-Datenbank.
- Sicherheitsrelevanz: Patch-Integrität wird per MD5-Hash überprüft (nur Lesen der Zieldatei im node_modules-Baum); verhindert Start mit nicht-gepatchter Browser-Automationsbibliothek.
- Geschäftslogik: Fail-fast beim Start: keine teilfunktionsfähige Bereitstellung, wenn die Browser-Automation nicht vollständig installiert ist.
- Algorithmen: MD5-Dateihash; Plattform-basierte Pfadauflösung (win32/darwin/linux).
- verwendete Datenmodelle: Patchliste (Quelle→Ziel-Dateinamen) aus dem Postinstall-Skript; Ergebnisobjekt (ok, errors).
- Abhängigkeiten: `fs`, `path`, `os`, `crypto`, Logger, Patchliste des Postinstall-Skripts (belegt durch Importliste Zeilen 6–11).
- Rust-Relevanz: Als Start-Validator: Pfad-/Dateichecks (std::path), SHA-256 statt MD5 (Sicherheit), plattformabhängige Pfade über cfg; fail-fast mit definiertem Exit-Code und einem gesammelten Fehlerbericht.

#### Evidence
- Evidence-ID: EV-WEB2API-000223 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/preflight.js | Zeilenbereich: 48–101 | Beziehung: Nutzt die Patchliste aus `scripts/postinstall.js` (CAMOUFOX_PATCHES, Zeile 24) | Typ: Config | Aussage: Prüft native SQLite-Bindings, Patch-Anwendung per MD5-Vergleich gegen die Patch-Quelldateien, Camoufox-Executable (plattformabhängig), `version.json` und GeoLite2-Datenbank; sammelt alle Fehler im Ergebnis.
- Evidence-ID: EV-WEB2API-000224 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/preflight.js | Zeilenbereich: 103–122 | Beziehung: Wird in `src/server/server.js` (Zeile 22) beim Start ausgeführt | Typ: Call | Aussage: Bei negativem Ergebnis wird die Fehlerliste geloggt und der Prozess mit Exit-Code 78 beendet (kein Auto-Restart durch den Supervisor); bei Erfolg wird „Preflight OK" geloggt.
- Negativnachweis: Keine Persistenz, keine Netzwerk-Operationen, keine laufenden Prüfungen (nur einmalig beim Start). Belegt durch vollständige Lektüre der 122 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 6d66ba809e536a6b86c4be8525d401fbfd9db506ac3ec0fb25c3207ad00da83b |
| Byte Size | 4012 |
| Line Count | 122 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/queue.js

- Zweck: Auftrags-Warteschlangen-Manager: serialisiert konkurrierende Generate-Aufträge, begrenzt die Parallelität, pflegt Historien- und Statistik-Einträge und betreut den Streaming-Herzschlag (belegt durch Dateiinhalt, 368 Zeilen).
- Verantwortlichkeit: Steuert den Lebenszyklus jedes Auftrags (Einreihung → Verarbeitung → Response/Fehler → Aufräumen), koordiniert die Pool-Initialisierung und reicht Cookies-Abfragen an die Backend-Schicht durch.
- Eingaben: Aufträge (prompt, imagePaths, modelId, modelName, id, isStreaming, reasoning), Queue-Konfiguration (maxConcurrent, queueBuffer, keepaliveMode), Callbacks (initBrowser, generate, config, navigateToMonitor, getCookies).
- Ausgaben: Chat-Completion-Responses (JSON/SSE), Fehlerantworten, Historien-/Statistik-Einträge, Queue-Status.
- Datenfluss: `addTask` → Queue-Array → `processQueue` (Schleife) → `processTask` → Pool-Init (einmalig) → Generate-Callback → Erfolg: Response + Historien-Update (async, nicht blockierend) + Statistik; Fehler: Fehlerantwort + Statistik + Historien-Update; am Ende Temp-Dateien-Bereinigung.
- Persistenz: Indirekt: Historien-Erstellung/-Updates (SQLite) und Tagesstatistik über die Utils; Temp-Bilddateien werden nach Verarbeitung gelöscht.
- Zustände: Modul-intern: Queue-Array, Liste der in Verarbeitung befindlichen Aufträge, Zähler `processingCount`, Pool-Kontext (null bis zur ersten Initialisierung); Task-Zustand: pending → success/failed.
- APIs: addTask, getStatus, getDetailedStatus, canAcceptNonStreaming, initializePool, getPoolContext, getWorkerCookies.
- Ereignisse: Heartbeat-Timer (3 s) für Streaming-Aufträge; Queue-Leerlauf löst einen optionalen Monitor-Navigations-Callback aus.
- Nebenwirkungen: Startet Pool beim ersten Auftrag (bzw. explizit via initializePool); löscht Temp-Bilddateien; schreibt Historien- und Statistikdaten; sendet Herzschläge in Streaming-Responses; Fehler-Responses verwenden Status 502 (retryable: 503).
- Fehlerfälle: Generate-Fehler → 502/503 mit `GENERATION_FAILED`; unerwartete Exceptions → 500 `INTERNAL_ERROR`; Historien-Schreibfehler werden nur geloggt (nicht fatal); fehlender getCookies-Callback → Fehler beim Durchreichen.
- Sicherheitsrelevanz: Keine direkte Auth; Fehlertexte des Backends werden in API-Antworten übernommen (potenzielles Informations-Leak — im Rust-Rewrite kategorisieren statt durchreichen); Historien enthält Prompts und Medien (Datenschutz-Aspekt).
- Geschäftslogik: Effektive Queuegröße = maxConcurrent + queueBuffer (0 = unbegrenzt); Nicht-Streaming-Kapazitätsprüfung; streaming ohne Kapazitätslimit; Reasoning-Inhalte werden getrennt erfasst und in der Historien gespeichert, Base64-Bilder werden nicht in die Historien geschrieben (nur URLs); Markdown-Image-Format konfigurierbar.
- Algorithmen: FIFO-Queue mit rekursiver Drain-Schleife; paralleler Durchsatz maxConcurrent; asynchrones, nicht-blockierendes Historien-Media-Postprocessing.
- verwendete Datenmodelle: TaskContext (HTTP-Objekte, prompt, Bildpfade, Modell, id, streaming, reasoning), QueueConfig, PoolContext, Historien-Record, Statistik-Zähler.
- Abhängigkeiten: Logger, Respond-Modul, Fehlermodul, Statistik-Utils, Historien-Utils (belegt durch Importliste Zeilen 6–18).
- Rust-Relevanz: Im Rust-Rewrite: tokio-basierte Auftrags-Warteschlange (mpsc + worker-Tasks), Semaphore für maxConcurrent, Abbruch-/Timeout-Handling, Actor- oder Shared-State-Muster statt Closures; Heartbeat über tokio-interval; Historien-Writes als Owned-Task ohne Blockierung der Antwort.

#### Evidence
- Evidence-ID: EV-WEB2API-000225 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/queue.js | Zeilenbereich: 56–75, 281–284 | Beziehung: Wird von `src/server/server.js` (Zeile 79) erzeugt | Typ: Call | Aussage: Fabrik für den Queue-Manager; effektive Queuegröße = maxConcurrent + queueBuffer (0 = unbegrenzt); `addTask` reiht ein und stößt die Verarbeitung an.
- Evidence-ID: EV-WEB2API-000226 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/queue.js | Zeilenbereich: 96–243 | Beziehung: Nutzt Generate-Callback, Historien- und Statistik-Utils | Typ: Call | Aussage: Pro Auftrag: Historien-Record (pending), Heartbeat-Timer (3 s) für Streaming, einmalige Pool-Initialisierung, Generate-Aufruf, Erfolgs-/Fehlerpfad mit Statistik- und Historien-Updates sowie Antwort-Formatierung (JSON/SSE, optional Reasoning, Markdown-Bilder konfigurierbar).
- Evidence-ID: EV-WEB2API-000227 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/queue.js | Zeilenbereich: 248–275 | Beziehung: Interne Drain-Schleife | Typ: Call | Aussage: Verarbeitet Aufträge bis maxConcurrent; bei Leerlauf optionaler Monitor-Navigations-Callback; Temp-Dateien werden nach der Verarbeitung bereinigt; Fehler in der Historien-Aktualisierung sind nicht fatal.
- Evidence-ID: EV-WEB2API-000228 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/queue.js | Zeilenbereich: 290–367 | Beziehung: Konsumiert von Admin-Router und OpenAI-Router | Typ: API | Aussage: Status (queueLength/processing/total), detaillierter Status (laufende/wartende Aufträge mit Modell/Streaming), Kapazitätsprüfung für Nicht-Streaming, explizite Pool-Initialisierung, Pool-Kontextzugriff und Cookies-Durchreichung sind implementiert.
- Negativnachweis: Keine eigene Datenbank (nur über Historien-Utils), keine Timer-Persistenz, kein verteiltes Queuing (nur In-Process). Belegt durch vollständige Lektüre der 368 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 4c4477c725cf7fc65d94495ccfaebb89c77af26f2262f2d295b34920dc5162a6 |
| Byte Size | 12339 |
| Line Count | 368 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/respond.js

- Zweck: Zentrales Response-Modul: JSON-, SSE-, Heartbeat- und Fehler-Antworten sowie OpenAI-kompatible Completion-Formatter (belegt durch Dateiinhalt, 159 Zeilen).
- Verantwortlichkeit: Einheitliche, OpenAI-kompatible Antwortserialsierung; vermeidet doppeltes Schreiben auf beendete Responses.
- Eingaben: HTTP-Response-Objekt, Status, Payload, Fehleroptionen (code/message/status/isStreaming), Inhalte für Completion-Formatter.
- Ausgaben: JSON-Antworten, SSE-Events (`data: <json>`), SSE-Done (`data: [DONE]`), Heartbeats (Kommentar- oder Content-Modus), OpenAI-kompatible Chat-Completion-Objekte (Chat-Completion / Chunk).
- Datenfluss: Aufrufer → Formatter → Serialisierung → Response-Stream.
- Persistenz: Keine.
- Zustände: Keine; Guard gegen bereits beendete Responses (`writableEnded`).
- APIs: JSON-/SSE-/Heartbeat-/Fehler-Sender; Completion-Formatter (final + chunk).
- Ereignisse: Keine.
- Nebenwirkungen: Schreibt in den HTTP-Response-Stream; im Streaming-Fehlerfall wird ein Fehler-Event gefolgt von `[DONE]` gesendet.
- Fehlerfälle: Bereits beendete Responses werden still ignoriert; unbekannter Fehlercode fällt auf `server_error`/500 zurück.
- Sicherheitsrelevanz: Einheitliches Fehlerformat verhindert das Leaken interner Fehlerdetails; Fehlerantworten enthalten nie Stack-Traces.
- Geschäftslogik: OpenAI-Format: id-Präfix, object-Typ (chat.completion / chat.completion.chunk), Timestamps, choices-Array mit finish_reason; Reasoning-Inhalte als separates Feld; Heartbeat-Modi: Kommentar-Zeile vs. leeres Delta-Chunk.
- Algorithmen: Keine (Formatierung/Serialisierung).
- verwendete Datenmodelle: Chat-Completion-Antwortobjekt; Streaming-Chunk; SSE-Event; Fehler-Payload (message, type, code).
- Abhängigkeiten: Fehlermodul (Details/Typ/Status-Lookup) (belegt durch Importliste Zeile 6).
- Rust-Relevanz: axum-Response-Typen mit serde; SSE über Stream (channel) statt schreibendem Response; Fehler als `IntoResponse`; Completion-Format als eigene Structs mit serde-Defaults.

#### Evidence
- Evidence-ID: EV-WEB2API-000229 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/respond.js | Zeilenbereich: 14–66 | Beziehung: Wird von allen Servern-Modulen importiert | Typ: API | Aussage: JSON-Antworten, SSE-Events, SSE-Abschlussmarker und Heartbeat (Kommentar- bzw. Content-Modus mit leerem Delta-Chunk) sind implementiert; alle Sender prüfen den beendeten Zustand der Response.
- Evidence-ID: EV-WEB2API-000230 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/respond.js | Zeilenbereich: 77–158 | Beziehung: Nutzt Fehler-Detail-Lookup | Typ: Call | Aussage: Einheitliche OpenAI-formatierte Fehlerantworten (Streaming: Fehler-Event + Done; sonst JSON mit Status); Completion- und Chunk-Formatter inkl. optionaler Reasoning-Inhalte sind implementiert.
- Negativnachweis: Keine Persistenz, keine Zustände, keine Netzwerklogik außer Response-Schreiben. Belegt durch vollständige Lektüre der 159 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 65d9f970fcc066fc2d55368edef31d6b181b0074da6c186354ab667c91a213de |
| Byte Size | 4959 |
| Line Count | 159 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/server/server.js

- Zweck: Server-Einstiegspunkt: Preflight, Backend-Ladung, Komponentenverkabelung (Queue/Router), Safe-Mode-Logik, HTTP-Server und VNC-Upgrade-Handling (belegt durch Dateiinhalt, 192 Zeilen).
- Verantwortlichkeit: Startreihenfolge und Fehlerbehandlung beim Booten; verdrahtet Backend-Fähigkeiten (initBrowser, generate, getModels, getImagePolicy, getModelType, navigateToMonitor, getCookies) mit Queue und Router.
- Eingaben: CLI-Argumente (`-login`), Umgebungsvariablen (Supervisor-Erkennung), Konfiguration aus dem Backend, Backend-Laufzeitobjekt.
- Ausgaben: Laufender HTTP-Server auf konfiguriertem Port; VNC-Upgrade-Behandlung; Log-Ausgaben zum Start; Prozess-Exit 78 bei Konfigurationsfehlern.
- Datenfluss: Start → Preflight → Backend (getBackend) → Konfigurationsextraktion → Queue-Manager → Global-Router → HTTP-Server (listen) ; Upgrades auf `/admin/vnc` → VNC-Proxy.
- Persistenz: Indirekt: lädt Tagesstatistik (Dateien) und initialisiert die Historien-Datenbank beim Start (Fehler nur als Warnung).
- Zustände: SafeMode-Flag + Begründung (gesetzt bei Pool-Init-Fehler); Login-Modus (aus CLI-Argumenten); Supervisor-Erkennung.
- APIs: HTTP-Endpunkte gemäß Router; WebSocket-Upgrade `/admin/vnc`.
- Ereignisse: Server-Listen; Upgrade-Requests; Pool-Initialisierungs-Ergebnis.
- Nebenwirkungen: Vorwärmen des Pools beim Start (oder Safe-Mode); Laden der Tagesstatistik; Erstellen/Initialisieren der Historien-Datenbank; Prozess-Exit 78 bei fehlender Konfiguration; VNC-Proxies bei Upgrade.
- Fehlerfälle: Backend-/Config-Ladefehler → Exit 78 (kein Supervisor-Restart); Historien-DB-Fehler → Warnung, Weiterlauf; Pool-Init-Fehler → Safe-Mode (HTTP+Admin verfügbar, `/v1` 503).
- Sicherheitsrelevanz: Auth-Token wird aus der Konfiguration übernommen und an Router/Middleware gereicht; Safe-Mode verweigert die OpenAI-API (fail-open für Admin/WebUI, fail-closed für Generate); VNC-Upgrade nur auf einem Pfad.
- Geschäftslogik: Login-Modus (Start mit `-login`) erlaubt Browser-Logins, deaktiviert `/v1`; Supervisor-Modus erkennt die Prozessumgebung für Restart-/Stop-IPC; dynamische Importe nach dem Preflight verzögern die Initialisierung bis nach der Integritätsprüfung.
- Algorithmen: Keine (Verkabelung/Startsequenz).
- verwendete Datenmodelle: Router-Kontext; Queue-Config; Safe-Mode-Signal (enabled, reason).
- Abhängigkeiten: `http`, Preflight, Backend (dynamisch), Logger, Server-Fassade, IPC-Utils, Statistik-Utils, Historien-Utils, VNC-Proxy (dynamisch) (belegt durch Importe Zeilen 18–29 und 166–177).
- Rust-Relevanz: Als Start-/Composition-Root des Rust-Rewrites: `main` mit Config-Load, Runtime-Setup, Router-Aufbau und Graceful-Shutdown; Pool-Init als async-Start mit Fehlerzustand statt Safe-Mode-Flag (ausdrücklicher Service-Zustand).

#### Evidence
- Evidence-ID: EV-WEB2API-000231 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/server.js | Zeilenbereich: 18–73 | Beziehung: Ruft Preflight und Backend-Ladung auf | Typ: Call | Aussage: Startet mit Preflight (Exit 78 bei Fehlschlag), lädt das Backend dynamisch (Exit 78 bei Config-Fehlern), extrahiert Port/Auth/Keepalive/Concurrency/QueueBuffer/ImageLimit aus der Konfiguration.
- Evidence-ID: EV-WEB2API-000232 | Repository: WebAI2API | Commit: content-copy | Datei: src/server/server.js | Zeilenbereich: 74–188 | Beziehung: Verdrahtet Queue, Router und HTTP-Server | Typ: API | Aussage: Erzeugt Queue-Manager mit Backend-Callbacks, erkennt Login-Modus, installiert den Global-Router, startet nach dem Laden der Tagesstatistik, initialisiert die Historien-DB (Warnung statt Abbruch), wärmt den Pool vor (Fehler → Safe-Mode mit 503 für `/v1`) und behandelt WebSocket-Upgrades ausschließlich auf `/admin/vnc` mit dem VNC-Proxy.
- Negativnachweis: Keine eigene Persistenz-Implementierung, kein TLS (HTTP), kein Cluster/Worker-Threading. Belegt durch vollständige Lektüre der 192 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | e59b362fda27e4171b65834fb5ebe878b31600b2900e10381ba680bbaee9abea |
| Byte Size | 6082 |
| Line Count | 192 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/config/index.js

- Zweck: Konfigurationslader: Pfadauflösung mit Priorität und Migration, YAML-Parsing, Validierung, Defaults und Ableitung abgeleiteter Felder (instances → workers-Flattung, maxConcurrent) (belegt durch Dateiinhalt, 351 Zeilen).
- Verantwortlichkeit: Einmalige (modul-gespeicherte) Konfigurationsextraktion; stellt sicher, dass die Konfiguration strukturell konsistent und für Backend/Server nutzbar ist.
- Eingaben: `config.yaml` aus `data/` oder Wurzel, `config.example.yaml`, Umgebung (Docker-Pfad), Benutzerverzeichnis-Markierungen.
- Ausgaben: Vollständig aufgelöstes, validiertes und mit Defaults angereichertes Konfigurationsobjekt inkl. geflatteter Worker-Liste und dynamischem maxConcurrent.
- Datenfluss: Pfadauflösung (Daten- > Wurzel- > Beispiel; Migration per rename/copy) → YAML-Parse → Docker-Pfad-Korrektur → Basisvalidierung (port, auth, keepalive, pool.strategy, failover, queue) → instances→workers-Flattung (mit Vererbung userDataDir/Proxy) → gemini_biz-Pflichtfeld → Log-Level → Cache.
- Persistenz: Lesend (einmalig); Migration: verschiebt Wurzel-config.yaml nach data/ oder kopiert das Beispiel (Schreibseiteneffekt beim ersten Lauf).
- Zustände: Modul-Level-Cache (Konfiguration wird nur einmal geladen); aktiver Config-Pfad.
- APIs: Konfigurationszugriff (load), Pfadabfrage, Hilfsfunktionen für Proxy-/Benutzerdaten-Auflösung.
- Ereignisse: Migration-/Kopierereignisse beim ersten Start (loggepflichtig).
- Nebenwirkungen: Datei-Migration/Kopie (rename/copy) beim ersten Lauf; Log-Warnungen bei schwachen Auth-Einstellungen; Log-Level-Anpassung des Loggers.
- Fehlerfälle: Fehlende Konfiguration (Throw), YAML-Parse-Fehler, fehlende Pflichtfelder (`server.port`, `backend.pool.instances` nicht leer), ungültiger Port, fehlende `entryUrl` bei gemini_biz-Workern, doppelte Worker-Namen, invalide Instance/Worker-Strukturen.
- Sicherheitsrelevanz: Warnungen bei fehlendem, standardmäßigem oder kurzem Auth-Token; bewusst keine Einführung von Defaults für das Auth-Token; Passwörter werden in Proxy-Konfigurationen übernommen (Config-Datei = Geheimnisträger).
- Geschäftslogik: Prioritäts-Pfadauflösung mit automatischer Migration; Instance-Proxy-Auflösung (Instance explizit deaktiviert → direkt; Instance aktiviert → Instance; sonst global; sonst direkt); Worker-Name-Globalität; merge-Worker benötigen mergeTypes; maxConcurrent wird aus der Worker-Anzahl abgeleitet (nicht konfigurierbar).
- Algorithmen: Flattung einer Instance/Worker-Hierarchie in eine flache Worker-Liste unter Vererbung von userDataDir und aufgelöstem Proxy; Enum-Whitelist-Prüfungen mit Fallback-Defaults.
- verwendete Datenmodelle: Konfigurationsbaum (server, browser, backend.pool (instances/workers/failover/strategy), backend.adapter, queue, logLevel); geflatteter Worker (name, type, mergeTypes, mergeMonitor, instanceName, userDataMark, userDataDir, resolvedProxy).
- Abhängigkeiten: `fs`, `path`, YAML-Parser, Logger (belegt durch Importliste Zeilen 10–14).
- Rust-Relevanz: Config-Schicht im Rust-Rewrite mit serde-yaml/`config`-Crate, PathBuf-Basis und Default-Deserialisierung; Validierung als typsichere Phase (Result statt Mutation); atomare Migration (write-temp + rename) statt rename/copy im Live-Betrieb; Enum-Typen statt String-Whitelists.

#### Evidence
- Evidence-ID: EV-WEB2API-000233 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/index.js | Zeilenbereich: 16–74 | Beziehung: Exportiert Pfadauflösung; wird von Manager und Backend genutzt | Typ: Config | Aussage: Konfigurationspfad wird mit Priorität `data/config.yaml` → Wurzel-`config.yaml` → `config.example.yaml` aufgelöst; Wurzel-Konfiguration wird nach `data/` migriert, Beispiel wird kopiert; Pfad wird gecacht.
- Evidence-ID: EV-WEB2API-000234 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/index.js | Zeilenbereich: 200–345 | Beziehung: Wird von `src/backend/index.js` (Zeile 34) als Backend-Konfiguration geladen | Typ: Config | Aussage: Parst YAML, korrigiert den Browser-Pfad im Container, validiert Port/Auth, setzt Keepalive-/Pool-/Failover-/Queue-Defaults, erzwingt nicht-leere Instances, flattet Instances zu Workern und berechnet maxConcurrent aus der Worker-Anzahl; Pflichtfeld-Prüfung für gemini_biz.
- Evidence-ID: EV-WEB2API-000235 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/index.js | Zeilenbereich: 81–194 | Beziehung: Interne Aufbereitung | Typ: Call | Aussage: Benutzerdaten-Verzeichnis-Auflösung (mit Markierung), Proxy-Auflösung mit Instance-Priorität, Instance-/Worker-Strukturvalidierung und Flattung mit Worker-Name-Globalitätsprüfung sind implementiert.
- Negativnachweis: Keine Schreibfunktion für reguläre Config-Änderungen (explizite Trennung: Loader nur lesend; Schreiben im Manager), kein Secret-Handling außer Übernahme in die Struktur. Belegt durch vollständige Lektüre der 351 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 118b128deff29a1dd89dab7031ee90defad731c82f66ec4ed816dc6a2cb04946 |
| Byte Size | 13214 |
| Line Count | 351 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/config/manager.js

- Zweck: Konfigurations-Manager: segmentierte Lese-/Schreibzugriffe auf die YAML-Config für die Admin-API (belegt durch Dateiinhalt, 333 Zeilen).
- Verantwortlichkeit: Transformiert Roh-YAML in die von der Admin-Oberfläche erwarteten Views und schreibt Validierte Änderungen zurück; liest bewusst ohne Cache (immer frische Werte).
- Eingaben: Roh-Config-Datei; Änderungspayloads je Segment (Server, Browser, Queue, Instances, Adapters, Pool).
- Ausgaben: Segment-Views (Server, Browser, Queue, Instances, Adapters, Pool); persistierte Config-Datei.
- Datenfluss: readRawConfig → View-Transformation → Antwort; bzw. Payload → Merge in Roh-Struktur → yaml-Serialisierung → Datei.
- Persistenz: Direkte Datei-Persistenz der vollständigen YAML (`data/config.yaml`) bei jeder Speicherung; Kommentare gehen bei Serialisierung verloren (dokumentierte Nebeneigenschaft).
- Zustände: Keine eigenen (bewusst cachelos); Zustand liegt in der Datei.
- APIs: Get-/Save-Funktionen für Server, Browser, Queue, Instances, Adapters, Pool.
- Ereignisse: Config-Save (geloggt); Admin-WebUI-Änderungen.
- Nebenwirkungen: Überschreibt die Config-Datei; Kommentar-Verlust; `waitTimeout` wird von Sekunden in Millisekunden konvertiert.
- Fehlerfälle: Fehlende Config-Datei (Throw beim Lesen); fehlende Segmente werden mit leeren Strukturen toleriert; ungültige Werte werden bei Get mit Defaults gefüllt (Validierung liegt im Validator, nicht hier).
- Sicherheitsrelevanz: Schreibt das Auth-Token (`server.auth`) und Proxy-Passwörter in die Config-Datei; Get-Views geben `authToken` explizit zurück (nur über die geschützte Admin-API erreichbar).
- Geschäftslogik: Get-Views mit Defaults (z. B. keepaliveMode comment, imageMarkdown false, Proxy-Felder, failover-Defaults); Save-Merge: nur übergebene Felder ändern; Adapters-Merge bewahrt andere Adapter-Konfigurationen; Instances-Save normalisiert die Struktur (nur aktive Proxy-Felder, merge-Felder bedingt).
- Algorithmen: Selektiver Feld-Merge; Sekunden→Millisekunden-Konvertierung; View-Normalisierung.
- verwendete Datenmodelle: Segment-Modelle der Admin-Oberfläche (siehe Router); Roh-YAML-Baum.
- Abhängigkeiten: `fs`, `path`, YAML-Parser, Logger, Pfadauflösung aus `config/index` (belegt durch Importliste Zeilen 6–10).
- Rust-Relevanz: Read-Modify-Write auf YAML mit Sperre (mutex/File-Lock) und atomarer Ersetzung; View- und Domänenmodell getrennt (serde für Views, Domänenmodell mit serde_default); Kommentar-Preservation nur, falls gewünscht (sonst dokumentierter Verlust).

#### Evidence
- Evidence-ID: EV-WEB2API-000236 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/manager.js | Zeilenbereich: 16–74 | Beziehung: Nutzt Pfadauflösung aus `config/index` | Typ: Call | Aussage: Liest die Roh-Config ungecacht, serialisiert sie mit 2-Intervall-Einrückung und schreibt sie zurück; Server-View (Port, Auth-Token, Keepalive, Log-Level, Image-Markdown) und selektiver Save-Merge sind implementiert.
- Evidence-ID: EV-WEB2API-000237 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/manager.js | Zeilenbereich: 76–171 | Beziehung: Admin-Router-Integration | Typ: Call | Aussage: Browser-View (Pfad, Headless, Fission, Cursor-Humanisierung, CSS-Injection-Optionen, Proxy mit Auth-Bool) sowie Queue-View (Buffer, Bildlimit) mit Save-Merges sind implementiert.
- Evidence-ID: EV-WEB2API-000238 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/manager.js | Zeilenbereich: 173–333 | Beziehung: Admin-Router-Integration | Typ: Call | Aussage: Instances-View/Save (mit Normalisierung und Bedingungslogik für merge), Adapters-View/Save (Merge statt Overwrite) und Pool-View/Save (Strategie, Wait-Timeout-Sekunden→ms, Failover-Optionen) sind implementiert.
- Negativnachweis: Keine Validierungslogik in dieser Datei (Validierung delegiert an `config/validator`), keine Default-Erzeugung der Config-Datei (Loader), keine atomare Schreibsemantik. Belegt durch vollständige Lektüre der 333 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 09eae0ebd244b1dd117505c059f33274ec4800dbdbdd6c08064c50a1417ab3cd |
| Byte Size | 10026 |
| Line Count | 333 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/config/validator.js

- Zweck: Frontend-Validierung von Config-Segmenten gegen strenge Typ-, Bereichs- und Referenzregeln (belegt durch Dateiinhalt, 297 Zeilen).
- Verantwortlichkeit: Verhindert das Persistieren ungültiger Konfigurationen über die Admin-API; stellt die Konsistenz von Instance-/Worker-Strukturen und Pool-Werten sicher.
- Eingaben: Segment-Payloads (Server, Browser, Instances, Pool, Adapters), Adapter-Registry (gültige Typen).
- Ausgaben: Validierungsergebnis `{valid, errors[]}`.
- Datenfluss: Payload → Feldweise Prüfung → Fehlersammlung → Entscheidung.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Validatoren für Server-, Browser-, Instances-, Pool- und Adapters-Config.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Zahlreiche, in Fehlerstrings beschriebene Fälle (Typfehler, Bereichsverletzungen, Duplikate, unbekannte Adapter-Typen, leere Listen, ungültige Regex-Zeichen in Benutzerdaten-Markierungen, nicht-https-URL).
- Sicherheitsrelevanz: `userDataMark` wird gegen eine Whitelist (alphanumerisch, `_`, `-`) validiert — verhindert Pfad-Injektion in Benutzerdaten-Verzeichnisse; Proxy-Port-Bereiche; Adapter-Typen werden gegen die Registry geprüft (kein beliebiger Typ).
- Geschäftslogik: Worker-Namen müssen global (über alle Instances) eindeutig sein; `merge`-Worker benötigen gültige, nicht-merge `mergeTypes` und einen `mergeMonitor`, der in `mergeTypes` enthalten ist; Instances dürfen nicht leer sein; Keepalive-/Log-Level-/Strategie-Enums werden whitelist-geprüft; Auth-Token nur als leer oder ≥10 Zeichen akzeptiert.
- Algorithmen: Set-basierte Duplikatprüfung; Regex-Whitelist; Enum-Mitgliedschaft; Bereichsprüfungen.
- verwendete Datenmodelle: Segment-Payloads (siehe Manager); Validierungsergebnis (valid, errors).
- Abhängigkeiten: Backend-Registry (Adapter-IDs) (belegt durch Importliste Zeile 6).
- Rust-Relevanz: Im Rust-Rewrite: Typ-basierte Deserialisierung mit serde-Validierung (Bereiche, Enums, Längen) ersetzt den Großteil; Querverweise (Worker-Typ gegen Registry, merge-Abhängigkeit) als Post-Deserialisierungs-Validierungsphase mit Sammel-Fehlern (aggregated errors).

#### Evidence
- Evidence-ID: EV-WEB2API-000239 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/validator.js | Zeilenbereich: 13–115 | Beziehung: Wird vom Admin-Router vor Save aufgerufen | Typ: Call | Aussage: Server-Validator (Port-Bereich, Auth-Token-Mindestlänge, Keepalive-/Log-Enums, Queue-Felder, Markdown-Bool) und Browser-Validator (Typen, Proxy-Protokoll-Wahl, Port-Bereich) sind implementiert.
- Evidence-ID: EV-WEB2API-000240 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/validator.js | Zeilenbereich: 122–229 | Beziehung: Nutzt die Adapter-Registry für gültige Typen | Typ: Call | Aussage: Instances-Validator erzwingt nicht-leere Strukturen, globale Worker-Namens-Eindeutigkeit, Registry-geprüfte Adapter-Typen, merge-Spezialregeln, `userDataMark`-Whitelist und Proxy-Bereiche.
- Evidence-ID: EV-WEB2API-000241 | Repository: WebAI2API | Commit: content-copy | Datei: src/config/validator.js | Zeilenbereich: 236–297 | Beziehung: Wird vom Admin-Router vor Save aufgerufen | Typ: Call | Aussage: Pool-Validator (Strategie-Enum, Failover-Integer-Bereiche) und Adapters-Validator (gemini_biz-URL muss https sein) sind implementiert.
- Negativnachweis: Keine Persistenz, keine Mutation der Payloads, keine Dateizugriffe. Belegt durch vollständige Lektüre der 297 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 96a591cb08e28a0120f577322e5a546ed54a33e3b06a3e059ec9bdcccf03e652 |
| Byte Size | 11435 |
| Line Count | 297 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/utils/constants.js

- Zweck: Zentraler Konstantenpool der Backend-Utils: Timeouts, Retry-Konfiguration und Humanisierungs-Delays (belegt durch Dateiinhalt, 89 Zeilen).
- Verantwortlichkeit: Einzige Quelle für zeitliche Schwellen und Retry-Entscheidungen der Browser-Automationsschicht.
- Eingaben: Keine.
- Ausgaben: Konstante Konfigurationswerte für Konsumenten.
- Datenfluss: Konstante Importe → Nutzung in Laufzeitlogik.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Timeout-Enum (Klick/Warte/Navigation/Scroll/Upload/OAuth/API-Antwort/Heartbeat/Poll), Retry-Enum (Versuche, Basis-Delay, retrybare Fehlertypen), Humanisierung-Delays (kurz/mittel/lang/Tippen).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine direkte; deterministische Timeouts begrenzen die Dauer riskanter Operationen.
- Geschäftslogik: Welche Fehlertypen als retrybar gelten (Netzwerk, Timeout, Page-Crash); Timeout-Definitionen für die Interaktion mit der Browser-Seite.
- Algorithmen: Keine.
- verwendete Datenmodelle: Konstante Enums/Tupel (Delay-Bereiche min/max).
- Abhängigkeiten: Keine (wird u. a. von `backend/utils/page.js`, `backend/engine/utils.js`, `backend/strategies/failover.js` importiert — belegt durch Import-Suche).
- Rust-Relevanz: Als Konstanten in Rust mit typisierten Enums (Duration-Konstanten, Retry-Policy-Struct) übernehmen; Delay-Bereiche als Verteilungs-Parameter für humanisierte Interaktion.

#### Evidence
- Evidence-ID: EV-WEB2API-000242 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/constants.js | Zeilenbereich: 14–89 | Beziehung: Wird von Backend-Seiten-/Engine-/Failover-Modulen importiert | Typ: Config | Aussage: Definiert Timeouts (25 s Klick, 20 s Navigation/Eingabe, 30 s Scroll, 60 s Upload/OAuth, 120 s API-Antwort, 3 s Heartbeat, 0,5 s Poll), Retry-Regeln (max. 2 Versuche, 1 s Basis-Delay, Netzwerk/Timeout/Crash retrybar) und Humanisierungs-Delay-Bereiche.
- Negativnachweis: Keine Logik, keine Persistenz, keine Abhängigkeiten. Belegt durch vollständige Lektüre der 89 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 03fbbfe74e73137db40820ee22c8bb668bfa4edfee1554343f9dacc7ac802029 |
| Byte Size | 1985 |
| Line Count | 89 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/utils/history.js

- Zweck: Request-Historien-Persistenz: SQLite-Datenbank für Anfragen/Antworten, lokale Medienablage und Retry-Downloads (belegt durch Dateiinhalt, 508 Zeilen).
- Verantwortlichkeit: Verwaltet den kompletten Lebenszyklus von Historieneinträgen (create/update/list/detail/delete), Medien-Materialisierung und Statistiken; bietet Modellliste und Medienverzeichniszugriff.
- Eingaben: Records (id, model, prompt, Bilder, streaming), Updates (Status, Antworttext, Reasoning, Medien, Fehler, Dauer), Filter/Paging, Daten-URIs bzw. URLs für Medien, Download-Funktionen.
- Ausgaben: Erstellte IDs, aktualisierte Records, Listen mit Paging, Details, Löschzahlen, Medienpfade, Statistiken, Modelllisten.
- Datenfluss: Init (Datenbank + Schema) → CRUD via vorbereitete Statements → JSON-Serialisierung eingebetteter Listen (input_images, response_media) → Dateisystem für Medien; Retry: Record → Indexprüfung → Datei-Existenzprüfung → Browser- oder HTTP-Download → Speicherung → Record-Update.
- Persistenz: SQLite-Datei `data/history/history.db`; Medien `data/history/media/`; Schema mit Indizes auf created_at, status, model_id.
- Zustände: Modul-Global `db`-Handle (lazy, einmalige Initialisierung); Record-Status pending/success/failed; Medien-Status pending/downloaded/failed/external.
- APIs: Init, Create, Update (dynamische Feldmenge), List (Filter/Paging), Detail, Delete (IDs), Delete-by-Date-Range, Medien-Speicherung, Medien-Verarbeitung, Retry-Download, Statistik, Modellliste, Medienverzeichnis.
- Ereignisse: Medien-Download-Ergebnisse; asynchrones Medien-Postprocessing (nicht-blockierend für die Response).
- Nebenwirkungen: Dateisystem-Schreibzugriffe (Datenbank, Medien); Datei-Löschungen beim Record-Delete; HTTP-Downloads im Retry-Fall (mit User-Agent-Header, 60 s Timeout).
- Fehlerfälle: Nicht initialisierte DB (Throw), ungültige JSON in gespeicherten Listen (tolerantes Parsing), fehlende Dateien beim Löschen (ignoriert), fehlende originalUrl bei Retry, HTTP-Fehlerstatus beim Retry, Datei-Existenzprüfung im Retry.
- Sicherheitsrelevanz: Prompt- und Antwortdaten (inkl. Reasoning) werden dauerhaft gespeichert (Datenschutz); Medien werden lokal abgelegt und über die Admin-API ausgeliefert; Retry-HTTP-Downloads führen authentifizierte URL-Fetches ohne Credential-Übergabe durch.
- Geschäftslogik: Base64-Bilder werden nicht als Text in die DB geschrieben (nur Verweise/URLs); Medien aus Markdown werden extrahiert und lokal gespeichert; Statistik (total/success/failed/avgDuration) mit Datumsfilter; Datumsgrenzen werden auf Tagesanfang/-ende normalisiert; Retry bevorzugt den Browser-Kontext, fällt auf HTTP mit Desktop-User-Agent zurück.
- Algorithmen: Dynamische WHERE-/UPDATE-Bildung mit Parametrierung; SQL-Aggregation für Statistiken; Regulärer Ausdruck zur Markdown-Bild-Extraktion; MIME→Endungs-Mapping; Daten-URI-Parsing.
- verwendete Datenmodelle: Historien-Record (id, created_at, model_id, model_name, prompt, input_images, response_text, reasoning_content, response_media, status, error_message, duration_ms, is_streaming); Medien-Info (type, originalUrl, localPath, status); Statistik-Summary.
- Abhängigkeiten: `better-sqlite3`, `fs/promises`, `path`, Logger (belegt durch Importliste Zeilen 6–9).
- Rust-Relevanz: Im Rust-Rewrite: `rusqlite` oder `sqlx` (sqlite) mit Migrationsschema, prepared statements, JSON-Spalten als serde; Medienablage über std::fs mit PathBuf-Normalisierung; Retry-Download über reqwest (mit konfigurierbarem UA); async, aber DB im gleichen Runtime-Task oder Pool.

#### Evidence
- Evidence-ID: EV-WEB2API-000243 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/history.js | Zeilenbereich: 22–66 | Beziehung: Wird von `src/server/server.js` (Zeile 141) beim Start initialisiert | Typ: Persistenz | Aussage: Erstellt Datenbank und Schema (requests-Tabelle, Indizes auf created_at/status/model_id) in `data/history/history.db`; Zugriff nach Initialisierung über Pflichtprüfung.
- Evidence-ID: EV-WEB2API-000244 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/history.js | Zeilenbereich: 73–212 | Beziehung: Wird vom Queue-Manager und Admin-Router genutzt | Typ: Call | Aussage: Record-Anlage, dynamisches Feld-Update, gefilterte Liste mit Paging (Status, Modell, Freitext, Datumsbereich) und Detail-Abfrage mit JSON-Dekodierung eingebetteter Listen sind implementiert.
- Evidence-ID: EV-WEB2API-000245 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/history.js | Zeilenbereich: 219–508 | Beziehung: Medien- und Statistiklogik | Typ: Persistenz | Aussage: Löschung nach IDs/Date-Range (inkl. Medien-Datei-Bereinigung), Medien-Speicherung aus Daten-URIs, Markdown-Medien-Extraktion, Retry-Download (Browser-First, HTTP-Fallback), Statistikaggregation und Modell-Distinct-Liste sind implementiert.
- Negativnachweis: Keine Verschlüsselung der DB/Medien, keine externen DB-Systeme, kein Backup-Mechanismus. Belegt durch vollständige Lektüre der 508 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | fd898865b9a95b5b7c25d61df66f6b240f5b5492d5c190bad5dd1b0b8bf6660e |
| Byte Size | 16069 |
| Line Count | 508 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/utils/ipc.js

- Zweck: IPC-Kommunikation mit dem Supervisor-Prozess über Unix-Socket/Netzwerk-Socket (belegt durch Dateiinhalt, 113 Zeilen).
- Verantwortlichkeit: Senden von Restart-/Stop-Signalen und Abfrage der VNC-Informationen an den überwachenden Supervisor.
- Eingaben: Env-Variable `SUPERVISOR_IPC` (Socket-Pfad), optionale Extra-Argumente für Restart.
- Ausgaben: Boolean-Erfolg (Restart/Stop), VNC-Info-Objekt oder null.
- Datenfluss: Kommando-String → Socket-Verbindung → Supervisor → Antwort (bei VNC: JSON über `end`-Event).
- Persistenz: Keine.
- Zustände: Keine (jede Operation öffnet eine eigene Verbindung).
- APIs: Restart-Signal (mit Argumenten), Stop-Signal, Supervisor-Erkennung, VNC-Info-Abfrage.
- Ereignisse: Socket-connect, -data, -end, -error; 3-s-Timeout bei VNC-Abfrage.
- Nebenwirkungen: Kontaktiert den Supervisor-Socket; bei Verbindungsfehlern toleranter Abschluss (false/null statt Exception).
- Fehlerfälle: Fehlender `SUPERVISOR_IPC` (false/null + Warnung), Verbindungsfehler, nicht-parsbares VNC-JSON (null), Timeout (null).
- Sicherheitsrelevanz: Protokoll ist unauthentifiziert auf dem lokalen Socket (nur über Dateisystemrechte abgesichert); keine Secrets im Protokoll.
- Geschäftslogik: Restart-Kommando kann mit durch Leerzeichen getrennten Argumenten versehen werden; VNC-Info nur unter Supervisor verfügbar.
- Algorithmen: JSON-Parsing der VNC-Antwort; Timeout-Handling.
- verwendete Datenmodelle: VNC-Info (enabled, port, display, xvfbMode); Kommando-Strings (RESTART[:arg], STOP, GET_VNC_INFO).
- Abhängigkeiten: `net` (belegt durch Importliste Zeile 6).
- Rust-Relevanz: Im Rust-Rewrite: Unix-Domain-Socket (tokio::net::UnixStream) mit klarem Request/Response-Protokoll (z. B. NEWLINE-terminiert oder length-prefixed statt Leerzeichen-Join), Timeouts und Fehler-Enum; Supervisor-Prozess als separater Binary-Teil.

#### Evidence
- Evidence-ID: EV-WEB2API-000246 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/ipc.js | Zeilenbereich: 13–113 | Beziehung: Wird vom Admin-Router (Restart, VNC) und vom VNC-Proxy genutzt | Typ: Call | Aussage: Restart-/Stop-Signale (mit Argument-Join für Restart), Supervisor-Erkennung über die Env-Variable und eine JSON-basierte VNC-Info-Abfrage mit 3-s-Timeout sind implementiert; Fehler führen zu toleranten Rückgabewerten.
- Negativnachweis: Keine Persistenz, kein Bidirektionales Long-Lived-Protokoll (pro Operation eine Verbindung), keine Authentifizierung. Belegt durch vollständige Lektüre der 113 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | d316b253ad4f769f3b2a0a2c0689a6b064db71a81943125ff3abed41b4f0caff |
| Byte Size | 2882 |
| Line Count | 113 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/utils/logger.js

- Zweck: Strukturiertes Logging: gefärbte, level-basierte Konsolenausgabe mit Timestamp/Modul/Meta sowie Datei-Mirroring mit Größen-Rotation (belegt durch Dateiinhalt, 243 Zeilen).
- Verantwortlichkeit: Einheitliche Log-Schnittstelle aller Module; Filterung nach Level; Persistenz in `data/logs/system.log`; Admin-Zugriff auf Logs (Lesen/Löschen).
- Eingaben: Level, Modul, Nachricht, Meta-Objekt; Env-Variable `LOG_LEVEL`; Laufzeit-Level-Setzung.
- Ausgaben: Konsolenzeilen (ANSI-gefärbt) und Dateizeilen (ohne Farbe) im Format `YYYY-MM-DD HH:mm:ss.SSS [LVL] [Modul] [front-meta] msg | k=v...`.
- Datenfluss: Log-Aufruf → Level-Filter → Formatierung (Timestamps, Meta-Serialisierung, Newline-Sanitisierung) → Konsole (je Level stdout/stderr) + Datei-Append (mit Rotation).
- Persistenz: Datei-Persistenz `data/logs/system.log`; Rotation: bei ≥5 MB wird die aktuelle Datei zu `system.log.old` (altes .old gelöscht); Log-Pfad-Abfragen; Lesen der letzten N Zeilen; Löschen beider Dateien.
- Zustände: Aktuelles Log-Level (initial aus `LOG_LEVEL`, sonst info; Laufzeit-Änderung möglich).
- APIs: debug/info/warn/error (mod, msg, meta), Level-Setter, Log-Pfad-Abfragen, Log-Lesen (letzte N), Log-Löschen.
- Ereignisse: Rotation (5-MB-Schwelle).
- Nebenwirkungen: Schreibt stets auf die Festplatte; verschiebt/löscht Log-Dateien; Fehler beim Schreiben/Rotation werden still ignoriert (nicht-fatal).
- Fehlerfälle: Log-Datei-/Verzeichnisprobleme (ignoriert), ungültiges Env-Level (Fallback auf info), zirkuläre Meta-Objekte (als `[Circular]` markiert), Error-Objekte im Meta (nur Message).
- Sicherheitsrelevanz: Logs können sensible Daten enthalten (Prompts, Modellnamen, Fehlertexte); Zugriff über Admin-API; keine Secret-Redaktion implementiert (im Rust-Rewrite bewusst behandeln).
- Geschäftslogik: Front-Meta-Felder (id, adapter, model) werden prominent in Klammern gesetzt, restliches Meta als `k=v`; Newlines werden in der Datei zu ` ↵ ` sanitisiert (Ein-Zeilen-Log).
- Algorithmen: Level-Filterung über Indexvergleich; ANSI-Farbwahl je Level; Zeitformatierung mit Pad; Byte-Größen-Vergleich für Rotation; Zeilen-Slice für das Lesen.
- verwendete Datenmodelle: Log-Eintrag (Timestamp, Level, Modul, Meta); Log-Abfrageergebnis (logs, total, file).
- Abhängigkeiten: `process`, `fs`, `path` (belegt durch Importliste Zeilen 10–12); wird von nahezu allen Server-/Config-/Backend-Modulen importiert (Import-Suche belegt 40+ Nutzer).
- Rust-Relevanz: Im Rust-Rewrite: `tracing`/`tracing-subscriber` mit env-filter und Datei-Writer (rollover in eigener Implementierung oder `rolling-file`-Crate); strukturierte Felder statt String-Interpolation; keine Geheimnisse ohne Redaction.

#### Evidence
- Evidence-ID: EV-WEB2API-000247 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/logger.js | Zeilenbereich: 14–114 | Beziehung: Importiert von >40 Modulen (Server/Config/Backend) | Typ: Import | Aussage: Level-basierte Filterung (Env `LOG_LEVEL`, Laufzeit-Setter), Zeitformatierung, ANSI-Färbung, 5-MB-Rotation (`system.log` → `.old`) und Datei-Mirroring sind implementiert; Schreibfehler sind nicht-fatal.
- Evidence-ID: EV-WEB2API-000248 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/logger.js | Zeilenbereich: 118–242 | Beziehung: Admin-Router nutzt Log-Lesen/-Löschen | Typ: Call | Aussage: Formatiert Front-Meta prominent und Rest-Meta als `k=v` (Error/Circular-Sondercases), sanitisiert Newlines, schreibt in Konsole + Datei und bietet Pfadabfrage, Rücklesen der letzten N Zeilen sowie Löschen der Log-Dateien an.
- Negativnachweis: Keine asynchrone Logging-Warteschlange, keine Remote-Logging, keine Secret-Redaktion. Belegt durch vollständige Lektüre der 243 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | ce9a2b9ab789c49b1a14375b1456b5fceec8b7c3a489349a9b4af063292d9358 |
| Byte Size | 6728 |
| Line Count | 243 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/utils/proxy.js

- Zweck: Proxy-Adapter: Konvertierung von HTTP-/SOCKS5-Proxy-Konfigurationen in browser- und Playwright/Camoufox-taugliche Formate inkl. SOCKS5-zu-HTTP-Brücke (belegt durch Dateiinhalt, 150 Zeilen).
- Verantwortlichkeit: Aufbereitung der Proxy-Konfiguration für die Browser-Engine und Lifecycle-Verwaltung der local anonymisierten Proxy-Brücke.
- Eingaben: Proxy-Konfigurationsobjekt (enable, type, host, port, user, passwd), globale Config.
- Ausgaben: Proxy-URL-Strings (mit/ohne Auth, http/socks5), lokale anonymisierte HTTP-Proxy-URL (bei SOCKS5), Playwright/Camoufox-taugliche Proxy-Repräsentation, null bei Deaktivierung.
- Datenfluss: Config → URL-Bau (Auth-URL-Encoding) → direkt (HTTP) oder proxy-chain-Anonymisierung (SOCKS5 → lokaler HTTP-Proxy) → Ausgabe; Cleanup schließt die Brücke.
- Persistenz: Keine.
- Zustände: Modul-Globaler Proxy-Zustand (anonymisierte URL + Original-URL) für die spätere Bereinigung.
- APIs: URL-Bau, HTTP-Proxy-Auflösung, Browser-Proxy-Auflösung, Cleanup, Config-Extraktion.
- Ereignisse: SOCKS5-Konvertierung; Cleanup.
- Nebenwirkungen: Startet/beendet einen lokalen Proxy-Server über proxy-chain (Netzwerk-/Port-Ressourcen); Logging der Proxy-URL (enthält ggf. Auth-Daten im Log).
- Fehlerfälle: SOCKS5-Konvertierungsfehler (Fehler wird weitergereicht), nicht unterstützter Typ (Warnung + null), fehlende/deaktivierte Config (null).
- Sicherheitsrelevanz: Proxy-Zugangsdaten werden in URL-Strings eingebettet (URL-Encoding von User/Passwort); Log-Ausgaben der Proxy-URL können Zugangsdaten exponieren — Sicherheitsrelevante Schwäche, im Rust-Rewrite redigieren; keine persistente Speicherung.
- Geschäftslogik: Nur aktivierte Proxies werden berücksichtigt; HTTP-Proxies werden unverändert durchgereicht; SOCKS5 wird einmalig in einen lokalen HTTP-Proxy übersetzt; Camoufox-spezifische URL-Behandlung (Sonderfall `new URL().origin` bei socks5).
- Algorithmen: URL-Komposition; Auth-URL-Encoding; HTTP(S)-Brücken-Management.
- verwendete Datenmodelle: Proxy-Config (enable, type, host, port, user, passwd); Proxy-Zustand (anonymizedUrl, originalUrl).
- Abhängigkeiten: `proxy-chain`, Logger (belegt durch Importliste Zeilen 6–7); Konsument: `src/backend/engine/launcher.js` (Import-Suche).
- Rust-Relevanz: Im Rust-Rewrite: Proxy-URL-Komposition als typisierte Struktur; SOCKS5→HTTP-Brücke entfällt, da die Browser-Steuerung (CDP) und der Netzwerk-Stack SOCKS5 direkt unterstützen (oder eigene minimale Brücke); Credentials mit URL-Encoding und Redaction im Logging.

#### Evidence
- Evidence-ID: EV-WEB2API-000249 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/proxy.js | Zeilenbereich: 25–82 | Beziehung: Wird von `src/backend/engine/launcher.js` (Zeile 20) importiert | Typ: Call | Aussage: Baut Proxy-URLs (mit/ohne Auth, http/socks5) und löst den effektiven HTTP-Proxy auf: HTTP direkt, SOCKS5 über die lokale Anonymisierungs-Brücke (proxy-chain); deaktivierte Konfiguration liefert null.
- Evidence-ID: EV-WEB2API-000250 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/proxy.js | Zeilenbereich: 90–150 | Beziehung: Browser-Engine-Integration | Typ: Call | Aussage: Liefert die browser-taugliche Proxy-Repräsentation (URL-String mit Auth-Encoding, Camoufox-Sonderfall), bereinigt die lokale Brücke über den modul-globalen Zustand und extrahiert die aktive Proxy-Config aus der Gesamtkonfiguration.
- Negativnachweis: Keine Persistenz, keine Proxy-Rotation, keine Validierung von Ziel-/Loopback-Adressen. Belegt durch vollständige Lektüre der 150 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | b75908cb21cd50199f97d723198ccebba0113664a19779d26dcf61a148049639 |
| Byte Size | 5142 |
| Line Count | 150 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/utils/stats.js

- Zweck: Tagesstatistik: persistente Erfolgs-/Fehlerzähler pro Kalendertag mit Datumswechsel, Bereichs-Aggregation und Löschung (belegt durch Dateiinhalt, 183 Zeilen).
- Verantwortlichkeit: Führt die Statistik der Generate-Aufträge und stellt sie der Admin-API bereit.
- Eingaben: Ereignis-Aufrufe (Erfolg/Fehler), Datumsbereiche (YYYY-MM-DD).
- Ausgaben: Tageszähler, Bereichs-Summen (success, failed, days), Löschzahlen.
- Datenfluss: Zähler-Inkrement → Datumswechsel-Check → Datei-Save `data/logs/stats_<date>.json`; Bereichsabfrage: Datumsschleife → Datei-Load → Summierung; Löschung: Dateien entfernen + ggf. Cache-Reset.
- Persistenz: JSON-Datei je Datum unter `data/logs/`; in-memory Cache für den heutigen Tag.
- Zustände: `todayStats`-Zähler und `todayDate` (Modul-Global); Datumswechsel löscht/neu lädt den Cache.
- APIs: Tageslast, Erfolgs-Inkrement, Fehler-Inkrement, Tagesabfrage, Bereichs-Abfrage, Bereichs-Löschung.
- Ereignisse: Datumswechsel (Rollover mit Speichern des Vortags); Tagesstatistik-Neuladung.
- Nebenwirkungen: Schreibt bei jedem Inkrement die Tagesdatei (Sync-basiert über Promises); löscht Statistikdateien im Bereich; bei Dateifehlern tolerante Defaults.
- Fehlerfälle: Fehlende/korrumpierte Tagesdateien (Default-Zähler, Datei wird beim Inkrement neu geschrieben), nicht existierende Dateien im Bereich (übersprungen), gelöschte heutige Datei (Cache-Reset).
- Sicherheitsrelevanz: Keine; enthält nur Zähler (keine personenbezogenen Daten).
- Geschäftslogik: Datumswechsel-Semantik: Vor-Inkrement wird der alte Tag persistiert und der heutige neu geladen; Tagesabfrage bei Wechsel liefert Null-Werte bis zur nächsten Schreiboperation.
- Algorithmen: ISO-Datumsformatierung (lokale Zeit), Datums-Iteration über Date-Bereich, JSON-Persistierung/-Parsing.
- verwendete Datenmodelle: Tageszähler (success, failed); Bereichsergebnis (success, failed, days); Löschergebnis (deleted).
- Abhängigkeiten: `fs/promises`, `path` (belegt durch Importliste Zeilen 6–7); Konsumenten: Queue-Manager, Server-Start, Admin-Router.
- Rust-Relevanz: Im Rust-Rewrite: Tagesstatistik aus SQLite oder einzelnen Dateien mit atomarem Write (write-temp+rename); Datumswechsel über Chrono/Time; in-memory Cache mit Mutex/RwLock; asynchrones Inkrementieren ohne globalen Singleton (in App-State).

#### Evidence
- Evidence-ID: EV-WEB2API-000251 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/stats.js | Zeilenbereich: 17–89 | Beziehung: Wird vom Server-Start (Zeile 137) und Queue-Manager genutzt | Typ: Persistenz | Aussage: Persistiert Zähler als JSON je Datum unter `data/logs/`, hält einen In-Memory-Cache für heute und führt beim Datumswechsel Speicherung des Vortags, Reset und Neuladen durch.
- Evidence-ID: EV-WEB2API-000252 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/stats.js | Zeilenbereich: 91–183 | Beziehung: Admin-Router und Queue-Manager | Typ: Call | Aussage: Erfolgs-/Fehler-Inkremente (mit Datei-Save), Tagesabfrage mit Wechsel-Check, Datumsbereichs-Aggregation (success/failed/days) und Bereichs-Löschung inkl. Cache-Reset sind implementiert.
- Negativnachweis: Keine Nutzer-/Auftrags-Details in der Statistik, keine Verzahnung mit der Historien-DB (separate Zählungen). Belegt durch vollständige Lektüre der 183 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 273b95c8a6938e7c18c7db656b74bad065a4a5fa86274bd2f5170f46c12f07ff |
| Byte Size | 4867 |
| Line Count | 183 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |

---

## src/utils/systemInfo.js

- Zweck: Systemzustands- und Datenordner-Verwaltung: Statusaggregation (CPU/Memory/Uptime/Mode/Version) sowie Auflistung, Bereinigung und Löschung von Benutzerdaten- und Temp-Ordnern (belegt durch Dateiinhalt, 256 Zeilen).
- Verantwortlichkeit: Liefert der Admin-API die System-Metrikdaten und führt destruktive, aber abgesicherte Dateisystem-Operationen auf dem Datenordner aus.
- Eingaben: Worker-Konfigurationen (für userDataDir-Zuordnung und In-Use-Erkennung), Temp-Verzeichnispfad, Folder-Namen (bei Löschung).
- Ausgaben: Statusobjekt (Modus, Version, System, Uptime, CPU%, Memory-MB), Ordnerliste (Name, Pfad, Größe formatiert/roh, zugehörige Instance), Löschergebnis (success, deleted, errors), Bereinigungszahlen.
- Datenfluss: Status: os-APIs + Prozess-Env + package.json-Version → Aggregation; Ordner: readdir(data) → Filter `camoufoxUserData*` → Größenberechnung (Tiefenbegrenzung) → Worker-Zuordnung; Löschung: Präfix-/In-Use-Prüfung → rekursive Löschung.
- Persistenz: Keine eigene (nur lesend/schreibend am Dateisystem).
- Zustände: Startzeit (Modul-Konstante), letzte CPU-Stichprobe (Modul-Global) für Deltaberechnung.
- APIs: Status, Datenordner-Liste, Datenordner-Löschung, Temp-Bereinigung.
- Ereignisse: Keine.
- Nebenwirkungen: Löscht rekursiv Ordner (nur mit `camoufoxUserData`-Präfix, nur nicht aktive); löscht Dateien im Temp-Verzeichnis; Logging der Löschungen/Bereinigungen.
- Fehlerfälle: Fehlendes Daten-/Temp-Verzeichnis (leere Ergebnisse), Ordner in Verwendung (Fehlerliste), nicht erlaubter Ordnername (Fehlerliste), nicht existenter Ordner (Fehlerliste), Rechenfehler bei CPU-Deltas (Null-Delta → 0).
- Sicherheitsrelevanz: Destruktive Löschung ist doppelt abgesichert (Präfix-Whitelist + In-Use-Prüfung); verhindert das Löschen fremder Verzeichnisse.
- Geschäftslogik: Modus-Erkennung über Env (Xvfb aktiv, Headless aktiv); CPU-Auslastung als Delta zweier Stichproben; nur Verzeichnisse mit Browser-Benutzerdaten-Präfix werden verwaltet; Instances werden über den userDataDir-Basisnamen zugeordnet.
- Algorithmen: CPU-Delta (idle/total-Anteil über alle Kerne), rekursive Ordnergröße mit Maximaltiefe 3, Größenformatierung (B/KB/MB/GB), Basisnamen-Zuordnung.
- verwendete Datenmodelle: Systemstatus-Objekt (status, version, systemVersion, uptime, cpuUsage, memoryUsage), Ordner-Eintrag (name, path, size, sizeBytes, instance), Löschergebnis (success, deleted, errors).
- Abhängigkeiten: `os`, `fs`, `path`, Logger (belegt durch Importliste Zeilen 6–9); Konsument: Admin-Router.
- Rust-Relevanz: Im Rust-Rewrite: `sysinfo`-Crate für CPU/Memory/System, std::fs mit Tiefenbegrenzung, PathBuf-Validierung vor Löschung (Canonicalize + Präfix-Prüfung), Prozess-Uptime über std::time::Instant.

#### Evidence
- Evidence-ID: EV-WEB2API-000253 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/systemInfo.js | Zeilenbereich: 12–86 | Beziehung: Wird vom Admin-Router (Zeile 92) für `/admin/status` genutzt | Typ: Call | Aussage: Aggregiert Status (Xvfb-/Headless-Modus aus Env, Versions-/System-/Uptime-Angaben, CPU-Auslastung als Delta zweier Stichproben, Memory in MB) und Version aus package.json.
- Evidence-ID: EV-WEB2API-000254 | Repository: WebAI2API | Commit: content-copy | Datei: src/utils/systemInfo.js | Zeilenbereich: 88–216 | Beziehung: Admin-Router-Endpunkte `/admin/data-folders` und `/admin/cache/clear` | Typ: Call | Aussage: Listet Browser-Benutzerdaten-Ordner mit Größenberechnung (Tiefe 3) und Instance-Zuordnung, löscht Ordner nur mit Präfix- und In-Use-Schutz und bereinigt Temp-Dateien; Hilfsfunktionen für Größen- und Formatierung.
- Negativnachweis: Keine Netzwerk-, Prozess- oder Container-Metriken über das Env-Flag hinaus, keine persistenten Zustände. Belegt durch vollständige Lektüre der 256 Zeilen.

#### Read Evidence
| Feld | Wert |
|---|---|
| File Hash | 2fabb86861fc964a1a0925d859ee698c37334c1cd02c75543d91513d53c5d5e4 |
| Byte Size | 7225 |
| Line Count | 256 |
| Encoding | UTF-8 |
| Read Timestamp | 2026-08-05 |
| Reader Result | OK |
