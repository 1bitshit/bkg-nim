# MindFoundry — Repo-Übersicht

Belegbasis: `/home/workspace/work/proxy/bkg-nim-studio/source/mindfoundry`

## Zweck

MindFoundry ist ein persistent laufender, autonomer Agent („Long Agent"), der aus Chat-Verläufen, Dokumenten und Operations-Historie pro Teammitglied ein strukturiertes Wissensprofil („Replicant") aufbaut und dann betriebliche Fragen im Namen dieser Teammitglieder mit Zitaten beantwortet (belegt durch `README.md`). Die Referenz-Instanz ist „NeMo Lodge", ein simuliertes 250-Zimmer-Hotel mit 8 Staff-Replicants, 2.500 Buchungspersonen, 3.150 Reservierungen und 500 Betriebs-Vorfällen über zwei simulierte Tage (belegt durch `docs/architecture.md`, `hotel_sim/generate.py`).

Das Repository ist ein Hackathon-Beitrag (NVIDIA Agent Hackathon 2026) und enthält die vollständige Pipeline: Datengenerierung, Retrieval-API, Nemotron-via-NVIDIA-NIM-Synthese, NemoClaw-Policy-Gate mit PII-Redaktion, Audit-Log, Evaluations-Harness, Discord-Integration und Dashboard (belegt durch `README.md`, `SUBMISSION-PACKAGE.md`).

## Verantwortlichkeit

Jede Datei hat eine klar abgegrenzte Verantwortlichkeit (Details in `files.md`):

- `hotel_sim/generate.py` — erzeugt die kanonische SQLite-Simulation (PMS + Operations).
- `hotel_sim/replicants.py` — baut Replicant-Profile und führt deterministisches Retrieval durch.
- `hotel_sim/live_ops.py` — aggregiert Live-Operations-Status aus SQLite + Discord-Post-Status.
- `hotel_sim/policy_gate.py` — PII-Erkennung und -Redaktion für jeden ausgehenden Text.
- `hotel_sim/evaluate.py` — bewertet Routing/Policy/Privacy/Halluzination über Vorfälle.
- `api/server.py` — lokale HTTP-Retrieval-API (`127.0.0.1:8765`).
- `scripts/*` — Discord-Integration, RAG-Bridge (NIM), Workspace-Provisionierung, Demo, Exporte.
- `ui/index.html` — „HotelSim Control Room"-Dashboard.
- `data/` — Policies (Markdown), Replicant-Memories (JSONL), Event-Stream (JSONL).
- `reports/` — Evaluationsergebnisse, Live-Ops-Summaries, Audit- und Zustandsdateien.
- `docs/` — Architektur-, Produkt-, Demo-, Policy- und NVIDIA-Dokumentation.
- `tests/` — pytest-/Standalone-Tests für Live-Ops und Simulation.

## Komponenten

1. **Knowledge Gatherer** — liest Discord-Nachrichten, Google-Workspace-Daten und Policy-Docs in strukturiertes Signal (belegt durch `docs/mindfoundry-architecture.svg`, Box „1 Knowledge Gatherer").
2. **Replicant Updater** — extrahiert dauerhafte Fakten pro Teammitglied mit Quelle + Freshness in SQLite/JSONL-Replicant-Memory (belegt durch `docs/mindfoundry-architecture.svg`, Box „2 Replicant Updater"; `scripts/update_replicants_from_discord.py`).
3. **Nemotron-RAG-Bridge** — ruft die lokale Retrieval-API, übergibt Zitate an Nemotron via NVIDIA NIM und synthetisiert eine zitierte Antwort (belegt durch `scripts/nemotron_rag_bridge.py`, `docs/mindfoundry-architecture.svg`, Box „3 Nemotron RAG + Actions").
4. **NemoClaw-Policy-Engine (Policy Gate)** — redigiert Passwörter, Gast-E-Mails, Telefonnummern, Zahlungsdaten, Ausweis-/Passfelder und `internal_notes` vor jedem ausgehenden Versand (belegt durch `hotel_sim/policy_gate.py`, `docs/openshell-policy-neemo-lodge.yaml`).
5. **Audit-Log** — jede Gate-Entscheidung wird an `reports/policy-gate-events.jsonl` und den Discord-Kanal `#agent-audit-log` angehängt (belegt durch `hotel_sim/policy_gate.py`, `scripts/discord_utils.py`).
6. **HotelSim-Evaluation** — deterministischer Baseline-Agent + Scoring über Routing, Policy-Grounding, Privacy und Halluzination (belegt durch `hotel_sim/evaluate.py`).
7. **HotelSim Control Room** — Web-Dashboard über die lokale API (belegt durch `ui/index.html`, `api/server.py`).

## Datenfluss

1. `hotel_sim/generate.py` erzeugt `data/hotel_sim.sqlite` (Rooms, Bookers, Reservations, Staff, Incidents, Messages, Audit-Logs) und `data/messages/two_day_event_stream.jsonl` wird davon exportiert (`scripts/export_event_stream.py`).
2. `hotel_sim/replicants.py` liest Staff + Incidents aus SQLite und `data/replicants/discord_memories.jsonl` und baut pro Staff-Mitglied eine Memory-Liste (Profil, Verhalten, Workspace-Identität, Erfahrung, Fälle, Discord-Memories).
3. `api/server.py` stellt diese Retrieval-Funktion als HTTP-API bereit (`/rag/query`, `/replicants`, `/incidents/*`, `/rooms/*`, `/guests/*`, `/policies/*`, `/dashboard/summary`, `/health`).
4. `scripts/nemotron_rag_bridge.py` liest Fragen aus dem Discord-Kanal `#nemotron-rag`, ruft `/rag/query`, ruft NVIDIA NIM (Chat-Completions), lässt das Ergebnis durch `policy_gate.gate_text`, postet die Antwort nach `#nemotron-rag` und einen Audit-Eintrag nach `#agent-audit-log`.
5. `scripts/drip_discord_incidents.py` / `seed_discord_incidents.py` posten simulierte Vorfälle aus SQLite in Discord-Kanäle; `scripts/update_replicants_from_discord.py` liest Staff-Antworten zurück und hängt Memory-Einträge an `discord_memories.jsonl`.
6. `hotel_sim/live_ops.py` aggregiert gepostete Vorfälle, Severity-Mixe, Owner-Queues und Memory-Signale zu einer Zusammenfassung, die in RAG-Antworten eingebettet wird.

## Persistenz

- **SQLite**: `data/hotel_sim.sqlite` — kanonische Quelle für Räume, Buchungspersonen, Reservierungen, Staff, Incidents, Messages, Audit-Logs (belegt durch `hotel_sim/generate.py` Schema-DDL; per `.gitignore` von Git ausgeschlossen, wird regeneriert).
- **JSONL**: `data/replicants/discord_memories.jsonl` (160 Memory-Einträge, belegt durch Zählung), `data/messages/two_day_event_stream.jsonl` (500 Events), `reports/policy-gate-events.jsonl` (zur Laufzeit angehängt, von Git ausgeschlossen).
- **JSON-Zustandsdateien** in `reports/`: `discord-drip-state.json`, `discord-seeded-incidents.json`, `nemotron-rag-bridge-state.json`, `replicant-updater-state.json`, `workspace-provisioning.json`, `eval-baseline-500.json`, `live-ops-summary.txt` (belegt durch die jeweiligen `save`-Funktionen in den Scripts).
- **Markdown-Policies**: `data/policies/{privacy,refunds,routing}.md` — Retrieval-gegroundete SOP-Quelle.

## Abhängigkeiten

- Python 3.11+ (belegt durch `README.md`).
- Externe Pakete (belegt durch `requirements.txt`): `google-api-python-client`, `google-auth`, `google-auth-oauthlib`, `google-auth-httplib2`, `PyYAML`. Der NIM-Client und die Discord-Integration nutzen ausschließlich die Standardbibliothek (`urllib`, `sqlite3`, `json`).
- Externe Dienste: NVIDIA NIM (`https://integrate.api.nvidia.com/v1`), Discord HTTP API v10 (`https://discord.com/api/v10`), Google Workspace (Admin/Directory, Drive, Docs; belegt durch `scripts/provision_workspace.py`).
- Laufzeit-Agent-Framework: OpenClaw (Herzschlag, Cron-Jobs, Discord-Integration, SQLite/JSONL-Persistenz; belegt durch `README.md` und `SUBMISSION-PACKAGE.md` — im Repo selbst nicht als Code enthalten).

## Eingaben / Ausgaben

**Eingaben:**
- Umgebungsvariablen: `NVIDIA_NIM_API_KEY` (Pflicht für Live-Synthese), `NVIDIA_NIM_MODEL` (Default `nvidia/llama-3.3-nemotron-super-49b-v1.5`), `NVIDIA_NIM_BASE`, `NVIDIA_NIM_DISABLE`, `HOTELSIM_PORT` (Default 8765), `HOTELSIM_API`, `DISCORD_BOT_TOKEN` (belegt durch `scripts/nemotron_rag_bridge.py`, `README.md`).
- Discord-Bot-Token aus `~/.openclaw/openclaw.json` (belegt durch `scripts/discord_utils.py`).
- Kanal-IDs aus `reports/discord-secondoffice-channels.json` (belegt durch `scripts/discord_utils.py`).
- Google-Workspace-Zugangsdaten unter `~/.openclaw/credentials/google-client-secret.json` und `~/.openclaw/secure/hotel-sim/*` (belegt durch `scripts/provision_workspace.py`).

**Ausgaben:**
- `reports/eval-baseline-500.json` (500 bewertete Vorfälle, Score 4/4 bei allen, belegt durch Zählung).
- `reports/live-ops-summary.txt` (gerenderte Live-Ops-Zusammenfassung).
- `reports/workspace-provisioning.json`, `reports/*.json` Zustandsdateien.
- Discord-Nachrichten (Antworten, Drip-Vorfälle, Audit-Zeilen).
- HTTP-API-Antworten (JSON).

## Zustände

- **Incident-Status**: `open`, `in_progress`, `resolved`, `escalated` (belegt durch `hotel_sim/generate.py`).
- **Room-Status**: `vacant_clean`, `vacant_dirty`, `occupied`, `out_of_order` (belegt durch `hotel_sim/generate.py`).
- **Gate-Entscheidungen**: `allow`, `allow_with_redactions` (belegt durch `hotel_sim/policy_gate.py`; `deny` erscheint in der YAML-Policy für Passwort-Klassen, belegt durch `docs/openshell-policy-neemo-lodge.yaml`).
- **Reservation-Status**: `checked_in` (nur für 15.–16. Juni 2026), sonst `confirmed`, `completed`, `cancelled` (belegt durch `hotel_sim/generate.py`).
- **Provisioning-Zustand**: Workspace `provisioned: True/False` (belegt durch `hotel_sim/replicants.py` `load_workspace`).
- **RAG-Bridge-Zustand**: Liste bereits beantworteter Frage-IDs (`answered`), `sent_log` (belegt durch `scripts/nemotron_rag_bridge.py`).
- **Drip-Zustand**: `posted` (bereits gepostete Incident-IDs), `cursor_created_at`, `sent_log` (belegt durch `scripts/drip_discord_incidents.py`).

## Sicherheitsrelevanz

- PII-Redaktion: Passwörter, Kreditkarten-ähnliche Nummern, Taiwan-Telefonnummern, E-Mail-Adressen, Pass-/ID-Felder, `internal_notes` werden durch Regex erkannt und durch `[KIND REDACTED]` ersetzt (belegt durch `hotel_sim/policy_gate.py`).
- Rollenbasierte Egress-Policies: `public_discord`, `finance_private`, `manager_private` mit unterschiedlichen Freigaben (belegt durch `hotel_sim/policy_gate.py` `ROLE_PERMISSIONS`).
- Filesystem-/Netzwerk-Allowlists und Prozess-Denylist in `docs/openshell-policy-neemo-lodge.yaml` (Default-Deny).
- Generierte Passwörter werden in `~/.openclaw/secure/hotel-sim/fake-staff-passwords.json` mit `chmod 0600` abgelegt (belegt durch `scripts/prepare_workspace_imports.py`, `scripts/provision_workspace.py`).
- Evaluations-Sensitivitätsmuster erkennen Telefonnummern, Gast-E-Mail-Adressen und Bezahl-/Steuer-Schlüsselbegriffe (belegt durch `hotel_sim/evaluate.py` `SENSITIVE_PATTERNS`).
- `.gitignore` schließt `.env`, Secrets, Service-Account-JSON, Token- und Key-Dateien von Git aus (belegt durch `.gitignore`).
- Die evaluierte Baseline weist 0 Leak-Treffer über 500 Vorfälle auf (belegt durch `reports/eval-baseline-500.json`, Zählung `leak_hits` leer).
