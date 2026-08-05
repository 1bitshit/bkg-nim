# WebAI2API — Teil 5: Wurzeldateien, Skripte, Patches und CI

Alle Pfade relativ zu `repos/WebAI2API/`. Umfang: 19 Dateien (Wurzel, scripts/, patches/, .github/workflows/).
Analyse nur von Fähigkeiten und Konzepten — keine Codeübernahme. Commit-Kennung für alle Belege: `content-copy`.

---

## supervisor.js

- Zweck: Prozess-Wachhund (Supervisor), der den API-Server als Kindprozess startet, überwacht und bei abnormalem Beenden automatisch neu startet; auf Linux zusätzlich Start der virtuellen Anzeige (Xvfb) und eines VNC-Servers (belegt durch Dateiinhalt, 428 Zeilen).
- Verantwortlichkeit: Lebenszyklus-Management des Serverprozesses; IPC-Schnittstelle für externe Neustart-/Stopp-Kommandos; Kollisionsvermeidung für Ports und X-Display-Nummern; VNC-Statusauskunft.
- Eingaben: Kommandozeilenargumente (einschließlich der Schalter für virtuelle Anzeige und VNC sowie Login-Parameter-Weitergabe); IPC-Kommandos über lokalen Socket beziehungsweise Named Pipe; Exit-Codes und Signale des Kindprozesses; Dateisystemzustand der X-Server-Lock-/Socket-Dateien.
- Ausgaben: gestarteter beziehungsweise neu gestarteter Serverprozess; IPC-Antworten (OK, UNKNOWN_COMMAND, VNC-Status als JSON); zeitgestempelte Logzeilen mit Wachhund-Kennzeichnung; Prozess-Exit-Codes.
- Datenfluss: Kommandozeile → Supervisor → Spawn des Serverprozesses; WebUI/Fremdprozess → IPC-Socket → Supervisor → Signal/Neustart; Supervisor → xvfb-run → X-Server; x11vnc → VNC-Port; Kindprozess erhält die IPC-Pfad-Umgebungsvariable.
- Persistenz: Keine dauerhafte Speicherung; temporäre IPC-Socket-Datei, die beim Start neu angelegt und vorher gelöscht wird.
- Zustände: laufend/gestoppt; Neustart-Flag (verhindert Doppel-Restarts); zwischengespeicherte Neustart-Argumente; VNC-Statusstruktur (aktiv, Port, Display, Anzeige-Modus); Referenz auf den Kindprozess; Tabelle der als fatal klassifizierten Exit-Codes.
- APIs: IPC-Kommandoprotokoll über Unix-Socket (Windows: Named Pipe) mit den Kommandos RESTART, RESTART mit Argumenten, STOP, GET_VNC_INFO.
- Ereignisse: Kindprozess-Exit; Kindprozess-Spawn-Fehler; IPC-Datenempfang; System-Signale (Unterbrechen/Beenden); Exit von xvfb-run und x11vnc.
- Nebenwirkungen: erzeugt und entfernt Socket-Dateien; setzt Umgebungsvariablen für den Kindprozess; beendet Kindprozesse per Beenden-Signal; setzt Display- und Anzeige-Lauf-Umgebungsvariablen.
- Fehlerfälle: fehlende Serverdatei → sofortiger Exit; als fatal klassifizierter Exit-Code (78) → kein automatischer Neustart und Weitergabe des Codes; Spawn-Fehler → Exit; fehlendes xvfb-run-Kommando → Exit mit Installationshinweis; nicht verfügbare Ports/Displays → Ausweichpfade (Zufallsdisplay, VNC-Skip).
- Sicherheitsrelevanz: VNC startet mit lokaler Bindung (nur Loopback), ohne Passwort, mit Freigabe mehrerer Clients und dauerhaftem Betrieb; Portprüfung ausschließlich auf Loopback; IPC-Endpunkt liegt im temporären Verzeichnis ohne Zugriffskontrolle — Negativbeleg: der IPC-Handler enthält keine Token- oder Authentifizierungsprüfung, jeder lokale Prozess kann Neustart/Stopp anstoßen.
- Geschäftslogik: Unterscheidung zwischen normalem Exit (kein Neustart, Wachhund beendet sich), abnormalem Exit (automatischer Neustart mit konfigurierter Verzögerung) und fatalem Exit-Code (kein Restart); Doppel-Neustart-Anforderungen werden während laufender Neustarts verworfen.
- Algorithmen: sequenzielle Portsuche ab Startwert mit begrenzter Versuchszahl; sequenzielle Wahl freier X-Display-Nummern ab 50 mit zufälligem Ausweichwert; Zerlegung der Neustart-Argumente aus dem IPC-Kommando.
- verwendete Datenmodelle: VNC-Statusstruktur (aktiv/Port/Display/Anzeige-Modus); IPC-Kommandostrings; Exit-Code-Tabelle.
- Abhängigkeiten: Node-Standardmodule für Prozess-Spawning, Netzwerk, Betriebssystem, Pfad und Dateisystem; externe Systemkommandos xvfb-run und x11vnc auf Linux.
- Rust-Relevanz: Das Wachhund-Muster (Kindprozess-Supervision, Neustartpolitik, Signal-Handling, IPC-Steuerkanal, Fatal-Code-Klassifikation) ist in Rust vollständig neu zu bauen (asynchrone Prozesssteuerung, Unix-Sockets, Signal-Behandlung); Start externer X- und VNC-Prozesse bleibt ein Prozess-Start-Feature.

Belege:
- EV-WEB2API-000401 | WebAI2API | content-copy | supervisor.js | 194-243 | Serverprozess-Supervision | Call | Aussage: Der Wachhund startet den API-Server als Kindprozess, vererbt die Umgebung plus IPC-Pfad und leitet bei abnormalem Exit einen automatischen Neustart mit Verzögerung ein; Exit-Code 78 ist als nicht erholbar klassifiziert und beendet den Wachhund.
- EV-WEB2API-000402 | WebAI2API | content-copy | supervisor.js | 137-181 | Steuerkanal | Event | Aussage: Der IPC-Server akzeptiert über einen lokalen Socket beziehungsweise Named Pipe die Kommandos RESTART, RESTART mit Argumenten, STOP und GET_VNC_INFO; unbekannte Kommandos erhalten UNKNOWN_COMMAND; es ist keine Authentifizierung vorgesehen.
- EV-WEB2API-000403 | WebAI2API | content-copy | supervisor.js | 287-385 | Xvfb/VNC-Verwaltung | Call | Aussage: Auf Linux startet der Wachhund eine virtuelle Anzeige über xvfb-run mit fester Bildschirmgröße und optional einen VNC-Server über x11vnc mit lokaler Bindung und ohne Passwort; verfügbare Ports und Display-Nummern werden kollisionsfrei gewählt.

Read-Evidence: File Hash: 3d7983371bdf794169be2dafe188956ed3c78eb828ef91baa0995d8c24cccdbb | Byte Size: 12051 | Line Count: 428 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## package.json

- Zweck: Manifest des Node-Pakets webai-2api (Version 3.0.0, MIT, ESM-Modulsystem); definiert Start- und Werkzeugskripte, interne Import-Aliase und die Laufzeit-Abhängigkeitsliste (belegt durch Dateiinhalt, 34 Zeilen).
- Verantwortlichkeit: koppelt npm-Aufrufe an die Skripte (Start → Supervisor; Initialisierung; Schlüsselerzeugung; Installations-Hook); definiert Pfad-Aliase für modulinterne Imports; deklariert die Abhängigkeiten für Browser-Automation, Proxy-Verbindungen, Bildverarbeitung, SQLite, YAML und interaktive Konsolenabfragen.
- Eingaben: Build-Zeit: Skriptkommandos von npm/pnpm; Laufzeit: keine direkten Eingaben.
- Ausgaben: Start-/Werkzeugkommandos; Auslösung des Installations-Hooks.
- Datenfluss: npm-Start → Supervisor; Initialisierungsskript; Schlüsselwerkzeug; npm-Installation → Installations-Hook.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine (Manifest ohne Laufzeit-API).
- Ereignisse: npm-Lifecycle-Hook nach Installation.
- Nebenwirkungen: Der Installations-Hook wendet nach der Installation Vendor-Patches an; das Modulsystem ist als ESM festgelegt.
- Fehlerfälle: Keine Laufzeit-Fehlerfälle.
- Sicherheitsrelevanz: Der Auth-Schlüssel wird durch ein separates Werkzeugskript mit kryptographischem Zufall erzeugt; die Abhängigkeitsgruppe enthält Proxy-Agents für HTTP und SOCKS5, woraus sich ein Proxy-Feature ergibt; das Manifest selbst enthält keine Geheimnisse.
- Geschäftslogik: Werkzeug-Gateway — alle zentralen Aufgaben (Start, Initialisierung, Schlüsselerzeugung, Patching) sind als Skripte erreichbar.
- Algorithmen: Keine.
- verwendete Datenmodelle: Manifest-Felder (Skripte, Import-Aliase, Abhängigkeiten).
- Abhängigkeiten: Node.js; Laufzeit-Abhängigkeiten: interaktive Konsolenabfragen, SQLite-Bindings, camoufox-Bibliothek, Archivkompression, Fingerprint-Erzeugung, Ghost-Cursor-Port, Scraping-HTTP-Client, HTTPS-Proxy-Agent, Playwright-Core, Proxy-Kette, Bildverarbeitung, SOCKS5-Proxy-Agent, YAML-Parser.
- Rust-Relevanz: Bestimmt den Werkzeug-/Feature-Umfang (Init-CLI, Schlüssel-Tool, Prozessstart, Patch-Anwendung), der im Rust-Rewrite eigene Binaries beziehungsweise Subkommandos erhält; die Abhängigkeitswahl (Proxy-Agents, YAML, Bildverarbeitung, SQLite) gibt die Crate-Orientierung vor.

Belege:
- EV-WEB2API-000404 | WebAI2API | content-copy | package.json | 8-34 | Skript- und Dependency-Manifest | Config | Aussage: Das Manifest bindet die Werkzeuge (Start, Schlüssel, Initialisierung, Installations-Hook) an npm-Skripte, definiert interne Import-Aliase und deklariert die Laufzeit-Abhängigkeiten einschließlich Proxy-Agents, SQLite, camoufox und Bildverarbeitung.

Read-Evidence: File Hash: e18acfd7ee58046f13f7f2976af70896cca92cb7a528aae917a331b312a78d5a | Byte Size: 934 | Line Count: 34 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## pnpm-workspace.yaml

- Zweck: pnpm-Workspace-Konfiguration: schließt das WebUI-Paket von der Wurzel-Arbeitsfläche aus und deklariert eine native Abhängigkeit als ignorierten Build-Kandidaten (belegt durch Dateiinhalt, 4 Zeilen).
- Verantwortlichkeit: steuert die pnpm-Installation an der Wurzel.
- Eingaben: Build-Zeit: pnpm.
- Ausgaben: Installationsentscheidung (welche Pakete zur Wurzel gehören).
- Datenfluss: pnpm → Workspace-Manifest.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Die SQLite-Bindung wird nicht automatisch gebaut — das Initialisierungsskript liefert vorgefertigte Binärdateien; das WebUI-Paket besitzt einen eigenen Workspace-Bereich.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Trennung der Dependency-Graphen von Wurzel und WebUI.
- Algorithmen: Keine.
- verwendete Datenmodelle: Liste ignorierter eingebauter Abhängigkeiten; Paket-Ausschlussmuster.
- Abhängigkeiten: pnpm mit Workspace-Unterstützung.
- Rust-Relevanz: Das Konzept der Graphen-Trennung (Backend versus WebUI) ist im Rust-Monorepo über getrennte Cargo-Workspaces beziehungsweise Profile abzubilden.

Beleg:
- EV-WEB2API-000405 | WebAI2API | content-copy | pnpm-workspace.yaml | 1-4 | Workspace-Konfiguration | Config | Aussage: Die Wurzel-Arbeitsfläche ignoriert den Auto-Build der SQLite-Bindung und schließt das WebUI-Paket vom Wurzel-Dependency-Graphen aus.

Read-Evidence: File Hash: 7c320b1487f1455e1e253d458a52d8bcfd061ed8a14f3f5d518fb3376f522e2b | Byte Size: 71 | Line Count: 4 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## pnpm-lock.yaml

- Zweck: Gepinnter Abhängigkeitsgraph (lockfileVersion 9.0) des Wurzel-Importers; verankert unter anderem die camoufox-Bibliothek 0.8.3 gekoppelt an Playwright-Core 1.57.0 sowie SQLite-Bindungen 12.5.0 (belegt durch Datei-Header, Zeilen 1-40; Gesamt 2149 Zeilen).
- Verantwortlichkeit: Reproduzierbare Installationen über den eingefrorenen Lockfile-Modus (belegt durch Dockerfile: pnpm-Install mit eingefrorenem Lockfile).
- Eingaben: Manifest und Paketregistrierungs-Metadaten.
- Ausgaben: Deterministische Installation.
- Datenfluss: Installationsgraph.
- Persistenz: Versionskontrolle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Der Lockfile-Modus bricht bei Abweichungen zum Manifest ab (belegt durch Dockerfile-Aufruf mit eingefrorenem Lockfile).
- Fehlerfälle: Lock-Drift bricht die Containerinstallation ab.
- Sicherheitsrelevanz: Pinned transitive Versionen inklusive nativer Binär-Abhängigkeiten; eine Integritätsprüfung des Inhalts ist hier nicht vorgesehen (bewusst nicht seziert).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Lockfile-Schema (Importer-Abschnitt, Versionsauflösungen, Peer-Kopplungen).
- Abhängigkeiten: pnpm; Wurzel-Manifest.
- Rust-Relevanz: Lockfile-Disziplin (eingefrorene Auflösungen, Reproduzierbarkeit) ist in der Rust-Build-Pipeline über eingefrorene Cargo-Lockfiles beziehungsweise Vendoring abzubilden.

Nicht-analysierbarer/Lockfile-Block:
- Typ: Lockfile
- Metadaten: Byte Size: 68951 | Line Count: 2149 | Encoding: UTF-8
- Verwendung: Gepinnter Dependency-Graph für den Wurzel-Importer; Inhalt wird gemäß Auftrag nicht zergliedert.

Beleg:
- EV-WEB2API-000406 | WebAI2API | content-copy | pnpm-lock.yaml | 1-40 | Lockfile | Schema | Aussage: Der Lockfile-Header deklariert das Format 9.0 und pinnt für den Wurzel-Importer die Abhängigkeiten einschließlich camoufox-js 0.8.3 in Peer-Kopplung an Playwright-Core 1.57.0 und SQLite-Bindungen 12.5.0; eine inhaltliche Zergliederung ist nicht vorgesehen.

Read-Evidence: File Hash: 6274d4765a40519e65b8d537a265dee11768db51483e8385391eefc9f5a64756 | Byte Size: 68951 | Line Count: 2149 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK (nur Header/Metadaten; Lockfile ohne Inhaltssektion)

---

## config.example.yaml

- Zweck: Referenz- und Beispielkonfiguration (YAML) für Server, Backend-Pool, Adapter, Warteschlange und Browser; dient beim ersten Start als Kopiervorlage für die effektive Konfigurationsdatei (belegt durch README und CHANGELOG 3.3.2; Datei 217 Zeilen).
- Verantwortlichkeit: definiert das öffentliche Konfigurationsschema mit Defaults und Kommentaren; dokumentiert unterstützte Adapter, Scheduler-Strategien, Failover-Regeln und Browser-Fingerprint-Optionen.
- Eingaben: Keine zur Laufzeit (Beispieldatei).
- Ausgaben: Vorlage für die tatsächliche Konfigurationsdatei.
- Datenfluss: Beispieldatei → Kopierlogik → effektive Konfigurationsdatei → Konfig-Lader.
- Persistenz: Vorlage; die wirksame Konfiguration liegt im Datenverzeichnis.
- Zustände: Keine.
- APIs: Keine direkt; beschreibt das Konfigurationsschema.
- Ereignisse: Keine.
- Nebenwirkungen: Wird beim Erststart in das Datenverzeichnis kopiert.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Der Auth-Schlüssel ist als Platzhalter mit Mindestlängen-Forderung hinterlegt; Proxy-Auth-Felder sind vorhanden; Kommentare markieren Fingerprint-Optionen mit Risikohinweisen (Auswirkungen auf Erkennbarkeit).
- Geschäftslogik: Scheduler-Strategien für die Pool-Verteilung; Failover-Regeln (Retry-Grenzen, separater Bild-Download-Retry); Warteschlangen-Pufferpolitik; Instanz-/Worker-Modell mit geteilten Browser-Daten pro Instanz; Merge-Worker mit kombinierten Backends und Leerlauf-Beobachtung.
- Algorithmen: Beschriebene Verteilstrategien (Least-Busy, Round-Robin, Zufall) und Failover-Wiederholungen.
- verwendete Datenmodelle: YAML-Topologie mit Server-, Backend-Pool-, Adapter-, Warteschlangen- und Browser-Abschnitten; Instanz-Worker-Struktur; Proxy-Objekte.
- Abhängigkeiten: YAML-Parser (laut package.json).
- Rust-Relevanz: Definiert das Konfig-Datenmodell (serde-Struktur) und die Defaults, die das Rust-Rewrite abbilden muss; Scheduler- und Failover-Semantik wird zur Policy-Konfiguration.

Beleg:
- EV-WEB2API-000407 | WebAI2API | content-copy | config.example.yaml | 1-217 | Konfigurationsschema | Config | Aussage: Die Beispieldatei definiert das Schema für Server (Port, Auth, Keepalive-Modus), Backend-Pool (Strategie, Failover, Instanzen/Worker inklusive Merge-Worker), Warteschlange (Puffer, Bildlimit) und Browser (Pfad, Headless, Humanisierung, Fission, CSS-Injection, globale und Instanz-Proxys) mit dokumentierten Defaults.

Read-Evidence: File Hash: 2d3ac8aaadc26f9915edac7fcf5c4547ffdc91e150bd2553fefabec1b6de4927 | Byte Size: 9267 | Line Count: 217 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## docker-compose.yaml

- Zweck: Container-Orchestrierung für den Dienst: startet das veröffentlichte Image mit Port-Mapping, Daten-Volume, erhöhtem Shared Memory und aktiviertem Container-Init (belegt durch Dateiinhalt, 11 Zeilen).
- Verantwortlichkeit: legt das Container-Laufzeitverhalten fest (Neustart-Policy, Persistenz-Volume, Init-Prozess).
- Eingaben: Docker Compose.
- Ausgaben: Container-Instanz.
- Datenfluss: Host-Volume verbindet Konfiguration, Browserdaten und Logs mit dem Container.
- Persistenz: Host-Volume mit dem Datenverzeichnis (effektive Konfigurationsdatei wird dort automatisch erzeugt).
- Zustände: Container-Lebenszyklus (Neustart sofern nicht gestoppt).
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: aktiviert den Init-Prozess des Containers (Zombie-Reaping).
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: exponiert ausschließlich den API-/WebUI-Port; der VNC-Port (5900) ist nicht nach außen gemappt — Negativbeleg: kein Port-Mapping für 5900 im Compose-Dateiinhalt; VNC bleibt damit nur container-intern beziehungsweise über andere Wege erreichbar.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Compose-Schema (Dienste, Ports, Volumes, Shared Memory, Init).
- Abhängigkeiten: Docker Compose; veröffentlichtes Image.
- Rust-Relevanz: Das Deployment-Muster (erhöhtes Shared Memory wegen Browser, Init-Prozess, Daten-Volume für Konfig und Zustand) ist im Rust-Container-Setting zu übernehmen.

Beleg:
- EV-WEB2API-000408 | WebAI2API | content-copy | docker-compose.yaml | 1-11 | Orchestrierung | Config | Aussage: Der Dienst startet aus dem veröffentlichten Image mit Port-Mapping 3000, einem Host-Volume für das Datenverzeichnis (dort entsteht die effektive Konfigurationsdatei), 2 GB Shared Memory und aktivem Init; ein Mapping für den VNC-Port existiert nicht.

Read-Evidence: File Hash: b11515e15e0cfce3d72e07d7289f7724ac5e9ee1d43d4de5d15b1924fefb7bec | Byte Size: 308 | Line Count: 11 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## Dockerfile

- Zweck: Baut das Docker-Image: Basis Node 22 (Debian Bookworm), Systembibliotheken für GUI-Browser (virtuelle Anzeige, VNC, GTK, NSS, Audio), pnpm-Installation mit eingefrorenem Lockfile, Initialisierung über das Init-Skript und Start mit virtueller Anzeige und VNC (belegt durch Dateiinhalt, 37 Zeilen).
- Verantwortlichkeit: reproduzierbarer Image-Build einschließlich Browser-Systemabhängigkeiten und Initialisierung.
- Eingaben: Build-Kontext; Manifest, Lockfile, Skript- und Patch-Verzeichnisse; Initialisierungsskript.
- Ausgaben: Docker-Image mit API und WebUI; offene Ports 3000 und 5900.
- Datenfluss: Build-Kontext → Basis-Image → Systempakete → pnpm-Installation (eingefroren) → Kopieren des Quellcodes → Initialisierung → Startkommando.
- Persistenz: Keine im Build; Laufzeit-Volume für Daten.
- Zustände: Build-Phasen.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Die Umgebungsvariable für übersprungene Playwright-Browser-Downloads unterbindet fremde Browser-Downloads; das Init-Skript lädt die projekt eigene Browser-Laufzeit und Geo-Datenbank nach.
- Fehlerfälle: Die eingefrorene Lockfile-Installation bricht bei Abweichungen zum Lockfile ab.
- Sicherheitsrelevanz: Kein Multi-Stage-Build und keine Nicht-Root-Benutzerdirektive — Negativbelege: der Prozess läuft als Root; Systempakete stammen aus den Debian-Repositorien; der Paketcache wird nach der Installation entfernt.
- Geschäftslogik: Das Image enthält die vollständige GUI-Infrastruktur und startet diese standardmäßig (virtuelle Anzeige und VNC).
- Algorithmen: Keine.
- verwendete Datenmodelle: Dockerfile-Anweisungen.
- Abhängigkeiten: Docker; Basis-Image Node 22 Bookworm; pnpm; Xvfb- und VNC-Systempakete; Python und Build-Werkzeuge.
- Rust-Relevanz: Das Rust-Rewrite benötigt für browserbasierte Features die gleichen Systemabhängigkeiten; Basis-Image-Wahl und Härtung (Multi-Stage, Nicht-Root) sind im Rust-Setting neu zu entscheiden.

Beleg:
- EV-WEB2API-000409 | WebAI2API | content-copy | Dockerfile | 1-37 | Image-Build | Config | Aussage: Das Image installiert GUI- und Browser-Systempakete (virtuelle Anzeige, VNC, GTK/NSS-Bibliotheken), installiert die Abhängigkeiten über pnpm mit eingefrorenem Lockfile, führt das Initialisierungsskript aus und startet den Dienst standardmäßig mit virtueller Anzeige und VNC; der Prozess läuft als Root ohne Multi-Stage-Build.

Read-Evidence: File Hash: 41aafa631b4696f770631207c8e5f76ceea931e2671a68065b0eb65f974cdc32 | Byte Size: 856 | Line Count: 37 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## README.md

- Zweck: Chinesische Einstiegs- und Betriebsdokumentation: Projektüberblick, Feature-Liste, unterstützte Zielseiten-Matrix, Deployment (manuell und Docker), Schnellstart inklusive Login-Initialisierung, API-Referenz, Hardware-Empfehlungen, Lizenz und Haftungsausschluss (belegt durch Dateiinhalt, 315 Zeilen).
- Verantwortlichkeit: öffentliche Anleitung und Anforderungsbeschreibung.
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Keine zur Laufzeit.
- Datenfluss: Kein Laufzeit-Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Dokumentiert den Chat-Vervollständigungs-Endpunkt, den Modelllisten-Endpunkt, den Cookie-Endpunkt sowie die zwei Keepalive-Modi (Kommentar und Inhalt) und das WebUI (belegt durch Dateiinhalt).
- Ereignisse: Keine.
- Nebenwirkungen: verweist auf eine externe Dokumentationsseite und GitHub-gehostete Abbildungen.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: warnt vor unverschlüsselter WebUI-Übertragung, empfiehlt HTTPS/SSH-Tunnel; dokumentiert die API-Token-Nutzung und einen Haftungsausschluss bezüglich Nutzungsbedingungen und Kontosperren.
- Geschäftslogik: Kernversprechen: webbasierte KI-Dienste werden als OpenAI-kompatible API bereitgestellt; Empfehlung für dauerhaften Non-Headless-Betrieb über eine virtuelle Anzeige; Streaming mit Keepalive gegen Zeitüberschreitungen; Bildformate werden serverseitig vereinheitlicht.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Primäre Anforderungsquelle für das Rust-Rewrite (Endpunkte, Keepalive-Modi, Instanz-Isolation, Login-Workflow, Bild-Normalisierung).

Beleg:
- EV-WEB2API-000410 | WebAI2API | content-copy | README.md | 1-315 | API-Dokumentation | API | Aussage: Die Dokumentation beschreibt den Chat-Vervollständigungs-Endpunkt, den Modelllisten-Endpunkt und den Cookie-Endpunkt sowie zwei SSE-Keepalive-Modi (Kommentar beziehungsweise Inhalt) und die WebUI-Erreichbarkeit; sie dokumentiert die Bildformate PNG/JPEG/GIF/WebP mit serverseitiger Vereinheitlichung.

Read-Evidence: File Hash: 4e82ad9aa1e802e462c60d1b388e380dbe96b2b971a8da73092d11ff1f92ec6a | Byte Size: 10466 | Line Count: 315 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## README_EN.md

- Zweck: Englische Übersetzung der chinesischen Dokumentation; als maschinelle Übersetzung deklariert (belegt durch Dateiinhalt, 318 Zeilen).
- Verantwortlichkeit: englischsprachige Anleitung.
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Keine zur Laufzeit.
- Datenfluss: Kein Laufzeit-Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Gleicher dokumentierter Endpunktumfang wie die chinesische Fassung (Chat-Vervollständigung, Modellliste, Cookies, Keepalive-Modi).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: identische Warnungen zu unverschlüsselter Übertragung, HTTPS/SSH-Tunnel und Haftungsausschluss.
- Geschäftslogik: identisch zur chinesischen Fassung; keine über die deutsche... keine über die chinesische Fassung hinausgehenden technischen Behauptungen (Negativbeleg: Inhalt deckungsgleich, nur Übersetzung).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Zweite Anforderungsquelle mit identischem Umfang; keine eigenständigen Capabilities über die chinesische Fassung hinaus.

Beleg:
- EV-WEB2API-000411 | WebAI2API | content-copy | README_EN.md | 1-318 | Dokumentationsübersetzung | API | Aussage: Die englische Fassung ist eine als maschinell deklarierte Übersetzung mit identischem Inhalt inklusive Endpunkt- und Keepalive-Beschreibung; sie enthält keine eigenständigen technischen Aussagen über die chinesische Fassung hinaus.

Read-Evidence: File Hash: d08b855e017bb712d7e6fbfa0fa8f8a10279a7aeb5078ec7df44ca8d111e2f74 | Byte Size: 12034 | Line Count: 318 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## CHANGELOG.md

- Zweck: Versionshistorie im Keep-a-Changelog-Format mit semantischer Versionierung von 2.0.0 (2025-12-06) bis 3.6.7 (2026-04-24); dokumentiert die Migration von Puppeteer auf die Camoufox-Basis, den Wachhund, das WebUI, die Pool-Architektur und die Adapter-Historie (belegt durch Dateiinhalt, 512 Zeilen).
- Verantwortlichkeit: belegt die Entwicklungsgeschichte und Verhaltensänderungen; Anforderungsquelle.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Versionskontrolle.
- Zustände: Keine.
- APIs: Referenziert den Cookie-Endpunkt ab Version 3.0.0 (belegt durch Dateiinhalt).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: dokumentiert Fixes für unauthentifizierte SOCKS5-Proxies, Token-Problemfälle und Risiken der CSS-Injection; ein Eintrag markiert eine Adapter-Korrektur explizit als unsichere Anpassung ohne Zugriff des Autors auf ein Abo-Konto (belegt durch Dateiinhalt).
- Geschäftslogik: Chronologie der Fähigkeiten: Pool-basierte Fehlerumleitung, Merge-Worker, Reasoning-Extraktion, Bild-Download-Retry mit wachsender Wartezeit, SQLite-basierte Anfragehistorie, Keepalive-Streaming, Cookie-Export, Wartung durch Beobachtungs-Patches.
- Algorithmen: Erwähnt wachsende Wartezeiten beim Bild-Download-Retry, dynamische Neuauflösung von Laufzeitzeitgebern gemäß eingehendem Stream und Keepalive-Intervalle.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Versionschronologie der Fähigkeiten dient als Checkliste für Rust-Feature-Anforderungen.

Beleg:
- EV-WEB2API-000412 | WebAI2API | content-copy | CHANGELOG.md | 1-512 | Versionshistorie | Event | Aussage: Die Historie dokumentiert die Fähigkeiten-Evolution von der Puppeteer-Basis über die Migration zur Camoufox-Basis, die Einführung des Wachhunds und des WebUI, die Pool-Architektur mit Fehlerumleitung und Merge-Workern bis zur Streaming-Keepalive- und Historie-Funktionalität; ein Adapter-Eintrag ist als unsichere Anpassung markiert.

Read-Evidence: File Hash: 60ba1dc78eaf26b6be9d4edc868b91521c7c19642d0156d93157695b2f8667b8 | Byte Size: 14949 | Line Count: 512 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## LICENSE

- Zweck: MIT-Lizenz mit Urheberrechtshinweis (2025) (belegt durch Dateiinhalt, 21 Zeilen).
- Verantwortlichkeit: Rechtsrahmen für Nutzung und Weitergabe.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Versionskontrolle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Haftungsausschluss des MIT-Textes (keine Gewährleistung).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine direkt; MIT ist für das Lizenzschema der Rust-Neuentwicklung zu beachten.

Beleg:
- EV-WEB2API-000413 | WebAI2API | content-copy | LICENSE | 1-21 | Lizenz | Config | Aussage: Die Datei enthält den vollständigen MIT-Lizenztext mit Urheberrechtshinweis und Haftungsausschluss.

Read-Evidence: File Hash: 3d093bffda16f153493d054b3dad8beada98dc6a41bc7500d27d722d3eb9e4a3 | Byte Size: 1063 | Line Count: 21 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## .gitignore

- Zweck: Schließt Knoten-Module, Datenverzeichnis, Testverzeichnis, lokale Konfigurationsdatei und Browser-Verzeichnis von der Versionskontrolle aus (belegt durch Dateiinhalt, 4 Zeilen).
- Verantwortlichkeit: Versionskontrolle der Build-/Laufzeit-Artefakte.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Versionskontrolle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Der ausgeschlossene Browser-Ordner im Projektwurzelverzeichnis bestätigt den Portabel-Pfad, den die gepatchten Module verwenden (Querverweis auf Patch-Module).
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Die lokale Konfigurationsdatei (enthält Auth-Token und Proxy-Zugangsdaten) ist von der Versionskontrolle ausgeschlossen — Positivbeleg für Geheimnis-Hygiene.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Ausschlussmuster.
- Abhängigkeiten: Git.
- Rust-Relevanz: Muster für die .gitignore des Rewrites (Geheimnisdateien, Build- und Browser-Verzeichnisse).

Beleg:
- EV-WEB2API-000414 | WebAI2API | content-copy | .gitignore | 1-4 | Versionskontrolle | Config | Aussage: Ausgeschlossen sind Knoten-Module, Daten-, Test- und Browser-Verzeichnis sowie die lokale Konfigurationsdatei, die die Auth- und Proxy-Geheimnisse trägt.

Read-Evidence: File Hash: 9758ce1e999e871234fa22b11a0fb146069d22345b74c945d5bd1175a3c82868 | Byte Size: 47 | Line Count: 4 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## scripts/genkey.js

- Zweck: Konsolenwerkzeug zur Erzeugung eines API-Schlüssels im Format Präfix plus 24 zufällige Bytes als Hexadezimalstring (belegt durch Dateiinhalt, 21 Zeilen).
- Verantwortlichkeit: Auth-Schlüsselerzeugung für die Serverkonfiguration.
- Eingaben: Keine (keine Kommandozeilenargumente ausgewertet).
- Ausgaben: Schlüssel auf der Standardausgabe mit Einfügehinweis für die Konfiguration.
- Datenfluss: kryptographischer Zufallsgenerator → Hexadezimalcodierung → Standardausgabe.
- Persistenz: Keine — der Schlüssel wird nicht gespeichert, der Nutzer übernimmt ihn manuell.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: verwendet den kryptographischen Zufallsgenerator der Laufzeit; der Schlüssel wird ausschließlich auf der Standardausgabe ausgegeben und nicht protokolliert.
- Geschäftslogik: Werkzeug für das Auth-Einrichtungsszenario.
- Algorithmen: kryptographischer Zufallsgenerator.
- verwendete Datenmodelle: Schlüsselformat (Präfix + Hexadezimalstring).
- Abhängigkeiten: Krypto-Modul der Node-Laufzeit.
- Rust-Relevanz: In Rust über den kryptographischen Zufall eines Getrandom-Crates zu implementieren; Vorgabe: 48 Hex-Zeichen nach Präfix.

Beleg:
- EV-WEB2API-000415 | WebAI2API | content-copy | scripts/genkey.js | 8-17 | Schlüsselerzeugung | API | Aussage: Das Werkzeug erzeugt einen Auth-Schlüssel aus 24 kryptographisch zufälligen Bytes, hexadezimal codiert und mit Präfix versehen, und gibt ihn ausschließlich auf der Standardausgabe aus.

Read-Evidence: File Hash: 67745cc8f8183364992dd0ba34c5140b1b4b5c6fd1ef66362f6b7a26cf09d5a6 | Byte Size: 615 | Line Count: 21 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## scripts/init.js

- Zweck: Konsolen-Initialisierer, der die Laufzeit-Abhängigkeiten vorbereitet: vorgefertigte SQLite-Binärdatei (ABI-genau), die Camoufox-Browser-Laufzeit, die Geo-Datenbank und macOS-Sonderfälle (belegt durch Dateiinhalt, 681 Zeilen).
- Verantwortlichkeit: Plattform- und ABI-Prüfung; Download-Infrastruktur (HTTP/SOCKS5-Proxies, Wiederholungen, Umleitung, Fortschritt, Leerlauf-Zeitüberschreitung); Installation der Artefakte in die Projektverzeichnisse; interaktive Modi für Proxy- und Einzelschritte.
- Eingaben: Kommandozeilenargumente für Proxy und benutzerdefinierten Modus; interaktive Eingaben; Systeminformationen (Plattform, Architektur, ABI).
- Ausgaben: installierte Artefakte (native SQLite-Binärdatei, Browser-Verzeichnis, Versionsdatei, Geo-Datenbank); Logausgaben.
- Datenfluss: Systeminformationen → Validierung → Download (optional über Proxy) → Entpacken → Kopieren in die Projektziele → Bereinigung.
- Persistenz: schreibt Dateien in das temporäre Datenverzeichnis, das Knoten-Modul-Verzeichnis und das Browser-Verzeichnis.
- Zustände: Installationsfortschritt; Wiederholungszähler.
- APIs: externe Download-Quellen über GitHub-Releases (SQLite-Bindungen, Camoufox-Browser, Geo-Datenbank).
- Ereignisse: Download-Ereignisse (Daten, Abschluss, Fehler), Umleitungsantworten, Leerlauf-Timer.
- Nebenwirkungen: verändert die installierten Module und das Browser-Verzeichnis; erzeugt die Versionsdatei mit festen Versionsangaben; legt das temporäre Datenverzeichnis an; entfernt temporäre Dateien nach der Installation.
- Fehlerfälle: nicht unterstützte Plattform beziehungsweise Architektur → Abbruch; nicht unterstützte ABI-Version → Abbruch; unvollständiger Download → Fehler und Löschung der Datei; nicht-erfolgreiche HTTP-Antwort → Fehler; Leerlauf ohne Daten (3 Minuten) → Fehler; fehlende native Binärdatei nach dem Entpacken → Fehler.
- Sicherheitsrelevanz: Downloads erfolgen ohne Signatur- oder Hash-Prüfung — Negativbeleg: keine Integritätsverifikation im Skript; Proxy-Zugangsdaten können in der Download-URL transportiert werden; der Download-Aufruf täuscht ein Wget-Feld der Benutzeragenten vor.
- Geschäftslogik: bereitet die browser- und SQLite-abhängige Laufzeit vor; der benutzerdefinierte Modus erlaubt Einzelschritte; macOS erhält eine Sonderbehandlung für die Browser-App-Struktur; eine bereits vorhandene Geo-Datenbank wird ohne Zwang übersprungen.
- Algorithmen: Download mit linear wachsender Wartezeit zwischen Wiederholungen, rekursiver Umleitungsverfolgung, Fortschrittsmeldung mit zeitlicher Drosselung und Leerlauf-Überwachung; URL-Konstruktion über Plattform-/Architektur-/ABI-Abbildungen; ABI-Liste mit expliziten unterstützten Werten.
- verwendete Datenmodelle: Plattform- und Architektur-Abbildungen, unterstützte ABI-Liste, Versionsdateistruktur.
- Abhängigkeiten: HTTP/HTTPS-Module der Laufzeit, Archivkompression, interaktive Konsolenabfragen, SOCKS5- und HTTPS-Proxy-Agents, internes Logging-Modul.
- Rust-Relevanz: Das Konzept der ABI-genau aufgelösten vorgefertigten nativen Abhängigkeiten und die Download-/Wiederholungs-/Proxy-Infrastruktur ist in Rust neu zu bauen (asynchrone HTTP-Client mit Rustls, parallele Downloads); eine Integritätsverifikation per Hash ist als Verbesserung für das Rewrite zu planen.

Belege:
- EV-WEB2API-000416 | WebAI2API | content-copy | scripts/init.js | 105-144 | Plattform-/ABI-Prüfung | Config | Aussage: Der Initialisierer prüft Betriebssystem, Architektur und Node-ABI gegen explizite Unterstützungslisten und bricht bei Nicht-Unterstützung mit klarer Meldung ab.
- EV-WEB2API-000417 | WebAI2API | content-copy | scripts/init.js | 153-354 | Download-Infrastruktur | Call | Aussage: Der Download läuft mit optionaler HTTP-/SOCKS5-Proxy-Unterstützung, Wiederholungen mit wachsender Wartezeit, Umleitungsverfolgung, Fortschrittsanzeige und Leerlauf-Zeitüberschreitung; es ist keine Integritätsprüfung vorgesehen.
- EV-WEB2API-000418 | WebAI2API | content-copy | scripts/init.js | 356-630 | Installationsschritte | Call | Aussage: Der Initialisierer installiert die SQLite-Binärdatei an den ABI-/Plattform-Zielort, entpackt die Camoufox-Browser-Laufzeit in das Projektverzeichnis, erzeugt eine Versionsdatei und lädt die Geo-Datenbank nach; macOS erhält eine Sonderbehandlung für die App-Struktur.

Read-Evidence: File Hash: 03cd4aa0e9de8801ea91cc7bfe4429e1a91123035913436994ca9af2c5ad4965 | Byte Size: 23648 | Line Count: 681 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## scripts/postinstall.js

- Zweck: Installations-Hook, der drei gepatchte Moduldateien aus dem Patch-Verzeichnis in das installierte Bibliotheksverzeichnis kopiert (belegt durch Dateiinhalt, 70 Zeilen).
- Verantwortlichkeit: automatisches Anwenden der Vendor-Patches nach der Installation; exportiert die Zuordnung für ein Selbstprüfungsmodul.
- Eingaben: Dateisystemzustand (Patch-Verzeichnis, Zielverzeichnis); exportierte Zuordnungstabelle.
- Ausgaben: kopierte Patchdateien im Zielverzeichnis.
- Datenfluss: Patch-Verzeichnis → Kopieroperation → Zielverzeichnis.
- Persistenz: verändert die installierten Module.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Lifecycle-Hook der Paketinstallation.
- Nebenwirkungen: überschreibt Dateien der installierten Bibliothek; wird nur bei Direktaufruf als Hauptmodul ausgeführt; bei fehlendem Zielverzeichnis wird mit Warnung übersprungen.
- Fehlerfälle: fehlendes Zielverzeichnis → Warnung und Überspringen; fehlende Quelldatei → Warnung und Fortsetzen; Kopierfehler → Fehlerprotokoll je Datei.
- Sicherheitsrelevanz: Das Verfahren ersetzt Dateien einer installierten Abhängigkeit ohne Herkunfts- oder Integritätsprüfung der Patchdateien — Negativbeleg: ausschließlich eine Kopieroperation, keine Signaturprüfung; dadurch werden die SOCKS5- und Portabel-Änderungen reproduzierbar eingespielt.
- Geschäftslogik: Vendor-Patching als Installationsschritt.
- Algorithmen: Keine.
- verwendete Datenmodelle: Zuordnungstabelle von Quelldatei zu Zieldatei.
- Abhängigkeiten: Dateisystem-, Pfad- und URL-Module der Laufzeit.
- Rust-Relevanz: Das Konzept des Dependency-Patchings als Installationsschritt wird in Rust durch eigene Implementierung ersetzt; Cargo-Patch-Mechanismen für analoge Bibliotheken; reproduzierbares Patching in der Build-Pipeline.

Beleg:
- EV-WEB2API-000419 | WebAI2API | content-copy | scripts/postinstall.js | 24-64 | Patch-Anwendung | Call | Aussage: Der Hook kopiert drei gepatchte Module der Camoufox-Bibliothek in das installierte Bibliotheksverzeichnis; fehlende Quellen werden je Datei gemeldet, eine Integritätsprüfung findet nicht statt.

Read-Evidence: File Hash: fb74bfef89ad22dd1fbc1563460cf5b4af85da8f4c4fb12aedc2cbb5b9a7d658 | Byte Size: 2194 | Line Count: 70 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## patches/camoufox-js@0.8.3.locale.patched.js

- Zweck: Gepatchtes Modul für Sprach-/Region-/Geolokalisierung der Browser-Umgebung: (1) Portabel-Modus — die Geo-Datenbank wird zuerst aus dem Projektverzeichnis geladen; (2) das Geolokalisierungsergebnis wird auf eine feste Sprach-/Region-Kombination (en-US) fixiert, während die IP-basierten Koordinaten und die Zeitzone erhalten bleiben (belegt durch Dateiinhalt, 289 Zeilen; Patchmarkierungen Zeilen 130-135 und 187-191).
- Verantwortlichkeit: Konfiguration von Sprache, Region und Geolokalisierung; statistische Auswahlverfahren; Geo-Datenbank-Beschaffung.
- Eingaben: Sprach-/Region-Strings; IP-Adresse; Dateisystemzustand der Geo-Datenbank.
- Ausgaben: Browser-Konfiguration (Sprach- und Regionseinträge, Längen-/Breitengrad, Zeitzone); Geo-Datenbank-Download beziehungsweise -Entfernung.
- Datenfluss: IP → Datenbankabfrage → Geolokalisierungsobjekt → Konfigurationsabbildung → Browser-Konfiguration; Sprachliste → Auswahllogik → Konfiguration.
- Persistenz: Geo-Datenbankdatei (Projektverzeichnis oder Installationscache).
- Zustände: Keine.
- APIs: Geo-Datenbank-Abfrage; GitHub-Releases-Abfrage für die Datenbank.
- Ereignisse: Download-Ereignisse.
- Nebenwirkungen: lädt bei fehlender Datenbank automatisch nach; erzeugt beziehungsweise entfernt die Datenbankdatei.
- Fehlerfälle: unbekannte Region beziehungsweise Sprache → eigene Fehlerobjekte; IP ohne Ortsinformation → Fehler; gesetzte Download-Skip-Umgebungsvariable → Überspringen des Datenbank-Downloads.
- Sicherheitsrelevanz: Der Datenbank-Download bezieht Binärdaten aus GitHub ohne Hash-Prüfung — Negativbeleg: keine Integritätsverifikation; die erzwungene feste Sprach-/Region-Kombination beeinflusst die Fingerprint-Konsistenz der Umgebung.
- Geschäftslogik: Statistische Sprachwahl nach Bevölkerungs- und Alphabetisierungsgewichten; im WebAI2API-Kontext wird die Sprach-/Region-Kombination gezielt fixiert, um konsistente Umgebungen zu erzwingen, während Koordinaten und Zeitzone IP-abhängig bleiben.
- Algorithmen: gewichtete Zufallsauswahl über Unicode-Territoriumsstatistik; Sprach-/Region-Normalisierung; Fallback-Ketten bei fehlender Region oder Sprache.
- verwendete Datenmodelle: Sprach-/Region-Objekt (Sprache, Region, Schrift), Geolokalisierungsobjekt (Koordinaten, Zeitzone, Genauigkeit), Konfigurationsabbildungen, XML-basierte Statistikdaten.
- Abhängigkeiten: Dateisystem- und Pfadmodule der Laufzeit; Sprach-Tags-Bibliothek; Geo-Datenbank-Leser; XML-Parser; interne Module der Bibliothek (Fehler, IP, Installationsverwaltung, Hilfsfunktionen, Warnungen).
- Rust-Relevanz: Das Konzept der IP-geolokalisierten Fingerprint-Umgebung und der statistischen Sprach-/Region-Selektion ist als Rust-Neuimplementierung zu bauen; Geo-Datenbank-Suche über ein MMDB-Crate; gewichtete Zufallsauswahl mit eigenem Algorithmus.

Beleg:
- EV-WEB2API-000420 | WebAI2API | content-copy | patches/camoufox-js@0.8.3.locale.patched.js | 130-191 | Portabel-Modus und Locale-Fixierung | Config | Aussage: Das gepatchte Modul bevorzugt die Geo-Datenbank im Projektverzeichnis gegenüber dem Installationscache und fixiert das Geolokalisierungsergebnis auf eine feste Sprach-/Region-Kombination (en-US), während Längen-/Breitengrad und Zeitzone aus der IP-Abfrage erhalten bleiben.

Read-Evidence: File Hash: 84017bd2cb5dee90e240e894ef17767c1d76eff2edfc57e306bbbd39f7faa19c | Byte Size: 9975 | Line Count: 289 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## patches/camoufox-js@0.8.3.pkgman.patched.js

- Zweck: Gepatchtes Installations- und Versionsmodul der Browser-Laufzeit: der Installationspfad bevorzugt das Projektverzeichnis gegenüber dem Benutzer-Cache (Portabel-Modus); enthält Versionsprüfung, Architektur-/System-Matrizen, GitHub-Release-Auflösung und Installations-/Entpackworkflow (belegt durch Dateiinhalt, 351 Zeilen; Patchmarkierung Zeilen 25-28).
- Verantwortlichkeit: Auffinden und Prüfen der Browser-Installation; Installations- und Download-Workflow; Launch-Pfadauflösung.
- Eingaben: Plattform, Architektur, Versionsdatei, GitHub-Release-API.
- Ausgaben: Installationspfad, Startpfad, Installation und Downloads.
- Datenfluss: Pfadauflösung → Versionsprüfung → Release-Auflösung → Download → Entpacken → Versionsdatei → Startpfad.
- Persistenz: Browser-Verzeichnis (Projekt oder Benutzer-Cache).
- Zustände: Versionsprüfungszustände; Fetcher-Initialisierung.
- APIs: GitHub-Releases-API für die Browser-Quelle.
- Ereignisse: Wiederholungs- und Download-Ereignisse.
- Nebenwirkungen: löscht und installiert bei Bedarf das Zielverzeichnis neu; setzt Ausführungsrechte auf Nicht-Windows-Systemen.
- Fehlerfälle: nicht unterstütztes System, Architektur oder Version; fehlende Versionsdatei; fehlendes Start-Binary; fehlgeschlagene Download-/Release-Abfragen nach begrenzten Wiederholungen.
- Sicherheitsrelevanz: installiert ausführbare Binärdaten aus einer Drittquelle ohne Signaturprüfung — Negativbeleg: keine Checksummen-Verifikation im Patch.
- Geschäftslogik: Versionen werden sortiert verglichen und gegen minimale beziehungsweise maximale Versionsbeschränkungen geprüft; Release-Teile werden numerisch konvertiert mit Zeichen-Ausweichverfahren.
- Algorithmen: sortierbarer Versionsvergleich (auf feste Länge gepolsterte Release-Teile, Buchstaben über Zeichencode-Offset), Pfadwahl nach Vorhandensein, Asset-Auswahl über Namensmuster.
- verwendete Datenmodelle: Versionsobjekt (Release, Version, sortierte Teile), Installationspfad, System-/Architektur-Abbildungen, Startdatei-Abbildung, System-/Architektur-Matrix.
- Abhängigkeiten: Prozess-, Dateisystem-, System-, Pfad- und Timer-Module der Laufzeit; ZIP-Bibliothek; Fortschrittsanzeige; interne Module der Bibliothek.
- Rust-Relevanz: Versionsprüfungs- und Download-/Installationslogik für ein externes Browser-Binary als Rust-CLI; semantische Versionsvergleich durch ein semver-Crate; Integritätsprüfung als geplante Verbesserung.

Beleg:
- EV-WEB2API-000421 | WebAI2API | content-copy | patches/camoufox-js@0.8.3.pkgman.patched.js | 25-28 | Installationspfad-Überschreibung | Config | Aussage: Das gepatchte Modul wählt den Installationspfad bevorzugt aus dem Projektverzeichnis und nur fallweise aus dem Benutzer-Cache, wodurch die Browser-Laufzeit portabel neben dem Projekt liegen kann; Downloads erfolgen ohne Integritätsprüfung.

Read-Evidence: File Hash: 7446fda709457b13fa5518aa7f4fff21a833a7e87aa400c1f13cb907d39f16c3 | Byte Size: 11925 | Line Count: 351 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## patches/camoufox-js@0.8.3.utils.patched.js

- Zweck: Gepatchtes Modul zur Erzeugung der Browser-Startkonfiguration: aggregiert Fingerprint, Schriftarten, WebGL- und Canvas-Parameter, Firefox-Einstellungen, Umgebungsvariablen, Proxy, Geolokalisierung und Sprache; enthält den WebAI2API-Fix für das SOCKS5-Proxy-Problem (belegt durch Dateiinhalt, 518 Zeilen; Fixmarkierung Zeilen 1-17 und 508-512).
- Verantwortlichkeit: vollständige Erzeugung und Validierung der Browser-Launch-Optionen.
- Eingaben: Launch-Parameter (Konfiguration, Fingerprint, Proxy, Geolokalisierung, Sprache, Humanisierung, Addons, Headless-Modus, ...); Eigenschaftsdatei; Schriftarten-Konfiguration; WebGL-Beispieldaten.
- Ausgaben: Launch-Optionen (Startpfad, Argumente, Umgebungsvariablen einschließlich segmentierter Konfigurationsvariablen, Firefox-Einstellungen, Proxy-Angabe mit korrigiertem Serverfeld, Headless-Flag).
- Datenfluss: Parameter → Konfigurationsaggregation → Fingerprint-Einspritzung → WebGL-Probennahme → Validierung → Umgebungsvariablen-Segmentierung → Launch-Optionen.
- Persistenz: Keine Laufzeit-Persistenz.
- Zustände: Keine.
- APIs: Schnittstellen zu Fingerprint-, IP-, Sprach-, Addon- und WebGL-Modulen.
- Ereignisse: Keine.
- Nebenwirkungen: erzeugt segmentierte Konfigurations-Umgebungsvariablen; setzt eine Schriftkonfigurationsumgebungsvariable auf Linux; validiert die Konfiguration und erzeugt Warnungen bei Inkonsistenzen.
- Fehlerfälle: unbekannte Konfigurationsschlüssel beziehungsweise falsche Typen → Validierungsfehler; Nicht-Firefox-Fingerprint → Ablehnung; nicht unterstütztes System → Fehler; widersprüchliche Schriftoptionen → Fehler.
- Sicherheitsrelevanz: Canvas-Anti-Fingerprinting (zufälliger Offset), WebGL-Blockierungsoptionen, Bild- und WebRTC-Blockierungsoptionen; der SOCKS5-Fix behebt ein Fehlverhalten bei nicht-standardisierten Protokollen, das zu einem Proxy-Host-Fehler im Browser führte.
- Geschäftslogik: Zusammenführen von Nutzerkonfiguration mit generierten Fingerprint-Daten; Typprüfung der Konfiguration gegen die Eigenschaftsdatei; Warnpolitik bei manuellen Konfigurationsangaben, die Fingerprint-Leaks begünstigen; Konsistenzsicherung zwischen Proxy und Geolokalisierung.
- Algorithmen: Konfigurations-Segmentierung in Umgebungsvariablen mit plattformabhängiger Segmentgröße; zufällige Canvas-Offsets; Konfigurations-Zusammenführungsregeln; Systemerkennung aus Benutzeragent; gewichtete Fingerprint-Aggregation.
- verwendete Datenmodelle: Konfigurationsabbildungen (Schlüssel/Wert), Launch-Optionen-Objekt, Proxy-URL-Objekt, Schriftarten-Konfiguration.
- Abhängigkeiten: Dateisystem- und Pfadmodule der Laufzeit; Benutzeragent-Parser; interne Module der Bibliothek (Addons, Fehler, Fingerprints, IP, Sprach, Installationsverwaltung, Warnungen, WebGL, Schriftarten).
- Rust-Relevanz: Die Startkonfigurations-Fähigkeit (Konfigurationsvalidierung, Umgebungsvariablen-Segmentierung, Fingerprint-Einspritzung, Proxy-URL-Normalisierung) ist als Rust-Modul neu zu bauen; serde für die Typprüfung; eigene Fehlerhierarchie; Proxy-URL-Normalisierung als eigenes Feature.

Beleg:
- EV-WEB2API-000422 | WebAI2API | content-copy | patches/camoufox-js@0.8.3.utils.patched.js | 495-517 | SOCKS5-Proxy-Korrektur | API | Aussage: Das gepatchte Modul normalisiert die Proxy-Serverangabe durch Kombination von Protokoll und Host statt der URL-Origin-Eigenschaft, wodurch nicht-standardisierte Protokolle wie SOCKS5 korrekt an den Browser übergeben werden; der Fix behebt einen Proxy-Host-Fehler der Browser-Laufzeit.

Read-Evidence: File Hash: b8c008d1e70edf388606113f1d5364659ab7c95f09708136d7a4ef0fe87fe2bc | Byte Size: 17623 | Line Count: 518 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK

---

## .github/workflows/docker-publish.yml

- Zweck: Manuell ausgelöste GitHub-Action zur Mehrarchitektur-Veröffentlichung des Docker-Images (amd64 und arm64) mit QEMU/Buildx, Docker-Hub-Anmeldung und Tag-Erzeugung (neueste Version plus kurzer Commit-Hash) (belegt durch Dateiinhalt, 68 Zeilen).
- Verantwortlichkeit: Veröffentlichungspipeline und Versorgung der Container-Registry.
- Eingaben: manueller Auslöser mit optionalem Grund; GitHub-Secrets für die Registrierungsanmeldung; Commit-Metadaten.
- Ausgaben: veröffentlichtes Image in der Registry.
- Datenfluss: Auslöser → Auschecken → QEMU → Buildx → Anmeldung → Metadaten → Build und Push → Registry.
- Persistenz: Container-Registry (Docker Hub).
- Zustände: CI-Job.
- APIs: Docker Hub; GitHub-Actions-Infrastruktur.
- Ereignisse: manueller Workflow-Auslöser.
- Nebenwirkungen: schreibt in die Registry-Paketrechte.
- Fehlerfälle: fehlende Anmelde-Secrets lassen die Anmeldung scheitern.
- Sicherheitsrelevanz: ausschließlich manuelle Auslösung — Negativbeleg: keine automatischen Auslöser über Push oder Pull-Requests im Workflow; Secrets werden als Anmeldedaten verwendet; es gibt keine Signatur-, SBOM- oder Sicherheits-Scanschritte — Negativbelege im Workflow-Inhalt.
- Geschäftslogik: Veröffentlichungsautomatisierung über Mehrarchitektur-Build mit Cache.
- Algorithmen: Tag-Erzeugung (neueste Version plus kurzer Commit-Hash).
- verwendete Datenmodelle: Workflow-YAML (Umgebung, Jobs, Schritte).
- Abhängigkeiten: GitHub Actions; veröffentlichte Aktionsversionen für Auschecken, QEMU, Buildx, Anmeldung, Metadaten und Build/Push.
- Rust-Relevanz: Das CI/CD-Muster (manueller Mehrarchitektur-Build mit GitHub-Cache) ist für das Rust-Rewrite übernehmbar; Rust-Images benötigen gegebenenfalls eine eigene Plattform-Matrix und Härtungsschritte.

Beleg:
- EV-WEB2API-000423 | WebAI2API | content-copy | .github/workflows/docker-publish.yml | 1-68 | Veröffentlichungspipeline | Config | Aussage: Die Workflow-Datei veröffentlicht das Image ausschließlich bei manuellem Auslöser für amd64 und arm64 mit Docker-Hub-Anmeldung, Tag-Erzeugung (neueste Version plus kurzer Commit-Hash) und GitHub-Cache; automatische Auslöser sowie Signatur-/SBOM-Schritte sind nicht enthalten.

Read-Evidence: File Hash: 27b756af7d610f83a32a82fb2e5c3af78013b4fbabcca25676534cff3e7ecf07 | Byte Size: 1859 | Line Count: 68 | Encoding: UTF-8 | Read Timestamp: 2026-08-05 | Reader Result: OK
