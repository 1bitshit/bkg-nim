# PolicyNIM — Repo-Übersicht

## Zweck

PolicyNIM ist eine policy-bewusste Engineering-Preflight-Schicht für KI-Coding-Agenten (belegt durch `README.md`). Es verwandelt ein kleines Markdown-Policy-Korpus in belegte (zitierte) Implementierungsleitfäden. Kernidee: Der Agent ruft vor der Implementierung belegte Policy-Evidenz ab, erzeugt mit Zitaten versehene Leitfäden und schlägt **fail-closed** fehl, wenn die verfügbare Grounding-Basis zu schwach für eine vertrauenswürdige Antwort ist (belegt durch `docs/architecture.md`).

## Verantwortlichkeit

- Retrieval-gestützte, zitierbare Implementierungsleitfäden für KI-Coding-Agenten liefern
- Fail-closed-Garantien durchsetzen: Setup-Fehler und schwache Evidenz werden nie als plausible Antwort maskiert
- Zwei Benutzeroberflächen: JSON-first CLI (`interfaces/cli.py`) und MCP-Server (`interfaces/mcp.py`)
- Lokale Policy-Entscheidungen zur Laufzeit (allow/confirm/block) mit unveränderlicher SQLite-Evidenz
- Gehostetes Beta-Portal mit GitHub-OAuth-Login, API-Key-Ausgabe und Tagesquota (SQLite-gestützt)
- Offline-/Live-Evaluationsworkflow mit Rerank-Vergleich, Conformance-Scoring und Phoenix-UI-Option

## Komponenten (Verzeichnisstruktur)

| Verzeichnis | Verantwortlichkeit |
|---|---|
| `src/policynim/` | Python-Paket mit Core-Modulen (`settings.py`, `types.py`, `contracts.py`, `errors.py`, `config_discovery.py`, `runtime_paths.py`, `agent_workflows.py`) |
| `src/policynim/ingest/` | Dokument-Discovery, Markdown-Parsing, Sektionsextraktion, Chunking (`loader.py`, `parser.py`, `chunking.py`) |
| `src/policynim/providers/` | NVIDIA-Adapter: Chat/Generation, Embeddings, Reranking, Eval, Guardrails (`nvidia.py`, `nvidia_eval.py`, `nvidia_guardrails.py`) |
| `src/policynim/storage/` | SQLite-Vektorindex (`sqlite_vec.py`, `index_store.py`), Runtime-Evidenz (`runtime_evidence.py`), Beta-Auth (`auth_store.py`) |
| `src/policynim/services/` | Anwendungsorchestrierung: Ingest, Search, Router, Compiler, Preflight, Regeneration, Runtime-Decision/Execution, Eval, Evidence-Trace, Health, Beta-Auth, Dump |
| `src/policynim/interfaces/` | Transport-Einstiegspunkte: `cli.py`, `mcp.py` |
| `src/policynim/templates/` | Jinja2-Templates für Beta-Portal und Guardrails-Preflight-Output |
| `src/policynim/assets/` | Statische Assets (CSS, JS, Bilder) für das Beta-Portal |
| `policies/` | Das ausgelieferte synthetische Policy-Korpus (Markdown + Frontmatter) |
| `evals/` | Gold-Eval-Suite (`default_cases.json`) |
| `tests/` | Automatisierte Tests (offline, live, docker_live markiert) |
| `docs/` | Projekt-Dokumentation (Architektur, Workflows, Hosting, Release) |

## Datenfluss

### Corpus → Index (Ingest)
1. Policy-Dateien aus gebündeltem Korpus oder `POLICYNIM_CORPUS_DIR` entdecken
2. Markdown parsen und normalisieren (`ingest/parser.py`)
3. Fehlende Metadaten ableiten
4. Heading-bewusste Sektionen mit stabilen 1-basierten Zeilenspannen extrahieren (`ingest/chunking.py`)
5. `runtime_rules`-Frontmatter in persistiertes Runtime-Rules-Artefakt kompilieren
6. Deterministische Chunk-IDs bauen
7. Chunk-Text durch NVIDIA-Embeddings einbetten (`providers/nvidia.py`)
8. Lokale SQLite-Tabelle durch neue eingebettete Zeilen ersetzen (`storage/sqlite_vec.py`)

### Suche/Route/Compile/Preflight
1. Query/Task durch NVIDIA-Embeddings einbetten
2. Dichte Kandidaten aus lokalem SQLite-Index holen
3. Optional nach Policy-Domain filtern
4. Kandidaten durch NVIDIA-Reranking neu ordnen
5. Router: deterministisches `TaskProfile` + `PolicySelectionPacket`
6. Compiler: zitiergestütztes `CompiledPolicyPacket`
7. Preflight: belegtes `PreflightResult` mit Zitaten; schwache Evidenz → `insufficient_context=true`
8. `--trace`: `PolicyEvidenceTrace` aus bereits materialisierten Daten (kein Re-Run)
9. `--regenerate`: begrenzte Retry-Schleife (max. 3) mit typisierten Conformance-Triggern

### Runtime-Entscheidungen (lokal, ohne NVIDIA)
1. `runtime_rules`-Artefakt laden
2. Aktion (shell_command/file_write/http_request) gegen kompilierte Regeln matchen
3. Entscheidung allow/confirm/block mit Zitaten
4. Optionale Ausführung der bereinigten Aktion
5. Unveränderliche Evidenz-Events an SQLite-Evidenz-Store anhängen
6. `evidence report` fasst eine Session zusammen

## Persistenz

- `data/index.sqlite3` — lokaler Vektorindex (SQLite + sqlite-vec)
- `data/runtime/runtime_rules.json` — kompiliertes Runtime-Rules-Artefakt
- `data/runtime/runtime_evidence.sqlite3` — unveränderliche Runtime-Evidenz-Events
- `data/evals/workspace/` — Eval-Artefakte (JSON/HTML), Phoenix-Zustand unter `phoenix/`
- Beta-Auth-SQLite (`POLICYNIM_BETA_AUTH_DB_PATH`, Standard `data/runtime/auth.sqlite3`) — Accounts, API-Keys (gehasht), Audit-Events, Tagesquota
- Konfigurationsdateien: `.env` (Checkout), Plattform-Config-`config.env` (Standalone), via `POLICYNIM_CONFIG_FILE` übersteuerbar

## Abhängigkeiten (Top-Level, belegt durch `pyproject.toml`)

- `arize-phoenix==13.21.0`, `arize-phoenix-evals==2.12.0` (Eval-UI, Offline-/Phoenix-Reporting)
- `httpx==0.27.2` (HTTP-Client für NVIDIA und Runtime-HTTP-Aktionen)
- `itsdangerous==2.2.0` (Signierung, Beta-Session-Cookies)
- `jinja2==3.1.6` (Beta-Portal-Templates)
- `markdown-it-py==4.0.0` (Markdown-Parsing)
- `mcp[cli]==1.26.0` (MCP-Server-SDK)
- `openai==2.29.0` (NVIDIA-OpenAI-kompatible Chat-/Embeddings-API)
- `pandas==2.2.3`, `platformdirs==4.5.0`, `pydantic==2.12.5`, `pydantic-settings==2.13.1`
- `sqlite-vec==0.1.9` (Vektor-Suche in SQLite)
- `typer==0.24.1` (CLI-Framework)
- Optional: `nemoguardrails[nvidia]==0.21.0`, `nemo-evaluator==0.2.5`, `nvidia-nat-eval==1.6.0`, `nvidia-simple-evals==26.3`, `nemo-evaluator-launcher==0.2.4`, `nvidia-nat[eval]==1.6.0`
- Python `>=3.11,<3.13`

## Eingaben

- CLI-Kommandos (typer/click-basiert), MCP-Tool-Calls (`policy_preflight`, `policy_search`), HTTP-Routen (`/healthz`, `/mcp`, `/beta`, OAuth-Callbacks)
- Env-Variablen: `NVIDIA_API_KEY`, `POLICYNIM_*` (vollständig in `settings.py`)
- JSON-Eingaben für `runtime decide`/`runtime execute` (`--input <path|->`, `RuntimeActionRequest`)
- Korpus-Markdown mit YAML-Frontmatter (`policies/`)

## Ausgaben

- JSON-first CLI-Ausgaben (`SearchResult`, `PolicySelectionPacket`, `CompiledPolicyPacket`, `PreflightResult`, `PreflightEvidenceTraceResult`, `PreflightRegenerationResult`, `EvalRunResult`, `RuntimeDecisionResult`, `RuntimeExecutionResult`, `RuntimeEvidenceSessionSummary`)
- MCP-Tool-Ergebnisse mit identischen Typformen wie CLI
- HTTP-JSON: `/healthz` (`HealthCheckResult`), `/mcp`-Fehler (`401/403/429` mit `{"error": ...}`)
- Eval-Artefakte: JSON + HTML unter `data/evals/workspace/`, Phoenix-Spans
- Release-Artefakte: Wheel, sdist, Standalone-Bundles + `SHA256SUMS`

## Zustände

- **Konfigurationszustände**: Entwicklung (Checkout-`.env`), Standalone (Plattform-Config), gehostet (Railway `PORT` injiziert, `0.0.0.0`-Bind)
- **Indexzustände**: fehlend/leer → explizite Fehler oder `503`/Startup-Fail-fast; vorhanden → bereit
- **Grounding-Zustände**: `insufficient_context=true/false` als Result-Flag (kein Setup-Fehler)
- **Runtime-Entscheidungen**: `allow`, `confirm`, `block` (allow ist No-Match-Ergebnis, nicht authored effect)
- **Execution-Outcomes**: `allowed`, `confirmed`, `blocked`, `refused`, `failed`
- **Beta-Accounts**: `active`, `suspended`
- **Auth-Entscheidungen**: `authorized`, `unauthorized`, `suspended`, `quota_exceeded`; Quellen `api_key`/`break_glass`
- **Regenerations-Stops**: `passed`, `max_regenerations`, `no_material_trigger`, `insufficient_context`

## Sicherheitsrelevanz

- Fail-closed-Philosophie durchgängig: Konfiguration, Index, Zitat-Validierung, generierte Constraints
- Secrets-Handling: `NVIDIA_API_KEY` nie in Logs/Artefakten; Init-Config atomar mit Dateirechten; Audit-Log redigiert Secret-Werte (belegt durch `docs/workflows.md` und `storage/auth_store.py`-Analyse)
- Hosted-Auth schützt nur `/mcp`; `/healthz` bleibt öffentlich; Bearer-Token-Vergleich mit Konstantzeitvergleich (zu verifizieren in `interfaces/mcp.py`)
- Beta-Portal: GitHub-OAuth, API-Key-Hashing, Tagesquota, Login-Rate-Limit, Session-Cookie-Signatur (`itsdangerous`)
- Public-Endpoint-Regeln (Policy `SEC-PUBLIC-ENDPOINT-002`): keine rohen Exceptions, Pfade, Token in öffentlichen Antworten
- Runtime-Execution: nur bereinigte Aktionen, Shell-Timeout (`POLICYNIM_RUNTIME_SHELL_TIMEOUT_SECONDS`), Exit-Codes für blocked/refused/failed

## Lizenz

MIT License, Copyright (c) 2026 Nnenna Ndukwe (belegt durch `LICENSE`).
