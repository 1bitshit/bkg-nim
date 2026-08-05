# WebAI2API — Web-UI (webui/)

Repository: `WebAI2API`
Commit: content-copy (lokal geklonte Arbeitskopie)
Datei-für-Datei-Analyse des Web-Frontends. Alle Pfade relativ zu `repos/WebAI2API/webui/`.
Belegquellen sind als `EV-WEB2API-0003xx`-Evidence-Blöcke referenziert.
Insgesamt 40 Dateien, vollständig gelesen (22 Quell-/Konfigurationsdateien + 18 Build-Artefakte unter `dist/`).

---

## .npmrc

- Zweck: Projektlokale pnpm-Konfiguration mit zwei Schaltern (belegt durch Dateiinhalt, 3 Zeilen).
- Verantwortlichkeit: Steuert das Lockfile- und Versionsverhalten des pnpm-Workspace für dieses Paket.
- Eingaben: Keine zur Laufzeit; Build-/Install-Zeit: pnpm.
- Ausgaben: Keine zur Laufzeit; beeinflusst Lockfile-Erzeugung.
- Datenfluss: Kein Laufzeit-Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: `shared-workspace-lockfile=false` erzwingt ein eigenes Lockfile pro Workspace-Paket; `use-workspace-root-version=false` verbietet die Version-Übernahme von der Workspace-Wurzel (belegt durch Dateiinhalt).
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pnpm.
- Rust-Relevanz: Konzept der eigenständigen, vom Monorepo-Wurzel entkoppelten Abhängigkeitsauflösung (eigenes Lockfile pro Paket) ist als Build-/Cargo-Workspace-Designfrage für die Rust-Neuentwicklung zu berücksichtigen; Dateiinhalt selbst wird nicht übernommen.

Read Evidence:
File Hash: a0141949f9aa0d896ddab1e8725e02e61585d705620f91e9ca44b5266ab2fe34
Byte Size: 123
Line Count: 3
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000301

---

## index.html

- Zweck: Statischer HTML-Einstiegspunkt der SPA mit UTF-8, Favicon-Verknüpfung, Viewport-Meta, Titel `WebAI2API`, Mount-Punkt `<div id="app">` und Modul-Referenz auf `src/main.js` (belegt durch Dateiinhalt, 15 Zeilen).
- Verantwortlichkeit: Liefert die Ladefläche, über die die Vue-Anwendung gebootstrappt wird.
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Render-Container und Initialladepfad für den Browser.
- Datenfluss: Browser lädt `/favicon.png` + `/src/main.js` → Vue-App mountet in `#app`.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `public/favicon.png`, `src/main.js`.
- Rust-Relevanz: Konzept des schlanken HTML-Einstiegspunkts, der eine Client-Anwendung lädt, ist als statische Auslieferung (einzelne HTML-Seite + Bundle) in der Rust-Serverimplementierung neu zu denken; die HTML-Datei selbst wird nicht übernommen.

Read Evidence:
File Hash: 7e9259288636283da861ccd33fef26e45d750e50e37e6d9fe1b66462958f2366
Byte Size: 335
Line Count: 15
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000302

---

## package.json

- Zweck: Paket-Manifest des Web-UI (`private`, ohne Versionsnummer) mit Laufzeit- und Entwicklungs-Abhängigkeiten sowie npm-Skripten für dev/build/preview (belegt durch Dateiinhalt, 23 Zeilen).
- Verantwortlichkeit: Deklariert den Frontend-Stack (Vue 3, Router, Pinia, Ant Design Vue, noVNC) und die Vite-Build-Skripte.
- Eingaben: Keine zur Laufzeit; Build-Zeit: npm/pnpm + Registry.
- Ausgaben: Build-Artefakte unter `dist/`.
- Datenfluss: Build-Graph; Skripte `dev` (Vite), `build` (Vite build), `preview` (Vite preview).
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Installiert das Modulbündel; `build` erzeugt die ausgelieferten Bundles.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: `private: true` verhindert versehentliche Veröffentlichung; Laufzeit-Abhängigkeit auf `@novnc/novnc` (VNC-Client) und `ant-design-vue` ist Teil des Exponierungs-Profils.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Laufzeit: `@ant-design/icons-vue ^7.0.1`, `@novnc/novnc 1.4.0`, `ant-design-vue ^4.2.6`, `pinia ^3.0.4`, `vue ^3.5.24`, `vue-router ^4.6.4`; Dev: `@vitejs/plugin-vue ^6.0.1`, `vite ^7.2.4` (belegt durch Dateiinhalt).
- Rust-Relevanz: Die Fähigkeitsliste (SPA, Zustandsverwaltung, UI-Komponentenbibliothek, VNC-Client, Routing) definiert den Rahmen für eine Rust-Web-Frontend-Neuentwicklung (z. B. Yew/Leptos + WebSocket-VNC); die konkreten JS-Pakete werden nicht übernommen.

Read Evidence:
File Hash: f2829b42a478efb4f2704e8f84258bc40fdd5321b335278459c81555f89d54a6
Byte Size: 461
Line Count: 23
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000303

---

## pnpm-lock.yaml

- Zweck: Pinned Dependency-Graph des Web-UI-Pakets im Lockfile-Format 9.0 mit exakten Auflösungen aller Laufzeit- und Dev-Pakete (belegt durch Dateiinhalt, 1136 Zeilen).
- Verantwortlichkeit: Reproduzierbare Installationen für das Web-UI.
- Eingaben: `package.json` + Registries.
- Ausgaben: Deterministischer Modulbaum.
- Datenfluss: Installations-Graph.
- Persistenz: Versionskontrolle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Pinnt transitive Versionen (z. B. `vue` 3.5.25, `vue-router` 4.6.4, `pinia` 3.0.4, `@ant-design/icons-vue` 7.0.1, `@novnc/novnc` 1.4.0, `ant-design-vue` 4.2.6, `vite` 7.3.0, `@vitejs/plugin-vue` 6.0.3) (belegt durch Dateiinhalt).
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Enthält die tatsächlich installierten Versionen des Lieferumfangs; ist die Grundlage für Supply-Chain-Prüfungen.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pnpm.
- Rust-Relevanz: Lockfile-Disziplin (exakte, nachvollziehbare transitive Versionen) ist als Vorbild für `Cargo.lock`-Verwaltung im Rust-Rewrite relevant; kein Code übernommen.

Read Evidence:
File Hash: b2cfb22da5260c2f6778ad71adfb6978c2d6b68af86c65b7a366d5483e0c2660
Byte Size: 35949
Line Count: 1136
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000304

---

## pnpm-workspace.yaml

- Zweck: Workspace-Marker-Datei (eine Zeile: `packages:`), die dieses Verzeichnis als pnpm-Workspace kennzeichnet (belegt durch Dateiinhalt, 1 Zeile).
- Verantwortlichkeit: Deklariert den Workspace-Kontext; Packages-Liste ist leer.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Aktiviert pnpm-Workspace-Semantik für das Verzeichnis.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pnpm.
- Rust-Relevanz: Keine technische Fähigkeit; rein deklarative Workspace-Kennzeichnung — in Rust-Begriffen: Cargo-Workspace-Entscheidung, kein zu übernehmender Inhalt.

Read Evidence:
File Hash: 11c9ba769c156f38b55880ec6e76fe59d8bfe38ebd728b286161e9e73a3def59
Byte Size: 17
Line Count: 1
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000305

---

## vite.config.js

- Zweck: Vite-Build- und Dev-Server-Konfiguration des Web-UI: Vue-Plugin, Pfad-Alias `@`, flaches Build-Ausgabeschema und API-Proxy für die Entwicklung (belegt durch Dateiinhalt, 35 Zeilen).
- Verantwortlichkeit: Bestimmt das Build-Verhalten (Bundle-Namen) und die Dev-Server-Verdrahtung zum Backend.
- Eingaben: Quellbaum; Build-Zeit: Vite/Rollup.
- Ausgaben: Build-Konfiguration, Ausgabedateien unter `dist/`.
- Datenfluss: Dev-Server-Proxy: `/admin` und `/v1` → `http://127.0.0.1:3000` (changeOrigin) — die gesamte Backend-Kommunikation des Frontends läuft im Dev-Modus durch diesen Proxy.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine eigene; definiert den Proxy-Pfad zu den Admin-/v1-Endpunkten des Backends.
- Ereignisse: Keine.
- Nebenwirkungen: `build.rollupOptions.output.entryFileNames/assets` → `assets/[name].[ext]` erzeugt flache, sprechende Bundle-Namen (belegt durch Dateiinhalt und die 1:1-Namen der Dateien in `dist/assets/`); Alias `@` → `./src`.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Der Proxy reicht `/admin` (administrative Endpunkte) an das Backend weiter — im Dev-Modus ohne zusätzliche Absicherung; Produktionsauslieferung erfolgt vermutlich direkt über das Backend (nicht durch diese Datei belegt).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `@vitejs/plugin-vue`, `vite`; Dev-Server auf `127.0.0.1:5173`.
- Rust-Relevanz: Das Fähigkeitskonzept „flache, deterministische Asset-Namen + Proxy auf Backend-Routen" ist als Auslieferungs-/Routing-Design in der Rust-Neuentwicklung zu bewerten; Konfigurationsinhalt wird nicht übernommen.

Read Evidence:
File Hash: debb0a6a1e783340a825e5b57b1cbd421d98e6f13e5496d354824d5535da9024
Byte Size: 693
Line Count: 35
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000306

---

## src/main.js

- Zweck: Bootstrap der Vue-Anwendung: Router mit 9 lazy-geladenen Routen, Pinia-Store-Installation, globale Registrierung von Ant Design Vue und Mount auf `#app` (belegt durch Dateiinhalt, 30 Zeilen).
- Verantwortlichkeit: Zentraler App-Einstieg; definiert die Navigationsstruktur des gesamten Frontends.
- Eingaben: Komponentenpfade (lazy), Router-Konfiguration.
- Ausgaben: Gemountete Anwendung; Routen: `/` (dash), `/settings/server`, `/settings/workers`, `/settings/browser`, `/settings/adapters`, `/tools/display`, `/tools/cache`, `/tools/logs`, `/tools/request` (belegt durch Dateiinhalt).
- Datenfluss: Router (createWebHistory) → Lazy-Import der Seitenkomponenten → Antd global → `#app`.
- Persistenz: Keine.
- Zustände: Router-History.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Registriert `ant-design-vue` global (alle Komponenten verfügbar); aktiviert Client-seitige History-Routing.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Seitenstruktur: 1 Dashboard, 4 Einstellungsseiten, 4 Werkzeugseiten (belegt durch Dateiinhalt).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `vue`, `pinia`, `vue-router`, `ant-design-vue`, `@/App.vue`, `@/components/*`, `@/stores/settings`.
- Rust-Relevanz: Die Fähigkeitsstruktur „lazy Seiten-Navigation mit 9 Views" (Dashboard, Einstellungen, Tools) ist das Navigationsskelett für eine Rust-Frontend-Neuimplementierung; die konkrete Router-API wird nicht übernommen.

Read Evidence:
File Hash: 58025b3cc9445530b6b6b100caeef5146ebf984e717b070f5cba70d532a6239d
Byte Size: 1261
Line Count: 30
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000307

---

## src/stores/settings.js

- Zweck: Pinia-Store `settings`: zentrale Client-Konfiguration, Admin-Token-Verwaltung und alle CRUD-Aufrufe gegen die `/admin/*`-Konfigurationsendpunkte des Backends (belegt durch Dateiinhalt, 237 Zeilen).
- Verantwortlichkeit: Einziger Eigentümer des Admin-Tokens im Client (inkl. Persistenz in `localStorage`) und der Konfigurationszustände für Server, Browser, Worker/Instanzen, Pool und Adapter.
- Eingaben: Server-Antworten von `/admin/status`, `/admin/config/server`, `/admin/config/browser`, `/admin/config/instances`, `/admin/config/pool`, `/admin/adapters`, `/admin/config/adapters`; User-Aktionen (Save/Update).
- Ausgaben: Fetch-/Save-Ergebnisse (`true`/`false`), Fehlermeldungen über Ant-Design-`message`/`Modal`; Zustandsaktualisierungen.
- Datenfluss: Komponente ruft Action → `fetch` mit Bearer-Header → Backend-Endpunkt → JSON-Fehlermapping über `handleResponse` → Store-State und UI-Rückmeldung.
- Persistenz: Schreibende Persistenz des Tokens in `localStorage` unter `admin_token` (setzen/entfernen) (belegt durch Dateiinhalt, `setToken`).
- Zustände: `token`; `serverConfig` (Objekt); `browserConfig`; `workerConfig` (Array); `poolConfig` (mit Defaults `strategy:'least_busy'`, `waitTimeout:120`, `failover:{enabled:false,maxRetries:3,imgDlRetry:false,imgDlRetryMaxRetries:2}`); `adapterConfig`; `adaptersMeta` (Array).
- APIs: Backend-Endpunkte (je einmal GET/PUT bzw. POST): `/admin/status`, `/admin/config/server`, `/admin/config/browser`, `/admin/config/instances`, `/admin/config/pool`, `/admin/adapters`, `/admin/config/adapters` (belegt durch Dateiinhalt).
- Ereignisse: Keine eigenen.
- Nebenwirkungen: `handleResponse` zeigt `message.success` bei Erfolg und `message.error`/`Modal.error` bei Fehlern; `checkAuth` setzt bei Status 401 den Token nicht selbst zurück (Rückgabe `false`).
- Fehlerfälle: 401 bei `checkAuth` → `false`; Netzwerkfehler → abgefangen, `false` bzw. Modal „保存失败 (网络异常)"; nicht-OK-Status → Fehlermeldung aus `data.error?.message`/`data.message`/Status-Text.
- Sicherheitsrelevanz: Token wird als `Authorization: Bearer <token>` in jedem Request mitgeführt; Token-Persistenz im Browser-`localStorage` (XSS-Angriffsfläche — beobachtet, keine Bewertung); leere Config-Objekte werden über `|| {}` abgesichert.
- Geschäftslogik: Save-Actions merged Konfigurationsobjekte in die bestehenden Store-Objekte (`this.serverConfig = {...this.serverConfig, ...config}`-Muster); `getHeaders` setzt Content-Type und Bearer-Auth.
- Algorithmen: Fehler-Parsing-Hierarchie (`error.message` → `message` → HTTP-Status); Default-Zusammenführung des PoolConfig bei fehlenden Feldern.
- verwendete Datenmodelle: Konfigurationsobjekte `serverConfig`, `browserConfig`, `poolConfig` (mit `failover`-Unterobjekt), `workerConfig` als Array von Instanz-Objekten, `adapterConfig` (Schlüssel = Adapter-ID), `adaptersMeta` (Array von Adapter-Metadaten mit `id`, `displayName`, `configSchema` u. a.).
- Abhängigkeiten: `pinia`, `ant-design-vue` (`message`, `Modal`).
- Rust-Relevanz: Die Fähigkeiten „Token-Session-Verwaltung mit Client-Persistenz", „Konfigurations-CRUD gegen eine Admin-API" und „einheitliches Fehler-Rückmeldungs-Schema" sind als Rust-Client-Schicht (z. B. ein API-Client-Modul mit Fehler-Enum) neu zu entwerfen; das localStorage-Konzept entspricht in Rust-Web-Frontends einer bewussten Session-Speicherstrategie.

Read Evidence:
File Hash: cae35b4975527643f00155a26e260931dfd8100025e911189fcc0d88b1df5145
Byte Size: 8827
Line Count: 237
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000308
- EV-WEB2API-000309
- EV-WEB2API-000310

---

## src/stores/system.js

- Zweck: Pinia-Store `system`: Laufzeit-Status und Statistiken des Backends sowie Dienst-Steuerung (Status abfragen, Statistik, Neustart, Stopp) (belegt durch Dateiinhalt, 120 Zeilen).
- Verantwortlichkeit: Hält den globalen Systemzustand (Version, Uptime, CPU, Speicher, Safe-Mode, Statistik) und kapselt die vier Admin-Aktionen.
- Eingaben: Antworten von `/admin/status`, `/admin/stats`, `/admin/restart`, `/admin/stop`; User-Aktionen.
- Ausgaben: Zustandsaktualisierungen via `$patch`; Rückgabe `true`/`false` und `message`-Feedbacks.
- Datenfluss: Komponente → Action → `fetch` mit Store-übergreifendem Auth (`useSettingsStore().getHeaders()`) → Backend → `$patch` auf den State.
- Persistenz: Keine.
- Zustände: `status`, `version` (Default `1.0.0`), `systemVersion`, `uptime`, `cpuUsage`, `memoryUsage{total,used,free}`, `safeMode{enabled,reason}`, `stats{totalRequests,successRate,activeWorkers,totalWorkers,avgResponseTime,success,failed}`.
- APIs: Backend-Endpunkte: `/admin/status` (GET), `/admin/stats` (GET), `/admin/restart` (POST), `/admin/stop` (POST) (belegt durch Dateiinhalt).
- Ereignisse: Keine.
- Nebenwirkungen: `restartService`/`stopService` lösen Dienstoperationen aus; zeigen `message`-Erfolgs-/Fehlermeldungen.
- Fehlerfälle: Netzwerk-/HTTP-Fehler → `message.error` mit Fehlertext; Rückgabe `false` bei Fehlschlag.
- Sicherheitsrelevanz: Keine eigenen Sicherheitsmechanismen; Auth erfolgt über den Settings-Store (Bearer-Token).
- Geschäftslogik: `restartService(options)` sendet optionale Parameter (für Login-/Neustart-Modi) als JSON-Body.
- Algorithmen: Keine.
- verwendete Datenmodelle: Zustandsobjekte für Status/Memory/Safe-Mode/Stats wie oben.
- Abhängigkeiten: `pinia`, `ant-design-vue` (`message`), `./settings`.
- Rust-Relevanz: Konzept eines „System-Dashboard-Datenmodells" (Status/Statistik/Steuerung) mit CRUD-artiger Steuerung ist als Rust-State-Struktur + Admin-Client neu zu entwerfen; die JS-Store-Form wird nicht übernommen.

Read Evidence:
File Hash: c668c463942ef479d7dadab249874c15dc2004238eb660131748c3cc91049c19
Byte Size: 3811
Line Count: 120
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000311

---

## src/App.vue

- Zweck: Rahmenkomponente der gesamten Anwendung: Auth-Gate, responsive Sidebar/Menü, Header mit Logout und API-Test-Drawer, Backend-Verbindungsüberwachung (belegt durch Dateiinhalt, 684 Zeilen).
- Verantwortlichkeit: Kontrolliert den Anwendungsfluss (Login-Sperre bis Token validiert), die Navigation zwischen allen Routen und den globalen API-Selbsttest.
- Eingaben: Backend-Antworten von `/admin/status`, `/v1/models`, `/v1/cookies`, `/v1/chat/completions`; Benutzerinteraktionen (Menü, Logout, Test).
- Ausgaben: Sichtbarkeit des Login-Modals; Navigation; Testergebnisse; Logout-Flow.
- Datenfluss: Start → Token vorhanden? → `checkAuth` → gültig: App laden, sonst Login; 5-Sekunden-Verbindungs-Check gegen `/admin/status` (AbortSignal.timeout(5000)); bei Verbindungsverlust Warn-Modal, bei Wiederkehr Seiten-Reload.
- Persistenz: Keine direkt; nutzt Settings-Store-Token aus localStorage.
- Zustände: `isInitializing`, `loginVisible`, `selectedKeys`, `collapsed`, `isMobile`, `apiTestDrawer`, `apiTestResults`, `chatStreamContent`, `connectionCheckInterval`, `disconnectModalShown`.
- APIs: Backend: `/admin/status` (Verbindungs-Check), `/v1/models` (GET), `/v1/cookies` (GET), `/v1/chat/completions` (POST, auch streaming) (belegt durch Dateiinhalt).
- Ereignisse: `resize` (Responsive-Sidebar); Browser-Stream-Ereignisse (SSE) beim Chat-Test.
- Nebenwirkungen: Logout setzt Token zurück und öffnet nach 500 ms das Login-Modal; Verbindungs-Wiederherstellung löst `window.location.reload()` aus; API-Test-Drawer streamt Chat-Antworten und akkumuliert `delta.content`.
- Fehlerfälle: Verbindungsabbruch → Warn-Modal (einmalig pro Ausfallphase); ungültiger Token → Login; SSE-`[DONE]` wird ignoriert; JSON-Parsing-Fehler im Stream werden übersprungen.
- Sicherheitsrelevanz: Auth-Gate verhindert die Anzeige der App ohne gültigen Token; `chatStreamContent` wird als HTML gerendert (Rendering-Praxis beobachtet, keine Bewertung); Test-Endpunkte exponieren `/v1/*` inkl. Cookies.
- Geschäftslogik: Menü-Key→Routen-Mapping (`dash→/`, `history→/tools/request`, `settings-server→/settings/server`, usw.); API-Test in drei Modi (models/cookies/chat); Chat-Test unterstützt multimodalen Content (Text + `image_url` als base64) und Streaming.
- Algorithmen: SSE-Parsing: Puffer-Split an Zeilenumbrüchen, `data: `-Präfix, `[DONE]`-Terminator, JSON-Delta-Extraktion; base64→Blob-Konvertierung für Bilddaten; Markdown-Bild-URL-Extraktion per Regex.
- verwendete Datenmodelle: `apiTestResults` (je Modus: status/data/error), `chatModelList`/`chatImageList`, `menuRoutes`-Mapping.
- Abhängigkeiten: `vue`, `vue-router`, `pinia`, `ant-design-vue`, `@ant-design/icons-vue`, `./stores/settings`, `./components/auth/LoginModal.vue`.
- Rust-Relevanz: Die Fähigkeiten „Auth-Gate vor App-Inhalt", „globale Verbindungsüberwachung mit Auto-Recovery", „Client-seitiger API-Selbsttest inkl. SSE-Stream-Anzeige" sind als Rust-Frontend-Architektur neu zu entwerfen (Session-Guard, Health-Polling, Stream-Parser als eigene Module).

Read Evidence:
File Hash: 127778f55ca7f2a334809a837d203177cc25ccf6bcb69bb9c4258d6e4eef8d1b
Byte Size: 24288
Line Count: 684
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000312
- EV-WEB2API-000313

---

## src/components/auth/LoginModal.vue

- Zweck: Modal-Dialog zur Token-Anmeldung: Eingabe, Validierung gegen das Backend und Rollback bei Fehlschlag (belegt durch Dateiinhalt, 82 Zeilen).
- Verantwortlichkeit: Einziger Anmelde-Einstieg; validiert das eingegebene Token über `checkAuth`.
- Eingaben: Prop `visible`; Token-Eingabe; Resultat von `settingsStore.checkAuth()`.
- Ausgaben: Emits `update:visible` (Schließen) und `success` (angemeldet); setzt/entfernt den Token im Store.
- Datenfluss: Eingabe → `setToken` → `checkAuth` (GET `/admin/status`) → bei Erfolg Emits + Schließen; sonst alten Token wiederherstellen.
- Persistenz: Über den Settings-Store (localStorage).
- Zustände: `loading`; lokaler `token`-Ref.
- APIs: Indirekt `/admin/status` über `checkAuth`.
- Ereignisse: Emits `success`/`update:visible`.
- Nebenwirkungen: Ersetzt bei Validierungsfehler den Token wieder durch den vorherigen.
- Fehlerfälle: Leeres Token → Warnung; Validierung fehlgeschlagen → Fehlermeldung + Token-Rollback; Netzwerk-/Ausnahme → generische Fehlermeldung.
- Sicherheitsrelevanz: Kein Passwort, nur Token; keine Rate-Limitierung oder Sperrlogik im Frontend (beobachtet, keine Bewertung); Modal ist nicht schließbar (kein Cancel), solange nicht validiert.
- Geschäftslogik: Validierung = reine Existenz-/Statusprüfung des Backends (HTTP 200 = gültig); es gibt keine lokale Token-Format-Validierung.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `vue`, `ant-design-vue`, `@/stores/settings`.
- Rust-Relevanz: Konzept „Token-Authentifizierungsdialog mit Backend-Validierung und Zustands-Rollback" ist als Rust-Anmeldekomponente mit explizitem Auth-Ergebnis-Typ neu zu entwerfen.

Read Evidence:
File Hash: 48f9193c6f883d7de4d287a18675f7a0fc2103c178c0fa29de4070ec741d9557
Byte Size: 2686
Line Count: 82
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000314

---

## src/components/dash.vue

- Zweck: Dashboard-Übersicht: Systemstatus, Statistiken und Live-Aufgabenwarteschlange mit 5-Sekunden-Polling (belegt durch Dateiinhalt, 273 Zeilen).
- Verantwortlichkeit: Aggregiert `fetchStatus`/`fetchStats` (System-Store) und `/admin/queue` zu einer Übersichtsseite.
- Eingaben: Antworten von `/admin/queue`, System-Store-Daten.
- Ausgaben: Kennzahlen-Karten, Queue-Tabellen (processing/waiting), Lade-/Status-Farben.
- Datenfluss: Polling-Timer (5000 ms) → `refreshData` = `Promise.all`(fetchStatus, fetchStats, fetchQueue) → UI.
- Persistenz: Keine.
- Zustände: `queueData`, `queueStats{processing,waiting,total}`, `timer`.
- APIs: Backend: `/admin/queue` (GET) (belegt durch Dateiinhalt); nutzt Store-Actions für `/admin/status` und `/admin/stats`.
- Ereignisse: Timer-Ereignisse alle 5 s (onMounted gestartet, onUnmounted gestoppt).
- Nebenwirkungen: Polling erzeugt laufende Backend-Last; Status-Zuordnung `processing`/`waiting` je Aufgabeneintrag.
- Fehlerfälle: `fetchQueue`-Fehler → `message.error`; Queue-Listen fehlen → leere Arrays.
- Sicherheitsrelevanz: Keine eigenen; Auth via Store-Header.
- Geschäftslogik: Queue-Aufgaben erhalten abhängig von ihrer Herkunft (`processingTasks`/`waitingTasks`) einen Status; Kennzahlen-Formatierung für Uptime/Speicher/Prozent-Farben.
- Algorithmen: `formatUptime` (Sekunden → d/h/m/s); `formatMemory` (MB → GB mit Nachkommastelle); `getLoadColor` (Schwellwerte für Auslastungsfarbe); `getStatusConfig` (Status→Farbe/Text-Mapping).
- verwendete Datenmodelle: Queue-Datenobjekte `{...task, status:'processing'|'waiting'}`.
- Abhängigkeiten: `vue`, `@/stores/system`, `@/stores/settings`, `ant-design-vue`.
- Rust-Relevanz: Fähigkeit „Live-Dashboard mit periodischem Polling und Statusvisualisierung" ist als Rust-Komponente mit Polling-Task und klar typisierten Queue-Datensätzen neu zu entwerfen.

Read Evidence:
File Hash: 90d788c9e4d968990519a7b32da168876682ddd01525a9eed7231be1a464fcd4
Byte Size: 12101
Line Count: 273
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000315

---

## src/components/settings/adapters.vue

- Zweck: Adapter-Verwaltung: Anzeige der Adapter-Metadaten, Modell-Filter-Konfiguration (Whitelist/Blacklist) und Bearbeitung der Adapter-Konfiguration per Drawer (belegt durch Dateiinhalt, 226 Zeilen).
- Verantwortlichkeit: Übersetzt `adaptersMeta` und `adapterConfig` des Stores in bearbeitbare Modellfilter und Konfigurationsformulare.
- Eingaben: `fetchAdaptersMeta`, `fetchAdapterConfig`; Benutzeraktionen (Modell umschalten, Konfiguration speichern).
- Ausgaben: `saveAdapterConfig`; aktualisierter Store-Zustand; Fehlermeldungen.
- Datenfluss: onMounted → Store-Actions → Tabellen/Drawer; Save → Store-Action → Backend → Refresh.
- Persistenz: Keine direkt (über Store/Backend).
- Zustände: `drawerVisible`, `currentAdapter`, `currentConfig` (reactive), `modelFilter{mode,list}`.
- APIs: Indirekt `/admin/adapters`, `/admin/config/adapters` über Store.
- Ereignisse: Keine.
- Nebenwirkungen: `toggleModel` manipuliert die Filterliste; Moduswechsel ersetzt die Liste.
- Fehlerfälle: Fehlende `modelFilter` in Adapter-Meta → Default `{mode:'blacklist', list:[]}`; Konfiguration ohne `configSchema` → Formular ohne dynamische Felder.
- Sicherheitsrelevanz: Modellfilter wirkt als Client-seitige Sichtbarkeitssteuerung; Autorität liegt beim Backend.
- Geschäftslogik: `isModelEnabled`: Whitelist = nur gelistete aktiv, Blacklist = gelistete deaktiviert; `onModeChange` initialisiert die Liste neu; Save merged Adapter-Konfiguration.
- Algorithmen: Listendifferenz beim Umschalten (Hinzufügen/Entfernen); Mapping von `configSchema`-Feldern auf Formular-Controls.
- verwendete Datenmodelle: Adapter-Meta `{id, displayName, configSchema, modelFilter?}`; Adapter-Konfigurationsobjekte.
- Abhängigkeiten: `vue`, `@/stores/settings`, `ant-design-vue`.
- Rust-Relevanz: Fähigkeit „schema-getriebene Konfigurationsformulare mit Modellfilter-Policy" ist als Rust-Komponente mit typisierten Adapter-Definitionen neu zu entwerfen.

Read Evidence:
File Hash: 19b936203ba48d1785409c84d4257132bc835e0a79a5c5f088a86080708f51a4
Byte Size: 9447
Line Count: 226
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000316

---

## src/components/settings/server.vue

- Zweck: Server-Konfigurationsseite: Port, Auth-Token, Keepalive-Modus, Log-Level, Warteschlangen-Puffer, Bild-Limit und Bild-Markdown-Schalter (belegt durch Dateiinhalt, 178 Zeilen).
- Verantwortlichkeit: Formular für `serverConfig` mit Validierung und Sicherheitsbestätigung beim Leeren des Tokens.
- Eingaben: `fetchServerConfig`; Formulareingaben.
- Ausgaben: `saveServerConfig`.
- Datenfluss: onMounted → Fetch → `Object.assign` auf Formular; Save → Validierung → ggf. Sicherheits-Modal → Speichern.
- Persistenz: Keine direkt (via Store/Backend).
- Zustände: `formData` (reactive).
- APIs: Indirekt `/admin/config/server` über Store.
- Ereignisse: Keine.
- Nebenwirkungen: Leerer Token löst Sicherheits-Warnmodal aus („API und WebUI ohne Authentifizierung … nicht in öffentlichen Netzen") (belegt durch Dateiinhalt).
- Fehlerfälle: Token-Länge 1–9 Zeichen → Abbruch mit Fehlermeldung („mindestens 10 Zeichen oder leer") (belegt durch Dateiinhalt); leeres Token → Bestätigung erforderlich.
- Sicherheitsrelevanz: Client-seitige Mindestlängen-Validierung des Admin-Tokens; explizite Warnung vor ungeschütztem Betrieb.
- Geschäftslogik: Formulardefaults (`port:5173`, `keepaliveMode:'comment'`, `logLevel:'info'`, `queueBuffer:2`, `imageLimit:5`, `imageMarkdown:false`); 4-Spalten-Layout (a-row/a-col).
- Algorithmen: Keine.
- verwendete Datenmodelle: `serverConfig`-Objekt mit obigen Feldern.
- Abhängigkeiten: `vue`, `ant-design-vue` (`Modal`, `message`), `@/stores/settings`.
- Rust-Relevanz: Fähigkeit „Server-Konfigurationsformular mit Validierungsregeln und Sicherheits-Warnflüssen" ist als Rust-Formular-View mit Validierungs-Enum neu zu entwerfen.

Read Evidence:
File Hash: d760746494793ee30ee861733c757f4cfcc8fb0e2e7dd5a8defa1b515c35ddcc
Byte Size: 8044
Line Count: 178
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000317

---

## src/components/settings/browser.vue

- Zweck: Browser-Konfigurationsseite: Browser-Pfad, Headless-Modus, Fission, Cursor-Humanisierung, CSS-Optimierungen und globaler Proxy (belegt durch Dateiinhalt, 289 Zeilen).
- Verantwortlichkeit: Formular für `browserConfig` inkl. verschachtelter `cssInject`- und `proxy`-Objekte.
- Eingaben: `fetchBrowserConfig`; Formulareingaben.
- Ausgaben: `saveBrowserConfig`.
- Datenfluss: onMounted → Fetch → Felder auf Formular mappen; Save → Config-Objekt zusammenbauen → Speichern.
- Persistenz: Keine direkt (via Store/Backend).
- Zustände: `formData` (reactive).
- APIs: Indirekt `/admin/config/browser` über Store.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlende Felder in `browserConfig` → Defaults (`fission !== false` → true; `humanizeCursor ?? false`; Proxy-Defaults `127.0.0.1:7890`).
- Sicherheitsrelevanz: Proxy-Zugangsdaten (`proxyUser`/`proxyPasswd`) werden im Formular gehalten und an das Backend übertragen.
- Geschäftslogik: `humanizeCursor` unterstützt drei Zustände (`false`=aus, `true`=ghost-cursor, `'camou'`=Camoufox-intern); `cssInject`-Flags (animation/filter/font) werden in ein Unterobjekt zusammengefasst; `fission`-Default true.
- Algorithmen: Mapping flach→verschachtelt und zurück.
- verwendete Datenmodelle: `browserConfig` mit `{path, headless, cssInject:{animation,filter,font}, fission, humanizeCursor, proxy:{enable,type,host,port,auth,username,password}}`.
- Abhängigkeiten: `vue`, `@/stores/settings`, `ant-design-vue`.
- Rust-Relevanz: Fähigkeit „verschachteltes Browser-/Proxy-Konfigurationsformular mit Defaults und Proxy-Anmeldedaten" ist als Rust-Formular-View mit verschachtelten typisierten Strukturen neu zu entwerfen.

Read Evidence:
File Hash: 4d54cb3e2c6a4521dae5899f284be8c61160ee8821d528880d1e88290ee32516
Byte Size: 15238
Line Count: 289
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000318

---

## src/components/settings/workers.vue

- Zweck: Worker-/Pool-Konfigurationsseite: Pool-Strategie, Failover, Instanzliste mit CRUD, Batch-Proxy-Bearbeitung und Merge-Adapter-Auswahl (belegt durch Dateiinhalt, 693 Zeilen).
- Verantwortlichkeit: Verwaltet `workerConfig` (Instanzen) und `poolConfig`; ermöglicht Einzel- und Stapeloperationen auf Instanzen.
- Eingaben: `fetchWorkerConfig`, `fetchPoolConfig`, `fetchAdaptersMeta`; Benutzeraktionen.
- Ausgaben: `saveWorkerConfig`, `savePoolConfig`.
- Datenfluss: onMounted → `Promise.all` der drei Store-Actions → Tabellen/Drawers; Speichern → Store-Actions.
- Persistenz: Keine direkt (via Store/Backend).
- Zustände: `poolConfig` (computed get/set), `instanceData`, `selectedRowKeys`, `batchProxyForm`, `editForm`, `workerForm`, Drawer-/Modal-Sichtbarkeiten.
- APIs: Indirekt `/admin/config/instances`, `/admin/config/pool`, `/admin/adapters` über Store.
- Ereignisse: Keine.
- Nebenwirkungen: Batch-Proxy-Anwendung modifiziert alle ausgewählten Instanzen; Batch-Löschen entfernt sie.
- Fehlerfälle: Keine ausgewählten Instanzen → Warnung; fehlende Meta → Adapter-Displayname fällt auf `id` zurück.
- Sicherheitsrelevanz: Batch-Proxy-Formular enthält Proxy-Zugangsdaten für viele Instanzen auf einmal.
- Geschäftslogik: `merge` wird als Aggregat-Modus an erster Stelle der Adapteroptionen angeboten und von `mergeableAdapterOptions` ausgeschlossen (keine Merge-Verschachtelung); neue Instanzen erhalten zufällige Namenssuffixe; Instanzen werden über `id`/`name` identifiziert; Worker-Formular unterstützt mehrere Worker pro Instanz.
- Algorithmen: Filter-/Mapping-Operationen auf Listen; Zufallssuffix (`Math.random().toString(36)`); Key-Bestimmung `id || name`.
- verwendete Datenmodelle: `poolConfig`, Instanz-Objekte (mit `id`, `name`, `type`, `mergeTypes`, `workerCount`, `proxy` u. a.), `workerForm` mit `workers[]`.
- Abhängigkeiten: `vue`, `@/stores/settings`, `ant-design-vue` (`Modal`).
- Rust-Relevanz: Fähigkeit „Tabellen-CRUD mit Stapeloperationen und optionale Aggregat-Modi" ist als Rust-Listen-Editor mit klaren Domain-Typen neu zu entwerfen.

Read Evidence:
File Hash: c4361019b74b7bc40a60b11dbeb461d934cb37195caa478e93db49cf11455f70
Byte Size: 29592
Line Count: 693
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000319

---

## src/components/tools/cache.vue

- Zweck: Cache-/Instanzverwaltung: gestufter Dienst-Neustart mit Fortschrittsanzeige, Dienst-Stopp, Cache-Bereinigung und Verwaltung der Instanz-Datenordner (belegt durch Dateiinhalt, 444 Zeilen).
- Verantwortlichkeit: Orchestriert administrative Operationen am laufenden Backend (Restart mit Wiederherstellungs-Polling, Cache-Löschen, Ordner-Löschen).
- Eingaben: `/admin/config/instances`, `/admin/cache/clear`, `/admin/data-folders`, `/admin/data-folders/delete`; `systemStore.restartService`/`stopService`.
- Ausgaben: Neustart-Fortschritt (4 Schritte), Bestätigungen, Listen der Datenordner.
- Datenfluss: Restart: Bestätigung → `restartService` → 3 s warten → bis zu 20× `fetchStatus`-Polling (2 s Abstand) → Erfolg; Ordner: `/admin/data-folders` → Auswahl → `/admin/data-folders/delete` → Neu-Laden.
- Persistenz: Keine direkt (operationell gegen Backend-Daten).
- Zustände: `currentStep`, `restarting`, `restartSteps[]`, `instanceFolders`, `selectedFolders`, `pendingRestartOptions`.
- APIs: Backend: `/admin/config/instances` (GET), `/admin/cache/clear` (POST), `/admin/data-folders` (GET), `/admin/data-folders/delete` (POST) (belegt durch Dateiinhalt); Store: `restartService`, `stopService`.
- Ereignisse: Keine eigenen.
- Nebenwirkungen: Löscht serverseitig Instanz-Datenordner bzw. Cache; stößt Backend-Neustart an; zeigt `message`-Rückmeldungen.
- Fehlerfälle: Restart-Verbindungsfehler → Schritt 2 als Fehler markiert, Abbruch; Wiederherstellungs-Polling erschöpft → trotzdem „fertig" (Fortführung beobachtet); keine Ordner ausgewählt → Warnung.
- Sicherheitsrelevanz: Ordner-Löschung entfernt möglicherweise gespeicherte Login-Sessions/Profile (Destruktivoperation mit Bestätigungs-Modals).
- Geschäftslogik: Neustart-Flow als Zustandsmaschine mit 4 Schritten (vorbereiten → senden → warten → fertig); Delete überträgt `{folders:[...]}`.
- Algorithmen: Polling-Schleife mit Retry-Zähler und Sleep-Intervallen.
- verwendete Datenmodelle: `restartSteps` (Array mit status), Instanz-/Ordnerlisten, `{folders:[names]}`.
- Abhängigkeiten: `vue`, `ant-design-vue`, `@/stores/system`, `@/stores/settings`.
- Rust-Relevanz: Fähigkeit „operationelle Dienststeuerung mit Fortschritts-Zustandsmaschine und Wiederherstellungs-Polling" ist als Rust-Komponente mit explizitem Zustands-Enum neu zu entwerfen.

Read Evidence:
File Hash: 8308da0ac192dc963b9094a97f6b7239645880f9bdb9635abfa27c07ac28b7ef
Byte Size: 17483
Line Count: 444
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000320

---

## src/components/tools/display.vue

- Zweck: Remote-Display-Ansicht: VNC-Verbindung zur Browser-Instanz über noVNC (RFB-Client) mit Statusabfrage und Vollbildmodus (belegt durch Dateiinhalt, 220 Zeilen).
- Verantwortlichkeit: Baut die VNC-WebSocket-Verbindung zum Backend auf und verwaltet den RFB-Lebenszyklus.
- Eingaben: `/admin/vnc/status`; WebSocket `/admin/vnc?token=...`; noVNC-RFB-Ereignisse.
- Ausgaben: Verbindungszustand (`disconnected|connecting|connected|error`), Fehlermeldungen, Vollbildumschaltung.
- Datenfluss: onMounted → `fetchVncStatus` → bei enabled `connectVnc` → dynamischer Import von `@novnc/novnc/core/rfb.js` → WebSocket-URL `ws(s)://host/admin/vnc?token=<token>` → RFB-Ereignisse → UI.
- Persistenz: Keine.
- Zustände: `vncStatus`, `connectionState`, `errorMessage`, `isFullscreen`, `rfb`.
- APIs: Backend: `/admin/vnc/status` (GET), WebSocket-Endpunkt `/admin/vnc` (Token als Query-Parameter) (belegt durch Dateiinhalt).
- Ereignisse: noVNC-Ereignisse `connect`, `disconnect`, `credentialsrequired`; Vollbild-`fullscreenchange`.
- Nebenwirkungen: Öffnet persistente WebSocket-Verbindung; sendet leere Credentials bei Anforderung; skaliert Viewport.
- Fehlerfälle: `vncStatus.enabled` false → kein Verbindungsversuch; unerwartetes `disconnect` mit `clean===false` → Fehlermeldung; dynamischer Import/Verbindungsfehler → `error`.
- Sicherheitsrelevanz: Token wird direkt in die WebSocket-URL eingebettet (Query-Parameter — beobachtet, keine Bewertung); Protokollwahl `wss` bei https, sonst `ws`; RFB-Option `wsProtocols: ['binary']`.
- Geschäftslogik: `scaleViewport=true`, `clipViewport=false`, `resizeSession=false`; Verbindung nur bei aktiviertem VNC.
- Algorithmen: Keine.
- verwendete Datenmodelle: `vncStatus`-Objekt (mit `enabled`); RFB-Instanz.
- Abhängigkeiten: `vue`, `@novnc/novnc` (dynamisch), `@/stores/settings`.
- Rust-Relevanz: Fähigkeit „Remote-Viewer mit WebSocket-basiertem RFB-Client, Statusabfrage und Vollbild" ist als Rust-Web-Viewer mit eigenem WebSocket-/RFB-Handling neu zu entwerfen (kein noVNC-Code übernehmen).

Read Evidence:
File Hash: 4a11f5170c099c5199df04c1c52329637849a9863c6bad1bbf9ad4ff8522786a
Byte Size: 8123
Line Count: 220
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000321

---

## src/components/tools/logs.vue

- Zweck: Log-Ansicht: Abruf, Parsing, Filterung, Suche, Auto-Refresh, Export und Datumsbereichs-Statistik der System-Logs (belegt durch Dateiinhalt, 536 Zeilen).
- Verantwortlichkeit: Stellt Backend-Logs strukturiert dar und analysiert Erfolgs-/Fehlerstatistiken über Zeitfenster.
- Eingaben: `/admin/logs?lines=500`, `/admin/stats/range?start=...&end=...`; Benutzerfilter.
- Ausgaben: Gefilterte Log-Liste, Level-Verteilung, Bereichsstatistik, Export-Datei.
- Datenfluss: onMounted → `fetchLogs` → `parseLogs` → Filter/Suche → UI; Auto-Refresh alle 5 s; Bereichsstatistik bei Datumsauswahl.
- Persistenz: Keine; Export als Blob-Download (`system-<datum>.log`).
- Zustände: `logs`, `total`, `autoRefresh`, `refreshInterval`, `searchText`, `levelFilter`, `dateRange`, `rangeStats`.
- APIs: Backend: `/admin/logs` (GET `lines=500`, DELETE zur Bereinigung), `/admin/stats/range` (GET mit `start`/`end` als YYYY-MM-DD) (belegt durch Dateiinhalt).
- Ereignisse: Timer (Auto-Refresh).
- Nebenwirkungen: Löscht bei Bestätigung alle Systemlog-Dateien (DELETE `/admin/logs`); löst Dateidownload aus.
- Fehlerfälle: Logs-Abruf fehlgeschlagen → `message.error`; Parsing nicht passender Zeilen → Fallback-Eintrag mit Level INFO; kein Datumsbereich → Statistik zurückgesetzt.
- Sicherheitsrelevanz: Log-Ansicht exponieren potenziell sensitive Systemmeldungen; Bereinigung ist destruktiv (Bestätigungsmodal).
- Geschäftslogik: Log-Format `YYYY-MM-DD HH:MM:SS.mmm [LEVEL] [MODUL] Nachricht`; Level `INFO/WARN/ERRO/DBUG` mit Farb-/Icon-Mapping; neueste Einträge zuerst (Reverse).
- Algorithmen: Regex-Parsing (Zeile → time/level/module/message); Textsuche (case-insensitive); Intervall-Steuerung.
- verwendete Datenmodelle: Log-Einträge `{id,time,level,module,message,raw}`; `rangeStats{success,failed,days}`.
- Abhängigkeiten: `vue`, `ant-design-vue`, `@/stores/settings`.
- Rust-Relevanz: Fähigkeit „strukturierte Log-Anzeige mit Filterung, Suche, Export und Zeitfenster-Statistik" ist als Rust-View mit eigenem Log-Parser und Statistik-Aggregation neu zu entwerfen.

Read Evidence:
File Hash: b7b3fd3a2f89b538f8c07411697ef0e38b7b2424ae2e562e8924eebae3eca396
Byte Size: 14762
Line Count: 536
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000322

---

## src/components/tools/request.vue

- Zweck: Request-Inspektor: historische Anfragen mit Pagination, Filterung, Detailansicht, Medienvorschau, Neuversand und eigener Anfrage-Sendefunktion inkl. Streaming und Reasoning-Modus (belegt durch Dateiinhalt, 1575 Zeilen).
- Verantwortlichkeit: Größtes Werkzeugmodul; vereint Lesen/Suchen/Löschen von Verlaufseinträgen, Medien-Download mit Blob-Caching und das direkte Senden von Chat-Anfragen.
- Eingaben: `/admin/history` (+Query: page/pageSize/status/model/search/startDate/endDate), `/admin/history/stats`, `/admin/history/models`, `/admin/history/{id}`, `/admin/history/media/{filename}`, `/admin/history/{id}/retry-media`, `/v1/models`, `/v1/chat/completions`; Datei-Uploads (Bilder).
- Ausgaben: Tabellen, Statistik-Summary (`total/success/failed/avgDuration`), Detail-Drawer, Medienvorschau, Sendeergebnisse.
- Datenfluss: Paginierte Liste → Thumbnail-Vorladen → Detail → Medien-Blob-URLs; Senden: `fetch('/v1/chat/completions')` fire-and-forget → Auto-Refresh nach 1 s; Retry-Medien per POST.
- Persistenz: Blob-URL-Cache in `mediaCache` (Laufzeit); Keine dauerhafte Client-Persistenz.
- Zustände: `records`, `total`, `page`, `pageSize(50)`, Filter-Refs, `stats`, `currentRecord`, `mediaCache`, `sendModelList`, `sendImageList`, `sendStreamMode`, `sendReasoningMode`.
- APIs: Backend: `/admin/history` (GET/POST DELETE `{ids}`), `/admin/history/stats` (GET), `/admin/history/models` (GET), `/admin/history/{id}` (GET), `/admin/history/media/{filename}` (GET), `/admin/history/{id}/retry-media` (POST), `/v1/models` (GET), `/v1/chat/completions` (POST) (belegt durch Dateiinhalt).
- Ereignisse: `watch` auf Filter-Änderungen; Auto-Refresh-Intervalle.
- Nebenwirkungen: Löscht Verlaufseinträge inkl. zugehöriger Medien (Bestätigung); lädt Medien herunter und cached sie; sendet echte Chat-Anfragen an `/v1`.
- Fehlerfälle: Bild-Upload nur PNG/JPEG/GIF/WebP und max. 10 Bilder; leeres Prompt/kein Modell → Warnung; Lösch-/Netzwerkfehler → Meldungen; fehlgeschlagene Medien → Download-Retry.
- Sicherheitsrelevanz: Medien-Abruf erfolgt authentifiziert über `/admin/history/media`; Blob-URLs bleiben im Speicher; das Senden an `/v1` nutzt denselben Bearer-Token.
- Geschäftslogik: Senden baut multimodalen Content (`text` + `image_url` base64), optional `reasoning:true`; fehlgeschlagene Originale werden beim Neuversand gelöscht; Statistik basiert auf Datumsbereich.
- Algorithmen: Query-Builder (URLSearchParams); Blob-Caching mit Cache-Key = Dateiname; Thumbnail-Vorladen iterativ; Pagination.
- verwendete Datenmodelle: Verlaufseinträge `{id, model_id|model_name, prompt, status, responseMedia:[{localPath,status}]}`; Statistik-Objekt.
- Abhängigkeiten: `vue`, `ant-design-vue`, `@/stores/settings`.
- Rust-Relevanz: Fähigkeiten „paginierter Verlaufs-Browser mit Filterung", „authentifizierter Medien-Stream mit Client-Cache", „multimodales Chat-Senden mit optionalem Reasoning/Streaming" sind als eigenständige Rust-Module neu zu entwerfen.

Read Evidence:
File Hash: 3ccd78e5e24324a9e7c42650d588bb524b959c3f3ad6be64145ed81fa581fa44
Byte Size: 47958
Line Count: 1575
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000323
- EV-WEB2API-000324

---

## public/favicon.png

- Zweck: Browser-Favicon (PNG, 256×256, 8-bit RGBA) — in `index.html` als `<link rel="icon" href="/favicon.png">` referenziert (belegt durch `index.html` und Dateityp).
- Verantwortlichkeit: Visuelles Branding im Browser-Tab.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Statisch vom Web-Server ausgeliefert.
- Persistenz: Binär-Asset in Versionskontrolle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `index.html`.
- Rust-Relevanz: Statisches Asset, wird in Rust-Neuimplementierung als eigenes Icon ersetzt (kein Übernehmen).

Read Evidence:
File Hash: c10b26f8a788c15e70e42fa94c500ae82491e9ff3136a39c165871ddf0286d65
Byte Size: 70830
Line Count: 280
Encoding: Binär (PNG)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000325

---

## dist/index.html

- Zweck: Build-Ausgabe des HTML-Einstiegs: verweist auf `/assets/index.js` (Modul, crossorigin) und `/assets/index.css` sowie auf `/favicon.png` (belegt durch Dateiinhalt, 16 Zeilen).
- Verantwortlichkeit: Produktionsfähiger Einstiegspunkt, der die kompilierten Bundles lädt.
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Einstieg für den ausgelieferten Build.
- Datenfluss: Browser lädt Bundles aus `dist/assets/`.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `/assets/index.js`, `/assets/index.css`, `/favicon.png` (aus `public/` kopiert).
- Rust-Relevanz: Belegt das flache Build-Schema; in Rust-Neuentwicklung wird die Auslieferung eigener Assets geplant.

Read Evidence:
File Hash: d95c44865c54f6e0da53991621e9746763b5f1a7169f209143a69e90f29fd984
Byte Size: 414
Line Count: 16
Encoding: UTF-8
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000326

---

## dist/favicon.png

- Zweck: Kopie des Favicons im Ausgabeverzeichnis (PNG 256×256 RGBA, identischer Hash wie `public/favicon.png`) (belegt durch Dateityp und Hash-Vergleich).
- Verantwortlichkeit: Statisches Asset im Produktionsbuild.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Ausgeliefert als `/favicon.png`.
- Persistenz: Build-Artefakt.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `public/favicon.png`.
- Rust-Relevanz: Statisches Asset (siehe `public/favicon.png`).

Read Evidence:
File Hash: c10b26f8a788c15e70e42fa94c500ae82491e9ff3136a39c165871ddf0286d65
Byte Size: 70830
Line Count: 280
Encoding: Binär (PNG)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000327

---

## dist/assets/index.js

- Zweck: Haupt-Bundle des Builds (minifiziert, 1.560.543 Bytes, 475 Zeilen): enthält die Anwendungslogik, den Router, Stores und Antd-Runtime; deklariert per `__vite__mapDeps` die lazy Chunks (`assets/dash.js`, `system.js`, `DesktopOutlined.js`, `server.js`, `display.js`, `cache.js`, `logs.js`, `request.js`) (belegt durch Dateiinhalt-Kopf).
- Verantwortlichkeit: Lade- und Routing-Zentrale des Produktionsbuilds.
- Eingaben: Route-Auflösung, Store-Nutzung.
- Ausgaben: Lazy-Ladung der Seiten-Chunks.
- Datenfluss: Browser lädt `index.js` → deklarierte Chunk-Map → bedarfsweise Route-Chunks.
- Persistenz: Keine.
- Zustände: Keine eigenen.
- APIs: Enthält die aufgerufenen Backend-Pfade (`/admin/...`, `/v1/...`) als Zeichenketten (belegt durch grep in der Datei).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine (compilierte Client-Datei).
- Geschäftslogik: Entspricht dem kompilierten Gegenstück von `main.js`/`App.vue`/Stores.
- Algorithmen: Keine (minifiziert).
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Alle übrigen Bundles; referenziert `assets/*.css` je Route.
- Rust-Relevanz: Belegt den Lazy-Load-Mechanismus; dient als Verifikation der Quellanalyse, nicht als Vorlage.

Read Evidence:
File Hash: 3ea1861757f8a95fb01f1c77fadf58556e65a1f6427bc9a266c74d1d08413212
Byte Size: 1560543
Line Count: 475
Encoding: UTF-8 (minifiziert, teils ASCII)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000328

---

## dist/assets/system.js

- Zweck: Kompilierter Store-Chunk `system` (minifiziert, 1401 Bytes): enthält den System-Store-Zustand und dessen Aktionen (belegt durch Dateiinhalt-Kopf mit `i("system",{state...})`).
- Verantwortlichkeit: Geteilter Chunk, den `dash.js` und `cache.js` importieren (belegt durch Importzeilen in beiden Chunks).
- Eingaben: Store-Aufrufe aus Route-Chunks.
- Ausgaben: Systemzustand.
- Datenfluss: Route-Chunk → `import{u}from"./system.js"` → Store.
- Persistenz: Keine.
- Zustände: Wie `src/stores/system.js`.
- APIs: Wie Quelle (`/admin/status`, `/admin/stats`, `/admin/restart`, `/admin/stop`).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/stores/system.js`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Wie Quelle.
- Abhängigkeiten: `./index.js`.
- Rust-Relevanz: Verifikation der Store-Analyse; kein Übernehmen.

Read Evidence:
File Hash: 2bfd5d136ab19f9ec67dad455c3d26a389874c861d06c5e86c5b46668c015593
Byte Size: 1401
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000329

---

## dist/assets/dash.js

- Zweck: Kompilierter Chunk der Dashboard-Seite (minifiziert, 14023 Bytes): importiert den System-Store (`./system.js`) und das Icon `DesktopOutlined` (belegt durch Dateiinhalt-Kopf).
- Verantwortlichkeit: Dashboard-Route.
- Eingaben: `/admin/queue`.
- Ausgaben: Dashboard-UI.
- Datenfluss: Wie `src/components/dash.vue`.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `/admin/queue`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/dash.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`, `./system.js`, `./DesktopOutlined.js`.
- Rust-Relevanz: Verifikation der Dashboard-Analyse; kein Übernehmen.

Read Evidence:
File Hash: 3daa257d8b754e1df2202e546479c166643a501c0abcb0f2b650a0a9054b796d
Byte Size: 14023
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000330

---

## dist/assets/cache.js

- Zweck: Kompilierter Chunk der Cache-Seite (minifiziert, 12202 Bytes): importiert den System-Store (`./system.js`) (belegt durch Dateiinhalt-Kopf).
- Verantwortlichkeit: Cache-/Instanzverwaltungs-Route.
- Eingaben: `/admin/config/instances`, `/admin/cache/clear`, `/admin/data-folders`.
- Ausgaben: Cache-UI.
- Datenfluss: Wie `src/components/tools/cache.vue`.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Wie Quelle.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/tools/cache.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`, `./system.js`.
- Rust-Relevanz: Verifikation der Cache-Analyse; kein Übernehmen.

Read Evidence:
File Hash: fdac6c819feda33afe1db0203a328d9eed97deb8787b361602dc4490f7d52091
Byte Size: 12202
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000331

---

## dist/assets/DesktopOutlined.js

- Zweck: Kompilierter Icon-Chunk (minifiziert, 1013 Bytes): enthält die SVG-Definition des Antd-Icons `DesktopOutlined`, von `dash.js` und `display.js` genutzt (belegt durch Dateiinhalt-Kopf mit `icon:{tag:"svg",...}`).
- Verantwortlichkeit: Geteilter Icon-Ressourcen-Chunk.
- Eingaben: Keine.
- Ausgaben: SVG-Symbol.
- Datenfluss: Route-Chunks importieren den Icon-Chunk.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`.
- Rust-Relevanz: Keine (UI-Icon).

Read Evidence:
File Hash: 21512199f14481640bce55b2b4dcd3b04b69a32242d54bb4b50d18deb539a2cd
Byte Size: 1013
Line Count: 1
Encoding: ASCII
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000332

---

## dist/assets/display.js

- Zweck: Kompilierter Chunk der Display-Seite (minifiziert, 8714 Bytes): importiert das Icon `DesktopOutlined`; enthält die VNC-Status- und Verbindungslogik (belegt durch Dateiinhalt-Kopf und grep nach „VNC").
- Verantwortlichkeit: VNC-Viewer-Route.
- Eingaben: `/admin/vnc/status`.
- Ausgaben: VNC-UI.
- Datenfluss: Wie `src/components/tools/display.vue`.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `/admin/vnc/status`, WS `/admin/vnc`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/tools/display.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`, `./DesktopOutlined.js`.
- Rust-Relevanz: Verifikation der VNC-Analyse; kein Übernehmen.

Read Evidence:
File Hash: 9d195f4998d505ce1edf80e16f34bc3aa28869b51dbe7328f52b70fd42c106f4
Byte Size: 8714
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000333

---

## dist/assets/rfb.js

- Zweck: Kompilierter noVNC-RFB-Chunk (minifiziert, 168531 Bytes, 5 Zeilen): implementiert den Remote-Frame-Buffer-Client (WebSocket-Protokoll, Log-Subsystem) — Quelle ist `@novnc/novnc/core/rfb.js` (belegt durch Dateiinhalt-Kopf und package.json-Dependency).
- Verantwortlichkeit: Netzwerk- und Protokollschicht der Remote-Anzeige.
- Eingaben: WebSocket-Verbindung.
- Ausgaben: Bild-/Eingabeströme.
- Datenfluss: Browser ↔ Backend-VNC-WebSocket.
- Persistenz: Keine.
- Zustände: Verbindungslogik.
- APIs: Keine eigenen; RFB-Client.
- Ereignisse: RFB-Ereignisse.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine (Drittbibliothek).
- Geschäftslogik: Drittbibliotheks-Code, nicht Teil der Projektlogik.
- Algorithmen: RFB-Protokoll (Drittanbieter).
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: keins (self-contained).
- Rust-Relevanz: Für das Rust-Rewrite muss ein eigener (oder Crate-basierter) RFB/WebSocket-Client bereitgestellt werden; der noVNC-Code wird nicht übernommen.

Read Evidence:
File Hash: bcd9bc2536bd523a73e399cddc14c4a5bc1da74fd0e4e15e11fa7f7a150ea516
Byte Size: 168531
Line Count: 5
Encoding: ASCII (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000334

---

## dist/assets/adapters.js

- Zweck: Kompilierter Chunk der Adapter-Verwaltung (minifiziert, 7021 Bytes) — Gegenstück von `src/components/settings/adapters.vue` (belegt durch Dateiinhalt-Kopf und grep nach „configSchema").
- Verantwortlichkeit: Adapter-Settings-Route.
- Eingaben: `/admin/adapters`, `/admin/config/adapters`.
- Ausgaben: Adapter-UI.
- Datenfluss: Wie Quelle.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Wie Quelle.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/settings/adapters.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`.
- Rust-Relevanz: Verifikation der Adapter-Analyse; kein Übernehmen.

Read Evidence:
File Hash: 5b0ef77a103975d6c85a17d70f62031071fd474b7f920c520ea3879f342a516c
Byte Size: 7021
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000335

---

## dist/assets/browser.js

- Zweck: Kompilierter Chunk der Browser-Settings (minifiziert, 10424 Bytes) — Gegenstück von `src/components/settings/browser.vue` (belegt durch Dateiinhalt-Kopf und grep nach „proxyHost").
- Verantwortlichkeit: Browser-Settings-Route.
- Eingaben: `/admin/config/browser`.
- Ausgaben: Browser-UI.
- Datenfluss: Wie Quelle.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Wie Quelle.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/settings/browser.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`.
- Rust-Relevanz: Verifikation der Browser-Settings-Analyse; kein Übernehmen.

Read Evidence:
File Hash: ddebb44f831f50f83726c89fadc0e9f269189fc35126998a1129dad6791e42dd
Byte Size: 10424
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000336

---

## dist/assets/server.js

- Zweck: Kompilierter Chunk der Server-Settings (minifiziert, 6011 Bytes) — Gegenstück von `src/components/settings/server.vue` (belegt durch Dateiinhalt-Kopf und grep nach „authToken").
- Verantwortlichkeit: Server-Settings-Route.
- Eingaben: `/admin/config/server`.
- Ausgaben: Server-UI.
- Datenfluss: Wie Quelle.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Wie Quelle.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/settings/server.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`.
- Rust-Relevanz: Verifikation der Server-Settings-Analyse; kein Übernehmen.

Read Evidence:
File Hash: ac9deda96540bee3a65e36c2413189ec2053609cfa05820fc5e775b0fe69b45d
Byte Size: 6011
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000337

---

## dist/assets/workers.js

- Zweck: Kompilierter Chunk der Worker-Settings (minifiziert, 19233 Bytes) — Gegenstück von `src/components/settings/workers.vue` (belegt durch Dateiinhalt-Kopf und grep nach „workerCount").
- Verantwortlichkeit: Worker-Settings-Route.
- Eingaben: `/admin/config/instances`, `/admin/config/pool`.
- Ausgaben: Worker-UI.
- Datenfluss: Wie Quelle.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Wie Quelle.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/settings/workers.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`.
- Rust-Relevanz: Verifikation der Worker-Settings-Analyse; kein Übernehmen.

Read Evidence:
File Hash: 79b05e3060b66c23037f293af3b09c57dc8f8c5e9f934f26aef9be771d9d33c1
Byte Size: 19233
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000338

---

## dist/assets/logs.js

- Zweck: Kompilierter Chunk der Log-Seite (minifiziert, 9549 Bytes, 2 Zeilen) — Gegenstück von `src/components/tools/logs.vue` (belegt durch Dateiinhalt-Kopf und grep nach „stats/range").
- Verantwortlichkeit: Log-Route.
- Eingaben: `/admin/logs`, `/admin/stats/range`.
- Ausgaben: Log-UI.
- Datenfluss: Wie Quelle.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Wie Quelle.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/tools/logs.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`.
- Rust-Relevanz: Verifikation der Log-Analyse; kein Übernehmen.

Read Evidence:
File Hash: dcaca48f76b842332a99c69494b2e4aa4a08daef1619471b1c5734a570d252f0
Byte Size: 9549
Line Count: 2
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000339

---

## dist/assets/request.js

- Zweck: Kompilierter Chunk der Request-Seite (minifiziert, 24182 Bytes) — Gegenstück von `src/components/tools/request.vue` (belegt durch Dateiinhalt-Kopf und grep nach „history").
- Verantwortlichkeit: Request-Inspektor-Route.
- Eingaben: `/admin/history*`, `/v1/chat/completions`.
- Ausgaben: Request-UI.
- Datenfluss: Wie Quelle.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Wie Quelle.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Kompiliertes Gegenstück von `src/components/tools/request.vue`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `./index.js`.
- Rust-Relevanz: Verifikation der Request-Analyse; kein Übernehmen.

Read Evidence:
File Hash: f31d0a0beb4dc0454365dc9e49c0c2dd671750776ceb19e02c943eb2f1ef89e0
Byte Size: 24182
Line Count: 1
Encoding: UTF-8 (minifiziert)
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000340

---

## dist/assets/index.css

- Zweck: Zentrales kompiliertes CSS-Bundle (3255 Bytes): Basis-Styles inkl. scrollbar- und Layout-Regeln (belegt durch Dateiinhalt-Kopf; `data-v-*`-Scoped-Attribute).
- Verantwortlichkeit: Globale Styles der App.
- Eingaben: Keine.
- Ausgaben: Styling.
- Datenfluss: Von `dist/index.html` via `/assets/index.css` geladen.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (kompiliertes Styling).

Read Evidence:
File Hash: 4979340dfd5ae34c06eae3e5417f7d9810413121de11da6ff8609496b1b3f77b
Byte Size: 3255
Line Count: 1
Encoding: ASCII
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000341

---

## dist/assets/logs.css

- Zweck: Kompiliertes CSS-Styling der Log-Seite (2732 Bytes) (belegt durch Dateiinhalt; lazy geladen mit `logs.js`).
- Verantwortlichkeit: Log-Ansichts-Styling.
- Eingaben: Keine.
- Ausgaben: Styling.
- Datenfluss: Von `logs.js` als lazy CSS geladen.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (kompiliertes Styling).

Read Evidence:
File Hash: 9c09988725dbb233b98d0f6a09d5b71e8412c5fe8b230832e7193fa281c2cf2a
Byte Size: 2732
Line Count: 1
Encoding: ASCII
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000342

---

## dist/assets/request.css

- Zweck: Kompiliertes CSS-Styling der Request-Seite (5930 Bytes) (belegt durch Dateiinhalt; lazy geladen mit `request.js`).
- Verantwortlichkeit: Request-Ansichts-Styling.
- Eingaben: Keine.
- Ausgaben: Styling.
- Datenfluss: Von `request.js` als lazy CSS geladen.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (kompiliertes Styling).

Read Evidence:
File Hash: b97edcf4a27d2c9d83ab250ba93bd3f85ca1760d640ee8730a4be1dde79f22c0
Byte Size: 5930
Line Count: 1
Encoding: ASCII
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000343

---

## dist/assets/server.css

- Zweck: Kompiliertes CSS-Styling der Server-Settings (47 Bytes; Regel `.ant-input-number[data-v-...]{width:100%}`) (belegt durch Dateiinhalt; lazy geladen mit `server.js`).
- Verantwortlichkeit: Server-Settings-Styling (minimal).
- Eingaben: Keine.
- Ausgaben: Styling.
- Datenfluss: Von `server.js` als lazy CSS geladen.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (kompiliertes Styling).

Read Evidence:
File Hash: 158cd99ff4f139c93d8d585daabe503c59a5d65b56cfbe4971cd6e2b38f73fb3
Byte Size: 47
Line Count: 1
Encoding: ASCII
Read Timestamp: 2026-08-05
Reader Result: OK

Evidence:
- EV-WEB2API-000344

---

# Evidence-Blöcke (EV-WEB2API-000301 … 000344)

Evidence-ID: EV-WEB2API-000301
Repository: WebAI2API
Commit: content-copy
Datei: webui/.npmrc
Zeilenbereich: 1-3
Beziehung: Selbstständige pnpm-Paketkonfiguration im Workspace
Typ: Config
Aussage: Die Datei deaktiviert `shared-workspace-lockfile` und `use-workspace-root-version`, wodurch das Web-UI ein eigenes Lockfile führt und nicht von der Wurzelversion des Monorepos abhängt.

Evidence-ID: EV-WEB2API-000302
Repository: WebAI2API
Commit: content-copy
Datei: webui/index.html
Zeilenbereich: 1-15
Beziehung: Einstiegspunkt für `src/main.js` und `public/favicon.png`
Typ: Config
Aussage: Statischer HTML-Entry mit Mount-Punkt `#app`, Favicon-Verknüpfung und Modul-Referenz auf `src/main.js`; alle weiteren Seiten werden clientseitig über den Router geladen.

Evidence-ID: EV-WEB2API-000303
Repository: WebAI2API
Commit: content-copy
Datei: webui/package.json
Zeilenbereich: 1-23
Beziehung: Grundlage von `pnpm-lock.yaml` und der Build-Skripte
Typ: Config
Aussage: Das Manifest deklariert Vue 3 + vue-router 4 + pinia 3 + ant-design-vue 4 + @ant-design/icons-vue 7 + @novnc/novnc 1.4 als Laufzeit-Stack und vite 7 + @vitejs/plugin-vue 6 als Dev-Stack; Skripte dev/build/preview.

Evidence-ID: EV-WEB2API-000304
Repository: WebAI2API
Commit: content-copy
Datei: webui/pnpm-lock.yaml
Zeilenbereich: 1-1136
Beziehung: Deterministische Auflösung von package.json
Typ: Config
Aussage: Das Lockfile (Version 9.0) pinnt exakte Versionen inkl. transitiver Pakete (u. a. vue 3.5.25, vue-router 4.6.4, pinia 3.0.4, @ant-design/icons-vue 7.0.1, @novnc/novnc 1.4.0, ant-design-vue 4.2.6, vite 7.3.0, @vitejs/plugin-vue 6.0.3).

Evidence-ID: EV-WEB2API-000305
Repository: WebAI2API
Commit: content-copy
Datei: webui/pnpm-workspace.yaml
Zeilenbereich: 1
Beziehung: Workspace-Kennzeichnung des Verzeichnisses
Typ: Config
Aussage: Die Datei markiert das Verzeichnis als pnpm-Workspace mit leerer Packages-Liste; keine Laufzeitfunktion.

Evidence-ID: EV-WEB2API-000306
Repository: WebAI2API
Commit: content-copy
Datei: webui/vite.config.js
Zeilenbereich: 1-35
Beziehung: Steuert `dist/assets/*`-Namen und Dev-Proxy zu Backend
Typ: Config
Aussage: Flaches Ausgabeschema `assets/[name].[ext]` erzeugt die sprechenden Bundle-Namen (1:1 zu Routen/Stores); der Dev-Server (127.0.0.1:5173) proxyt `/admin` und `/v1` an `http://127.0.0.1:3000` mit changeOrigin; Alias `@` → `./src`.

Evidence-ID: EV-WEB2API-000307
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/main.js
Zeilenbereich: 8-30
Beziehung: Registriert Router, Pinia, Antd; mountet App
Typ: Import
Aussage: Neun Routen (Dashboard, 4 Settings, 4 Tools) sind lazy-importiert; createWebHistory-Routing, globale Antd-Registrierung und Mount auf `#app`.

Evidence-ID: EV-WEB2API-000308
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/stores/settings.js
Zeilenbereich: 5-22, 25-40
Beziehung: State-Definition und Token-/Header-Logik
Typ: Schema
Aussage: Der Store hält `token` (aus localStorage `admin_token`), Server-/Browser-/Adapter-Konfiguration, Worker-Instanzliste und Pool-Konfiguration (Defaults: least_busy, waitTimeout 120, Failover maxRetries 3); `getHeaders` setzt `Authorization: Bearer <token>`.

Evidence-ID: EV-WEB2API-000309
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/stores/settings.js
Zeilenbereich: 42-78
Beziehung: checkAuth/handleResponse als zentrale Fehler- und Auth-Schicht
Typ: API
Aussage: `checkAuth` prüft den Token per GET `/admin/status` (401 ⇒ false); `handleResponse` mappt Fehler hierarchisch (`data.error.message` → `data.message` → HTTP-Status) und zeigt Antd-Feedbacks.

Evidence-ID: EV-WEB2API-000310
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/stores/settings.js
Zeilenbereich: 79-237
Beziehung: Config-Actions gegen `/admin/config/*`
Typ: API
Aussage: Der Store lädt und speichert Server-, Browser-, Instanz-, Pool- und Adapter-Konfiguration über GET/PUT auf `/admin/config/server`, `/admin/config/browser`, `/admin/config/instances`, `/admin/config/pool`, `/admin/config/adapters` sowie `/admin/adapters`; Save-Actions mergen Konfigurationsobjekte in den Store-State.

Evidence-ID: EV-WEB2API-000311
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/stores/system.js
Zeilenbereich: 1-120
Beziehung: Status-/Statistik-/Steuerungs-Actions; genutzt von dash.vue und cache.vue
Typ: API
Aussage: Der Store ruft `/admin/status`, `/admin/stats`, `/admin/restart` und `/admin/stop`; hält Version, Uptime, CPU, Speicher, Safe-Mode und Statistiken (totalRequests, successRate, activeWorkers, avgResponseTime, success, failed).

Evidence-ID: EV-WEB2API-000312
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/App.vue
Zeilenbereich: 318-391
Beziehung: Auth-Gate und Verbindungsüberwachung beim App-Start
Typ: Call
Aussage: Ohne Token bzw. bei fehlgeschlagenem `checkAuth` wird das Login-Modal geöffnet; alle 5 s wird `/admin/status` (AbortSignal.timeout(5000)) gepollt; bei Verbindungsabbruch erscheint ein Warn-Modal, bei Wiederkehr ein Seiten-Reload; Logout setzt den Token zurück und öffnet nach 500 ms das Login.

Evidence-ID: EV-WEB2API-000313
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/App.vue
Zeilenbereich: 57-283
Beziehung: API-Test-Drawer im Header
Typ: API
Aussage: Der Drawer testet GET `/v1/models`, GET `/v1/cookies` und POST `/v1/chat/completions` inkl. SSE-Stream-Parsing (`data:`-Zeilen, `[DONE]`, Delta-Akkumulation) und multimodalen Inhalten (Text + base64-`image_url`).

Evidence-ID: EV-WEB2API-000314
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/auth/LoginModal.vue
Zeilenbereich: 20-47
Beziehung: Token-Validierung über den Settings-Store
Typ: Call
Aussage: Anmeldedialog setzt den Token, validiert ihn via `checkAuth` (GET `/admin/status`), emittet bei Erfolg `success` und stellt bei Fehlschlag den vorherigen Token wieder her.

Evidence-ID: EV-WEB2API-000315
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/dash.vue
Zeilenbereich: 23-93
Beziehung: Queue-Polling und Aggregation von System-Store-Daten
Typ: API
Aussage: Das Dashboard ruft `/admin/queue` ab, trennt processing/waiting-Tasks und aktualisiert alle 5 s Status/Statistik/Queue; Formatierung von Uptime und Speicher sowie Statusfarben.

Evidence-ID: EV-WEB2API-000316
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/settings/adapters.vue
Zeilenbereich: 20-116
Beziehung: Modellfilter und Adapter-Konfigurationseditor
Typ: Call
Aussage: Whitelist/Blacklist-Filterung je Adapter, Modell-Umschaltung, Konfigurations-Drawer und Save über den Settings-Store; Fehlende `modelFilter`-Definitionen werden mit `{mode:'blacklist', list:[]}` defaultisiert.

Evidence-ID: EV-WEB2API-000317
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/settings/server.vue
Zeilenbereich: 9-52
Beziehung: Server-Config-Formular mit Sicherheitsvalidierung
Typ: Config
Aussage: Formularfelder port/authToken/keepaliveMode/logLevel/queueBuffer/imageLimit/imageMarkdown; Token von 1–9 Zeichen wird abgelehnt; leeres Token erfordert Bestätigung eines Sicherheitswarn-Modals (unbeschützter Betrieb).

Evidence-ID: EV-WEB2API-000318
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/settings/browser.vue
Zeilenbereich: 8-77
Beziehung: Browser-Config-Formular mit verschachtelten Objekten
Typ: Config
Aussage: Felder path/headless/fission/humanizeCursor (drei Zustände) sowie cssInject{animation,filter,font} und proxy{enable,type,host,port,auth,username,password}; fission-Default true, Proxy-Default 127.0.0.1:7890.

Evidence-ID: EV-WEB2API-000319
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/settings/workers.vue
Zeilenbereich: 8-54, 85-268
Beziehung: Pool-Config und Instanz-CRUD mit Stapeloperationen
Typ: Call
Aussage: Pool-Config (get/set auf Store) wird über `savePoolConfig` gespeichert; Instanzen werden über `saveWorkerConfig` erstellt/editiert/gelöscht; Batch-Proxy und Batch-Löschen wirken auf ausgewählte Zeilen; `merge` wird als Aggregat-Modus vorangestellt und von Merge-Kandidaten ausgeschlossen.

Evidence-ID: EV-WEB2API-000320
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/tools/cache.vue
Zeilenbereich: 67-245
Beziehung: Restart-/Cache-/Ordner-Operationen
Typ: API
Aussage: Neustart-Flow mit 4 Schritten (vorbereiten → senden via `restartService` → 3 s warten → bis 20× Status-Polling), Stopp via `stopService`, Cache-Bereinigung über POST `/admin/cache/clear` und Ordner-Verwaltung über GET `/admin/data-folders` / POST `/admin/data-folders/delete`.

Evidence-ID: EV-WEB2API-000321
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/tools/display.vue
Zeilenbereich: 29-127
Beziehung: VNC-Statusabfrage und noVNC-Verbindungsaufbau
Typ: Import
Aussage: `/admin/vnc/status` steuert die Sichtbarkeit; `@novnc/novnc/core/rfb.js` wird dynamisch importiert und via `ws(s)://host/admin/vnc?token=<token>` mit `wsProtocols:['binary']` verbunden; Zustände disconnected/connecting/connected/error; Vollbildumschaltung.

Evidence-ID: EV-WEB2API-000322
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/tools/logs.vue
Zeilenbereich: 41-211
Beziehung: Log-Abruf, -Parsing, -Filter und Bereichsstatistik
Typ: API
Aussage: GET `/admin/logs?lines=500` mit Regex-Parsing (`Zeit [LEVEL] [Modul] Nachricht`), Level-/Suchfilter, Auto-Refresh alle 5 s, Export als Blob-Download, DELETE `/admin/logs` zur Bereinigung und `/admin/stats/range` für Erfolgs-/Fehlerstatistik über Datumsfenster.

Evidence-ID: EV-WEB2API-000323
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/tools/request.vue
Zeilenbereich: 90-340
Beziehung: Verlaufs-Browser mit Pagination, Detail und Medien
Typ: API
Aussage: GET `/admin/history` (Query: page/pageSize/status/model/search/startDate/endDate), `/admin/history/stats`, `/admin/history/models`, `/admin/history/{id}`, authentifizierter Medienabruf `/admin/history/media/{filename}` mit Blob-URL-Cache sowie DELETE `/admin/history` mit `{ids}`; Retry fehlgeschlagener Medien via POST `/admin/history/{id}/retry-media`.

Evidence-ID: EV-WEB2API-000324
Repository: WebAI2API
Commit: content-copy
Datei: webui/src/components/tools/request.vue
Zeilenbereich: 561-720
Beziehung: Chat-Senden an /v1
Typ: API
Aussage: Sendet POST `/v1/chat/completions` fire-and-forget mit model/messages/stream und optional `reasoning:true`; multimodaler Content aus Text + bis zu 10 Bildern (PNG/JPEG/GIF/WebP als base64); nach dem Senden Auto-Refresh der Liste; fehlgeschlagene Originale werden beim Neuversand still gelöscht.

Evidence-ID: EV-WEB2API-000325
Repository: WebAI2API
Commit: content-copy
Datei: webui/public/favicon.png
Zeilenbereich: 1-280
Beziehung: Referenziert von `index.html`
Typ: Persistenz
Aussage: Binäres PNG-Asset (256×256, RGBA) als Browser-Favicon; wird in den Build nach `dist/favicon.png` kopiert (identischer Hash).

Evidence-ID: EV-WEB2API-000326
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/index.html
Zeilenbereich: 1-16
Beziehung: Produktions-Entry des Builds
Typ: Config
Aussage: Lädt `/assets/index.js` als Modul (crossorigin) und `/assets/index.css`; Favicon `/favicon.png`; belegt das flache Build-Schema aus `vite.config.js`.

Evidence-ID: EV-WEB2API-000327
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/favicon.png
Zeilenbereich: 1-280
Beziehung: Kopie von `public/favicon.png`
Typ: Persistenz
Aussage: Identischer Hash wie das Quell-Favicon; statisches Ausgabe-Asset.

Evidence-ID: EV-WEB2API-000328
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/index.js
Zeilenbereich: 1-475
Beziehung: Haupt-Bundle mit Lazy-Chunk-Deklaration
Typ: Import
Aussage: Enthält die App-Logik und deklariert per `__vite__mapDeps` die Lazy-Chunks (dash, system, DesktopOutlined, server, display, cache, logs, request) — bestätigt die Lazy-Route-Struktur aus `main.js`.

Evidence-ID: EV-WEB2API-000329
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/system.js
Zeilenbereich: 1
Beziehung: Geteilter Store-Chunk, importiert von dash.js/cache.js
Typ: Import
Aussage: Kompiliertes Gegenstück des System-Stores; von `dash.js` und `cache.js` per `import{u}from"./system.js"` referenziert.

Evidence-ID: EV-WEB2API-000330
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/dash.js
Zeilenbereich: 1
Beziehung: Kompilierte Dashboard-Route
Typ: Call
Aussage: Importiert `./system.js` und `./DesktopOutlined.js`; enthält die `/admin/queue`-Logik — Verifikation von `src/components/dash.vue`.

Evidence-ID: EV-WEB2API-000331
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/cache.js
Zeilenbereich: 1
Beziehung: Kompilierte Cache-Route
Typ: Call
Aussage: Importiert `./system.js`; enthält die Cache-/Restart-/Ordner-Endpunktlogik — Verifikation von `src/components/tools/cache.vue`.

Evidence-ID: EV-WEB2API-000332
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/DesktopOutlined.js
Zeilenbereich: 1
Beziehung: Geteilter Antd-Icon-Chunk
Typ: Import
Aussage: SVG-Definition des Desktop-Icons; von dash.js und display.js importiert.

Evidence-ID: EV-WEB2API-000333
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/display.js
Zeilenbereich: 1
Beziehung: Kompilierte VNC-Display-Route
Typ: Call
Aussage: Enthält die VNC-Status-/Verbindungslogik (grep „虚拟显示器"/„VNC"); importiert `DesktopOutlined` — Verifikation von `src/components/tools/display.vue`.

Evidence-ID: EV-WEB2API-000334
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/rfb.js
Zeilenbereich: 1-5
Beziehung: noVNC-RFB-Client als Drittbibliotheks-Chunk
Typ: Import
Aussage: Kompilierte noVNC-RFB-Implementierung (WebSocket/RFB-Protokoll), bereitgestellt aus `@novnc/novnc/core/rfb.js`; Basis der Remote-Anzeige.

Evidence-ID: EV-WEB2API-000335
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/adapters.js
Zeilenbereich: 1
Beziehung: Kompilierte Adapter-Settings-Route
Typ: Call
Aussage: Enthält die Modellfilter-/Konfigurationslogik (grep „configSchema") — Verifikation von `src/components/settings/adapters.vue`.

Evidence-ID: EV-WEB2API-000336
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/browser.js
Zeilenbereich: 1
Beziehung: Kompilierte Browser-Settings-Route
Typ: Call
Aussage: Enthält die Browser-/Proxy-Konfigurationslogik (grep „proxyHost") — Verifikation von `src/components/settings/browser.vue`.

Evidence-ID: EV-WEB2API-000337
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/server.js
Zeilenbereich: 1
Beziehung: Kompilierte Server-Settings-Route
Typ: Call
Aussage: Enthält die Server-Config-Logik (grep „authToken") — Verifikation von `src/components/settings/server.vue`.

Evidence-ID: EV-WEB2API-000338
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/workers.js
Zeilenbereich: 1
Beziehung: Kompilierte Worker-Settings-Route
Typ: Call
Aussage: Enthält die Pool-/Instanzlogik (grep „workerCount") — Verifikation von `src/components/settings/workers.vue`.

Evidence-ID: EV-WEB2API-000339
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/logs.js
Zeilenbereich: 1-2
Beziehung: Kompilierte Log-Route
Typ: Call
Aussage: Enthält die Log-/Statistiklogik (grep „stats/range") — Verifikation von `src/components/tools/logs.vue`.

Evidence-ID: EV-WEB2API-000340
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/request.js
Zeilenbereich: 1
Beziehung: Kompilierte Request-Route
Typ: Call
Aussage: Enthält die Verlaufs-/Medien-/Sende-Logik (grep „history") — Verifikation von `src/components/tools/request.vue`.

Evidence-ID: EV-WEB2API-000341
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/index.css
Zeilenbereich: 1
Beziehung: Globales CSS-Bundle
Typ: Config
Aussage: Von `dist/index.html` via `/assets/index.css` geladen; enthält scoped `data-v-*`-Regeln und Scrollbar-/Layout-Styles.

Evidence-ID: EV-WEB2API-000342
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/logs.css
Zeilenbereich: 1
Beziehung: Lazy-CSS der Log-Route
Typ: Config
Aussage: Styling der Log-Ansicht, von `logs.js` geladen.

Evidence-ID: EV-WEB2API-000343
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/request.css
Zeilenbereich: 1
Beziehung: Lazy-CSS der Request-Route
Typ: Config
Aussage: Styling der Request-Ansicht, von `request.js` geladen.

Evidence-ID: EV-WEB2API-000344
Repository: WebAI2API
Commit: content-copy
Datei: webui/dist/assets/server.css
Zeilenbereich: 1
Beziehung: Lazy-CSS der Server-Settings-Route
Typ: Config
Aussage: Minimale Regel (Input-Breite), von `server.js` geladen.

---

# Zusammenfassung

- 40 Dateien gelesen, alle Status `FERTIG_ANALYSIERT`.
- 44 Evidence-Blöcke erstellt (EV-WEB2API-000301 … 000344).
- Verbotene Füllwörter kommen im Dokument nicht vor; alle Felder sind belegt.
- Negative Befunde wurden dokumentiert (keine Persistenz in System-Store, dash, adapters, browser, workers, logs, request; keine eigene Laufzeit-API in Konfig- und Asset-Dateien).
- Build-Artefakte wurden als solche klassifiziert und als Verifikation der Quellanalyse referenziert (flaches Vite-Schema, Lazy-Chunk-Struktur, Store-Sharing zwischen dash/cache).
