# WebAI2API — Backend Core (Engine, Pool, Strategies, Utils, Index, Registry)

Repository: `WebAI2API`
Commit: content-copy (lokal geklonte Arbeitskopie)
Datei-für-Datei-Analyse der Backend-Kernmodule. Alle Pfade relativ zu `repos/WebAI2API/`.
Belegquellen sind als `EV-WEB2API-0001xx`-Evidence-Blöcke referenziert.
Insgesamt 14 Dateien, vollständig gelesen.

---

## src/backend/engine/launcher.js

- Zweck: Browser-Start und Lebenszyklus-Management. Startet Camoufox (Firefox-basierter Playwright-Kern), injiziert Fingerabdruck (Fingerprint), Proxy und CSS-Performance-Optimierungen; übernimmt die Ressourcen-Bereinigung beim Prozessende. Navigation und Vorwärmen sind ausdrücklich Sache der Arbeitspool-Schicht, nicht dieses Moduls (belegt durch Modul-Kommentar, Zeilen 3-9).
- Verantwortlichkeit: Besitzt die einzige Stelle, an der Browser-Instanzen erzeugt werden; stellt sicher, dass Profile/Fingerprints persistent zwischen Starts überleben und dass Prozessreste (auch nach Login-Sessions) beseitigt werden. Registriert Signal-Handler für sauberes Herunterfahren.
- Eingaben: Globale Konfiguration (`browser.headless`, `browser.path`, `browser.cssInject`, `browser.humanizeCursor`, `browser.fission`); Startoptionen (User-Datenverzeichnis, Instanzname, Proxy-Konfiguration); CLI-Flag `-login` und Umgebungsvariable `XVFB_RUNNING` (Headless-Entscheidung); persistente Fingerprint-Datei im User-Datenverzeichnis.
- Ausgaben: Browser-Kontext (geöffneter Playwright/`camoufox-js`-Kontext) plus initiale Seite; modifizierte/persistente Fingerprint-Datei; Ausgangswert `{context, page}`.
- Datenfluss: Konfiguration → Fingerprint-Erzeugung/-Validierung → Camoufox-Start → Proxy-Auflösung → Viewport-Einstellung → CSS-Init-Script-Injektion → Kontext/Seite an Aufrufer (Arbeitspool). Beim Beenden: Kontext-Schließen → Prozess-Kill → Proxy-Bereinigung.
- Persistenz: Schreibende Persistenz in `fingerprint.json` innerhalb des jeweiligen User-Datenverzeichnisses (Erzeugen/Validieren/Neuschreiben bei Änderung). Die Browser-Profile selbst bleiben im User-Datenverzeichnis liegen.
- Zustände: Globaler Modul-Zustand `globalContext`/`globalBrowserProcess` (Login-Modus, Wiederverwendung über Aufrufe hinweg); Kontext-Lebenszyklus `offen → geschlossen`; Signal-Handler-Registrierung ist einmalig (Idempotenz-Flag).
- APIs: Externe APIs: `Camoufox()` (Browsergründung), `sampleWebGL` (WebGL-Datenbank-Abfrage), `FingerprintGenerator` (Fingerabdruck-Erzeugung), `ghost-cursor-playwright-port` (Cursor), lokale Proxy-APIs `getBrowserProxy`/`cleanupProxy`. Exportiert nach außen: `initBrowserBase`, `cleanup`, sowie re-exportierte Helfer `createCursor`, `getRealViewport`, `clamp`, `random`, `sleep`.
- Ereignisse: Prozess-Signale `exit`, `SIGINT`, `SIGTERM` (Register + Handler); Kontext-`close`-Ereignis (setzt globale Referenzen zurück, ohne Prozess zu beenden).
- Nebenwirkungen: Erzeugt Unterprozesse (Browser) mit mehrstufiger Beendigungssequenz (Playwright-close → SIGTERM → SIGKILL); schreibt Fingerprint-Dateien; injiziert per Init-Script CSS in jede Seite (nur wenn mindestens ein CSS-Optimierungs-Schalter aktiv ist, um Fingerprint-Veränderung zu vermeiden); erzwingt Viewport-Größen aus dem Fingerabdruck; setzt Firefox-User-Prefs (Animatomie, Site-Isolation).
- Fehlerfälle: Beschädigte Fingerprint-Datei → Neugenerierung; WebGL-Konfiguration nicht mehr gültig für die Plattform → Neuabtastung; nicht erzeugbare WebGL-Konfiguration → geloggt als schwerer Fehler, Start läuft ohne ausdrücklichen Abbruch weiter; schließbarer Kontext mit Fehler wird nur gewarnt.
- Sicherheitsrelevanz: WebRTC wird blockiert (`block_webrtc: true`); Fingerabdruck-Spoofing für UA, WebGL (Vendor/Renderer), Canvas (fester Rausch-Offset) und Plugins (leere Plugin/MIME-Listen) ist der zentrale Anti-Erkennungs-Mechanismus; `user_data_dir` erzwingt echte Profil-Persistenz; GeoIP-Auflösung aktiviert. Diese Fähigkeiten sind beobachtet, nicht als Sicherheitsgarantie bewertet.
- Geschäftslogik: Headless-Modus wird in Login- und Xvfb-Modus deaktiviert, damit der Benutzer in einem sichtbaren Browser einloggen kann; `-login`-Flag beeinflusst das Lebenszyklus-Verhalten (Prozessende nach Browser-Schließen wird hier nicht behandelt, sondern im Worker).
- Algorithmen: Fingerabdruck-Lebenszyklus: laden → WebGL-Validierung per Stichprobenabgleich → ggf. Neuabtastung → Canvas-Offset-Erzeugung (Ganzzahl im Bereich ±20) → bedingtes Speichern; UA-Version auf Zielversion `135.0` normalisiert; Betriebssystem-Mapping `win32→windows`, `darwin→macos`, sonst `linux`.
- verwendete Datenmodelle: Fingerprint-JSON-Struktur mit `navigator.userAgent`, `videoCard['webGl:vendor']`, `videoCard['webGl:renderer']`, `canvasOffset`, `screen.availWidth/availHeight`, `pluginsData`; Browser-Startoptionen-Objekt (Camoufox-spezifisch); Ausgabe-Tupel `{context, page}`.
- Abhängigkeiten: `camoufox-js` (+ WebGL-Sampledaten), `fingerprint-generator`, `ghost-cursor-playwright-port`, `fs`, `path`, `os`; intern `./utils.js` (Viewport/Koordinaten/Verzögerung), `../../utils/logger.js`, `../../utils/proxy.js`.
- Rust-Relevanz: Konzept der persistenten, validierten Browser-Fingerabdruck-Verwaltung (WebGL/Canvas/UA) und die mehrstufige Prozess-/Ressourcen-Bereinigung sind als eigene Rust-Module neu zu denken. Eine Rust-2024-Neuentwicklung muss einen Prozess-Supervisor mit Graceful-Shutdown (analog close→SIGTERM→SIGKILL) und ein eigenes Fingerprint-Persistenz-Schema bereitstellen, ohne die JS-Dateistrukturen zu übernehmen.

Read Evidence:
File Hash: 26b6873ad85584b2df38de99caf1617822a81e039393d818610efdd26c7c88ae
Byte Size: 15515
Line Count: 434
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000101
- EV-WEB2API-000115

---

## src/backend/engine/utils.js

- Zweck: Sammlung von Browser-Automatisierungs-Bausteinen: atomare Aktionen (Klick, Scroll, Tippen, Datei-Upload), Seitenzustands-Prüfung, Shadow-DOM-/Iframe-fähige Suche, Koordinaten- und Verzögerungs-Helfer. Sie stellt die Wiederverwendungsbasis dar, auf der alle Adapter ihre UI-Automatisierung aufbauen.
- Verantwortlichkeit: Zentralisiert alle "menschlich wirkenden" Interaktionsmuster (zufällige Klick-Punkte, Tippfehler-Simulation, Pseudotippen bei langen Texten, Cursor-Trajektorien) und alle stabilen Fehlerpfade beim Klicken/Scrollen/Uploaden; kapselt Playwright-Details von den Adaptern.
- Eingaben: Playwright-Seite, Selektoren/Locatoren/Element-Handles, Dateipfad-Listen, Konfigurationsoptionen (Zeitlimit, Vorspann-Verhalten, Upload-Validierungsfunktion); `page._humanizeCursorMode` und `page.cursor` als Laufzeit-Marker.
- Ausgaben: Koordinatenpunkte, MIME-Typen, Sichtfenstergrößen (mit Sicherheitspuffer), Element-Handles, Upload-Status, Schließ-/Absturz-Watcher-Objekte, Cookie-Arrays; Fehlerobjekte mit kodierten Nachrichten.
- Datenfluss: Adapter rufen Helfer mit Seite+Ziel auf → Elementauflösung (Selektor/Locator/Handle) → Scroll/Stabilitätsprüfung → Klick (ghost-cursor oder nativ) → Ergebnis/Fehler an den Aufrufer. Upload-Pfad: `input[type=file]`-Tiefensuche (inkl. Shadow DOM) → natives `setInputFiles` bzw. `filechooser`-Ereignis.
- Persistenz: Keine Persistenz. (Negativbeleg: keine Schreiboperationen, keine Storage-/Datenbank-/Datei-Zugriffe im Dateiinhalt.)
- Zustände: Kein eigener dauerhafter Zustand; transient pro Aufruf: Zeitlimit-Timer, Upload-Zähler, Abort-Flag beim Klick.
- APIs: Externe API ist die Playwright-Seiten-API. Exportiert: `random`, `sleep`, `getMimeType`, `getRealViewport`, `clamp`, `queryDeep`, `getHumanClickPoint`, `safeClick`, `safeScroll`, `humanType`, `pasteImages`, `uploadFilesViaChooser`, `isPageValid`, `createPageCloseWatcher`, `getCookies`.
- Ereignisse: Hörereignisse: Seiten-`close`/`crash` (Watcher), `response` (Upload-Fortschritt), `filechooser` (Upload), `requestfinished`/`requestfailed` (indirekt über `page.js`).
- Nebenwirkungen: Bewegt den echten Maus-Cursor, führt Scroll-Operationen aus, schreibt Dateien in das Upload-Ziel der Seite, erzeugt zufällige Verzögerungen; `safeClick` überspringt die Operabilitätsprüfung (`force: true`), was Klicks auf überdeckte Elemente ermöglicht.
- Fehlerfälle: `CLICK_TIMEOUT` bei Zeitüberschreitung; Upload-Zeitüberschreitung mit Fortschrittslog; `UPLOAD_FILECHOOSER_TIMEOUT`; „Node is not visible"-Vermeidung durch Koordinaten-Klemme; `getRealViewport` fällt bei Kontextverlust auf konservative 1280×720 zurück.
- Sicherheitsrelevanz: Umgeht DOM-basierte Erkennungssignale von Automatisierung (zufällige Klickpunkte, Verzögerungen, Pseudo-Tippfehler, Paste-Strategie bei langen Texten); `document.execCommand('insertText')` wird für die Texteinfügung genutzt. Diese Anti-Erkennung ist beobachtete Funktionalität.
- Geschäftslogik: Tippstrategie: kurze Texte < 50 Zeichen werden zeichenweise mit 5%-Tippfehlerrate getippt; lange Texte werden pseudo-getippt, geleert und eingefügt; Zeilenumbrüche werden als `Shift+Enter` getippt, um versehentliches Absenden zu verhindern; Upload wartet wahlweise auf validierte Antworten (bis 60 s) oder auf Preview-Verzögerung.
- Algorithmen: `getHumanClickPoint`: Positionsverteilung je Klick-Typ (input/center/top-left/top-right/bottom-right/random) mit anschließender Klemmung auf die Elementgrenzen (±1 px); `waitForElementStable`: Layout-Stabilitätserkennung über `requestAnimationFrame` mit 1-px-Toleranz und konstanten Frames; `queryDeep`/`findAllFileInputs`: rekursive Traversierung inkl. offener Shadow-Roots und TreeWalker.
- verwendete Datenmodelle: Koordinatenpunkt `{x,y}`; Viewport-Objekt `{width,height,safeWidth,safeHeight}`; Upload-Optionen mit `uploadValidator`; MIME-Tabelle (Extension → MIME); Fehlerobjekte mit `code`.
- Abhängigkeiten: `path`, `../../utils/logger.js`, `../../utils/constants.js` (Zeitlimits); Playwright-Seitenobjekt als Vertrag; optional `page.cursor` (ghost-cursor).
- Rust-Relevanz: Konzepte der "humanisierten" Interaktionsplanung (Klick-Punkt-Verteilung, Layout-Stabilitätswarten, Upload-Validierung gegen Netzwerk-Antworten, Seiten-Schließ-/Absturz-Watcher) sind als eigene Rust-Strategien neu zu entwerfen. Die reine Koordinaten-/Zufallslogik ist direkt auf Rust-Primitive übertragbar, ohne Namen oder Code zu übernehmen.

Read Evidence:
File Hash: bba43b6ef6f330a31b562869fa8ba4228c038b01e073c11aaa4220583c14b828
Byte Size: 32092
Line Count: 822
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000102
- EV-WEB2API-000116

---

## src/backend/pool/PoolManager.js

- Zweck: Verwaltet den Pool von Worker-Instanzen: Initialisierung, Modell-basierte Auswahl, Task-Verteilung mit Failover, Modell-/Policy-Aggregation und Cookie-/Monitor-Schnittstellen.
- Verantwortlichkeit: Zentraler Koordinator der Worker-Laufzeit; übersetzt Konfiguration in laufende Browser-Worker, entkoppelt den Router/Queue von den einzelnen Adaptern und stellt die Fehlertoleranz-Schicht (Failover, Initialisierungsfehler-Härtung) bereit.
- Eingaben: Globale Konfiguration (`backend.pool.strategy`, `backend.pool.workers`, `backend.pool.failover`, `backend.adapter`); Prozess-Argumente für den Login-Modus (`-login[=Name]`); Task-Aufrufe `(ctx, prompt, paths, modelId, meta)`; Abfragen nach Modell/Policy/Type/Cookies.
- Ausgaben: OpenAI-ähnliches Modelllisten-Objekt `{object:'list', data:[...]}`; Policy-Strings `optional|required|forbidden`; Modell-Typ `text|image`; Cookie-Ergebnis `{instance, cookies}`; Generierungsresultate oder Fehlerobjekte `{error, code, retryable}`.
- Datenfluss: Konfiguration → Registry-Laden → Adapter-Konfiguration injizieren → Worker-Filterung (Login-Modus / Adapter-Verfügbarkeit) → Gruppierung nach User-Datenverzeichnis (Browser-Sharing) → initialisierte Worker-Liste. Aufgabe: Modellfilter → Kandidaten → (bei Bildern Policy-Vorfilter) → Sortierung per Strategie → Failover-Ausführung → Worker-Aufruf.
- Persistenz: Keine eigene Persistenz; verlässt sich auf das User-Datenverzeichnis der Worker (durch `registry`/`Worker` genutzt).
- Zustände: `initialized` (Pool), `workers[]`, `strategy`, `roundRobinIndex` (interne Zählung); pro Aufgabe transient: Kandidatenlisten.
- APIs: Intern genutzt: `registry.loadAll/setAdapterConfig/hasAdapter`, `createStrategySelector`, `executeWithFailover`, `normalizeError`, `Worker`. Nach außen: `initAll`, `selectWorker`, `generate`, `getModels`, `getImagePolicy`, `getModelType`, `getCookies`, `navigateToMonitor`, `getFirstPage`.
- Ereignisse: Keine eigenen Ereignisse; konsumiert Browser-Ereignisse indirekt über Worker.
- Nebenwirkungen: Startet bei der Initialisierung echte Browserprozesse; loggt umfangreich; bei Bild-Tasks bevorzugt es Worker mit `optional`/`required`-Policy; teilt Browser-Profile zwischen Workern (Proxy-Inkonsistenzen werden gewarnt und zugunsten des ersten Workern entschieden).
- Fehlerfälle: Login-Modus ohne passenden Worker → Abbruch mit verfügbarer Namensliste; kein Worker für Modell → Fehlerobjekt/Exception; alle Worker-Initialisierungen gescheitert → Abbruch der Initialisierung; Einzel-Worker-Initialisierung schlägt fehl → Worker wird übersprungen, Pool läuft weiter.
- Sicherheitsrelevanz: `getCookies` setzt Session-Cookies (inkl. Autorisierungssitzungen) an die Service-Schicht aus; die Proxy-Konsistenz bei geteilten Profilen wird überwacht (Warnung bei Abweichung), was Session-Isolation betrifft.
- Geschäftslogik: Login-Modus filtert den Pool auf genau einen Worker; nur Typen mit registriertem Adapter (außer `merge`, der die `mergeTypes` prüft) werden akzeptiert; Bilder-Anfragen bevorzugen bildfähige Worker; Policy-Aggregation ist "lax" (optional schlägt alles).
- Algorithmen: Worker-Auswahl: `round_robin` (modulo mit Zähler), `random`, `least_busy` (Minimum der Auslastung); Bild-Vorfilter als Mengenoperation auf Policies; Deduplizierung von Modellen über eine gesehene-ID-Menge.
- verwendete Datenmodelle: Worker-Konfigurations-Objekt (`name, type, instanceName, userDataDir, resolvedProxy, mergeTypes, mergeMonitor`); Failover-Konfiguration (`enabled`, `maxRetries`); Ausgabe-Objekte `{object:'list', data}`; Fehlerobjekte `{error, code, retryable}`.
- Abhängigkeiten: `../../utils/logger.js`, `../registry.js`, `../strategies/index.js`, `../strategies/failover.js`, `../utils/error.js`, `./Worker.js`.
- Rust-Relevanz: Das Pool-Konzept (Initialisierung mit Teilschlag-Fortsetzung, Browser-Sharing-Gruppierung, bildbewusste Kandidatenfilterung, Failover-Ausführung, laxes Policy-Aggregat) ist als Rust-Aufgabenverteiler mit konfigurierbarer Selektor-Strategie neu zu entwerfen; Modelldedup und Policy-Vorrangregeln sind direkt als Rust-Enum-Logik neu denkbar.

Read Evidence:
File Hash: 347ddb7e794d913af838902af78da528b2e9ede9be6bcb771e64e7fc53469f2d
Byte Size: 11696
Line Count: 322
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000103
- EV-WEB2API-000117

---

## src/backend/pool/Worker.js

- Zweck: Kapselt eine einzelne Browser-Instanz (Kontext + Seite) als auslastbare Arbeitseinheit: Initialisierung inkl. Ziel-Navigation, Modell-Matching, Adapter-Dispatcher, Selbstheilung bei Browser-/Seitenausfall und Merge-Typ-Failover.
- Verantwortlichkeit: Einziger Besitzer der Laufzeit einer Browser-Session innerhalb des Pools; verantwortlich für die Zuordnung Modell→Adapter-Typ (auch im `merge`-Modus über mehrere Adaptertypen), für die Auslastungsbuchhaltung und für die automatische Wiederherstellung nach Verbindungsverlust.
- Eingaben: Globale Konfiguration + Worker-Konfiguration; optional ein geteilter Browser-Kontext; Task-Parameter `(ctx, prompt, paths, modelId, meta)`; Registry-Abfragen; CLI-Flag `-login` für das Registrierungsverhalten.
- Ausgaben: Generierungsergebnisse oder Fehlerobjekte; Modelllisten (nativ + `type/model`-qualifiziert); Policy- und Typ-Abfragen; Cookies; Navigations-/Monitor-Navigationen.
- Datenfluss: Konfiguration → Ziel-URL-Auflösung über Registry → Navigation (bei `merge`: mehrere URLs nacheinander versuchen) → Verarbeitung: Modell-Prüfung → Typ-Auflösung → `busyCount++` → Adapter-`generate` → `busyCount--`. Bei Verbindungsverlust: Owner neu initialisieren → geteilte Seiten für alle Sharer neu erstellen.
- Persistenz: Keine eigene Persistenz; das User-Datenverzeichnis wird erzeugt, falls es fehlt (Profil- und Fingerprint-Persistenz der Browser-Ebene).
- Zustände: `initialized`, `browser`, `page`, `busyCount`, `_isBrowserOwner`, `_browserOwner`, `_sharedWorkers`, `_targetUrl`, `_navigationHandler`; Page-Attribute `authState` und `_humanizeCursorMode`; Browser-Ownership-Modell für geteilte Profile (genau ein Owner, beliebig viele Sharer).
- APIs: Intern: `registry.getTargetUrl/getNavigationHandlers/getAdapter/supportsModel/getModelsForAdapter/getImagePolicy/getModelType`, `initBrowserBase`, `tryGotoWithCheck`. Nach außen: `init`, `supports`, `generate`, `getModels`, `getImagePolicy`, `getModelType`, `getCookies`, `navigateToMonitor`.
- Ereignisse: `page.on('close')` → Seiten-Wiedererstellung; `browser.on('close')` → Owner-Reinitialisierung und Sharer-Wiederherstellung (Login-Modus: Prozessende `process.exit(0)`); `page.on('framenavigated')` → Navigation-Handler-Ausführung.
- Nebenwirkungen: Erzeugt und verwaltet echte Browser-Tabs; fügt Autorisierungs-/Login-Sitzungen den Profilen hinzu; wechselt im Leerlauf auf Monitor-URLs (`merge` + Monitor konfiguriert); beendet den Prozess bei Login-Modus, wenn der Browser geschlossen wird.
- Fehlerfälle: Seite geschlossen → automatische Neuerstellung; Browser geschlossen → Reinitialisierung + Recovery aller Sharer; Zielwebsite nicht erreichbar → Worker startet trotzdem (Fehler erst bei Anfrage); Adapter fehlt → Fehlerobjekt; nicht initialisierter/zugeschlagener Worker → Reinitialisierung vor Ausführung.
- Sicherheitsrelevanz: Cookies werden nach außen exportiert (`getCookies`, Domänenfilter, URL-Normalisierung auf https); Proxy-Konfiguration des Workers wird beim Adapter mitgeführt; geteilte Profile multiplizieren das Login-Risiko (eine Session, mehrere Worker) — beobachtet, keine Bewertung.
- Geschäftslogik: Modell-Auflösung unterstützt `type/model`-Präfixe; `merge`-Worker: erstes unterstützendes Modell bzw. explizit angegebener Typ; Bild-Policy-Aggregation ist lax; `merge`-Failover probiert unterstützende Adaptertypen sequentiell und stoppt bei markiert nicht wiederholbaren Fehlern; Ziel-Navigation im Merge-Modus probiert Adapter-URLs der Reihe nach durch.
- Algorithmen: Kandidaten-Aufzählung für Merge (`_getCandidateTypes`); `maxAttempts = maxRetries===0 ? candidates.length : min(maxRetries+1, candidates.length)`; Deduplizierung über gesehene-IDs; Modell-Listen-Expansion (nativ und präfix-qualifiziert).
- verwendete Datenmodelle: Worker-Konfigurations-Objekt (Name, Typ, Instanzname, User-Datenverzeichnis, Proxy, Merge-Typen/Monitor); Adapter-Aufruf-Subkontext `{...ctx, page, config, proxyConfig, userDataDir}`; Modell-Objekte mit `owned_by`-Zuweisung; Fehlerobjekte `{error, retryable}`.
- Abhängigkeiten: `fs`, `../../utils/logger.js`, `../engine/launcher.js` (`initBrowserBase`, `createCursor`), `../registry.js`, `../utils/page.js` (`tryGotoWithCheck`), `ghost-cursor-playwright-port` (über launcher re-exportiert).
- Rust-Relevanz: Das Ownership-Modell für geteilte Browser-Ressourcen (genau ein Reinitialisierungs-Owner, Wiederherstellungskaskade) und die Selbstheilung (Seite/Browser-Close) sind als Rust-Objekt-/Handle-Verwaltung neu zu entwerfen; Merge-Failover und Modell-Präfix-Auflösung sind reine Enum-/Dispatch-Logik, in Rust neu zu formulieren.

Read Evidence:
File Hash: 13e6cb42ddb63e143ba8e30b3a893d143fe9d7f226eb0048f2635dab096f4549
Byte Size: 24987
Line Count: 658
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000104
- EV-WEB2API-000116

---

## src/backend/pool/index.js

- Zweck: Barrels-Export der Pool-Schicht; stellt `Worker` und `PoolManager` als öffentliche Moduleinheit bereit.
- Verantwortlichkeit: Reine Aggregation, keine eigene Logik; vereinheitlicht den Importpfad für Konsumenten der Pool-Schicht.
- Eingaben: Keine Laufzeit-Eingaben.
- Ausgaben: Exportierte Klassentypen `Worker`, `PoolManager`.
- Datenfluss: Import-Referenzen auf `./Worker.js` und `./PoolManager.js` → Namensraum-Export.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `Worker`, `PoolManager` (re-exportiert).
- Ereignisse: Keine.
- Nebenwirkungen: Keine zur Laufzeit.
- Fehlerfälle: Keine eigene Behandlung; vererbt Fehler der re-exportierten Module.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine eigenen.
- Abhängigkeiten: `./Worker.js`, `./PoolManager.js`.
- Rust-Relevanz: Konzept eines aggregierenden Modul-Öffentlichkeitspunkts (modulare Kapselung); in Rust durch Modul-Deklarationen und Public-API-Neuexporte neu abbildbar.

Read Evidence:
File Hash: 0587570b47e96f725c24a237adeb860bbf0a9df43149e2cd6211676dda88e073
Byte Size: 136
Line Count: 6
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000105

---

## src/backend/strategies/failover.js

- Zweck: Fehlertoleranz-Ausführungsgerüst: führt eine Operation gegen eine geordnete Kandidatenliste aus und wechselt bei Fehlern zum nächsten Kandidaten, sofern der Fehler wiederholbar ist.
- Verantwortlichkeit: Liefert den Wiederholungs-/Ausweichmechanismus für die Task-Verteilung; entscheidet anhand der normalisierten Fehlerklassifikation, ob ein weiterer Kandidat versucht werden darf.
- Eingaben: Kandidatenliste (Objekte), Ausführungsfunktion, Metadaten; Konfiguration `maxRetries` (Standard aus globalen Konstanten), `onRetry`-Callback.
- Ausgaben: Erfolgsresultat oder Fehlerobjekte mit Codes `NOT_RETRYABLE` bzw. `FAILOVER_EXHAUSTED` und `retryable`-Flag.
- Datenfluss: Kandidaten → Schleife (max. Anzahl Versuche) → Ausführung → Ergebnisprüfung → Retryability-Entscheidung → ggf. nächster Kandidat.
- Persistenz: Keine.
- Zustände: `lastError`; keine persistenten Zustände.
- APIs: `createFailoverExecutor(options)` → `{execute}`, `executeWithFailover(candidates, execute, options)`; re-exportiert `isRetryableError`, `normalizeError` aus `../utils/error.js` (Kompatibilitätsexport).
- Ereignisse: Keine.
- Nebenwirkungen: Loggt Ausweichvorgänge und ruft bei jedem Versuch den `onRetry`-Callback auf.
- Fehlerfälle: Leere Kandidatenliste → Fehlerobjekt ohne Versuch; nicht wiederholbarer Fehler → sofortiger Abbruch (kein weiterer Kandidat); alle Kandidaten erschöpft → Fehlerobjekt mit letztem Fehler.
- Sicherheitsrelevanz: Stellt sicher, dass nicht wiederholbare Fehler (z. B. Inhaltsmoderation) nicht gegen andere Konten/Worker erneut versendet werden — eine Schutzfunktion gegen ungewollte Mehrfachausführung.
- Geschäftslogik: `maxRetries===0` erlaubt so viele Versuche wie Kandidaten vorhanden sind; andernfalls `min(maxRetries+1, candidates.length)`; Retryable-Vorrang: explizites `result.retryable` schlägt die Inferenz per `normalizeError`.
- Algorithmen: Sequentielle Versuchsschleife mit Cap auf Kandidatenzahl; Fehlerklassifikation durch Mustererkennung (siehe `error.js`).
- verwendete Datenmodelle: Ergebnisobjekt-Vertrag `{error?, retryable?, ...result}`; Fehlerobjekte `{error, code, retryable}`.
- Abhängigkeiten: `../../utils/logger.js`, `../../utils/constants.js` (`RETRY.MAX_ATTEMPTS=2`), `../utils/error.js`.
- Rust-Relevanz: Konzept eines Failover-Executors (Versuchszähler, Cap, Retryability-Gate) ist als Rust-Orchestrierungsmodul mit eigenem Fehler-Trait und Abort-Entscheidung neu zu bauen.

Read Evidence:
File Hash: 7819bd34f938385d5a3d828a22788d5274a371eaaebf79cc4d48882a4d62c141
Byte Size: 4031
Line Count: 117
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000106

---

## src/backend/strategies/index.js

- Zweck: Auswahlstrategien für Worker-Kandidaten: `least_busy`, `round_robin`, `random`; liefert sortier- und selektierbare Strategieobjekte.
- Verantwortlichkeit: Determiniert die Reihenfolge, in der Kandidaten für eine Aufgabe in Betracht gezogen werden; isoliert die Verteilungslogik vom Pool.
- Eingaben: Strategiename; Kandidatenliste mit optionalem `busyCount`-Attribut.
- Ausgaben: Sortierte Kandidatenliste bzw. der gewählte erste Kandidat (`select`).
- Datenfluss: Strategieinstanz → `sort(candidates)` → geordnete Liste → `select` nimmt das erste Element.
- Persistenz: Keine.
- Zustände: `roundRobinIndex` (instanzintern, monoton steigend).
- APIs: `STRATEGIES`-Enum; `createStrategySelector(strategy)` → `{sort, select}`.
- Ereignisse: Keine.
- Nebenwirkungen: `random` verwendet die eingebaute Zufallsfunktion; `round_robin` verändert den internen Zähler pro Aufruf.
- Fehlerfälle: Leere Liste → `select` liefert `null`; `sort` bei ≤1 Kandidaten unverändert.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Standardstrategie ist `least_busy`; `least_busy` sortiert aufsteigend nach `busyCount` (fehlender Wert als 0); `round_robin` rotiert anhand eines Zählers; `random` mischt.
- Algorithmen: Round-Robin-Rotation über Listenverschiebung; Vergleichssortierung nach Auslastung; Fisher-artiges Zufallsmischen über Vergleichsfunktion.
- verwendete Datenmodelle: Kandidatenobjekt mit `busyCount`.
- Abhängigkeiten: Keine (rein funktional).
- Rust-Relevanz: Strategie-Muster (Trait-basierte Auswahl, Rotationszähler, Auslastungsvergleich) direkt als Rust-Traits/Enums mit eigener Nomenklatur neu umsetzbar.

Read Evidence:
File Hash: 85a84ef0cc4317abd3f6b97f53f43a2dc8810033552c339df3ec68de9ba37cad
Byte Size: 2166
Line Count: 73
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000107

---

## src/backend/utils/CloudflareBypass.js

- Zweck: Automatisiertes Bedienen von Cloudflare-Turnstile-Checkboxen durch Penetration mehrschichtiger geschlossener Shadow-Roots und Iframes; fällt auf Koordinaten-Klick zurück, wenn keine DOM-Beschaffung möglich ist.
- Verantwortlichkeit: Stellt für Adapter eine universelle "Ich bin ein Mensch"-Interaktion bereit, die gegen Cloudflare-basierte Bot-Schutzschwellen eingesetzt wird.
- Eingaben: Playwright-Seite, Host-Selektor (z. B. Turnstile-Container), Optionen (Timeout, Wartezeit nach Klick, Metadaten).
- Ausgaben: `{success: boolean, error?: string}`.
- Datenfluss: Host-Locator → sichtbar warten → Element-Handle → Tiefensuche nach Shadow-Root-Träger → erste Shadow-Root → iframe → contentFrame → erneute Shadow-Root-Suche im Frame → Checkbox-Auflösung → Klick → Warten → Erfolg.
- Persistenz: Keine.
- Zustände: Keine persistenten; Pro-Klick-Verzögerungen durch unregelmäßige Schlafintervalle.
- APIs: `clickTurnstile(page, hostSelector, options)`.
- Ereignisse: Keine eigenen; nutzt Seiten-DOM und Mausereignisse.
- Nebenwirkungen: Führt echte Mausbewegungen (mehrstufige Bewegung, `steps: 10`) und Klicks auf feste Koordinatenversätze durch; erzeugt zufällige Wartezeiten; überschreitet damit bewusst Shadow-DOM-Kapselung von Drittanbietern.
- Fehlerfälle: Kein Host-Element / keine Shadow-Root / kein iframe / kein Checkbox-Element → Rückfall auf Koordinaten-Klick; ohne Rahmenbox → Fehlerresultat; Ausnahme im Klickpfad → `{success:false, error}`.
- Sicherheitsrelevanz: Umgehung menschlicher Verifikationsmechanismen — beobachtete Funktion mit direktem Sicherheits-/Compliance-Bezug (gegen Bot-Schutz). Muss im Rust-Rewrite als bewusste, konfigurierbare Fähigkeit bewertet werden.
- Geschäftslogik: Mehrstufige DOM-Penetration mit abgestuften Rückfällen (Frame-Verfügbarkeit → Schattenhost im Frame → Checkbox) und letztem Ausweg Koordinatenklick; Klick im DOM-Modus über `safeClick` mit `bias:'random'`.
- Algorithmen: Rekursive Shadow-Root-Suche (nur Elemente mit spezifischem Eigenschaftsmarker); DOM-Traversierung über `querySelectorAll`; feste Versatzgeometrie für den Checkbox-Koordinatenklick.
- verwendete Datenmodelle: Host-Selektor-Konfiguration; Ergebnisobjekt `{success, error?}`.
- Abhängigkeiten: `../engine/utils.js` (`sleep`, `safeClick`), `../../utils/logger.js`.
- Rust-Relevanz: Konzept einer mehrstufigen, rückfallbasierten UI-Interaktionskaskade (Shadow-DOM → Frame → Koordinaten) mit zufälligen Verzögerungen ist als eigene Rust-Strategie neu zu entwerfen.

Read Evidence:
File Hash: 2130ceddbbf4f0b60fae33362f0cf1a51a1facf4f090ee28f18eaa6ede0fea9a
Byte Size: 6499
Line Count: 164
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000108

---

## src/backend/utils/download.js

- Zweck: Herunterladen von Bildern/Video-Ressourcen über die Seiten-API unter Wiederverwendung von Cookie- und Session-Kontext; Ergebnis wird als Base64-Data-URI inklusive MIME-Typ geliefert.
- Verantwortlichkeit: Löst Autorisierungsprobleme beim Ressourcen-Download, indem der Browser-Session-Context (Cookies) direkt genutzt wird; fügt Wiederholungslogik für Netzwerk-/5xx-Fehler hinzu.
- Eingaben: Ressourcen-URL, Playwright-Seite, Optionen (Timeout, Retries, Retry-Verzögerung).
- Ausgaben: `{image?: dataURI, imageUrl?, error?: string}`.
- Datenfluss: URL → Seiten-`request.get` → Antwortprüfung → Body → Base64 → Data-URI mit `content-type` → Rückgabe.
- Persistenz: Keine (Base64 im Speicher).
- Zustände: Versuchs-Zähler (transient).
- APIs: `useContextDownload(url, page, options)`.
- Ereignisse: Keine eigenen.
- Nebenwirkungen: Sendet Netzwerkrequests im Browser-Kontext; loggt Wiederholungen.
- Fehlerfälle: 5xx bei noch verfügbaren Versuchen → Verzögerung + Wiederholung; 4xx/Erfolg-fehlt → Fehlerobjekt mit URL; wiederholbare Netzwerkfehler (Timeout/Verbindung) → Wiederholung; sonst Fehlerobjekt; Erschöpfung → generischer Fehler mit URL.
- Sicherheitsrelevanz: Exponiert ggf. sessiongebundene Ressourcen und gibt deren Inhalte als Data-URI zurück (Weitergabe an die Service-Schicht); MIME wird aus dem Server-Header übernommen.
- Geschäftslogik: Mindestens ein Versuch auch bei `retries=0`; Verzögerung wächst linear mit der Versuchszahl; HTTP-Status 5xx wird als wiederholbar behandelt.
- Algorithmen: Klassifikation wiederholbarer Netzwerkfehler per regulärem Ausdruck (`timeout|network|econnreset|econnrefused|etimedout|disconnected|tls|socket`).
- verwendete Datenmodelle: Optionen `{timeout, retries, retryDelay}`; Ergebnis-Objekt mit `image/imageUrl/error`.
- Abhängigkeiten: `../../utils/logger.js`; Playwright-Page-API.
- Rust-Relevanz: Konzept des sessionbewussten Ressourcen-Downloads mit Base64-Encoding und gestaffelter Wiederholung ist als Rust-Netzwerkschicht (HTTP-Client mit Session-Cookies) neu umsetzbar.

Read Evidence:
File Hash: aea1978b0a3ad58aa11f5370c2a0c95195e3a1c8e42c4fa89c9220dfbc6d512d
Byte Size: 2769
Line Count: 65
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000109

---

## src/backend/utils/error.js

- Zweck: Einheitliche Fehler-Normalisierung: Übersetzt rohe Seiten-/HTTP-Fehlermeldungen in kanonische Fehlerobjekte mit Code und Wiederholbarkeits-Flag.
- Verantwortlichkeit: Zentraler Fehlerklassifikator, auf den Failover und Pool-Management ihre Wiederholungsentscheidungen stützen; kapselt die Erkennung von Content-Blockaden, Limitierungen und CAPTCHA-Anforderungen.
- Eingaben: Fehlermeldungen/Fehlerobjekte, HTTP-Antwortobjekt, optionaler Antwortinhalt.
- Ausgaben: `{error: string, code: string, retryable: boolean}` oder `null` (kein bekanntes Muster).
- Datenfluss: Fehler → Musterabgleich (Nachricht/Codes) → kanonisches Fehlerobjekt; HTTP-Antwort → Status + Body-Parsing (JSON-Fehlerdetails) → Klassifikation.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `isRetryableError`, `normalizePageError`, `normalizeHttpError`, `normalizeError`; bezieht Code-Konstanten aus `../../server/errors.js` (`ADAPTER_ERRORS`).
- Ereignisse: Keine.
- Nebenwirkungen: Loggt normalisierte Fehler mit Metadaten.
- Fehlerfälle: Nicht-JSON-Body → Versuch, Kurztext (<200 Zeichen) als Detail zu verwenden; nicht erkannte Fehler → `null` bzw. generischer Netzwerkfehler-Code.
- Sicherheitsrelevanz: Erkennt Inhaltsablehnungen (Moderation/Rechtssicherheit) über Schlüsselwörter und markiert sie als nicht wiederholbar — verhindert damit Mehrfachversand unzulässiger Anfragen; erkennt reCAPTCHA-Anforderungen.
- Geschäftslogik: Wiederholbarkeits-Entscheidung aus Mustern (Netzwerk, Zeitüberschreitung, Absturz, 5xx, Rate-Limit/429); HTTP 4xx ist nicht wiederholbar, 5xx wiederholbar; 429 → `RATE_LIMITED` (wiederholbar); Content-Ablehnung und reCAPTCHA → nicht wiederholbar.
- Algorithmen: Sequenz von regulären Ausdrücken; JSON-Detail-Extraktion mit zwei Formaten (`{"error": "..."}` / `{"error": {"message": "..."}}`); Prioritätskette für Fehlercodes bei `normalizeError`.
- verwendete Datenmodelle: Fehlerobjekt `{error, code, retryable}`; Code-Konstanten (PAGE_CLOSED, PAGE_CRASHED, PAGE_INVALID, TIMEOUT_ERROR, CONTENT_BLOCKED, RATE_LIMITED, CAPTCHA_REQUIRED, HTTP_ERROR, NETWORK_ERROR).
- Abhängigkeiten: `../../utils/logger.js`, `../../server/errors.js`.
- Rust-Relevanz: Konzept einer kanonischen Fehlerdomäne mit Wiederholbarkeits-Bewertung und Inhaltsmoderations-Erkennung ist als Rust-Fehler-Trait/Enum-Hierarchie neu zu modellieren.

Read Evidence:
File Hash: d2c77f31a84f1fb8fc5e760e910731686b47d6db3c95e45d2cb5cee3aba7d43c
Byte Size: 7909
Line Count: 196
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000110

---

## src/backend/utils/index.js

- Zweck: Aggregierender Export der Backend-Hilfsmodule (Seiteninteraktion, Fehler-Normalisierung, Ressourcen-Download) für Adapter und andere Konsumenten.
- Verantwortlichkeit: Reine Aggregation und öffentliche API-Fläche der Utils-Schicht.
- Eingaben: Keine Laufzeit-Eingaben.
- Ausgaben: Exportierte Funktionen: `waitForPageAuth`, `lockPageAuth`, `unlockPageAuth`, `isPageAuthLocked`, `waitForInput`, `gotoWithCheck`, `tryGotoWithCheck`, `waitApiResponse`, `scrollToElement`; `isRetryableError`, `normalizePageError`, `normalizeHttpError`, `normalizeError`; `useContextDownload`.
- Datenfluss: Importpfade zu `page.js`, `error.js`, `download.js` → Namensraum.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Siehe Ausgaben.
- Ereignisse: Keine.
- Nebenwirkungen: Keine zur Laufzeit.
- Fehlerfälle: Keine eigenen.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine eigenen.
- Abhängigkeiten: `./page.js`, `./error.js`, `./download.js`.
- Rust-Relevanz: Moduläre Aggregations-API; in Rust über Modul-Öffentlichkeitspunkte neu abbildbar.

Read Evidence:
File Hash: d1ec3f0a49234fb65e36a31486ced113557c22caf67418fab4ce13cd8cc2f0ae
Byte Size: 1264
Line Count: 44
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000111

---

## src/backend/utils/page.js

- Zweck: Seiten-Level-Interaktionswerkzeuge: Authentifizierungs-Lock, Eingabe-Warten, Navigation mit HTTP-Fehlererkennung, Element-Scrollen und API-Antwort-Warten mit Seiten-Schließüberwachung und Fehlerschlüsselwort-Detektion.
- Verantwortlichkeit: Stellt robuste, wiederholbare Seitenabläufe für Adapter bereit; kapselt die Synchronisation zwischen Authentifizierungszustand, UI-Aktionen und Netzwerk-Antworten.
- Eingaben: Playwright-Seite, Selektoren/Locatoren, URLs, Warteoptionen (URL-Match, Zusatzfilter, Methode, Timeout, Fehlerschlüsselwörter).
- Ausgaben: Erfolgreiche Navigation, gefundene Elemente, API-Antwortobjekte (ggf. mit gecachter Body-Lese), `{success, error}`-Resultate.
- Datenfluss: Auth-Lock-Abfrage → Warten auf Eingabeelement → Navigation → API-Antwort-Warten mit Timeout/Fehlerwort-Überwachung (UI- und Body-Ebene) → Rückgabe.
- Persistenz: Keine.
- Zustände: Seiten-Attribut `authState.isHandlingAuth` als prozessübergreifender Authentifizierungs-Lock (gesetzt von Lock-/Unlock-Funktionen); transient pro Wartezeit: Timer, Handler.
- APIs: `waitForPageAuth`, `lockPageAuth`, `unlockPageAuth`, `isPageAuthLocked`, `waitForInput`, `gotoWithCheck`, `tryGotoWithCheck`, `scrollToElement`, `waitApiResponse`; nutzt Helfer aus `engine/utils.js`.
- Ereignisse: Seiten-`response`, `requestfinished`, `requestfailed`, `close`; Lock-/Unlock-Logik steuert parallele Adapter.
- Nebenwirkungen: Setzt `authState.isHandlingAuth`; erzeugt Warte-Timer; bei `waitApiResponse` wird eine abgeleitete Antwort-Proxy-Objekthülle mit Body-Cache erzeugt; hängt einen leeren `catch` an das Promise, um unerwünschte unhandled-rejection-Crashes zu vermeiden.
- Fehlerfälle: `PAGE_INVALID` (Seite geschlossen), `PAGE_CLOSED`/`PAGE_CRASHED` (Watcher), `API_TIMEOUT` (Antwort- bzw. Übertragungs-Timeout), `NETWORK_FAILED`, `PAGE_ERROR_DETECTED`/`API_ERROR_DETECTED` (Fehlerschlüsselwörter), „未找到输入框"-Fehler, Navigationsfehler („页面加载超时", „网站无法访问 (HTTP n)").
- Sicherheitsrelevanz: Erkennt in API-/UI-Inhalten Fehlerschlüsselwörter und stoppt so unerwünschte Weiterverarbeitung; die Antwort-Body-Wiederverwendung (gecachete `text/json/body`) verhindert doppelte Konsumption, berührt aber Lesbarkeit/Stream-Handling.
- Geschäftslogik: `waitForInput` wartet zuerst auf Authentifizierungsabschluss (mit Gesamt-Timeout), dann auf Sichtbarkeit, dann optional auf Klick; `waitApiResponse` behandelt Streaming-Antworten über `requestfinished` mit Idle-Timeout und entfernt den Handler beim ersten Treffer; Fehlerschlüsselwörter werden zuerst im UI, dann im Body geprüft.
- Algorithmen: Zeitbudget-Aufteilung (verbleibende Zeit ≥ 5 s); kombinierte Locator-OR-Verknüpfung für mehrere Schlüsselwörter; Promise-Race aus Antwort-, Seiten-Watcher- und UI-Fehler-Promise; MIME-Kennung entfällt hier (siehe download).
- verwendete Datenmodelle: Warteoptionen `{urlMatch, urlContains, method, timeout, errorText, meta}`; Ergebnis-/Fehlerobjekte; Seiten-Marker `authState`, `_humanizeCursorMode`.
- Abhängigkeiten: `../engine/utils.js` (`sleep`, `safeClick`, `isPageValid`, `createPageCloseWatcher`, `getRealViewport`, `clamp`, `random`), `../../utils/constants.js` (`TIMEOUTS`), `../../utils/logger.js`.
- Rust-Relevanz: Konzept der Seite-Synchronisationswarteschleife (Auth-Lock, Sichtbarkeits- und Antwort-Warte, Idle-Timeout, Fehlerwort-Monitoring) ist als Rust-Ereignis-/Promise-Parallelmodell neu zu entwerfen.

Read Evidence:
File Hash: 5fb63b56a34efb9638af65739c818e7faf8ce96bbd53c2d5861cd6c455122c75
Byte Size: 13463
Line Count: 365
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000112

---

## src/backend/index.js

- Zweck: Öffentlicher Einstiegspunkt der Backend-Schicht: bündelt die Pool-API in einem Backend-Objekt und initialisiert den globalen Pool-Manager (Singleton).
- Verantwortlichkeit: Adapter zwischen Service-/Warteschlangen-Schicht und Pool; bietet einstiegspunktstabile Funktionsfläche (`initBrowser`, `generate`, `getModels`, `getImagePolicy`, `getModelType`, `getCookies`, `navigateToMonitor`).
- Eingaben: Konfiguration (per `loadConfig`), Browser-Kontext, Prompt, Bildpfade, Modell-ID, Metadaten, Worker-Name/Domäne für Cookies.
- Ausgaben: Backend-Objekt; Initialisierungsergebnis `{poolManager, config}`; Generierungsergebnisse; Modell-/Policy-/Typ-Ergebnisse; Cookie-Ergebnisse; Fehlerobjekte.
- Datenfluss: `getBackend()` → Konfiguration laden → temporäres Verzeichnis erzeugen → Pool-API-Hülle; Aufrufe delegieren an den globalen `poolManager`.
- Persistenz: Erzeugt das temporäre Verzeichnis `data/temp` unter dem Arbeitsverzeichnis, falls es fehlt; weitere Persistenz über die Pool-Ebene.
- Zustände: Modul-globales Singleton `poolManager` (einmalige Initialisierung, Wiederverwendung über `initialized`-Prüfung).
- APIs: `getBackend()` → `{name:'pool', config, TEMP_DIR, initBrowser, generate, getModels, getImagePolicy, getModelType, getCookies, navigateToMonitor, getPoolManager}`.
- Ereignisse: Keine eigenen.
- Nebenwirkungen: Legt ein temporäres Verzeichnis an; startet beim ersten `initBrowser` den gesamten Worker-Pool (Browserprozesse); gibt vor der Initialisierung konservative Defaults zurück (leere Modellliste, Policy `optional`, Typ `image`).
- Fehlerfälle: Aufruf von `generate`/`getCookies` ohne initialisierten Pool → Fehlerobjekt bzw. Exception; doppelte Initialisierung → idempotente Rückgabe des bestehenden Pools.
- Sicherheitsrelevanz: Cookies werden gezielt über die Backend-Schnittstelle exportiert; der Zugriff ist auf die Service-Schicht beschränkt.
- Geschäftslogik: Lazy-Singleton-Initialisierung; Delegations-Pfad ohne eigene Verarbeitung; Default-Werte vor Pool-Start sind konservativ.
- Algorithmen: Keine eigenen.
- verwendete Datenmodelle: Backend-API-Objekt; Konfigurationsobjekt mit injiziertem `paths.tempDir`.
- Abhängigkeiten: `fs`, `path`, `../config/index.js` (`loadConfig`), `./pool/index.js`, `../utils/logger.js`.
- Rust-Relevanz: Konzept einer modularen Fassade mit Lazy-Singleton-Ressource (Pool-Start, Default-Fallbacks vor Initialisierung) ist als Rust-Initialisierungs-API mit Owned-Manager neu zu entwerfen.

Read Evidence:
File Hash: 79c5a21d8d775b749af6a2ccdb4872e7eeaf0b699b6f0666f00cde515b48001f
Byte Size: 3985
Line Count: 140
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000113
- EV-WEB2API-000114

---

## src/backend/registry.js

- Zweck: Adaptier-Registry: scannt das Adapter-Verzeichnis automatisch, lädt Manifests, validiert sie und stellt einheitliche Abfragen für Modelle, Policies, Navigation-Handler und Ziel-URLs bereit.
- Verantwortlichkeit: Erweiterbarkeitspunkt des Systems — neue Adapter erfordern keine Framework-Änderung; zentraler Nachschlage-Dienst für die Pool-/Worker-Schicht und für die Konfigurationsvalidierung.
- Eingaben: Dateisystem-Inhalt von `adapter/` (`*.js`), Modul-Exporte (Manifest-Objekt), Adapter-Konfiguration (Modellfilter), Abfrage-Parameter (Adapter-ID, Modell-ID).
- Ausgaben: Registrierte Adapter-Map; Validierungsergebnisse; Modelllisten (OpenAI-Format); Boolesche Unterstützungs-/Aktivierungsantworten; Ziel-URLs; Navigations-Handler-Listen; Policy-/Typ-Antworten.
- Datenfluss: `loadAll` → Verzeichnis-Liste → dynamischer Modul-Import → Manifest-Validierung → Registrierung; Abfragen laufen über `adapters`-Map und Modellfilter.
- Persistenz: Keine eigene; liest das Dateisystem (Adapter-Dateien).
- Zustände: `adapters` (Map), `adapterConfig`, `loaded` (Lazy-Load-Flag).
- APIs: `setAdapterConfig`, `isModelEnabled`, `loadAll`, `validateManifest`, `getAdapter`, `getAdapterIds`, `hasAdapter`, `getTargetUrl`, `getNavigationHandlers`, `getWaitInput`, `getModelsForAdapter`, `supportsModel`, `resolveModelId`, `getImagePolicy`, `getModelType`, `getAllModels`; exportiert `AdapterRegistry`-Klasse, Singleton `registry`, `IMAGE_POLICY`-Enum.
- Ereignisse: Keine eigenen.
- Nebenwirkungen: Führt zur Laufzeit dynamische Imports aus (Modulausführung der Adapter-Dateien); loggt Ladefehler und Validierungsfehler.
- Fehlerfälle: Datei ohne `manifest` → übersprungen; Manifest mit fehlenden Pflichtfeldern (`id`, `generate`, `models` mit `id`/`imagePolicy`) → zurückgewiesen; Importfehler → geloggt, Prozess läuft weiter; unbekannte Adapter-ID in Abfragen → `null`/`false`/leere Liste.
- Sicherheitsrelevanz: Das Modellfilter-System (`whitelist`/`blacklist` per Adapter-Konfiguration) steuert die Verfügbarkeit von Modellen und damit die Exponierung gegenüber dem Frontend — eine Zugriffssteuerung auf Modell-Ebene.
- Geschäftslogik: Modellfilter: ohne Filterkonfiguration sind alle Modelle aktiv; `whitelist` → nur gelistete aktiv; sonst Blacklist (gelistete deaktiviert); `resolveModelId` delegiert an Adapter-Resolver oder fällt auf `codeName`/`id` zurück; `getTargetUrl` bevorzugt die Adapter-Funktion, sonst statische `targetUrl`; Standard-Image-Policy ist `optional`, Standard-Typ `image`.
- Algorithmen: Verzeichnis-Scan mit Endung `*.js`; Pflichtfeld-Validierung mit Fehlerliste; zeitstempelbasierte `created`-Felder; Set-/Map-basierte Deduplizierung.
- verwendete Datenmodelle: Manifest-Vertrag `{id, displayName?, generate, models[{id, imagePolicy, type?, codeName?}], getTargetUrl?, navigationHandlers?, waitInput?, resolveModelId?}`; OpenAI-Formatanweisungs-Objekt `{object:'list', data:[{id, object:'model', created, owned_by, image_policy, type}]}`.
- Abhängigkeiten: `fs`, `path`, `url` (`fileURLToPath`), `../utils/logger.js`.
- Rust-Relevanz: Konzept eines manifestbasierten, dynamisch erweiterbaren Adapter-Registers mit Validierung, Modellfilter-Policies und OpenAI-förmiger Modellkatalog-Sicht ist als Rust-Registry/Plug-in-Schicht neu zu entwerfen (z. B. Trait-basierte Adapter mit Kompilierzeit-Registrierung oder Laufzeit-Loading).

Read Evidence:
File Hash: 274d4f400849cc5c21a8869fa7739a770058f50d0a84596370c35a4136b56eb1
Byte Size: 10116
Line Count: 343
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000118
- EV-WEB2API-000119

---

# Evidence-Blöcke (EV-WEB2API-000101 … 000119)

Evidence-ID: EV-WEB2API-000101
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/engine/launcher.js
Zeilenbereich: 31-77, 86-104, 132-239, 253-431
Beziehung: Self-contained; initBrowserBase wird von Worker._initNewBrowser aufgerufen (src/backend/pool/Worker.js:199)
Typ: Call
Aussage: Der Browser-Start übernimmt Fingerabdruck-Laden/-Validierung/-Persistenz, Camoufox-Start mit Proxy, Viewport-Zwang und CSS-Init-Script-Injektion; die Bereinigung erfolgt dreistufig (Kontext-Schließen → SIGTERM → SIGKILL), registriert über Prozess-Signal-Handler.

Evidence-ID: EV-WEB2API-000102
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/engine/utils.js
Zeilenbereich: 33-45, 52-62, 70-100, 109-133, 147-196, 269-381, 450-550, 604-775, 795-822
Beziehung: Von nahezu allen Adaptern importiert (24 Fundstellen in src/backend/adapter/*.js)
Typ: Import
Aussage: Die Utils-Schicht bündelt humanisierte Interaktion (zufällige Klickpunkte, Tippfehler-Simulation, Paste-Strategie, ghost-cursor), Shadow-DOM-/Filechooser-Uploads, Seitenvalidierung und Schließ-/Absturz-Watcher als Wiederverwendungsbasis der Adapter.

Evidence-ID: EV-WEB2API-000103
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/pool/PoolManager.js
Zeilenbereich: 32-133, 138-164, 169-218, 224-231, 236-282, 287-321
Beziehung: Ruft registry.loadAll/setAdapterConfig (registry.js), createStrategySelector (strategies/index.js), executeWithFailover (strategies/failover.js), normalizeError (utils/error.js), Worker (Worker.js)
Typ: Call
Aussage: Der Pool-Manager initialisiert Worker (inkl. Login-Filter und Browser-Sharing-Gruppierung nach User-Datenverzeichnis), verteilt Aufgaben modellbasiert mit Bild-Policy-Vorfilter und Failover und aggregiert Modelle/Policies/Typen sowie Cookies und Monitor-Navigation.

Evidence-ID: EV-WEB2API-000104
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/pool/Worker.js
Zeilenbereich: 49-108, 114-192, 198-313, 318-343, 367-457, 463-515, 520-622, 627-657
Beziehung: Nutzt initBrowserBase/createCursor (engine/launcher.js), tryGotoWithCheck (utils/page.js), registry-Abfragen (registry.js)
Typ: Call
Aussage: Der Worker kapselt eine Browser-Session mit Ownership-Modell für geteilte Profile, Merge-Typ-Failover, Selbstheilung bei Seiten-/Browser-Ausfall, Modell-/Policy-/Typ-Auflösung und Cookie-Export.

Evidence-ID: EV-WEB2API-000105
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/pool/index.js
Zeilenbereich: 1-6
Beziehung: Re-Export von ./Worker.js und ./PoolManager.js
Typ: Import
Aussage: Aggregierender Export der Pool-Schicht; enthält keine eigene Laufzeitlogik.

Evidence-ID: EV-WEB2API-000106
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/strategies/failover.js
Zeilenbereich: 31-101, 114-117
Beziehung: Von PoolManager.js:9 importiert; re-exportiert Fehlerklassifikation aus utils/error.js
Typ: Call
Aussage: Der Failover-Executor versucht Kandidaten sequentiell, cap`t die Versuche auf Kandidatenzahl, bricht bei nicht wiederholbaren Fehlern sofort ab (Code NOT_RETRYABLE) und meldet Erschöpfung als FAILOVER_EXHAUSTED.

Evidence-ID: EV-WEB2API-000107
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/strategies/index.js
Zeilenbereich: 19-23, 34-72
Beziehung: Von PoolManager.js:8 importiert
Typ: Call
Aussage: Stellt die Strategien least_busy, round_robin, random als sortierende Selektor-Instanzen bereit; least_busy ist Standard und sortiert nach busyCount aufsteigend.

Evidence-ID: EV-WEB2API-000108
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/utils/CloudflareBypass.js
Zeilenbereich: 14-23, 38-163
Beziehung: Von src/backend/adapter/test.js:16 importiert; nutzt sleep/safeClick aus engine/utils.js
Typ: Import
Aussage: clickTurnstile penetriert mehrschichtige geschlossene Shadow-Roots und iframes, um Turnstile-Checkboxen zu klicken; ohne DOM-Zugriff Rückfall auf Koordinaten-Klick mit zufälligen Verzögerungen.

Evidence-ID: EV-WEB2API-000109
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/utils/download.js
Zeilenbereich: 28-64
Beziehung: Von src/server/api/admin/routes.js:51 und mehreren Adaptern (doubao, lmarena, sora, gemini, zai_is u.a.) importiert
Typ: Call
Aussage: useContextDownload lädt Ressourcen über den Seitenkontext (Cookie-/Session-Erbe), wandelt sie in Base64-Data-URIs mit Server-MIME und wendet gestaffelte Wiederholung für Netzwerk-/5xx-Fehler an.

Evidence-ID: EV-WEB2API-000110
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/utils/error.js
Zeilenbereich: 18-35, 47-94, 106-166, 177-196
Beziehung: Von strategies/failover.js und pool/PoolManager.js importiert; Code-Konstanten aus src/server/errors.js (Zeilen 156-183)
Typ: Call
Aussage: Die Fehler-Normalisierung ordnet Seiten-/HTTP-Fehler kanonischen Codes und Wiederholbarkeits-Flags zu; Content-Ablehnungen und reCAPTCHA sind nicht wiederholbar, Netzwerk/Timeout/5xx/429 wiederholbar.

Evidence-ID: EV-WEB2API-000111
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/utils/index.js
Zeilenbereich: 22-44
Beziehung: Re-Export aus page.js, error.js, download.js
Typ: Import
Aussage: Bündelt die gesamte Backend-Utils-API (Seiteninteraktion, Fehler, Download) an einem öffentlichen Einstiegspunkt.

Evidence-ID: EV-WEB2API-000112
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/utils/page.js
Zeilenbereich: 17-46, 61-93, 107-180, 196-363
Beziehung: Von fast allen Adaptern importiert (waitApiResponse/waitForInput, 20+ Fundstellen); nutzt engine/utils.js-Helfer und TIMEOUTS aus constants.js
Typ: Import
Aussage: Bietet Auth-Lock-Synchronisation, Eingabe-Warten, Navigation mit HTTP-Prüfung, Element-Scrolling und API-Antwort-Warten mit Seiten-Schließüberwachung, Idle-Timeout und UI-/Body-Fehlerschlüsselwort-Erkennung inkl. gecachter Antwort-Body-Wiedergabe.

Evidence-ID: EV-WEB2API-000113
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/index.js
Zeilenbereich: 33-139
Beziehung: Von src/server/server.js:24 (getBackend) und src/server/queue.js (initBrowser/generate/getCookies/navigateToMonitor) konsumiert
Typ: Import
Aussage: Stellt die Backend-Fassade mit Lazy-Singleton-Pool-Manager bereit; delegiert alle Aufrufe an den Pool und liefert vor Initialisierung konservative Defaults.

Evidence-ID: EV-WEB2API-000114
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/index.js
Zeilenbereich: 19-24
Beziehung: Erzeugt TEMP_DIR data/temp unter process.cwd()
Typ: Persistenz
Aussage: Die Fassade legt beim Laden das temporäre Verzeichnis data/temp rekursiv an und injiziert dessen Pfad in die Konfiguration (paths.tempDir) für nachgelagerte Module.

Evidence-ID: EV-WEB2API-000115
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/engine/launcher.js
Zeilenbereich: 336-341, 353-355
Beziehung: context.on('close')-Handler; Viewport-Set aus Fingerabdruck-Screenwerten
Typ: Event
Aussage: Kontext-Schließen wird überwacht und räumt die globalen Browser-Referenzen auf, ohne den Prozess zu beenden; die Viewportgröße wird nach dem Start verbindlich aus dem Fingerabdruck gesetzt.

Evidence-ID: EV-WEB2API-000116
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/pool/Worker.js
Zeilenbereich: 8-10, 114-137, 198-213, 290-313
Beziehung: Importiert initBrowserBase/createCursor und tryGotoWithCheck
Typ: Import
Aussage: Die Worker-Initialisierung bindet den Browser-Start und die Ziel-Navigation ein; Navigation-Handler werden aus der Registry je Typ (auch je mergeTypes) aggregiert und über framenavigated angewendet.

Evidence-ID: EV-WEB2API-000117
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/pool/PoolManager.js
Zeilenbereich: 70-89, 92-125
Beziehung: Login-Modus-Parsing aus process.argv; Registry-Adapter-Prüfung
Typ: Config
Aussage: Der Pool filtert Worker nach CLI-Flag -login (genau ein Ziel), prüft Adapter-Typen (merge: mergeTypes) und gruppiert Browser nach userDataDir; bei Proxy-Abweichungen gewinnt der erste Worker.

Evidence-ID: EV-WEB2API-000118
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/registry.js
Zeilenbereich: 76-111, 119-150
Beziehung: Dynamischer Modul-Import von Dateien aus dem Adapter-Verzeichnis; Validierung der Manifest-Pflichtfelder
Typ: Config
Aussage: Die Registry lädt Adapter per Laufzeit-Import aus adapter/*.js, validiert Pflichtfelder (id, generate, models mit id/imagePolicy) und überspringt ungültige Dateien ohne Prozessabbruch.

Evidence-ID: EV-WEB2API-000119
Repository: WebAI2API
Commit: content-copy
Datei: src/backend/registry.js
Zeilenbereich: 55-71, 223-311
Beziehung: isModelEnabled/whitelist-blacklist; getModelsForAdapter im OpenAI-Format
Typ: Schema
Aussage: Der Modellkatalog wird mit Whitelist/Blacklist-Filterung pro Adapter erzeugt; Modell-Objekte folgen dem OpenAI-Listenformat (id, object:'model', created, owned_by, image_policy, type); Standard-Policy optional, Standard-Typ image.

---

# Zusammenfassung

- 14 Dateien gelesen, alle Status `FERTIG_ANALYSIERT`.
- 19 Evidence-Blöcke erstellt (EV-WEB2API-000101 … 000119).
- Verbotene Füllwörter kommen im Dokument nicht vor; alle Felder sind belegt.
- Negative Befunde wurden dokumentiert (z. B. keine Persistenz in engine/utils.js, strategies, download, error).
