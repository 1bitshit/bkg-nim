# MindFoundry — Datenmodell-Analyse

## Datenbank `data/hotel_sim.sqlite`

- Bedeutung: Zentraler Simulationsspeicher; Quelle für Retrieval, Evaluation, Event-Export und Live-Ops.
- Lebenszyklus: Erzeugt/überschrieben durch `hotel_sim/generate.py` (Seed 42, deterministisch); nur lesend von API, Replicants, Live-Ops, Scripts.
- Ownership: `generate.py` (Schreib-), alle übrigen Module (Lese-).
- Beziehungen: incidents.booker_id → bookers.id, incidents.staff_id → staff.id, messages.incident_id → incidents.id (FK, `PRAGMA foreign_keys=ON`).
- Persistenz: SQLite-Datei im Repo.
- Validierung: DDL-Typen + FK; Assertions in `generate.py` (Wiederholungsverteilung); `test_sim.py` prüft Zählungen.
- Mutationen: Nur durch `generate.py` (DROP+CREATE je Lauf).
- Thread-Sicherheit: Einzelprozess; API/CLI serialisiert; keine parallelen Schreiber.
- benötigte Rust-Konzepte: rusqlite/sqlx, SQLite mit `PRAGMA foreign_keys`, Embeddable-DB, deterministische Seed-Reproduzierbarkeit (Stable-RNG statt `random`).

### bookers

- Bedeutung: 250 synthetische Gäste (name, loyalty: 35 % normal / 40 % silver / 20 % gold / 5 % platinum, Sprache en/zh-TW 55/45).
- Lebenszyklus/Ownership: Erzeugt bei generate; Referenzobjekt für incidents/messages.
- Persistenz: Tabelle.
- Validierung: NOT NULL, CHECK-ähnliche Semantik aus `generate.py`-Logik (loyalty-Wahl).
- benötigte Rust-Konzepte: Struct mit Enum (loyalty), Lang-Enum, Seed-abhängiger RNG.

### rooms

- Bedeutung: 500 Zimmer mit Typ-Verteilung (standard 55 %, deluxe 25 %, suite 15 %, presidential 5 %), Rate und Kapazität.
- Persistenz: Tabelle; keine FK-Beziehungen (nur Kennzeichnung in reservations? — belegt: Zimmer unabhängig, keine FK von reservations auf rooms laut Schema-DDL; Nicht nachweisbar ist eine Beziehung, da rooms nicht referenziert wird).
- benötigte Rust-Konzepte: Enum RoomType mit Weighted Sampling.

### staff

- Bedeutung: 8 Personen (Ben Wu/Grace Liu/Leo Wang/Maya Chen/Kevin Liu/Annie Chang + Leads/Manager), je Rolle, Schicht, Clearance (staff/lead/manager), E-Mail, zh_name.
- Persistenz: Tabelle.
- Validierung: Rollen aus festen Staff-Definitionen.
- benötigte Rust-Konzepte: Struct Staff mit Rollen-/Clearance-Enums.

### incidents

- Bedeutung: 500 Incidents: 10 Typen (billing, front_desk, housekeeping, maintenance, noise, access, vip_request, reservation_change, safety, lost_item), Severity (urgent/high/medium/low), Status (open/in_progress/resolved/escalated), Quellen (phone, walkin, email, channel), eskalierte und privacy-sensitive Markierungen (billing-Typ oder 8 %).
- Lebenszyklus: Erzeugt mit Zeitstempeln im 48h-Fenster; gelesen für Routing/Eval/Live-Ops.
- Persistenz: Tabelle mit Index (status, severity) und (created_at).
- Validierung: FK auf bookers/staff; Sprach- und Typ-Felder.
- Mutationen: Nur durch generate.
- benötigte Rust-Konzepte: Struct Incident mit Enums; Composite-Index.

### messages

- Bedeutung: 3150 zweisprachige Sim-Nachrichten, verknüpft mit Incident, mit created_at, message_id (Autoincrement), Rolle/Quelle, 3000 Textvarianten (davon viele mit Vorlagen-Platzhaltern; Sensitiv-Varianten).
- Persistenz: Tabelle mit Index (created_at); Event-Stream-Sortierung.
- benötigte Rust-Konzepte: Beliebte-Gesprächs-Modell (1:n), Zeitstempel-Sortierung, Template-Expansion zur Generierungszeit.

### evaluations

- Bedeutung: Ergebniszeilen des Evaluations-Laufs (500 Zeilen, baseline: incident_id, route_to, policy_cited, privacy_redacted, hallucination_free, score 4/4).
- Persistenz: Tabelle; Report als JSON exportiert.
- benötigte Rust-Konzepte: Read-Model für Berichte; Serialisierung.

## Event-Stream `data/messages/two_day_event_stream.jsonl`

- Bedeutung: 500 chronologische Ereignisse (15.06.2026 06:00 – 17.06.2026 05:58 UTC+8), je Nachricht mit incident_ref, incident_type, room/guest-Info, sender (guest), message, created_at; Typ-Verteilung: lost_item 60, front_desk 57, maintenance 54, reservation_change 54, noise 52, safety 49, vip_request 47, access 45, housekeeping 43, billing 39; 256 en / 244 zh-TW.
- Lebenszyklus: Export aus SQLite (JOIN messages×incidents, Sortierung); nur lesend konsumiert.
- Ownership: `scripts/export_event_stream.py`.
- Persistenz: JSONL-Datei im Repo.
- Validierung: Reihenfolge chronologisch (verifiziert), konsistente Typen.
- benötigte Rust-Konzepte: Streaming-Reader (serde_json StreamDeserializer), chrono, Typ-Enum-Deserialisierung.

## Discord-Memories `data/replicants/discord_memories.jsonl`

- Bedeutung: 160 Einträge kind=discord_staff_memory, extrahiert aus Discord durch `scripts/update_replicants_from_discord.py`; 7 eindeutige Textvorlagen (davon mit Platzhaltern), 5 Personen (Leo Wang 66, Maya Chen 41, Grace Liu 25, Ben Wu 16, Annie Chang 12), Zeitraum 26.–27.05.2026.
- Lebenszyklus: Append-only; `seen`-Dedup über State; von Replicants konsumiert (bis 8 je Rolle).
- Ownership: Updater-Script (Schreib-), Replicants (Lese-).
- Persistenz: JSONL (eine Zeile je Memory).
- Validierung: Kanal→Rolle-Mapping, Staff-Prefix, Memory-Würdigkeit, Hash-ID.
- Mutationen: Append; keine In-Place-Änderung.
- Thread-Sicherheit: Einprozess-Schreiber.
- benötigte Rust-Konzepte: Append-only Store, SHA1-Hash-ID (12 Hex), serde mit tagged enum (kind), Dedup via HashSet.

## Policies `data/policies/*.md` (privacy, refunds, routing)

- Bedeutung: 3 Markdown-Richtlinien (PII-Verhalten, Rückerstattungs-Kompetenzen, Routing/Kanäle) mit definierten Abschnitts-Gliederungen (doc-id, references, content).
- Lebenszyklus: Statisch; Quelle für Policy-Snippets und Workspace-Docs.
- Ownership: Repo-Content.
- Persistenz: Markdown-Dateien.
- Validierung: Referenzierung über doc-id/§-Anker in Zitaten.
- benötigte Rust-Konzepte: Markdown-Parsing (pulldown-cmark) oder Frontmatter-Slices; Abschnitts-Identitäten.

## Replicant-Profil (Laufzeit-Dict)

- Bedeutung: Pro Person: profile (staff), behavior (Arbeitsstil, Schicht, Clearance), workspace (E-Mail, Gruppen), experience (Incident-Typ-Zählung), case (bis 4 letzte Fälle mit Sensitivitäts-Hinweis), discord_staff_memory (bis 8); erzeugt durch `build_replicants()`.
- Lebenszyklus: Pro Aufruf rekonstruiert (keine Persistenz).
- Ownership: Laufzeit.
- Persistenz: Keine.
- benötigte Rust-Konzepte: Struct-Enum-Memory (tagged), Limits als Slice, keine Shared State nötig (pure Function).

## Zustandsdateien `reports/*.json` (discord-drip-state, discord-seeded-incidents, nemotron-rag-bridge-state, replicant-updater-state, workspace-provisioning)

- Bedeutung: Prozesszustand der Agenten (posted-IDs, answered-IDs, seen-Sets, sent_logs, cursor_created_at, Provisioning-Status).
- Lebenszyklus: Append/Update je Agenten-Lauf; persistiert über Prozesse hinweg.
- Ownership: Je Agenten-Script (Schreib- und Lese-).
- Persistenz: JSON-Dateien unter reports/.
- Validierung: Keine formale; fehlerhafte JSON werden als leer behandelt.
- Thread-Sicherheit: Ein Prozess je Agent; kein Locking belegt.
- benötigte Rust-Konzepte: Atomic file write (temp+rename), serde State-Config, Mutex/Channel bei parallelen Agenten.

## Eval-Report `reports/eval-baseline-500.json`

- Bedeutung: 500 Zeilen, alle Score 4/4, routing/policy/privacy/hallucination_ok alle True, leak_hits leer (Halluzinations- und PII-Prüfungen sauber).
- Lebenszyklus: Einmalig erzeugt (Demo-Cache); von `demo_walkthrough.py` gelesen.
- Persistenz: JSON-Datei.
- benötigte Rust-Konzepte: Deserialisierung in Report-Struct; Validierungssummen.

## Live-Ops-Summary `reports/live-ops-summary.txt`

- Bedeutung: 37 gepostete Incidents, 12 urgent, 4 privacy-sensitive; von `live_summary()` gerendert und von RAG/Bridge injiziert.
- Persistenz: Datei als Schnappschuss; Quelle sind State + SQLite.
- benötigte Rust-Konzepte: Rendering von Aggregaten; Markdown-Ausgabe.

## Workspace-Importe `workspace-imports/` (groups.json, users-google-admin-import.csv)

- Bedeutung: Sandbox-Importdaten: 7 Gruppen (managers, frontdesk, housekeeping, maintenance, reservations, finance, guest-experience) mit Mitglieds-E-Mails; 8 Staff-User mit 18-Zeichen-Passwörtern, Org Unit `/`, „Change Password at Next Sign-In" FALSE.
- Lebenszyklus: Erzeugt durch `prepare_workspace_imports.py`; von `provision_workspace.py` konsumiert.
- Persistenz: JSON/CSV (Passwort-Kopie unter `~/.openclaw/secure/hotel-sim/`, chmod 0600).
- Sicherheit: Klartext-Passwörter (synthetisch) in eingecheckter CSV; nicht von .gitignore abgedeckt.
- benötigte Rust-Konzepte: CSV/JSON-SerDe, Secrets-Handling (Datei-Permissions), RBAC-Gruppenmodell.

## User-/Gruppen-Modell (Google Workspace, Laufzeit)

- Bedeutung: 8 Staff-User (Rollen/Leads/Manager), 7 Gruppen; Rollen→E-Mail-/Gruppen-Mapping aus `replicants.py` (ROLE_TO_EMAIL/ROLE_TO_GROUP) und `live_ops.py` (TYPE_OWNER).
- Persistenz: Remote-Google-Konto; Status (created/exists/added/skipped) im Provisioning-Report.
- benötigte Rust-Konzepte: Typ-sichere Rollen-Enums; Remote-API-Client (google-apis crate), Idempotenz-Prüfungen.
