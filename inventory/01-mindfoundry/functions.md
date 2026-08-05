# MindFoundry — Funktionsanalyse

Beschrieben werden Fähigkeiten, nicht Code. Belege in Klammern.

## hotel_sim/policy_gate.py

### gate_text(text, destination) — Policy-Gate für ausgehenden Text

- Warum existiert sie: Sicherstellen, dass kein PII den Agenten Richtung Discord verlässt (belegt durch Docstring/Semantik und `README.md`).
- Welches Problem löst sie: Erzwingt die NemoClaw-Inference-Regel (Redaktion) auf Ebene einzelner Texte mit rollenabhängigen Freigaben.
- Welche Regeln implementiert sie: Klassenbasierte Redaktion (password, credit_card, taiwan_phone, email, passport_or_id, internal_note); Rollen-Matrix `public_discord`/`finance_private`/`manager_private`; Passwörter sind in keiner Rolle erlaubt; Staff-E-Mails (8-Adressen-Allowlist) bleiben in `public_discord` sichtbar; Entscheidung `allow`/`allow_with_redactions`.
- Welche Daten verändert sie: Ersetzt Treffer durch `[KIND REDACTED]`; hängt `GateResult`-JSON an `reports/policy-gate-events.jsonl` (Audit).
- Welche Zustände entstehen: Entscheidungszustand je Text; Redaktionsliste (Typ + Sample der ersten 24 Zeichen).
- Welche Algorithmen werden verwendet: Sechs Regex-Substitutionen (case-insensitive, Wortgrenzen; chinesische Schlüsselwörter 身分證/護照); E-Mail-Matching erfordert TLD; Kreditkarten-Regex verlangt Trennzeichen oder echte Kartengruppierung, um Discord-Snowflakes nicht zu treffen.
- Welche Fehler behandelt sie: Unbekannte Destination → Fallback `public_discord`; Verzeichnis-Erstellung für Audit-Datei.
- Welche Seiteneffekte besitzt sie: Audit-JSONL-Anhang (pro Gate-Aufruf eine Zeile).

### gate_payload(payload, destination) — Policy-Gate für strukturierte Payloads

- Warum existiert sie: Auch verschachtelte JSON-Strukturen (z.B. Antwort-Cards) müssen redigiert werden, bevor sie ausgehen.
- Welches Problem löst sie: Redaktion ohne manuelles Traversieren beliebig tiefer Listen/Dicts.
- Welche Regeln implementiert sie: Identische Klassen/Regeln wie `gate_text`, angewendet rekursiv auf jeden String.
- Welche Daten verändert sie: Tiefenkopie der Payload mit redigierten Strings (Original bleibt unverändert).
- Welche Zustände entstehen: Redigierte Payload.
- Welche Algorithmen werden verwendet: Rekursiver Walk über dict/list/str.
- Welche Fehler behandelt sie: Nicht-String-Skalare bleiben unverändert.
- Welche Seiteneffekte besitzt sie: Audit-Zeilen je String-Redaktion.

### _redact_match(kind, text, dest) — Klassen-Redaktionsengine

- Warum existiert sie: Gemeinsame Substitutionslogik für alle Klassen mit Rollen-Freigabe-Prüfung.
- Welches Problem löst sie: Erlaubt Klassen durchzulassen, wenn die Ziel-Destination die Freigabe besitzt.
- Welche Regeln implementiert sie: `allow_staff_email`, `allow_guest_email`, `allow_phone`, `allow_payment`, `allow_password`, `allow_internal_notes` je Rolle; Sample-Erfassung für Audit.
- Welche Daten verändert sie: Text (Stringersetzung), Redaktionsliste.
- Welche Zustände entstehen: Redaktionen-Liste.
- Welche Algorithmen werden verwendet: Regex-Substitution mit Closure `repl` je Klasse; E-Mail-Normalisierung (lower, Strip von Interpunktion).
- Welche Fehler behandelt sie: Keine.
- Welche Seiteneffekte besitzt sie: Keine (Audit erfolgt in `gate_text`).

## hotel_sim/replicants.py

### build_replicants() — Replicant-Konstruktion

- Warum existiert sie: Wandelt Staff + Incidents + Discord-Memories in pro-Person-Wissensprofile um.
- Welches Problem löst sie: Strukturierte, durchsuchbare „zweite Gehirne" je Teammitglied mit Quelle und Kontext.
- Welche Regeln implementiert sie: Memory-Arten: profile (SQLite-Staff), behavior (Arbeitsstil/Schicht/Clearance), workspace (E-Mail/Gruppe), experience (Incident-Mix), case (bis 4 letzte Fälle mit Sensitivitäts-Hinweis), discord_staff_memory (bis 8 je Rolle); Rollen→E-Mail-/Gruppen-Mapping.
- Welche Daten verändert sie: Keine (nur Lesen); erzeugt Replicant-Strukturen.
- Welche Zustände entstehen: Je Replicant: memory_count, memories-Liste.
- Welche Algorithmen werden verwendet: SQL (Incidents je Staff, limitiert, absteigend nach created_at), Typ-Zählung, Slice-Limits.
- Welche Fehler behandelt sie: Fehlende Dateien (exists-Checks), ungültige JSONL-Zeilen (übersprungen).
- Welche Seiteneffekte besitzt sie: Keine.

### retrieve(q, limit) — Deterministisches RAG-Retrieval

- Warum existiert sie: Liefert zitiertes Material (Replicant-Memories, Policy-Snippets, Live-Ops) für die Synthese, ohne LLM-Abhängigkeit im Retrieval.
- Welches Problem löst sie: Grounded Retrieval mit Routing-Vorschlag und Guardrail.
- Welche Regeln implementiert sie: Intent-Klassifikation (10 Typen via EN/ZH-Keywords), VIP-Intent erzwingt GM-Routing, Live-Trigger injiziert Live-Ops-Summary (Score 999), Policy-Bonus für privacy/refund/routing/escalat-Begriffe, Guardrail- und Draft-Text.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Score-Liste, Citations (Limit), recommended_route (Rolle/Person/E-Mail/Begründung).
- Welche Algorithmen werden verwendet: `tokenize` (EN+CJK-Token), `score_text` (2 Punkte exakter Treffer + 1 Punkt Präfix-Treffer für Terme >3 Zeichen), Intent-Bonus (+3 Memory / +4 Policy), absteigende Sortierung.
- Welche Fehler behandelt sie: Leere Queries → leere Citations möglich; keine Exception-Handling-Notwendigkeit.
- Welche Seiteneffekte besitzt sie: Keine.

### tokenize(q) / score_text(terms, text) / classify_query(q)

- Warum existieren sie: Basismechanik des deterministischen Retrievals.
- Welches Problem lösen sie: Sprachübergreifende (EN/ZH) Term-Erkennung und Ähnlichkeitsheuristik; Intent-Zuordnung.
- Welche Regeln implementieren sie: Wort-Tokenisierung inkl. CJK-Zeichenblock; Score-Regeln wie oben; Fallback-Klassifikation (`reservation_change` bei checkout/check, sonst `vip`).
- Welche Daten verändern sie: Keine.
- Welche Zustände entstehen: Term-Set, Score, Intent-Liste.
- Welche Algorithmen werden verwendet: Regex-Findall, Substring-Scoring, Keyword-Substring-Matching.
- Welche Fehler behandeln sie: Keine.
- Welche Seiteneffekte besitzen sie: Keine.

### policy_snippets() / discord_memories() / load_workspace()

- Warum existieren sie: Einheitliche Zugriffs-Pfade auf Policy-Docs, Discord-Memories und Workspace-Provisionierungsstatus.
- Welches Problem lösen sie: Retrieval-fähige Aufbereitung externer Wissensquellen; Statusanzeige der Google-Sandbox.
- Welche Regeln implementieren sie: Policy-Absätze (≤900 Zeichen) mit Quelldatei-Name; JSONL-zeilenweises Lesen; Provisioned-Zählung (users/groups/drives mit Status created/exists).
- Welche Daten verändern sie: Keine.
- Welche Zustände entstehen: Snippet-/Memory-/Workspace-Status-Dicts.
- Welche Algorithmen werden verwendet: Markdown-Absatz-Split, JSON-Parsing, Summierung.
- Welche Fehler behandeln sie: Fehlende Dateien, JSONDecodeError-Zeilen.
- Welche Seiteneffekte besitzen sie: Keine.

### summary()

- Warum existiert sie: Dashboard-/Demo-Zusammenfassung (Replicant-Zahl, Memory-Zahl, Workspace-Status, Beispiel-Query).
- Welches Problem löst sie: Schneller Gesamtüberblick für UI (`/replicants/summary`) und CLI.
- Welche Regeln implementiert sie: Beispielfrage „late checkout request from a VIP guest" mit Limit 5.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Summary-Dict.
- Welche Algorithmen werden verwendet: Delegation an build_replicants/retrieve.
- Welche Fehler behandelt sie: Keine.
- Welche Seiteneffekte besitzt sie: Keine.

## hotel_sim/live_ops.py

### live_summary(limit) / render_summary()

- Warum existieren sie: Liefern den „Was passiert gerade"-Signalblock für RAG-Antworten und Tests.
- Welches Problem lösen sie: Aggregation von geposteten Incidents, Severity-/Typ-Verteilungen, Owner-Queues, Urgent-Queue und Memory-Signalen aus SQLite + Zustandsdateien.
- Welche Regeln implementieren sie: Incident-Auswahl entweder über gepostete IDs oder über offene Status; `TYPE_OWNER`-Mapping; Urgent-Filter (severity urgent/high oder escalated, max 6); Limits (12/5/5); Privacy-Hinweis im Text.
- Welche Daten verändern sie: Keine.
- Welche Zustände entstehen: Summary-Dict, gerenderter Markdown-Text.
- Welche Algorithmen werden verwendet: Set-Union (posted IDs), Counter, defaultdict-Queues, sortierte Ausgabe.
- Welche Fehler behandeln sie: Fehlende Zustands-/Memory-Dateien, JSON-Fehler (Exception-Silencing).
- Welche Seiteneffekte besitzen sie: Keine.

### posted_ids() / recent_memories()

- Warum existieren sie: Zustands- und Memory-Zugriff.
- Welches Problem lösen sie: Deduplizierung der geposteten Incidents; letzte Memory-Signale.
- Welche Regeln implementieren sie: Liest seeded + drip State (`posted`-Listen); letzte N Memories (Default 8).
- Welche Daten verändern sie: Keine.
- Welche Zustände entstehen: ID-Set, Memory-Liste.
- Welche Algorithmen werden verwendet: JSON-Parsing, Slice.
- Welche Fehler behandeln sie: Fehlende Dateien/JSON-Fehler.
- Welche Seiteneffekte besitzen sie: Keine.

## hotel_sim/evaluate.py

### DeterministicHotelAgent.decide(incident, messages, policies)

- Warum existiert sie: Validierungs-Baseline ohne LLM, um den Scoring-Harness zu testen (belegt durch Klassen-Docstring).
- Welches Problem löst sie: Reproduzierbare Referenz für Routing/Policy/Privacy/Halluzination.
- Welche Regeln implementiert sie: `EXPECTED_ROUTE` (Typ→Staff-ID), Policy-Zitate je Typ (`POLICY_KEYWORDS`), Eskalation bei billing/safety/urgent, interne Notiz-Redaktion bei `contains_sensitive`, Sprach-/Typ-abhängige Gast-Antworten (en/zh-TW, billing/safety-Spezialtexte).
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Decision-Dict.
- Welche Algorithmen werden verwendet: Typ-Lookup, Policy-Filter, Sprach-Lookup.
- Welche Fehler behandelt sie: Leere Messages → Default-Sprache en.
- Welche Seiteneffekte besitzt sie: Keine.

### score(decision, incident)

- Warum existiert sie: Bewertet eine Agenten-Entscheidung gegen vier Kriterien.
- Welches Problem löst sie: Quantifizierung der Baseline-Metriken.
- Welche Regeln implementiert sie: Routing ok = Staff-ID gleich erwartet; Policy ok = mind. ein Zitat; Privacy ok = keine Sensitivitäts-Muster-Treffer (nur bei contains_sensitive); Halluzination = kein Refund-Begriff ohne `refunds.md`-Zitat.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Score-Dict (4 Bool-Kriterien + Summe).
- Welche Algorithmen werden verwendet: JSON-Serialisierung des Decisions für Regex-Suche; Regex-Musterliste; Halluzinations-Regex (refund|compensation|退款|補償).
- Welche Fehler behandelt sie: Leere Citations → policy_ok False.
- Welche Seiteneffekte besitzt sie: Keine.

### evaluate(limit, incident_type)

- Warum existiert sie: Führt die Evaluation über Incidents aus und aggregiert.
- Welches Problem löst sie: Gesamtraten + Einzelergebnisse für Berichte.
- Welche Regeln implementiert sie: Optionaler Typ-Filter; Chronologie-Limit; Aggregation mit Division-Schutz bei leerer Menge.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Summary-Dict (5 Raten + avg_score_4), Results-Liste.
- Welche Algorithmen werden verwendet: SQL-Limit/Order, Summen-/Ratenbildung.
- Welche Fehler behandelt sie: Leere Ergebnismenge.
- Welche Seiteneffekte besitzt sie: Schreibt Report-JSON bei `--out`.

## hotel_sim/generate.py

### generate(out, seed)

- Warum existiert sie: Deterministische Erzeugung der kompletten Simulation.
- Welches Problem löst sie: Reproduzierbare Datenbasis (Seed 42) für Retrieval, Evaluation und Demo.
- Welche Regeln implementiert sie: Schema-DDL (7 Tabellen, Indizes, FK), Skalen (250/2500/3150/8/500), Wiederholungsverteilung, gewichtete Zufallswahlen (Room-Typen, Sprachen 55/45, Loyalty, Severity, Status, Quellen, Nächte), Staff-Definitionen (Rollen, Schichten, Clearance), ROUTE (Typ→Staff), sensitive-Flags (billing oder 8 %), zweisprachige Vorlagentexte, 48h-Zeitfenster, Audit-Log-Zeilen.
- Welche Daten verändert sie: Schreibt/überschreibt die SQLite-Datei.
- Welche Zustände entstehen: Kompletter Datenbestand (Tabellen).
- Welche Algorithmen werden verwendet: `pick_weighted` (kumulative Gewichte), `random.choices`, gestaffelte Check-in-Daten (10–45 Tage × Index), Minuten-Random im 48h-Fenster.
- Welche Fehler behandelt sie: Assertion der Reservierungsverteilung; Fallback des Weighted-Pick.
- Welche Seiteneffekte besitzt sie: DROP+CREATE aller Tabellen; Datei-Schreiben.

### schema(con) / connect(db)

- Warum existieren sie: Schema-Aufbau und Verbindung mit FK-Aktivierung.
- Welches Problem lösen sie: Konsistente Tabellenstruktur und referenzielle Integrität.
- Welche Regeln implementieren sie: `PRAGMA foreign_keys=ON`; Indizes auf booker_id, incidents(status,severity), messages(created_at).
- Welche Daten verändern sie: Datenbank-Schema.
- Welche Zustände entstehen: Tabellen/Indizes.
- Welche Algorithmen werden verwendet: Executescript.
- Welche Fehler behandelt sie: Keine.
- Welche Seiteneffekte besitzt sie: DROP bestehender Tabellen.

## api/server.py

### Handler.do_GET / send_json / send_file / log_message

- Warum existieren sie: HTTP-Dispatch für alle Retrieval-Endpunkte.
- Welches Problem lösen sie: JSON-REST-Oberfläche für RAG, Dashboard, Datenabfragen und UI.
- Welche Regeln implementieren sie: Endpunkt-Semantik (siehe files.md); JSON UTF-8; UI-Datei bei `/`/`/dashboard`; 400/404/500-Fehlerzustände; Logging deaktiviert.
- Welche Daten verändern sie: Keine (read-only).
- Welche Zustände entstehen: HTTP-Responses.
- Welche Algorithmen werden verwendet: URL-Parsing, Query-Parsing, SQL-Lookups, Join für Reservierungs-Suche.
- Welche Fehler behandelt sie: Unbekannter Endpunkt, fehlender Query, fehlende Ressourcen, generische Exceptions.
- Welche Seiteneffekte besitzt sie: Keine.

## scripts/nemotron_rag_bridge.py

### nemotron_synthesize(question, retrieval)

- Warum existiert sie: Synthetisiert die zitierte Antwort über NVIDIA NIM (Nemotron) oder deterministischen Fallback.
- Welches Problem löst sie: Grounded, hallucinations-geschützte Antwort über Zitate; Graceful Degradation ohne Key/Netz.
- Welche Regeln implementiert sie: System-Prompt (nur Zitate, kein Erfinden von PII, [n]-Zitate, 4–8 Bullets); User-Prompt mit intents/route/guardrail/citations; Payload temperature 0.2, top_p 0.9, max_tokens 600, stream False; Key latin-1-Prüfung; Citation-Block (6×400 Zeichen); Antwort-Format; Quellenliste im Ergebnis.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Ergebnis-Dict (text, model, used_nim, error).
- Welche Algorithmen werden verwendet: HTTP POST (urllib), 1 Retry bei Timeout/URLError, Fallback-Renderer, JSON-Parsing.
- Welche Fehler behandelt sie: NIM_DISABLE, fehlender Key, nicht-latin-1-Key, HTTPError (mit Body-Auszug), Timeout/URLError (Retry), unerwartete Response-Struktur.
- Welche Seiteneffekte besitzt sie: Keine (reine Funktion).

### answer(q) / main()

- Warum existieren sie: End-to-End-Frageverarbeitung und Discord-Watcher-Loop.
- Welches Problem lösen sie: Neue Fragen aus `#nemotron-rag` beantworten, gaten, posten, auditieren, deduplizieren.
- Welche Regeln implementieren sie: `clean_question` (Mentions entfernen, Fallback-Frage), Live-Summary-Injektion bei Live-Intents, Gate mit `public_discord`, Bot-Nachrichten ignorieren, bereits beantwortete IDs überspringen, Audit-Zeile (NIM-Flag, Modell, Redaktionszahl, Fehler, Query-Auszug), Fehler-Post ohne Abbruch.
- Welche Daten verändert sie: State-Datei `nemotron-rag-bridge-state.json` (answered, sent_log).
- Welche Zustände entstehen: `answered`-Set, Antwort-/Audit-/Fehler-IDs.
- Welche Algorithmen werden verwendet: Reversed-Iteration über gelesene Nachrichten, Set-Dedup, Sortierung.
- Welche Fehler behandelt sie: Per-Nachrichten-Exceptions → Fehler-Post; Antwort-/Audit-Posts.
- Welche Seiteneffekte besitzt sie: Discord-Posts, State-Save.

## scripts/discord_utils.py

### send / read / request / channel_ids / bot_token

- Warum existieren sie: Einheitliche, gegatete Discord-Schnittstelle.
- Welches Problem lösen sie: Token-Handling, Kanal-Mapping, Redaktion, Chunking, Rate-Limit-Einhaltung.
- Welche Regeln implementieren sie: `gate_text` vor jedem Post; 1900-Zeichen-Chunking (Newline-Cut, Fallback 1900); 0,7 s Sleep zwischen Chunks; Audit-Post bei Redaktionen (nicht an den Zielkanal selbst); User-Agent `OpenClaw-HotelSim/1.0`; Token aus OpenClaw-Config.
- Welche Daten verändert sie: Discord-Server (Nachrichten); keine lokalen Dateien.
- Welche Zustände entstehen: Antworten (JSON) je Nachricht.
- Welche Algorithmen werden verwendet: Zeilen-Chunking, JSON-Encoding.
- Welche Fehler behandelt sie: HTTPError → RuntimeError mit Detail; Audit-Fehler werden verschluckt; fehlender Audit-Kanal übersprungen.
- Welche Seiteneffekte besitzt sie: Discord-Posts inkl. Audit-Kanal.

## scripts/update_replicants_from_discord.py

### main / classify / is_memory_worthy / make_memory / append_new

- Warum existieren sie: Autonome Wissensextraktion aus Discord.
- Welches Problem lösen sie: Konvertieren von Staff-Nachrichten in strukturierte Memories.
- Welche Regeln implementieren sie: Kanal→Rolle-Mapping; Staff-Prefix-Erkennung (maya:/leo:/… EN/ZH-Doppelpunkt); Bot-Filter (außer Staff-Prefix); Memory-Würdigkeit (≥30 Zeichen, keine Live-Event-/Escalation-Vorlagen, Signalwort-Treffer); SHA1-Memory-ID; Audit-Post je Rolle mit Zählung.
- Welche Daten verändert sie: `discord_memories.jsonl` (Append), State-Datei (`seen`, `updates`).
- Welche Zustände entstehen: `seen`-Set, neue Memory-Liste.
- Welche Algorithmen werden verwendet: Prefix-Regex, Signalwort-Regex (25 Begriffe), SHA1-Hash (12 Hex), Channel-Loop mit reversed-Read (25 je Kanal).
- Welche Fehler behandelt sie: Fehlende Kanäle übersprungen; leere Contents.
- Welche Seiteneffekte besitzt sie: JSONL-Append, Audit-Post (ohne Gate, audit=False).

## scripts/drip_discord_incidents.py / seed_discord_incidents.py

### main / incident_message / staff_followup / fmt

- Warum existieren sie: Befüllen der Discord-Kanäle mit simulierten Incidents (Drip laufend, Seed initial).
- Welches Problem lösen sie: Lebendige Ops-Oberfläche; Material für Replicant-Updater.
- Welche Regeln implementieren sie: Typ→Kanal-Mapping; Dedup über posted-IDs; Escalation-Mirror bei urgent/high/escalated; Staff-Followup-Texte je Typ/Severity (Ben/Annie/Kevin/Maya/Leo/Grace); Privacy-Marker und „do not expose private fields"-Instruktionen; Fallback-Kanal `nemo-lodge-lobby`; Limits (Drip 3, Seed 16; Seed-Sortierung nach Severity-Rang).
- Welche Daten verändert sie: State-Dateien (posted, sent_log, cursor_created_at), Discord-Nachrichten.
- Welche Zustände entstehen: posted-Set, sent-Log.
- Welche Algorithmen werden verwendet: SQL-Selektionen (Limit, Status-Filter, CASE-ORDER), Set-Union, Schleife bis Limit.
- Welche Fehler behandelt sie: Fehlende Kanäle → Lobby-Fallback; fehlende State-Dateien → Initialisierung (Seed-Datei als Startpunkt beim Drip).
- Welche Seiteneffekte besitzt sie: Discord-Posts (Incident + Followup + Mirror), State-Saves.

## scripts/demo_walkthrough.py

### main / step_health / step_rag / step_eval

- Warum existieren sie: Ein-Kommando-Nachweis des Stacks.
- Welches Problem lösen sie: Juroren-freundliche Verifikation ohne Discord.
- Welche Regeln implementieren sie: Reihenfolge Health → RAG → Adversarial-Probe → Eval; discord_utils-Stub für Offline-Import; NIM-Fallback-Hinweis; Eval-Cache via Report-Datei; Exit 1 bei Health-Fail.
- Welche Daten verändert sie: Erzeugt ggf. `reports/eval-baseline-500.json` (Subprozess).
- Welche Zustände entstehen: Keine persistenten außer Eval-Cache.
- Welche Algorithmen werden verwendet: URL-Encoding, JSON-Parsing, Subprozess-Check.
- Welche Fehler behandelt sie: API unerreichbar → klare Anweisung; NIM-Fehler → Hinweis, kein Abbruch.
- Welche Seiteneffekte besitzt sie: Eval-Subprozess, Konsolenausgabe.

## scripts/ensure_hotelsim_api.py

### main / healthy

- Warum existiert sie: Idempotenter API-Wächter.
- Welches Problem löst sie: Startet die API, falls nicht erreichbar.
- Welche Regeln implementiert sie: Health-Check mit 2-s-Timeout; Start als detached Subprozess; 20×0,25 s Polling; PID/Log-Persistenz.
- Welche Daten verändert sie: PID-/Log-Dateien; startet Server.
- Welche Zustände entstehen: Healthy/nicht healthy.
- Welche Algorithmen werden verwendet: Polling-Schleife.
- Welche Fehler behandelt sie: Nicht-Healthy nach Timeout → SystemExit.
- Welche Seiteneffekte besitzt sie: Prozessstart.

## scripts/prepare_workspace_imports.py

### pw / main

- Warum existiert sie: Erzeugt Import-Dateien + sichere Passwortablage.
- Welches Problem löst sie: Offline-Vorbereitung der Workspace-Provisionierung.
- Welche Regeln implementiert sie: 18-Zeichen-Passwörter mit Komplexität (lower+upper+digit); Admin-CSV-Format; Gruppen-Zuordnung; chmod 0600 für Passwortdatei.
- Welche Daten verändert sie: Drei Ausgabedateien.
- Welche Zustände entstehen: Keine.
- Welche Algorithmen werden verwendet: `secrets.choice`-Sampling mit Retry-Loop.
- Welche Fehler behandelt sie: Passwort-Komplexitätsverletzung → neuer Versuch.
- Welche Seiteneffekte besitzt sie: Datei-Schreiben + chmod.

## scripts/provision_workspace.py

### main / create_users / create_groups / create_drives_and_docs / create_doc / creds / delegated_creds / temp_password

- Warum existieren sie: Echte Provisionierung der Google-Workspace-Sandbox.
- Welches Problem lösen sie: User/Gruppen/Drives/Docs mit Rollen-Berechtigungen anlegen.
- Welche Regeln implementieren sie: Idempotenz (exists-Check vor create); Passwort-Persistenz chmod 0600; Drive-Freigaben (Policies reader, sonst writer); managers-Gruppe umfasst Leads+Finance; Doc-Inhalte aus Markdown-Policies + synthetische Texte; Delegation als Sandbox-Admin oder OAuth; dry-run/skip-drive-Optionen.
- Welche Daten verändert sie: Google-Workspace-Ressourcen; Report- und Token-/Passwortdateien.
- Welche Zustände entstehen: Status je Ressource (created/exists/added/skipped).
- Welche Algorithmen werden verwendet: OAuth-Token-Refresh, Service-Account-Delegation, Drive-Pagination, Namens-Dedup-Query, Batch-Update für Doc-Text.
- Welche Fehler behandelt sie: Fehlende Credentials → SystemExit; 404/400/403/409-Zustände → not_found/exists/skipped.
- Welche Seiteneffekte besitzt sie: Echte API-Änderungen (außer dry-run).

## scripts/export_event_stream.py

- Warum existiert sie: Materialisiert den chronologischen Event-Stream.
- Welches Problem löst sie: Repräsentation der 48h-Simulation als JSONL.
- Welche Regeln implementiert sie: JOIN messages×incidents, Sortierung created_at/message_id, `ensure_ascii=False`.
- Welche Daten verändert sie: Überschreibt die Ziel-JSONL.
- Welche Zustände entstehen: Keine.
- Welche Algorithmen werden verwendet: SQL-JOIN/ORDER.
- Welche Fehler behandelt sie: Keine.
- Welche Seiteneffekte besitzt sie: Datei-Write.
