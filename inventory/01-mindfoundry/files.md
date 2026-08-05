# MindFoundry — Datei-für-Datei-Analyse

Alle Pfade relativ zu `source/mindfoundry/`. Belegquellen in Klammern.

---

## .gitignore

- Zweck: Definiert, welche Dateien Git ignoriert, um Secrets, generierte Daten, Runtime-Zustand und Editor-/OS-Rauschen aus dem Repo fernzuhalten.
- Verantwortlichkeit: Versionskontrolle-Hygiene; verhindert das Einchecken von Zugangsdaten und regenerierbaren Artefakten.
- Eingaben: Keine (statische Konfiguration).
- Ausgaben: Keine.
- Datenfluss: Kein Laufzeit-Datenfluss; wirkt auf `git add`/`git status`.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Hält `data/hotel_sim.sqlite`, `reports/*.json`, `reports/*.jsonl`, `reports/*.pid`, Logs, `.venv/`, `__pycache__`, `.env`, `service-account*.json`, `client-secret*.json`, `google-credentials*.json`, `oauth-token*.json`, `discord-bot-token*`, `*.pem`, `*.key`, `.secret`, `.token` außerhalb der Versionskontrolle; `reports/.gitkeep` wird ausgenommen (belegt durch die Ignore-Regeln).
- Fehlerfälle: Keine (statische Datei).
- Sicherheitsrelevanz: Hoch — schließt Zugangsdaten- und Token-Dateien von der Versionskontrolle aus (belegt durch die Muster `service-account*.json`, `client-secret*.json`, `.env`, `*.token`).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Git.
- Rust-Relevanz: Konzeptionell übertragbar: Ein Rust-Neuprojekt benötigt dieselbe Ignore-Strategie (Cargo-Target-Verzeichnis, Secrets, generierte Daten). Konkret: Keine Rust-Entscheidungen ableitbar; als `.gitignore`-Vorlage für das Rust-Repo übernehmbar. Rust-Relevanz: Keine.

---

## LICENSE

- Zweck: Legt die MIT-Lizenz für das Projekt fest (Copyright 2026 Roger Lee).
- Verantwortlichkeit: Rechtliche Nutzungsbedingungen (Verwendung, Kopie, Modifikation, Verkauf, Unterlizenzierung; Haftungsausschluss „AS IS").
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine direkte; definiert Haftungsausschluss.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Die MIT-Lizenz ist lizenzkompatibel mit einer Neuentwicklung; das Rust-Projekt benötigt eine eigene Lizenzdatei. Konkrete Rust-Entscheidungen: keine.

---

## Makefile

- Zweck: Judge-freundliche Build-/Setup-/Runtime-Orchestrierung für einen frischen Klon (belegt durch Kopfkommentar).
- Verantwortlichkeit: Erzeugt venv, installiert Abhängigkeiten, generiert die SQLite-Simulation, startet/stoppt die Retrieval-API im Hintergrund, führt Demo/Eval/Smoke/Discord-Drip/Diagnostik/Cleanup aus.
- Eingaben: Umgebungsvariablen `NVIDIA_NIM_API_KEY`, `NVIDIA_NIM_MODEL` (Default `nvidia/llama-3.3-nemotron-super-49b-v1.5`), `HOTELSIM_PORT` (Default 8765), `PYTHON` (Default `python3`), `VENV` (Default `.venv`); Targets `setup`, `venv`, `install`, `generate`, `api`, `stop`, `demo`, `eval`, `smoke`, `discord`, `doctor`, `clean`, `reset`, `help`.
- Ausgaben: `.venv/`, `data/hotel_sim.sqlite`, `reports/hotelsim-api.pid`, `reports/hotelsim-api.log`, `reports/eval-baseline-500.json`, Konsolenausgaben; Hintergrund-API-Prozess.
- Datenfluss: `make setup` → venv + pip; `make generate` → `python3 -m hotel_sim.generate`; `make api` → `nohup python3 api/server.py` mit Health-Polling über `curl /health`; `make demo` → prüft API-Key, startet API, führt `scripts/demo_walkthrough.py` aus; `make eval` → `python3 -m hotel_sim.evaluate --limit 500 --out reports/eval-baseline-500.json`.
- Persistenz: PID-Datei `reports/hotelsim-api.pid`, Log-Datei `reports/hotelsim-api.log`, generierte SQLite.
- Zustände: API läuft/nicht läuft (PID-Check mit `kill -0`); venv vorhanden/nicht vorhanden; SQLite vorhanden/nicht vorhanden; NIM-Key gesetzt/nicht gesetzt (belegt durch `doctor`-Target).
- APIs: Keine eigenen; ruft `scripts/demo_walkthrough.py`, `hotel_sim.evaluate`, `hotel_sim.generate`, `scripts/drip_discord_incidents.py` auf; pollt `http://127.0.0.1:$(HOTELSIM_PORT)/health`.
- Ereignisse: API-Start, API-Stopp, Setup-Abschluss, Demo-Ausführung.
- Nebenwirkungen: Erzeugt venv, installiert Pakete, schreibt PID/Log-Dateien, löscht bei `clean`/`reset` generierte Zustände und das venv.
- Fehlerfälle: Fehlender `NVIDIA_NIM_API_KEY` bricht `make demo` mit Anweisung ab; API-Start-Fehler zeigt die letzten 20 Log-Zeilen und entfernt die PID-Datei; Port bereits belegt wird erkannt (`lsof`).
- Sicherheitsrelevanz: Gibt den NIM-API-Key nur weiter; keine Geheimnisse im Makefile selbst.
- Geschäftslogik: Erzwingt die Reihenfolge Setup → API → Demo; verlangt den NIM-Key vor Live-Synthese.
- Algorithmen: Health-Polling-Schleife (10×1s); Target-Dokumentations-Parsing via `awk` für `make help`.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `python3`, `pip`, `curl`, `nohup`, `lsof` (nur für Port-Check), `make`.
- Rust-Relevanz: Die Orchestrierung (Setup/Generate/Api/Demo/Eval/Smoke/Doctor) muss in einer Rust-Neuentwicklung über ein vergleichbares Build-Tool (z.B. Cargo-Tasks/XTasks) oder ein CLI mit identischen Subkommandos abgebildet werden; Health-Polling und PID-Handling sind wiederverwendbare Konzepte. Rust-Relevanz: Keine (reines Orchestrierungsskript, keine Programmlogik).

---

## README.md

- Zweck: Projekt-Überblick, Problemstellung, Architektur, NVIDIA-Tool-Zuordnung, Setup-/Demo-Anleitung, Evaluationsergebnisse, Repo-Layout.
- Verantwortlichkeit: Öffentliche Einstiegs- und Anleitungsdokumentation; beschreibt die Persistenz als „OpenClaw runtime: heartbeats, SQLite + JSONL".
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Keine zur Laufzeit.
- Datenfluss: Kein Laufzeit-Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Dokumentiert `http://127.0.0.1:8765` (Retrieval-API) und `https://integrate.api.nvidia.com/v1` (NIM).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Dokumentiert die Policy-Gate-Redaktion und verweist auf `docs/nvidia-tools-used.md` und das Policy-YAML.
- Geschäftslogik: Beschreibt die vier Baseline-Metriken (Routing/Policy/Privacy/No-Hallucination, je 100 %) und den autonomen Loop.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Dient als Anforderungsquelle (Capabilities, Umgebungsvariablen, Ports, Metriken) für das Rust-Rewrite. Konkrete Rust-Entscheidungen: keine.

---

## SUBMISSION-PACKAGE.md

- Zweck: Zusammenstellung aller Texte für das Airtable-Einreichungsformular des NVIDIA Agent Hackathon (Projektname, Tagline, Beschreibungen, Problemstellung, NVIDIA-Stack, Scoring-Selbstbewertung, Checkliste).
- Verantwortlichkeit: Einreichungs- und Marketing-Dokumentation.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Beschreibt NemoClaw-Guardrails und PII-Redaktion.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (reine Einreichungstexte). Rust-Relevanz: Keine.

---

## requirements.txt

- Zweck: Deklariert die Python-Laufzeitabhängigkeiten des Projekts.
- Verantwortlichkeit: Liefert die Pakete für Google-Workspace-Integration und PyYAML-Parsing; dokumentiert, dass der NIM-Client und die Discord-Helfer auf der Standardbibliothek basieren.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine (nur Installationsanweisung für pip).
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine direkte.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pip.
- Rust-Relevanz: Die Abhängigkeitsliste ist eine Capability-Quelle: Google-Workspace-APIs und YAML-Parsing müssen in Rust durch Crates abgedeckt werden (siehe `rust-foundation.md`). Rust-Relevanz: Keine (reine Deklaration).

---

## api/server.py

- Zweck: Lokale HTTP-Retrieval-API (ThreadingHTTPServer) auf `127.0.0.1:8765` (Port via `HOTELSIM_PORT`), die Replicant-Retrieval, SQLite-Abfragen, Policy-Docs, Dashboard-Summaries und die UI ausliefert.
- Verantwortlichkeit: Eindeutige Retrieval-Oberfläche für den Agenten; kapselt `hotel_sim.replicants` (build_replicants, retrieve, summary) hinter JSON-REST-Endpunkten.
- Eingaben: HTTP-GET-Requests; Query-Parameter `q` (Suchbegriff) und `limit` bei `/rag/query`, `/rooms`, `/incidents/open`; Pfadsegmente für `/rooms/<id>`, `/guests/<id>`, `/incidents/<id>`, `/policies/<name>`.
- Ausgaben: JSON (UTF-8, `ensure_ascii=False`), `ui/index.html` bei `/` und `/dashboard`, HTTP-Statuscodes 200/400/404/500.
- Datenfluss: GET → Endpunkt-Dispatch (`do_GET`) → SQLite-Abfragen (`rows`/`one`) bzw. `retrieve()`/`build_replicants()`/`summary()` → JSON-Response.
- Persistenz: Liest `data/hotel_sim.sqlite`, `data/policies/*.md`, `reports/baseline-eval.json` (bei `/evaluations/baseline`; falls nicht vorhanden 404).
- Zustände: Keine serverseitige Zustandshaltung (stateless pro Request).
- APIs: `/health`, `/dashboard/summary`, `/evaluations/baseline`, `/replicants`, `/replicants/summary`, `/rag/query`, `/rooms`, `/rooms/<id>`, `/reservations/search`, `/guests/<id>`, `/incidents/open`, `/incidents/<id>`, `/staff`, `/policies`, `/policies/<name>`, `/` und `/dashboard` (UI).
- Ereignisse: Jeder Request; Logging ist absichtlich deaktiviert (`log_message` leer).
- Nebenwirkungen: Keine Schreibzugriffe; liest nur.
- Fehlerfälle: Unbekannter Endpunkt → 404 `unknown_endpoint`; fehlender Query bei `/rag/query` → 400 `missing_query`; fehlende Ressourcen (`/policies/<name>`, `/evaluations/baseline`) → 404 `not_found`; allgemeine Exceptions → 500 mit Typ und Meldung.
- Sicherheitsrelevanz: Bindet ausschließlich an `127.0.0.1`; gibt Reservierungs-Suche inklusive E-Mail/Name zurück (nur lokal); keine Authentifizierung vorgesehen.
- Geschäftslogik: Endpunkt-Semantik für Retrieval (RAG-Query), Dashboard-Summaries, Reservierungs-Suche (joins bookers) und Incident-Details inklusive Messages.
- Algorithmen: Keine eigenen; delegiert an `retrieve()` in `hotel_sim/replicants.py`.
- verwendete Datenmodelle: SQLite-Tabellen rooms, bookers, reservations, staff, incidents, messages (Joins über booker_id/incident_id).
- Abhängigkeiten: `http.server`, `sqlite3`, `urllib.parse` (Standardbibliothek), `hotel_sim.replicants`.
- Rust-Relevanz: Die Endpunkt-Fläche, JSON-Serialisierung, Threading-Server-Verhalten und Fehler-Mappings (404/400/500) definieren die benötigte HTTP-Server-Capability; in Rust über einen Async-HTTP-Server (z.B. axum/hyper) mit JSON-Serialisierung (serde) und SQLite-Zugriff (rusqlite/sqlx) abbildbar. Konkrete Rust-Entscheidungen in `rust-foundation.md`.

---

## data/messages/two_day_event_stream.jsonl

- Zweck: Chronologischer 48-Stunden-Event-Stream von 500 Gast-Nachrichten (15.06.2026 06:00 – 17.06.2026 05:58), angereichert mit Incident-Kontext (belegt durch Dateiinhalt und Zählung).
- Verantwortlichkeit: Repräsentiert die zeitbasierte Operations-Simulation; Ausgabe von `scripts/export_event_stream.py` (JOIN messages × incidents, sortiert nach created_at).
- Eingaben: SQLite-Tabellen messages und incidents (Exportzeitpunkt).
- Ausgaben: Keine (reine Datenquelle).
- Datenfluss: SQLite → JSONL-Export → Retrieval/Demo-Kontext; von `hotel_sim`-Modulen direkt nicht gelesen (belegt durch Grep: keine Referenz in Python-Modulen; Export erfolgt via Script).
- Persistenz: JSONL-Datei; 500 Zeilen.
- Zustände: Jede Zeile enthält `status` des zugehörigen Incidents (`open`/`in_progress`/`resolved`/`escalated`), `severity`, `type`.
- APIs: Keine.
- Ereignisse: 500 Gast-Nachrichten-Ereignisse mit Zeitstempel.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine zur Laufzeit.
- Sicherheitsrelevanz: Enthält synthetische PII-Marker (`guestNNNN@example.test`, Telefonnummern) und `sensitive`-Flags; reales Testmaterial für Privacy-Evaluation.
- Geschäftslogik: Abbild der Incident-Message-Verknüpfung (ein Event je Incident).
- Algorithmen: Chronologische Sortierung (created_at, message_id) beim Export.
- verwendete Datenmodelle: Nachrichten-Ereignis: message_id, incident_id, sender_type, sender_id, channel, language, created_at, body, sensitive, type, severity, status, assigned_staff_id, room_id, reservation_id.
- Abhängigkeiten: `hotel_sim/generate.py` (SQLite-Erzeugung), `scripts/export_event_stream.py`.
- Rust-Relevanz: Definiert ein serialisierbares Event-Schema (serde-struct); Stream-Konsum und Zeilen-Parsing sind in Rust mit `serde_json`/`BufRead` abbildbar. Rust-Relevanz: Keine (reine Daten).

---

## data/policies/privacy.md

- Zweck: Datenschutz-Policy der Simulation: Prinzip der minimalen Informationsweitergabe, Gäste-Verbote und Rollen-Grenzen (Housekeeping/Maintenance/Front Desk/Finance/GM), inklusive chinesischer Kurzfassung.
- Verantwortlichkeit: Retrieval-gegroundete SOP-Quelle für Privacy-Entscheidungen; wird von `hotel_sim/replicants.py` (`policy_snippets`) und `hotel_sim/evaluate.py` (`load_policies`) eingelesen.
- Eingaben: Keine.
- Ausgaben: Policy-Abschnitte (bis 900 Zeichen je Absatz) als Retrieval-Zitate.
- Datenfluss: Datei → `policy_snippets()` → RAG-Citations bzw. Evaluator-Policy-Context.
- Persistenz: Markdown-Datei im Repo.
- Zustände: Keine.
- APIs: Über `/policies/privacy.md` in `api/server.py` abrufbar.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Kern-Spezifikation der Rollen-Grenzen (wer darf was sehen); Grundlage der Privacy-Evaluation und der Gate-Policy.
- Geschäftslogik: Regelt „minimum information needed for the recipient's role" und definiert Verbotsliste für Gäste-Auskünfte.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Inhalte müssen in der Rust-Version als Policy-Ressource (eingebettet oder als Asset) verfügbar sein; semantischer Inhalt ist fachlich, nicht technisch. Rust-Relevanz: Keine.

---

## data/policies/refunds.md

- Zweck: Erstattungs- und Kompensations-Policy: ohne GM-Genehmigung zulässige Fälle (Raumausfall > 2 h, Doppelbelastung, Service-Amenity ≤ TWD 500), nicht automatisch erstattungsfähige Fälle (Lärm ohne Verifikation, Late-Checkout-Verweigerung, Präferenz-Mismatch, Wetter/Flug), Eskalationspflicht (Cash-/OTA-Refund, Deposits, > TWD 500).
- Verantwortlichkeit: Grundlage gegen Halluzination von Erstattungsregeln; wird vom Evaluator als `refunds.md`-Zitatpflicht geprüft (Halluzinations-Risiko wenn `refund|compensation|退款|補償` ohne `refunds.md`-Zitat).
- Eingaben: Keine.
- Ausgaben: Policy-Abschnitte als Retrieval-Zitate.
- Datenfluss: Datei → `policy_snippets()`/`load_policies()` → RAG/Evaluation.
- Persistenz: Markdown-Datei.
- Zustände: Keine.
- APIs: `/policies/refunds.md`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Verhindert unautorisierte Kompensationszusagen (Schutz vor monetären Verlusten).
- Geschäftslogik: Drei Klassen: automatisiert zulässig / nicht automatisiert / Eskalation erforderlich.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Fachlicher Inhalt; in Rust als statische Policy-Ressource abbildbar. Rust-Relevanz: Keine.

---

## data/policies/routing.md

- Zweck: Incident-Routing-Regeln: Typ → verantwortliche Rolle (Maintenance → Maintenance Lead, Housekeeping → Housekeeping Lead, Front Desk → Front Desk Manager, Revenue & Reservations, Guest Experience, Finance/Admin, General Manager) plus Eskalationsregel bei fehlender/mehrdeutiger Policy.
- Verantwortlichkeit: Deterministische Routing-Basis; konsistent mit `hotel_sim/generate.py` (ROUTE), `hotel_sim/replicants.py` (ROUTE_HINTS), `hotel_sim/live_ops.py` (TYPE_OWNER) und `hotel_sim/evaluate.py` (EXPECTED_ROUTE).
- Eingaben: Keine.
- Ausgaben: Routing-Zitate für RAG und Evaluator.
- Datenfluss: Datei → `policy_snippets()`/`load_policies()`.
- Persistenz: Markdown-Datei.
- Zustände: Keine.
- APIs: `/policies/routing.md`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Indirekt — korrektes Routing verhindert, dass PII an falsche Rollen gelangt.
- Geschäftslogik: Typ→Rolle-Zuordnung plus Eskalations-Prinzip („escalate instead of inventing a rule").
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Fachlicher Inhalt; als statische Policy-Ressource abbildbar. Rust-Relevanz: Keine.

---

## data/replicants/discord_memories.jsonl

- Zweck: JSONL-Speicher extrahierter Discord-Staff-Memories (160 Einträge, 26.–27.05.2026), erzeugt durch `scripts/update_replicants_from_discord.py`.
- Verantwortlichkeit: Langfristige Replicant-Memory-Quelle aus Discord; wird von `hotel_sim/replicants.py` (`discord_memories`) und `hotel_sim/live_ops.py` (`recent_memories`) gelesen.
- Eingaben: Append-only durch `update_replicants_from_discord.py`.
- Ausgaben: Memory-Objekte für Replicant-Konstruktion und Live-Ops-Zusammenfassung.
- Datenfluss: Discord → Updater → JSONL → Replicants/Retrieval.
- Persistenz: JSONL-Datei; 160 Zeilen, Memory-IDs eindeutig (belegt durch Zählung).
- Zustände: Keine expliziten; `kind` ist durchgängig `discord_staff_memory`.
- APIs: Keine.
- Ereignisse: Keine (Datenbestand).
- Nebenwirkungen: Keine.
- Fehlerfälle: `hotel_sim/replicants.py` überspringt JSONDecodeError-Zeilen (fehlertolerantes Lesen).
- Sicherheitsrelevanz: Enthält Staff-Äußerungen aus Discord; PII-Redaktion greift erst beim Versand.
- Geschäftslogik: Wissenssignale (7 eindeutige Textvorlagen, belegt durch Zählung) aus Staff-Followups, die als „durable knowledge" gelten.
- Algorithmen: Keine (Daten).
- verwendete Datenmodelle: Memory: memory_id, discord_message_id, channel, person, role, source (`discord:<channel>`), kind, text, created_at.
- Abhängigkeiten: `scripts/update_replicants_from_discord.py`, `scripts/discord_utils.py`.
- Rust-Relevanz: Definiert ein append-only JSONL-Memory-Schema; Rust: `serde`-struct + `BufWriter`-Append mit Determinismus. Rust-Relevanz: Keine (reine Daten).

---

## docs/QUICKSTART.md

- Zweck: 60-Sekunden-Anleitung für Juroren (Klon → `make setup` → `make api` → `make demo`), inklusive erwarteter Ausgaben und Overrides (`NVIDIA_NIM_MODEL`, `HOTELSIM_PORT`).
- Verantwortlichkeit: Einstiegsdokumentation.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
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
- Rust-Relevanz: Dokumentiert erwartete Demo-Ausgaben (Health, RAG, Adversarial Probe, Eval) — Anforderungsquelle für das Rust-CLI. Rust-Relevanz: Keine.

---

## docs/architecture.md

- Zweck: Architektur-Beschreibung der Simulation: Vergleich mit realen Hotel-Systemen (PMS, Ops-Tools, API/Webhook, Messaging, SOP-KB) und gewählte Sim-Architektur (SQLite, lokale API, Markdown-Policies, JSONL-Event-Stream, Discord, Google Workspace) mit Skalenangaben (250 Räume, 2.500 Buchungspersonen, 3.150 Reservierungen mit Wiederholungsverteilung 2050/300/100/50, 8 Staff, 500 Vorfälle).
- Verantwortlichkeit: Architektur-Entscheidungsdokument.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Dokumentiert die korrigierte Reservierungszahl (3.150 statt 3.450).
- Algorithmen: Keine.
- verwendete Datenmodelle: Beschreibt die Datenbestände.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Schicht-Modell (PMS/Ops/API/Messaging/SOP) ist eine Anforderungsquelle für die Rust-Datenarchitektur. Rust-Relevanz: Keine (Doku).

---

## docs/dashboard-screenshot.png

- Zweck: Rasterbild (PNG, 1920×1080, RGBA, 41.454 Bytes, belegt durch `file`/`ls`) des „HotelSim Control Room"-Dashboards für Präsentationszwecke.
- Verantwortlichkeit: Visuelle Dokumentation der UI (`ui/index.html`); kein Laufzeit-Artefakt.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Binärdatei im Repo.
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
- Rust-Relevanz: Pixelinhalt nicht maschinell lesbar (Modell ohne Bildunterstützung); technische Bedeutung: dokumentiert das Dashboard, dessen Inhalte (Live-Kennzahlen, offene Incidents, Staff-Karten, RAG-Query, Guardrails, Policies) aus `ui/index.html` und `api/server.py` (`/dashboard/summary`) belegbar sind. Rust-Relevanz: Keine (Asset).

---

## docs/demo-script.md

- Zweck: Drehbuch für das Demo-Video (2:15–2:55), je Beat mit Zeitstempel, Show-Anweisungen und wörtlicher Narration; inkl. 30-Sekunden-Elevator-Pitch, Aufnahmetipps und Pre-Recording-Checkliste.
- Verantwortlichkeit: Präsentationsdokumentation.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
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
- Rust-Relevanz: Keine. Rust-Relevanz: Keine.

---

## docs/google-workspace-plan.md

- Zweck: Sandbox-Plan für die Fake-Staff-Workspace (Domain `snapdesign.tw`): User, Gruppen, Shared Drives/Docs, Berechtigungstests (Rollenübergriffe) und Setup-Sicherheit (temporäres Admin-Konto, scoped API-Zugriff).
- Verantwortlichkeit: Planungsdokument für die Workspace-Provisionierung.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Hoch — definiert Berechtigungsgrenzen (Housekeeping ≠ Finance, Maintenance ≠ Payments, Front Desk ohne volle Zahlungsdetails, Manager-all) und Agenten-Pflicht (redact/refuse bei Rollenübergriff).
- Geschäftslogik: Spiegel der SQLite-Simulation, nicht Ersatz.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Fachliches Konzept für RBAC-Szenarien. Rust-Relevanz: Keine.

---

## docs/mindfoundry-architecture.png

- Zweck: Raster-Rendering (PNG, 1800×2800, RGB, 300.986 Bytes, belegt durch `file`/`ls`) des Architektur-Diagramms für README/Demo.
- Verantwortlichkeit: Visualisierung der Agent-Architektur.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Binärdatei.
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
- Rust-Relevanz: Inhalt des Diagramms ist vollständig aus der SVG-Quelle `docs/mindfoundry-architecture.svg` lesbar (siehe dort): Agent-Loop (Knowledge Gatherer → Replicant Updater → Nemotron RAG + Actions), Quellen (Discord 10 Kanäle, Google Workspace), Persistenz (SQLite + Replicant-Memory, 8 Staff/68 Memories), Nemotron via NVIDIA NIM, NemoClaw Policy Engine (filesystem/network/inference/PII-Redaktion, Audit), Surfaces (Google Workspace, Control Room :8765), OpenClaw-Cron (Drip 15 min, RAG 2 min, Replicant-Updater 5 min), Baseline-Eval-Badge. Pixelinhalt des PNGs: Nicht nachweisbar (Modell ohne Bildunterstützung). Rust-Relevanz: Keine (Asset).

---

## docs/mindfoundry-architecture.svg

- Zweck: Vektorbasiertes Architektur-Diagramm (900×1400, dunkles Theme) mit dem kompletten Agent-Loop und Security-Gate (belegt durch Dateiinhalt).
- Verantwortlichkeit: Kanonische Architektur-Visualisierung; Quelle für die PNG-Rasterfassung.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: SVG-Datei.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Visualisiert die fünf Hauptkomponenten und deren Datenfluss.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Dient als Architektur-Beleg für `architecture.md`; keine direkte Rust-Entscheidung. Rust-Relevanz: Keine (Asset).

---

## docs/mindfoundry-preview.png

- Zweck: Social-Card/Preview-Bild (PNG, 2560×1280, RGB, 724.985 Bytes, belegt durch `file`/`ls`; in `SUBMISSION-PACKAGE.md` als 1280×640-Cover bezeichnet) für README und Airtable-Einreichung.
- Verantwortlichkeit: Marketing-Asset.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Binärdatei.
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
- Rust-Relevanz: Inhalt aus SVG-Quelle `docs/mindfoundry-preview.svg` lesbar: Titel „MindFoundry", Taglines („NemoClaw-secured long agent for teammate knowledge.", „Cited Nemotron answers. Policy-enforced privacy."), vier Stack-Pills (Nemotron/NemoClaw/NVIDIA NIM/OpenClaw), Stat-Zeile (100 % Routing/Policy/Privacy/No-Hallucination, 500 Incidents evaluated), Repo-URL und „NeMo Lodge demo · 8 staff replicants · bilingual". Pixelinhalt des PNGs: Nicht nachweisbar. Rust-Relevanz: Keine (Asset).

---

## docs/mindfoundry-preview.svg

- Zweck: Vektor-Social-Card (1280×640) für README und Submission (belegt durch Dateiinhalt).
- Verantwortlichkeit: Kanonische Marketing-Visualisierung.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: SVG-Datei.
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
- Rust-Relevanz: Keine. Rust-Relevanz: Keine (Asset).

---

## docs/nvidia-tools-used.md

- Zweck: Detaillierte Zuordnung der vier NVIDIA-Tools (Nemotron, NemoClaw, NVIDIA NIM, OpenClaw) zu ihren Rollen in MindFoundry; dokumentiert NIM-Call-Pattern (OpenAI-kompatible Chat-Completions mit System-/User-Prompt, temperature 0.2, max_tokens 600), Policy-Inhalte (Filesystem/Network/Inference, Rollen-Egress) und Degradationsverhalten (`NVIDIA_NIM_DISABLE=1`).
- Verantwortlichkeit: Nachweis der NVIDIA-Nutzung (Hackathon-Kriterium).
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine (dokumentiert `POST https://integrate.api.nvidia.com/v1/chat/completions`).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Dokumentiert Rollen-Egress-Matrix (`public_discord`/`finance_private`/`manager_private`) und PII-Erkennungsfelder.
- Geschäftslogik: Beschreibt die drei Nemotron-Einsatzorte (RAG-Synthese, Wissensextraktion, Routing/Action-Drafting) und NIM-Model-Discovery (`GET /v1/models`, ~25 Varianten).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Beleg für `nim.md`; Request-/Response-Aufbau ist Anforderungsquelle für den Rust-NIM-Client. Rust-Relevanz: Keine (Doku).

---

## docs/openshell-policy-neemo-lodge.yaml

- Zweck: Portables OpenShell/NemoClaw-Policy-Draft für den NeMo-Lodge-Agenten: Name `nemo-lodge-hotelsim-agent` v0.1, Filesystem-Allowlist (read/write/deny), Netzwerk-Allowlist (localhost:8765 GET, Discord GET/POST auf `/api/v10/channels/*/messages`, NVIDIA POST), Prozess-Allow/Deny, DLP (`default_action: redact_and_audit`, Klassen password/guest_email/phone/payment_card/passport_or_id/internal_notes) und Audit-Konfiguration (Log-Datei, Discord-Kanal, Event-Typen).
- Verantwortlichkeit: Deklarative Sicherheits-Policy; Laufzeit-Enforcement liegt in `hotel_sim/policy_gate.py` (die YAML wird nicht von `policy_gate.py` gelesen — belegt durch Quellcode; PyYAML wird laut `requirements.txt` nur für Code-Pfade genutzt, die die YAML parsen, „parsing is only needed for tests").
- Eingaben: Keine (statische Konfiguration).
- Ausgaben: Keine.
- Datenfluss: Kein Laufzeit-Datenfluss; dient als Policy-Spezifikation.
- Persistenz: YAML-Datei.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Höchste — definiert Default-Deny für Filesystem/Network/Process, DLP-Klassen mit Aktionen (deny/redact) und Audit-Event-Typen (`policy_denied`, `redaction`, `outbound_discord_post`, `external_network_denied`, `filesystem_denied`).
- Geschäftslogik: Rollen-Egress (public_discord-Kanäle), Passwort-Klasse mit Aktion `deny`, übrige Klassen `redact`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Policy-Schema (filesystem/network/process/data_loss_prevention/audit).
- Abhängigkeiten: OpenShell/NemoClaw (extern; im Repo nicht enthalten).
- Rust-Relevanz: Definiert die benötigte deklarative Policy-Engine-Capability; Rust: Policy-Parsing (serde_yaml), Policy-Enforcement als Trait. Konkrete Entscheidungen in `rust-foundation.md`.

---

## docs/product-description.md

- Zweck: Produktbeschreibung (One-Liner, Kurz-/Langfassung, „Why this wins", Team) für Präsentations- und Einreichungszwecke.
- Verantwortlichkeit: Marketing-/Produktdokumentation.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Beschreibt Guardrails und PII-Redaktion.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine. Rust-Relevanz: Keine.

---

## docs/test-cases/hotel-sim.md

- Zweck: Gherkin-artige Testfall-Szenarien für die Simulation: AC-Routing, Zahlungs-/Privacy-Leak-Schutz zu Housekeeping, Vermeidung halluzinierter Refund-Policy, zweisprachige Abfragen, zeitbasierter 48h-Lauf, Dashboard-Verhalten inkl. API-Ausfall-Edge-Case.
- Verantwortlichkeit: Test-Spezifikation für das Simulationsverhalten.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Beschreibt Edge Cases (anderer Gast fragt Raumstatus, interne Notizen mit Finanzdaten, über-generalisierte Refund-Policy, gemischte EN/ZH-LINE-Nachrichten, gleichzeitige dringende Incidents, API nicht verfügbar).
- Sicherheitsrelevanz: Formuliert die Privacy-/PII-Schutzszenarien.
- Geschäftslogik: Anforderungsliste für Routing-, Refund- und Bilingual-Verhalten.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Direkte Quelle für spätere Rust-Integrationstests (siehe `tests.md`). Rust-Relevanz: Keine (Doku).

---

## hotel_sim/generate.py

- Zweck: Deterministische Generierung der SQLite-Simulation (Seed 42): 250 Räume, 2.500 Buchungspersonen, 3.150 Reservierungen (Verteilung 2050×1 / 300×2 / 100×3 / 50×4), 8 Staff mit Rollen, 500 Incidents + 500 Messages + 500 Audit-Logs über zwei simulierte Tage ab 2026-06-15 06:00 (belegt durch Quellcode).
- Verantwortlichkeit: Erzeugt das kanonische Datenmodell (Schema-DDL) und die Datenverteilungen.
- Eingaben: CLI `--out` (Default `data/hotel_sim.sqlite`) und `--seed` (Default 42).
- Ausgaben: SQLite-Datei mit Tabellen rooms, bookers, reservations, staff, incidents, messages, audit_logs + Indizes; JSON-Summary (Zählwerte).
- Datenfluss: Seed → Zufallsgenerator → Tabellenbefüllung → Commit.
- Persistenz: `data/hotel_sim.sqlite`.
- Zustände: Room-Status (`vacant_clean`/`vacant_dirty`/`occupied`/`out_of_order`), Incident-Status (`open`/`in_progress`/`resolved`/`escalated`), Reservation-Status (`checked_in` nur 15.–16.06.2026, sonst `confirmed`/`completed`/`cancelled`), `contains_sensitive`-Flag.
- APIs: Keine.
- Ereignisse: Keine (reine Datenerzeugung).
- Nebenwirkungen: Überschreibt bestehende Tabellen (DROP TABLE IF EXISTS); schreibt SQLite-Datei.
- Fehlerfälle: Assertion auf Reservierungsverteilung (`len(counts)==2500 and sum(counts)==3150`); Zufallsauswahl `pick_weighted` fällt auf letztes Element zurück, falls kumulative Gewichte nicht greifen.
- Sicherheitsrelevanz: Erzeugt synthetische PII (E-Mail `guestNNNN@example.test`, Telefon `+886-900-NNNNNN`, Loyalty-Tier, `privacy_level`), `internal_notes` mit Hinweisen wie „Do not disclose stay details"; `contains_sensitive` für billing und 8 % Zufallsanteil.
- Geschäftslogik: Staff-Mapping (Rollen, Schichten, Clearance high für S001/S003/S008), Typ→Staff-Routing (ROUTE), gewichtete Severity- und Buchungsquellen-Verteilungen, zweisprachige Gast-Texte (en/zh-TW, 55/45), Nachrichten-Kanäle.
- Algorithmen: `pick_weighted` (kumulative Gewichtswahl), `random.choices` mit Gewichten, zeitliche Verteilung über `random.randint(0, 48*60-1)` Minuten, Wiederholungsbuchungen über gestaffelte Check-in-Daten (`+10..45 Tage × j`).
- verwendete Datenmodelle: SQLite-Schema: rooms(room_id PK, floor, room_number, room_type, status, privacy_zone); bookers(booker_id PK, full_name, zh_name, email, phone, language, loyalty_tier, privacy_level); reservations(reservation_id PK, booker_id FK, room_id FK, check_in, check_out, status, source, adults, children, rate_twd, internal_notes); staff(staff_id PK, name, zh_name, role, zh_role, responsibilities, personality, shift_start, shift_end, clearance); incidents(incident_id PK, reservation_id FK, room_id FK, type, severity, status, created_at, updated_at, assigned_staff_id FK, guest_visible_summary, internal_notes, contains_sensitive); messages(message_id PK, incident_id FK, sender_type, sender_id, channel, language, created_at, body, sensitive); audit_logs(audit_id PK, created_at, actor, action, resource_type, resource_id, decision). Indizes: booker_id, incidents(status, severity), messages(created_at). Fremdschlüssel aktiviert (`PRAGMA foreign_keys=ON`).
- Abhängigkeiten: stdlib (argparse, json, random, sqlite3, datetime, pathlib).
- Rust-Relevanz: Datengenerierung ist in Rust mit einem deterministischen PRNG (Seed), SQLite-Crates und serde-Schemata abbildbar; die Verteilungen und Constraints (Assertion) sind Anforderung für Generierungs-Tests. Konkrete Entscheidungen in `rust-foundation.md`.

---

## hotel_sim/evaluate.py

- Zweck: Evaluations-Harness: bewertet einen deterministischen Baseline-Agenten über Incidents aus SQLite gegen erwartetes Routing, Policy-Zitate, Privacy (Leak-Muster) und Halluzinationsrisiko; schreibt Report-JSON.
- Verantwortlichkeit: Quantifiziert die vier Baseline-Metriken (routing_accuracy, policy_grounding_rate, privacy_pass_rate, hallucination_pass_rate, avg_score_4).
- Eingaben: CLI `--limit` (Default 100), `--type` (Incident-Typ-Filter), `--out` (Report-Ziel); liest `data/hotel_sim.sqlite` und `data/policies/*.md`.
- Ausgaben: Report-JSON (`summary` + `results[]`), Konsolen-Summary.
- Datenfluss: SQLite-Incidents → Agent.decide(incident, messages, policies) → score(decision, incident) → Aggregation.
- Persistenz: `reports/eval-baseline-500.json` (500 Einträge, alle Score 4/4, belegt durch Zählung).
- Zustände: Ergebnis-JSON pro Incident (routing_ok/policy_ok/privacy_ok/hallucination_ok/score).
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Schreibt Report-Datei bei `--out`.
- Fehlerfälle: Leere Ergebnismenge → Division durch 0 wird per `if results else 0` abgefangen.
- Sicherheitsrelevanz: `SENSITIVE_PATTERNS` (Taiwan-Telefon, `guestNNNN@example.test`, Karten-/Zahlungs-/Tax-ID-Begriffe, „Do not disclose stay details") werden nur bei `contains_sensitive`-Incidents geprüft; `leak_hits` werden gelistet.
- Geschäftslogik: `EXPECTED_ROUTE` (Typ→Staff-ID), `POLICY_KEYWORDS` (Typ→erwartete Policy-Dateien: billing→refunds+privacy, noise→refunds+routing, maintenance→routing+refunds, housekeeping→routing+privacy, safety→routing+privacy), Eskalationsregel (`billing`/`safety`/`urgent`), Redaktionsregel für interne Notizen bei sensitiven Fällen, Halluzinations-Heuristik (Refund-Begriff ohne `refunds.md`-Zitat).
- Algorithmen: Deterministische Agenten-Entscheidung; Keyword-basiertes Policy-Zitat; Regex-Leak-Erkennung; Score-Summierung (4 Kriterien à 1 Punkt); Aggregation als Raten.
- verwendete Datenmodelle: Incident-/Message-Dicts aus SQLite; Decision-Dict (incident_id, route_to_staff_id, route_reason, policy_citations, guest_reply, internal_note, requires_escalation, redacted_sensitive); Score-Dict.
- Abhängigkeiten: stdlib (argparse, json, re, sqlite3, pathlib).
- Rust-Relevanz: Evaluations-Harness (deterministischer Agent + Score) ist in Rust als reine Funktionen (Input: Incident, Policies → Score) testbar; Heuristiken sind Anforderung für Unit-Tests. Konkrete Entscheidungen in `rust-foundation.md`.

---

## hotel_sim/live_ops.py

- Zweck: Aggregiert Live-Operations-Status aus SQLite und Discord-Post-Zustand (seeded/dripped IDs) zu einer Zusammenfassung inkl. Owner-Queues, Severity-Mix, Privacy-Hinweisen und Staff-Memory-Signalen.
- Verantwortlichkeit: Live-Signalquelle für den RAG-Agenten („what is happening right now") und Tests.
- Eingaben: SQLite (`data/hotel_sim.sqlite`), Zustandsdateien `reports/discord-seeded-incidents.json` und `reports/discord-drip-state.json`, Memory-Datei `data/replicants/discord_memories.jsonl`.
- Ausgaben: Dict `live_summary` bzw. gerenderter Markdown-Text `render_summary`.
- Datenfluss: posted_ids → Incident-SELECT (entweder nach geposteten IDs oder nach offenen Status) → Counter/Dict-Aggregation → Text.
- Persistenz: Liest Zustandsdateien; schreibt nichts.
- Zustände: Keine eigenen; abhängig von Incident-Status und geposteten IDs.
- APIs: Keine (Funktionsbibliothek).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlende Zustands-/Memory-Dateien werden toleriert (exists-Checks, Exception-Silencing beim JSON-Laden).
- Sicherheitsrelevanz: Markiert privacy-sensitive Incidents und instruiert „summarize only, do not expose private fields"; Summary nennt keine privaten Felder.
- Geschäftslogik: `TYPE_OWNER`-Mapping (Incident-Typ → Owner-String), Queue-Bildung pro Owner, Urgent-/Escalated-Filter (severity urgent/high oder status escalated), Limit 12 Incidents / 6 Queue-Einträge / 5 Memory-Signale.
- Algorithmen: Set-Union über posted IDs; Counter; defaultdict-Queue; Slice-Limits.
- verwendete Datenmodelle: Incident-Rows (dicts), Memory-Rows, Owner-Queue-Dict.
- Abhängigkeiten: stdlib (json, sqlite3, collections).
- Rust-Relevanz: Aggregationslogik als reine Rust-Funktionen (Counter/Queues aus SQLite-Rows); deterministisch testbar. Konkrete Entscheidungen in `rust-foundation.md`.

---

## hotel_sim/policy_gate.py

- Zweck: PII-Erkennungs- und Redaktionsschicht für jeden ausgehenden Text: Regex-Muster für password, credit_card, taiwan_phone, email, passport_or_id, internal_note; rollenbasierte Freigaben; Audit-Anhang an `reports/policy-gate-events.jsonl`.
- Verantwortlichkeit: Enforcement des NemoClaw-Inference-Gates im Code (komplementär zur YAML-Policy).
- Eingaben: Text (`gate_text`) oder beliebig verschachtelte JSON-artige Payloads (`gate_payload`), Ziel-Destination (Default `public_discord`).
- Ausgaben: `GateResult` (allowed, text, redactions, destination, decision) bzw. redigierte Payload; JSONL-Zeile im Audit-Log.
- Datenfluss: Text → Regex-Substitution mit Rollen-Policy → Redactions-Liste → Audit-Append → Ergebnis.
- Persistenz: Append an `reports/policy-gate-events.jsonl` (create parents).
- Zustände: Entscheidungen `allow` / `allow_with_redactions`; Rollen-Policies `public_discord`/`finance_private`/`manager_private`.
- APIs: `gate_text(text, destination)`, `gate_payload(payload, destination)`, Dataclass `GateResult`.
- Ereignisse: Jede Gate-Entscheidung erzeugt einen Audit-Eintrag.
- Nebenwirkungen: Schreibt Audit-JSONL; erzeugt `reports/`-Verzeichnis.
- Fehlerfälle: Unbekannte Destination fällt auf `public_discord` zurück; E-Mail-Matching erfordert TLD (2+ Großbuchstaben); Kreditkarten-Regex verlangt Trennzeichen oder echte Kartengruppierung, damit Discord-Snowflake-IDs nicht fälschlich redigiert werden (belegt durch Kommentar).
- Sicherheitsrelevanz: Kern der PII-Redaktion: Staff-E-Mails (Allowlist `ALLOW_EMAILS`, 8 Adressen) bleiben in `public_discord` sichtbar, Gäste-E-Mails nicht; Passwörter in keiner Rolle; `finance_private` erlaubt Zahlungsmetadaten, `manager_private` erlaubt `internal_notes`; Ersatztext `[KIND REDACTED]`.
- Geschäftslogik: Klassenbasierte Freigabe-Matrix; Sample-Erfassung (erste 24 Zeichen) für Audit-Zwecke.
- Algorithmen: Sechs Regex-Muster (case-insensitive, Wortgrenzen, ASCII-/Unicode-Zeichenklassen inkl. chinesischer Begriffe 身分證/護照); rekursiver Payload-Walk.
- verwendete Datenmodelle: `GateResult`-Dataclass (allowed, text, redactions[{type, sample}], destination, decision).
- Abhängigkeiten: stdlib (json, re, dataclasses, pathlib).
- Rust-Relevanz: Regex-Redaktions-Engine mit Rollen-Policy ist eine zentrale Capability; in Rust mit `regex`-Crate und typisierten Enums (Destination, RedactionKind) abbildbar; Thread-Sicherheit des Audit-Appends über Mutex/atomic append. Konkrete Entscheidungen in `rust-foundation.md`.

---

## hotel_sim/replicants.py

- Zweck: Baut Replicant-Profile pro Staff-Mitglied aus SQLite (Profil, Verhalten, Workspace-Identität, Erfahrung, Fälle) plus Discord-Memories und führt deterministisches Retrieval (Tokenisierung, Scoring, Intent-Klassifikation, Routing-Vorschlag) durch.
- Verantwortlichkeit: Kern des Wissens- und Retrieval-Systems (Replicant Updater-Ausgabe + RAG-Retrieval).
- Eingaben: SQLite (`staff`, `incidents`), `data/replicants/discord_memories.jsonl`, `data/policies/*.md`, `reports/workspace-provisioning.json`, Query-String `q` und Limit.
- Ausgaben: Replicant-Liste, Policy-Snippets, Retrieval-Dict (query, detected_intents, recommended_route, guardrail, draft_next_step, citations), Summary.
- Datenfluss: SQLite+JSONL → build_replicants → Memory-Chunks → score_text/classify_query → sortierte Citations + Route.
- Persistenz: Nur Lesen; kein Schreiben.
- Zustände: Keine persistenten Zustände; berechnete Scores je Chunk.
- APIs: `build_replicants()`, `retrieve(q, limit)`, `summary()`, `policy_snippets()`, `discord_memories()`, `load_workspace()`, `tokenize()`, `score_text()`, `classify_query()`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlende JSONL-/Report-Dateien werden toleriert (exists-Checks); JSONDecodeError-Zeilen in Memory-Datei werden übersprungen.
- Sicherheitsrelevanz: `guardrail`-Text und `draft_next_step` verankern die Privacy-Regel im Retrieval-Ergebnis; Case-Memories enthalten `contains_sensitive`-Hinweise; Workspace-Identität enthält E-Mail/Gruppe.
- Geschäftslogik: `ROLE_TO_EMAIL`/`ROLE_TO_GROUP` (Rolle→snapdesign.tw-E-Mail/-Gruppe), `TYPE_KEYWORDS` (10 Intent-Klassen mit EN/ZH-Keywords), `ROUTE_HINTS` (Intent→Rolle), VIP-Intent erzwingt zusätzlich General Manager, Live-Query-Trigger („what is happening", „right now", „live", „status", „summary", „going on", „urgent", „open incidents") injiziert Live-Ops-Zusammenfassung mit Score 999.
- Algorithmen: `tokenize` (Regex-Token EN + CJK), `score_text` (2 Punkte pro exakten Term-Treffer + 1 Punkt pro Präfix-Treffer >3 Zeichen), Intent-Bonus (+3 für Replicant-Memory, +4 für Policy-Snippets mit privacy/refund/routing/escalat), absteigende Sortierung, Citation-Limit.
- verwendete Datenmodelle: Staff-Rows, Incident-Rows, Memory-Dicts (kind, source, text), Chunk-Dicts (score, person, role, kind, source, text), Retrieval-Dict, Summary-Dict.
- Abhängigkeiten: stdlib (json, re, sqlite3, dataclasses, pathlib), `hotel_sim/live_ops.render_summary`.
- Rust-Relevanz: Deterministisches Retrieval (Tokenisierung, Scoring, Klassifikation) ist in Rust als reine, unit-testbare Funktionen abbildbar; Replicant-Struktur → typisierte Structs. Konkrete Entscheidungen in `rust-foundation.md`.

---

## reports/.gitkeep

- Zweck: Platzhalterdatei, damit das `reports/`-Verzeichnis in Git erhalten bleibt, obwohl `reports/*.json` und `reports/*.jsonl` ignoriert werden (belegt durch `.gitignore`-Ausnahme `!reports/.gitkeep`).
- Verantwortlichkeit: Versionskontroll-Hygiene.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Sicherheitsrelevanz/Geschäftslogik/Algorithmen/verwendete Datenmodelle/Abhängigkeiten: Keine (leere Datei).
- Rust-Relevanz: Keine.

---

## reports/eval-baseline-500.json

- Zweck: Kommittiertes Baseline-Evaluationsergebnis über 500 Incidents: Summary (evaluated 500, alle vier Raten 1.0, avg_score_4 4.0) plus 500 Ergebnis-Datensätze (belegt durch Zählung und Counter-Analyse).
- Verantwortlichkeit: Reproduzierbarer Nachweis der 100 %-Metriken für README/Demo/Submission.
- Eingaben: Erzeugt durch `hotel_sim/evaluate.py --limit 500 --out reports/eval-baseline-500.json`.
- Ausgaben: Keine.
- Datenfluss: Evaluator → JSON-Datei → Demo/README.
- Persistenz: JSON-Datei (12.748 Zeilen).
- Zustände: Je Datensatz routing_ok/policy_ok/privacy_ok/hallucination_ok/score (durchgängig 4), decision-Dict.
- APIs: `/evaluations/baseline` liest `reports/baseline-eval.json` — die kommittierte Datei heißt `eval-baseline-500.json`; `/evaluations/baseline` liefert bei fehlender Datei 404 (Namensdivergenz belegt durch `api/server.py`).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: `leak_hits` ist in allen 500 Datensätzen leer (0 Treffer, belegt durch Zählung).
- Geschäftslogik: Deterministische Baseline (Agent `DeterministicHotelAgent`) validiert den Scoring-Harness; Typ-Verteilung: lost_item 60, front_desk 57, maintenance 54, reservation_change 54, noise 52, safety 49, vip_request 47, access 45, housekeeping 43, billing 39 (belegt durch Zählung).
- Algorithmen: Score-Aggregation des Evaluators.
- verwendete Datenmodelle: Summary-Dict + Results-Liste (siehe `hotel_sim/evaluate.py`).
- Abhängigkeiten: `hotel_sim/evaluate.py`, `hotel_sim/generate.py` (Datenbasis).
- Rust-Relevanz: Dient als Gold-Datei für einen Rust-Evaluator-Regressionstest (100 %-Erwartung). Rust-Relevanz: Keine (Daten).

---

## reports/live-ops-summary.txt

- Zweck: Gerenderte Live-Ops-Zusammenfassung (37 gepostete Incidents, Severity-Mix urgent=12, 4 privacy-sensitive, Urgent-/Escalated-Queue, Owner-Queues, 5 Staff-Memory-Signale) — Ausgabe von `hotel_sim/live_ops.render_summary()`.
- Verantwortlichkeit: Nachweis des Live-Ops-Renderers.
- Eingaben: SQLite + Zustandsdateien (siehe `live_ops.py`).
- Ausgaben: Keine.
- Datenfluss: live_summary → Text.
- Persistenz: Textdatei.
- Zustände: Enthält Incident-Status und Severity.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Zeigt ausschließlich guest_visible_summary und Staff-Memory-Signale, keine privaten Felder.
- Geschäftslogik: Queue- und Mix-Darstellung.
- Algorithmen: Rendering des Summaries.
- verwendete Datenmodelle: Summary-Dict.
- Abhängigkeiten: `hotel_sim/live_ops.py`.
- Rust-Relevanz: Erwartete Textform als Test-Fixture für Rust. Rust-Relevanz: Keine (Daten).

---

## scripts/demo_walkthrough.py

- Zweck: End-to-End-Demo für Juroren: 1) API-Health-Check, 2) Nemotron-RAG-Antwort auf echte Ops-Frage, 3) Adversarial-PII-Probe mit Policy-Gate-Redaktion, 4) Baseline-Evaluation (gecached oder frisch via `-m hotel_sim.evaluate --limit 500`).
- Verantwortlichkeit: Automatisierter Nachweis des Gesamtstacks ohne Discord.
- Eingaben: `NVIDIA_NIM_API_KEY`, `NVIDIA_NIM_MODEL`, `HOTELSIM_PORT`; stubbt `discord_utils` (SimpleNamespace) für Offline-Import der Bridge.
- Ausgaben: Farbige Konsolenausgabe (deaktiviert bei Nicht-TTY), Exit-Code 0/1.
- Datenfluss: API `/health` → `/rag/query` → `nemotron_synthesize` → `gate_text` → Ausgabe; Eval-Datei lesen oder erzeugen.
- Persistenz: Liest/schreibt `reports/eval-baseline-500.json` (Cache).
- Zustände: Health ok/fail; NIM live/Fallback.
- APIs: `http://127.0.0.1:<port>/health`, `/rag/query`.
- Ereignisse: Keine.
- Nebenwirkungen: Startet bei Bedarf Evaluator-Subprozess und schreibt Report.
- Fehlerfälle: API nicht erreichbar → FAIL-Meldung mit Starthinweis, Exit 1; NIM-Fehler werden als Hinweis ausgegeben (kein Abbruch).
- Sicherheitsrelevanz: Führt die Adversarial-Probe aus und zeigt Redaktionen.
- Geschäftslogik: Reihenfolge der vier Demo-Schritte; Modellname aus Env.
- Algorithmen: Keine eigenen.
- verwendete Datenmodelle: Retrieval-/Synthesis-/GateResult-Dicts.
- Abhängigkeiten: stdlib; `scripts/nemotron_rag_bridge.py`, `hotel_sim.policy_gate`, `hotel_sim.evaluate`.
- Rust-Relevanz: Definiert den CLI-Demo-Workflow (Health → RAG → Probe → Eval) für das Rust-Rewrite. Konkrete Entscheidungen in `rust-foundation.md`.

---

## scripts/discord_utils.py

- Zweck: Discord-HTTP-API-Client (v10) auf Standardbibliothek: Bot-Token aus `~/.openclaw/openclaw.json`, Kanal-IDs aus `reports/discord-secondoffice-channels.json`, `send` (mit Policy-Gate, Chunking auf 1900 Zeichen, Rate-Sleep 0,7 s, optionalem Audit-Post), `read`.
- Verantwortlichkeit: Einzige Discord-Schnittstelle aller Scripts.
- Eingaben: Channel-ID, Content, Destination (Default `public_discord`), Audit-Flag.
- Ausgaben: API-Antworten (JSON), Liste geposteter Nachrichten.
- Datenfluss: Content → `gate_text` → Chunking → POST `/channels/{id}/messages`; bei Redaktionen zusätzlich Audit-Post an `#agent-audit-log`.
- Persistenz: Liest OpenClaw-Config und Kanal-Report; schreibt keine Dateien.
- Zustände: Keine.
- APIs: `bot_token()`, `headers()`, `request(method, path, body)`, `channel_ids()`, `send(...)`, `read(channel_id, limit)`.
- Ereignisse: Nachrichten-Posts.
- Nebenwirkungen: Postet Discord-Nachrichten; setzt Audit-Posts bei Redaktionen (Fehler beim Audit werden verschluckt).
- Fehlerfälle: `HTTPError` → RuntimeError mit Status und Antworttext; Discord-2000-Zeichen-Limit durch Chunking adressiert (Cut bei letztem Newline < 1900, Fallback 1900).
- Sicherheitsrelevanz: Jeder ausgehende Text passiert das Policy-Gate; User-Agent `OpenClaw-HotelSim/1.0`; Token aus OpenClaw-Config (nicht im Repo).
- Geschäftslogik: Audit-Post nur bei tatsächlichen Redaktionen; dedupliziert Kanal-IDs per Name→ID-Map.
- Algorithmen: Zeilenbasiertes Chunking; lineares Backoff nicht vorhanden (nur Sleep 0,7 s).
- verwendete Datenmodelle: Discord-Message-Objekte, Kanal-Report-JSON.
- Abhängigkeiten: stdlib (urllib, json, time, pathlib), `hotel_sim.policy_gate`.
- Rust-Relevanz: Discord-Client (Auth, POST/GET, Chunking, Rate-Limit-Sleep) als async Rust-Service mit reqwest; Audit-Integration. Konkrete Entscheidungen in `rust-foundation.md`.

---

## scripts/drip_discord_incidents.py

- Zweck: „Drip"-Loop: postet bis zu `limit` (Default 3) noch nicht gepostete Incidents aus SQLite in die typzugeordneten Discord-Kanäle, dazu eine simulierte Staff-Followup-Antwort und bei escalated/urgent/high einen Escalation-Mirror nach `manager-escalations`; persistiert Zustand.
- Verantwortlichkeit: Autonome Discord-Befüllung (15-Minuten-Cron laut README).
- Eingaben: SQLite, Zustandsdatei `reports/discord-drip-state.json` (+ Seeded-Status als Startpunkt), Discord-Kanal-IDs.
- Ausgaben: Gesendete Nachrichten-IDs, `posted_total` (JSON), Discord-Posts.
- Datenfluss: Incident-Rows → Typ→Kanal-Mapping → `send` → Zustands-Update.
- Persistenz: `reports/discord-drip-state.json` (posted, cursor_created_at, sent_log).
- Zustände: posted-Set (dedupliziert über Läufe hinweg); Fallback-Kanal `nemo-lodge-lobby` wenn Kanal fehlt.
- APIs: `send`/`channel_ids` aus `discord_utils`.
- Ereignisse: Incident-Post, Staff-Followup, Escalation-Mirror.
- Nebenwirkungen: Discord-Nachrichten + Staff-Antworten (diese werden später vom Replicant-Updater als Memories gelesen — belegt durch `update_replicants_from_discord.py`).
- Fehlerfälle: Keine expliziten; fehlender Kanal wird durch Lobby-Fallback abgefangen.
- Sicherheitsrelevanz: Nachricht enthält Privacy-Hinweis (`⚠️ privacy-sensitive`), Guest-Visible-Text und Agenten-Instruktion „If sensitive, summarize only".
- Geschäftslogik: `TYPE_CHANNEL`-Mapping (16 Typen inkl. lost_item/cleaning_delay/dirty_room/overbooking/payment_dispute/noise_complaint); `staff_followup` liefert rollentypische Antworttexte (Ben/Annie/Kevin/Maya/Leo/Grace) je Typ/Severity.
- Algorithmen: Sequentielle Iteration über Incidents bis Limit; Set-basierte Deduplizierung.
- verwendete Datenmodelle: Incident-Rows, State-Dict, sent-Einträge.
- Abhängigkeiten: stdlib, `discord_utils`.
- Rust-Relevanz: Cron-artiger Discord-Drip mit Zustandspersistenz als async Task + State-Store. Konkrete Entscheidungen in `rust-foundation.md`.

---

## scripts/ensure_hotelsim_api.py

- Zweck: Startet die Retrieval-API im Hintergrund, falls `/health` nicht antwortet; pollt bis zu 20×0,25 s; schreibt PID.
- Verantwortlichkeit: Idempotenter API-Wächter (Alternative zum Makefile-Target).
- Eingaben: Keine (fester URL `http://127.0.0.1:8765/health`).
- Ausgaben: Konsolenmeldungen („already healthy"/„started"/Fehler).
- Datenfluss: Health-Check → ggf. Subprozess-Start (`api/server.py`, start_new_session) → Polling.
- Persistenz: `reports/hotelsim-api.pid`, `reports/hotelsim-api.log`.
- Zustände: Healthy/nicht healthy.
- APIs: `/health`.
- Ereignisse: API-Start.
- Nebenwirkungen: Startet Serverprozess; schreibt PID/Log.
- Fehlerfälle: Kein Healthy nach 20 Versuchen → `SystemExit` mit Meldung.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Idempotenz (nicht starten, wenn bereits healthy).
- Algorithmen: Polling mit Timeout.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: stdlib (subprocess, sys, time, urllib).
- Rust-Relevanz: Prozess-Orchestrierung + Health-Polling als Rust-Spawn/Poll. Rust-Relevanz: Keine (reines Util).

---

## scripts/export_event_stream.py

- Zweck: Exportiert messages JOIN incidents aus SQLite chronologisch (created_at, message_id) nach `data/messages/two_day_event_stream.jsonl` (JSON pro Zeile).
- Verantwortlichkeit: Materialisierung des 48h-Event-Streams.
- Eingaben: `data/hotel_sim.sqlite`.
- Ausgaben: JSONL-Datei (500 Zeilen).
- Datenfluss: SQLite → SELECT mit JOIN → JSONL-Writes.
- Persistenz: `data/messages/two_day_event_stream.jsonl`.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Überschreibt die Zieldatei.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Exportiert synthetische PII-Felder.
- Geschäftslogik: Anreicherung von Messages mit Incident-Feldern (type, severity, status, assigned_staff_id, room_id, reservation_id).
- Algorithmen: Sortierung nach created_at, message_id.
- verwendete Datenmodelle: Message-Rows + Incident-Join-Felder.
- Abhängigkeiten: stdlib.
- Rust-Relevanz: SQL-Join-Export als Rust-Query + JSONL-Writer. Rust-Relevanz: Keine (reines Util).

---

## scripts/nemotron_rag_bridge.py

- Zweck: RAG-Bridge: liest neue Fragen aus `#nemotron-rag`, ruft die lokale Retrieval-API (`/rag/query`), synthetisiert via NVIDIA NIM (OpenAI-kompatible Chat-Completions), redigiert via `policy_gate.gate_text`, postet Antwort nach `#nemotron-rag` und Audit-Zeile nach `#agent-audit-log`; persistiert beantwortete IDs.
- Verantwortlichkeit: Nemotron-Synthese + end-to-end RAG-Loop (2-Minuten-Cron laut README).
- Eingaben: Env `NVIDIA_NIM_API_KEY`, `NVIDIA_NIM_MODEL`, `NVIDIA_NIM_BASE` (Default `https://integrate.api.nvidia.com/v1`), `NVIDIA_NIM_DISABLE`, `HOTELSIM_API`/`HOTELSIM_PORT`; Discord-Kanal `nemotron-rag`; Retrieval-Dict.
- Ausgaben: Gated-Antworttext, `used_nim`, `model`, `redactions`, `nim_error`; Discord-Posts; State-Update.
- Datenfluss: Discord-Read → clean_question → `/rag/query` (+ Live-Summary bei Live-Intents) → `nemotron_synthesize` → `gate_text` → Post + Audit → State-Save.
- Persistenz: `reports/nemotron-rag-bridge-state.json` (`answered`, `sent_log`).
- Zustände: `answered`-Set (bereits beantwortete Nachrichten-IDs).
- APIs: NIM `POST /chat/completions`; lokale API `GET /rag/query`.
- Ereignisse: Fragen-Beantwortung, Audit-Zeilen, Fehler-Posts.
- Nebenwirkungen: Discord-Posts (Antwort, Audit, Fehler-Meldung).
- Fehlerfälle: NIM-Disable/Key fehlt/Key nicht latin-1 → deterministischer Zitaten-Fallback mit `error`-Feld; HTTPError → sofortiger Fallback mit Status; Timeout/URLError → 1 Retry (2 Versuche), dann Fallback; überraschende Responses → Fallback mit Body-Auszug; Bridge-Fehler pro Nachricht → Fehler-Post und Fortsetzen.
- Sicherheitsrelevanz: System-Prompt verbietet Erfindung von Gastnamen/Passwörtern/Telefon/E-Mails/Zahlungen/IDs/internal_notes; Antworten passieren zwingend das Gate; Bot-Antworten werden übersprungen (Author-Bot-Check).
- Geschäftslogik: Citation-Block auf 6 Zitate à 400 Zeichen begrenzt (Rate-Limit-Schonung); temperature 0.2, top_p 0.9, max_tokens 600, stream False; Antwortformat-Vorgabe (1-Zeile + ≤6 Bullets mit [n] + Route); Quellenliste wird der Synthese angehängt.
- Algorithmen: Retry-Schleife (2 Versuche) bei transienten Fehlern; Regex-Bereinigung von Mentions (`<@!?\d+>`); Fallback-Renderer.
- verwendete Datenmodelle: Retrieval-Dict (citations, recommended_route, guardrail, detected_intents), NIM-Response (choices[0].message.content), State-Dict.
- Abhängigkeiten: stdlib (urllib, json, re, os), `discord_utils`, `hotel_sim.live_ops.render_summary`, `hotel_sim.policy_gate.gate_text`.
- Rust-Relevanz: NIM-Client (Request-/Response-Aufbau, Retry, Fallback) ist Kern der Rust-NIM-Capability; Details in `nim.md`/`rust-foundation.md`.

---

## scripts/prepare_workspace_imports.py

- Zweck: Bereitet Google-Workspace-Importdateien vor: `users-google-admin-import.csv` (8 Staff, generierte 18-Zeichen-Passwörter) und `groups.json` (7 Gruppen); speichert Passwörter sicher unter `~/.openclaw/secure/hotel-sim/fake-staff-passwords-prepared.json` (chmod 0600).
- Verantwortlichkeit: Offline-Vorbereitung der Workspace-Provisionierung.
- Eingaben: Statische STAFF-/GROUPS-Definitionen.
- Ausgaben: CSV, groups.json, Passwort-JSON; Konsolen-JSON (Pfade).
- Datenfluss: Passwortgenerator → CSV/JSON → Secure-Speicher.
- Persistenz: `workspace-imports/*`, Secure-Datei.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Schreibt drei Dateien; chmod 0600 auf Passwortdatei.
- Fehlerfälle: Passwortgenerator looped bis Komplexitätskriterien erfüllt sind (lower+upper+digit).
- Sicherheitsrelevanz: Hohe Relevanz: Passwörter nur in Secure-Verzeichnis außerhalb des Repos; CSV enthält Passwörter (Import-Zweck).
- Geschäftslogik: Gruppen-Zuordnung identisch zu `provision_workspace.py`; Admin-CSV-Format (First/Last/Email/Password/OrgUnit/Change-Password).
- Algorithmen: `secrets`-Zufallsgenerator mit Komplexitätsprüfung.
- verwendete Datenmodelle: STAFF-Tupel, GROUPS-Dict, CSV-Zeilen.
- Abhängigkeiten: stdlib (csv, json, secrets, string, os).
- Rust-Relevanz: CSV/JSON-Generierung + secrets (rand_core::OsRng) als Rust-Tool. Rust-Relevanz: Keine (reines Util).

---

## scripts/provision_workspace.py

- Zweck: Provisioniert die Google-Workspace-Sandbox: erstellt 8 User (Admin-Directory), 7 Gruppen + Mitgliedschaften, 5 Shared Drives + Berechtigungen (reader für Policies, writer sonst) und Docs (Policy-Inhalte aus `data/policies/*.md`, synthetische Board-/Finanztexte); schreibt Provisioning-Report.
- Verantwortlichkeit: Google-Workspace-Integration (Admin/Directory, Drive, Docs).
- Eingaben: CLI `--skip-drive`, `--dry-run`, `--delegated-admin` (Default `jarvis-hotelsim@snapdesign.tw`); Credentials (OAuth-Token oder Service-Account-Delegation); SCOPES: admin.directory.user/group/group.member, drive, documents, spreadsheets.
- Ausgaben: `reports/workspace-provisioning.json` (users, groups, drives_docs), Konsolen-JSON.
- Datenfluss: Credentials → API-Services → exists-Checks → create/insert → Report.
- Persistenz: Token (chmod 0600), `reports/workspace-provisioning.json`, `fake-staff-passwords.json` (chmod 0600).
- Zustände: `created`/`exists`/`added`/`skipped:<code>` je Ressource; Workspace `provisioned` wird in `replicants.load_workspace` aus dem Report abgeleitet.
- APIs: Google Admin Directory (users.get/insert, groups.get/insert, members.insert), Drive (drives.list/create, permissions.create, files.list/create), Docs (documents.batchUpdate).
- Ereignisse: User-/Gruppen-/Drive-/Doc-Erzeugung.
- Nebenwirkungen: Echte Änderungen im Google-Workspace-Sandbox (nur bei `--dry-run` unterdrückt).
- Fehlerfälle: Fehlende Client-Secret/Service-Account-Key → SystemExit; 404 beim Get → not_found-Handling; 400/403/409 bei Mitgliedschaften/Berechtigungen → `exists`/`skipped`; Doc-Duplikate werden per Namens-Query vermieden.
- Sicherheitsrelevanz: Rollenbasierte Drive-Freigaben (Policies reader, übrige writer); Passwörter chmod 0600; Token chmod 0600; delegierte Admin-Identität; temporärer Admin gemäß Plan.
- Geschäftslogik: DRIVES→Gruppen-Matrix (HotelSim Policies: alle; Finance: nur managers+finance), DOCS-Inhalte, managers-Gruppe umfasst Leads+Finance.
- Algorithmen: Pagination (drives.list nextPageToken), exists-Check vor create, Namens-Dedup für Docs.
- verwendete Datenmodelle: STAFF/GROUPS/DRIVES/DOCS-Strukturen, API-Response-Dicts, Report-Struktur.
- Abhängigkeiten: `google-api-python-client`, `google-auth`, `google-auth-oauthlib`, `google-auth-httplib2` (aus `requirements.txt`), stdlib.
- Rust-Relevanz: Google-API-Client (OAuth/Service-Account, Admin/Drive/Docs) als async Service; RBAC-Zuordnungen als Konfigurationsdaten. Konkrete Entscheidungen in `rust-foundation.md`.

---

## scripts/seed_discord_incidents.py

- Zweck: Seeding-Pendant zum Drip: postet bis zu 16 offene/in-progress/escalated Incidents (Sortierung nach Severity-Rang dann created_at) in typzugeordnete Kanäle inkl. Escalation-Mirror bei urgent/high/escalated; persistiert Zustand.
- Verantwortlichkeit: Initiale Befüllung der Discord-Kanäle.
- Eingaben: SQLite, `reports/discord-seeded-incidents.json`, Kanal-IDs.
- Ausgaben: Gesendete IDs, `posted_total`; Discord-Posts.
- Datenfluss: SELECT (Status-Filter, Severity-ORDER) → Typ→Kanal → send → State.
- Persistenz: `reports/discord-seeded-incidents.json`.
- Zustände: posted-Set.
- APIs: `send`/`channel_ids`.
- Ereignisse: Seeding-Posts + Escalation-Mirrors.
- Nebenwirkungen: Discord-Posts.
- Fehlerfälle: Fehlender Kanal → Lobby-Fallback.
- Sicherheitsrelevanz: Nachrichten enthalten Routing-Instruktion „Do not paste payment/ID/contact data into public channels" und Privacy-Marker.
- Geschäftslogik: `ESCALATE_SEVERITIES = {urgent, high}`; Status-Filter open/in_progress/escalated; LIMIT 16.
- Algorithmen: SQL-ORDER BY CASE severity.
- verwendete Datenmodelle: Incident-Rows, State-Dict.
- Abhängigkeiten: stdlib, `discord_utils`.
- Rust-Relevanz: Analog zu Drip; Seeding als Initialisierungs-Task. Rust-Relevanz: Keine (Skript).

---

## scripts/update_replicants_from_discord.py

- Zweck: Replicant-Updater: liest bis zu 25 Nachrichten je Kanal, überspringt Bot-Nachrichten (außer Staff-Prefix `maya:`/`leo:`/…), filtert „memory-worthy" Nachrichten (≥30 Zeichen, keine Live-Event-/Escalation-Vorlagen, Signal-Wort-Treffer), klassifiziert Person/Rolle (Staff-Prefix oder Kanal-Zuordnung) und hängt Memories an `discord_memories.jsonl`; Audit-Post bei neuen Memories.
- Verantwortlichkeit: Langfristige Wissensextraktion aus Discord (5-Minuten-Cron laut README).
- Eingaben: Discord-Kanäle (front-desk, housekeeping, maintenance, reservations-revenue, guest-experience, finance-admin, manager-escalations), `reports/replicant-updater-state.json`.
- Ausgaben: Neue Memory-Einträge (JSONL-Append), Konsolen-Zählung, Audit-Post.
- Datenfluss: read → seen-Dedup → Filter → classify → make_memory (SHA1-ID) → append → State.
- Persistenz: `data/replicants/discord_memories.jsonl` (append-only), `reports/replicant-updater-state.json`.
- Zustände: `seen` (verarbeitete Nachrichten-IDs), `updates`-Historie.
- APIs: `read`/`send` aus `discord_utils`.
- Ereignisse: Memory-Erzeugung, Audit-Post.
- Nebenwirkungen: JSONL-Append; Audit-Post ohne Gate (audit=False).
- Fehlerfälle: Fehlende Kanäle werden übersprungen; ungültige Nachrichten ohne content übersprungen.
- Sicherheitsrelevanz: Bot-Nachrichten ohne Staff-Prefix werden ignoriert (verhindert Self-Learning aus eigenen Antworten); Text wird als Staff-Wissen gespeichert (später Gate-geschützt beim Versand).
- Geschäftslogik: `CHANNEL_ROLE` (Kanal→Rolle/Person), `STAFF_NAME_ROLE` (Namensprefix→Rolle/Person), `SIGNAL_WORDS` (25 Begriffe: policy, privacy, guest-safe, route, routing, escalat, refund, folio, payment, repair, access, clean, room board, dispatch, vip, late checkout, do not post, private, sensitive, confirm, assign, triage); Memory-Kind `discord_staff_memory`; Quelle `discord:<channel>`.
- Algorithmen: SHA1-Hash (`channel:message_id`, 12 Hex-Zeichen) für Memory-ID; Prefix-Regex-Zuordnung; Zeitstempel aus Discord.
- verwendete Datenmodelle: Memory-Dict (siehe JSONL), State-Dict.
- Abhängigkeiten: stdlib, `discord_utils`.
- Rust-Relevanz: Wissensextraktion (Klassifikation, Dedup, append-only Storage) als Rust-Task mit Hash-basierter Deduplizierung. Konkrete Entscheidungen in `rust-foundation.md`.

---

## tests/test_live_ops.py

- Zweck: Testet Owner-Mapping (billing→Annie, noise→Kevin, safety→Maya, access→Leo), Live-Summary-Rendering (Header-Phrasen) und Billing-Routing über `retrieve` (Finance/Admin in recommended_route).
- Verantwortlichkeit: Regressionssicherung für `hotel_sim.live_ops`/`replicants`.
- Eingaben: Modulimporte; erwartet vorhandene SQLite- und Memory-Daten.
- Ausgaben: Pytest-Assertions bzw. „ok"-Ausgabe im Direktaufruf.
- Datenfluss: Kein externer Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine expliziten.
- Sicherheitsrelevanz: Indirekt — testet Routing-Zuordnungen, die PII-Schutz unterstützen.
- Geschäftslogik: Verifiziert Kern-Mappings des Incident-Routings.
- Algorithmen: Keine.
- verwendete Datenmodelle: TYPE_OWNER, render_summary-Ausgabe, retrieve-Ergebnis.
- Abhängigkeiten: `hotel_sim.live_ops`, `hotel_sim.replicants` (retrieve), Datenbestand.
- Rust-Relevanz: Vorlage für Rust-Unit-Tests der Owner-Mappings und Routing-Retrieval (Details in `tests.md`).

---

## tests/test_sim.py

- Zweck: Integrationstests: Tabellen-Zählungen (250/2500/3150/8/500/500), Wiederholungsbuchungs-Verteilung (300×2, 100×3, 50×4), Existenz privacy-sensitiver Incidents und beider Sprachen, API-Health + Staff-Endpunkt (Start des Servers als Subprozess, Terminierung im finally).
- Verantwortlichkeit: Validierung der generierten Simulation und der API-Startfähigkeit.
- Eingaben: Erwartet `data/hotel_sim.sqlite`; startet `api/server.py` temporär auf Port 8765.
- Ausgaben: Assertions; „ok" im Direktaufruf.
- Datenfluss: SQLite → COUNT-Abfragen; Subprozess-Server → HTTP.
- Persistenz: Keine (temporärer Serverprozess).
- Zustände: Keine.
- APIs: `/health`, `/staff`.
- Ereignisse: Server-Start/Stopp.
- Nebenwirkungen: Startet/terminiert einen Serverprozess.
- Fehlerfälle: Port 8765 belegt → Testfehler (kein Port-Override im Test).
- Sicherheitsrelevanz: Prüft, dass sensitive Incidents und zweisprachige Daten existieren (Privacy-Testbasis).
- Geschäftslogik: Verifiziert die Generierungs-Invarianten (exakte Zählwerte aus `generate.py`).
- Algorithmen: Aggregations-SQL (GROUP BY booker_id HAVING c=2/3/4).
- verwendete Datenmodelle: SQLite-Tabellen.
- Abhängigkeiten: stdlib (sqlite3, subprocess, sys, time, urllib, json, pathlib).
- Rust-Relevanz: Vorlage für Rust-Integrationstests der Datenbank und des HTTP-Servers (Details in `tests.md`).

---

## ui/index.html

- Zweck: „HotelSim Control Room"-Dashboard: Metrik-Karten (Rooms/Bookers/Reservations/Open Incidents), Live-Incident-Queue, Replicant-RAG-Abfragefeld, Privacy-/Halluzinations-Guardrail-Panel, Policy-Retrieval mit Anzeige, Staff-Karten; lädt per `Promise.all` sechs API-Endpunkte, Auto-Refresh 15 s.
- Verantwortlichkeit: Visualisierung des Simulations- und Agent-Zustands.
- Eingaben: API-Endpunkte `/health`, `/dashboard/summary`, `/incidents/open?limit=12`, `/staff`, `/policies`, `/replicants/summary`, `/rag/query?q=…`, `/policies/<name>`.
- Ausgaben: DOM-Rendering; keine Datei-Ausgaben.
- Datenfluss: fetch → JSON → DOM-Templates (kein Framework, Vanilla-JS).
- Persistenz: Keine.
- Zustände: „API live"/„API offline" (error-Klasse bei Fehlern); Severity-Klassen urgent/high.
- APIs: Konsumiert die lokale Retrieval-API.
- Ereignisse: Klick „Ask" (RAG), Policy-Klick, 15-s-Refresh.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlende API → Fehlertext in Health-Pill und Incident-Panel (kein Fake-Daten-Fallback, belegt durch catch-Block).
- Sicherheitsrelevanz: Zeigt sensitive-Marker (PII/clean) und Guardrail-Zusammenfassung; keine Auth-Schicht.
- Geschäftslogik: RAG-Ergebnisdarstellung (Intents, Route-Tags, Guardrail, Draft, Citations mit score), Staff-Karten (Name, zh_name, Rolle, Schicht, Clearance).
- Algorithmen: Keine.
- verwendete Datenmodelle: Summary-, Incident-, Staff-, Policy-, Replicant-Summary-, Retrieval-Dicts.
- Abhängigkeiten: Lokale API (`api/server.py`); Browser.
- Rust-Relevanz: UI-Anforderungen (Endpunkte, Felder, Fehlerverhalten) sind Anforderungsquelle für ein Rust-basiertes Dashboard (z.B. statische Assets hinter axum + JSON-API). Rust-Relevanz: Keine (HTML/JS-Asset).

---

## workspace-imports/groups.json

- Zweck: Gruppen→Mitglieder-Zuordnung der Workspace-Sandbox (7 Gruppen: managers, frontdesk, housekeeping, maintenance, reservations, finance, guest-experience), erzeugt durch `scripts/prepare_workspace_imports.py`.
- Verantwortlichkeit: Import-Daten für Google-Gruppen.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Generator → JSON.
- Persistenz: JSON-Datei.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Enthält E-Mail-Adressen der Staff-Sandbox.
- Geschäftslogik: Spiegel der RBAC-Gruppen aus `provision_workspace.py`.
- Algorithmen: Keine.
- verwendete Datenmodelle: Dict gruppe@domain → [member@domain].
- Abhängigkeiten: `scripts/prepare_workspace_imports.py`.
- Rust-Relevanz: RBAC-Gruppenmodell als Datenquelle. Rust-Relevanz: Keine (Daten).

---

## workspace-imports/users-google-admin-import.csv

- Zweck: Google-Admin-Import-CSV für 8 Staff-User mit generierten 18-Zeichen-Passwörtern, Org Unit `/`, „Change Password at Next Sign-In" FALSE; erzeugt durch `scripts/prepare_workspace_imports.py`.
- Verantwortlichkeit: Import-Daten für Google-Workspace-User.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Generator → CSV.
- Persistenz: CSV-Datei.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Enthält Klartext-Passwörter (synthetisch); die sichere Kopie liegt unter `~/.openclaw/secure/hotel-sim/` (chmod 0600); die CSV wird von `.gitignore` nicht abgedeckt und ist eingecheckt (belegt durch Dateiliste) — Kennwortmaterial ist synthetisch.
- Geschäftslogik: Vollständige User-Liste der 8 Staff-Rollen.
- Algorithmen: Keine.
- verwendete Datenmodelle: Admin-CSV-Zeilen.
- Abhängigkeiten: `scripts/prepare_workspace_imports.py`.
- Rust-Relevanz: CSV-Import-Format als Datenquelle. Rust-Relevanz: Keine (Daten).
