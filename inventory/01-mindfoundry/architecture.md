# MindFoundry — Architektur-Analyse

## Komponenten

### 1. Datenschicht (SQLite)
- `hotel_sim/generate.py`: deterministische Erzeugung (7 Tabellen: bookers, rooms, staff, incidents, messages, evaluations; Indizes; FK).
- `data/hotel_sim.sqlite`: Zentraler Speicher; Lese-Quelle aller höheren Module.
- `data/messages/two_day_event_stream.jsonl`: chronologischer 48h-Event-Export.
- `data/policies/*.md`: 3 Markdown-Richtlinien (privacy, refunds, routing).
- `data/replicants/discord_memories.jsonl`: append-only extrahierte Discord-Memories.

### 2. Wissens-/Agentenschicht (RAG)
- `hotel_sim/replicants.py`: Replicant-Profile (Pro-Person-Wissen aus Staff + Incidents + Discord), deterministisches Retrieval (`retrieve`: Tokenize + Score + Intent-Klassifikation + Routing-Vorschlag + Guardrail).
- `hotel_sim/live_ops.py`: Live-Signalaggregation (Severity-/Typ-Verteilungen, Owner-Queues, Urgent, Memory-Signale).
- `scripts/nemotron_rag_bridge.py`: Synthese über NVIDIA NIM (Nemotron) mit Zitat-Groundedness und deterministischem Fallback; Discord-Watcher-Loop.

### 3. Policy-/Guardrail-Schicht
- `hotel_sim/policy_gate.py`: klassenbasierte Redaktion (6 Klassen) mit Rollen-Matrix (`public_discord`, `finance_private`, `manager_private`); Audit-JSONL.
- `hotel_sim/evaluate.py`: Baseline-Agent + Score-Harness (Routing, Policy-Zitat, Privacy, Halluzination).

### 4. Integrationsschicht (Discord + Google Workspace)
- `scripts/discord_utils.py`: gegatete Discord-API (v10) mit Chunking/Rate-Limit/Token.
- `scripts/drip_discord_incidents.py`, `scripts/seed_discord_incidents.py`: Incident-Posting mit Dedup, Escalation-Mirror, Staff-Followups.
- `scripts/update_replicants_from_discord.py`: Wissensextraktion (SHA1-Dedup) → discord_memories.jsonl.
- `scripts/provision_workspace.py`, `scripts/prepare_workspace_imports.py`: Google-Workspace-Provisionierung (User/Gruppen/Drives/Docs, idempotent, RBAC).

### 5. API-/UI-Schicht
- `api/server.py`: stdlib-HTTP-Server (127.0.0.1:8765), 15 JSON-REST-Endpunkte, read-only, UI-Auslieferung.
- `ui/index.html`: Vanilla-JS-Dashboard (Metriken, Incident-Queue, RAG-Feld, Guardrail-Panel, Staff-Karten, Policy-Retrieval, Auto-Refresh 15 s).
- `scripts/ensure_hotelsim_api.py`: idempotenter API-Wächter.

### 6. Demo-/Test-Schicht
- `scripts/demo_walkthrough.py`: offline Nachweis (Health → RAG → Adversarial → Eval).
- `tests/test_sim.py`, `tests/test_live_ops.py`: Integrations-/Unit-Tests.
- `reports/eval-baseline-500.json`, `reports/live-ops-summary.txt`: Ergebnis-Artefakte.

## Module & Layer
- Layer-0 (Daten): SQLite + JSONL/Markdown.
- Layer-1 (Domäne): generate/evaluate/live_ops/replicants/policy_gate.
- Layer-2 (Integration): discord_utils, drip/seed, updater, bridge, provisioning.
- Layer-3 (Präsentation): api/server, ui/index.html, demo/ensure.
- Querschnitt: Policy-Gate (jede ausgehende Nachricht), Zustandsdateien (Agenten-Dedup), Audit-JSONL.

## Datenfluss
1. generate.py → SQLite (deterministisch, Seed 42).
2. export_event_stream.py → two_day_event_stream.jsonl.
3. retrieve(q) → Score-Liste aus Replicant-Memories + Policy-Snippets + Live-Ops (Score 999 bei Live-Trigger) → Citations → recommended_route.
4. Bridge: Frage (Discord) → clean → retrieve → [NIM-Synthese mit Citations oder Fallback] → gate → Post + Audit.
5. Drip/Seed: SQLite-Incidents → State-Dedup → Discord (Incident + Followup + Mirror) → Audit.
6. Updater: Discord-Nachrichten → Klassifikation → Memory-ID (SHA1) → append discord_memories.jsonl + State.
7. Demo: Health-Check → RAG-Beispiel → Adversarial-Probe → Eval (Subprozess) → Report.

## Eventfluss
- Discord: Nachricht in #nemotron-rag → Antwort/Audit/Fehler-Post; Incident-Kanäle → Incident + Followup + Mirror (Escalation); Staff-Kanäle → Audit-Post (Zählung).
- API: HTTP-GET → JSON (keine Events).
- UI: Polling alle 15 s (kein Push).

## APIs
- Lokal: /health, /dashboard/summary, /incidents/open, /incidents/closed, /staff, /policies, /policies/<name>, /replicants/summary, /rag/query, /messages, /reservations, /evaluations/baseline, /evaluations/routing, /evaluations/privacy, /evaluations/hallucination (nur GET; Details in files.md; Namensdivergenz: Code liest `reports/baseline-eval.json`, Datei heißt `eval-baseline-500.json`).
- Extern: NVIDIA NIM `https://integrate.api.nvidia.com/v1`, Modell `nvidia/llama-3.3-nemotron-super-49b-v1.5`; Discord API v10 `https://discord.com/api/v10`; Google-Workspace-APIs (Admin, Drive).

## State Machines
- Incident-Status: open → in_progress → resolved; escalated zusätzlich (escalation-Mirror bei urgent/high/escalated).
- Agenten: Init/Fehlen → Initialisierung (State-Datei), Lauf → Dedup → Post → Save; Prozess-Loop (Bridge-Watcher, Drip-Scheduler).

## Scheduler / Worker / Pipelines
- Bridge: Discord-Watcher-Loop mit Sleep (Zeitabstand in main; Details: while-Schleife mit `time.sleep`).
- Drip: Laufende Incident-Befüllung mit Cursor (cursor_created_at) und Limits (3/Lauf).
- Keine parallelen Worker; alles sequenziell pro Prozess.

## Queues
- Owner-Queues (live_ops): pro Mitarbeiter geordnete Incident-Listen (Limit 5).
- Urgent-Queue: severity urgent/high oder escalated (Limit 6).
- Discord-Chunking-Queue: 1900-Zeichen-Chunks mit 0,7 s Sleep.

## Memory-Systeme
- Short Memory: incident-Daten + letzte Discord-Messages (reversed-Read 25 je Kanal).
- Long Memory: SQLite (staff/incidents) + discord_memories.jsonl + Policies.
- Episodic: experience (Typ-Zählung), case (bis 4 Fälle je Person).
- Semantic: Replicant-Profile, Policy-Snippets.
- Facts: Staff-Masterdaten, TYPE_OWNER/ROUTE-Mappings.
- Knowledge Graph: Nicht nachweisbar (keine Graph-Struktur; flache Zuordnungen).
- Consolidation/Ranking/Retrieval: deterministisches Scoring (2-Punkt-exakt + 1-Punkt-Präfix, Intent-Boni), Limits, keine LLM-gebundene Re-Rankierung.

## RAG
- Ablauf: Intent-Klassifikation (10 Typen) → Candidate-Pool (Memories, Policies, Live-Ops) → Score → Top-k Citations → NIM-Synthese mit [n]-Zitatpflicht → Gate → Ausgabe.
- Retrieval deterministisch (keine Vektoreinbettung belegt); Synthese LLM-gebunden mit Fallback.

## Policy
- 3 Dokumente; Rollen-Matrix; Audit-JSONL je Gate-Aufruf; Redaktion `[KIND REDACTED]`; keine Auth-/Session-Schicht in der API.

## NVIDIA NIM
- Chat-Completion-Request (POST chat/completions, temperature 0.2, top_p 0.9, max_tokens 600, stream False); 1 Retry bei Timeout/URLError; HTTPError-Body-Auszug; Key latin-1-Prüfung; NIM_DISABLE-Schalter; Response-Feld `choices[0].message.content`.

## Tool Calling / Streaming
- Tool Calling: Nicht nachweisbar (keine Tool-Call-Strukturen im Code belegt).
- Streaming: Nicht nachweisbar (stream=False explizit).

## Sicherheit
- Policy-Gate erzwingt Redaktion; PII-Klassen; Staff-E-Mail-Allowlist; Passwörter chmod 0600 in `~/.openclaw/secure/`; synthetische Klartext-Passwörter in eingecheckter CSV; API ohne Auth (localhost-bound); Bot-Token aus OpenClaw-Config.

## Fehlerbehandlung
- Bridge: Per-Nachrichten-Try/Except → Fehler-Post; Fallback-Rendering bei NIM-Ausfall.
- Provisioning: API-Zustände (404/400/403/409) → not_found/exists/skipped.
- Gate/Utils: Fehlende Dateien/Kanäle → Init/Überspringen.
- Demo: API-Fail → Exit 1.

## Komponenten-/Modul-Graph (Zusammenfassung)
```
generate → SQLite → (export → event-stream)
                   ├─ replicants → (retrieve → citations)
                   ├─ live_ops → live-summary
                   ├─ evaluate → eval-report
                   ├─ drip/seed → Discord (+State)
                   └─ api/server → UI
replicants + live_ops + policies → bridge → NIM/Fallback → gate → Discord
discord (Staff-Kanäle) → updater → discord_memories.jsonl → replicants
prepare_workspace_imports → workspace-imports → provision_workspace → Google
```
