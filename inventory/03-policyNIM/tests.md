# Test-Analyse: PolicyNIM

Beleg-Regel: Jede Aussage ist durch die genannte Quelldatei gedeckt. Nicht Belegbares: "Nicht nachweisbar."

## Bestand an Testdateien

Das Repo enthält 40 pytest-Dateien unter `tests/` plus `tests/README.md` (belegt durch `find`). Zeilenzahlen via `wc -l` (belegt):

| Datei | Zeilen | Schwerpunkt |
|---|---|---|
| tests/README.md | 88 | Testplan, Marker, Opt-in-Env-Variablen |
| test_cli.py | 4252 | CLI-Verträge (init, doctor, quickstart, mcp-config, mcp-smoke, preflight, eval, beta-admin, runtime, evidence) |
| test_mcp.py | 1066 | MCP-Surface (policy_preflight, policy_search) + Hosted-HTTP-Runtime |
| test_settings_and_types.py | 972 | Konfigurationspräzedenz, Settings-/Types-Verträge |
| test_preflight_service.py | 756 | Grounding-Pipeline, Fail-closed-Matrix, Chunk-Kappung |
| test_runtime_decision_service.py | 595 | Runtime-Entscheidungen, Regel-Matching |
| test_runtime_execution_service.py | 560 | Ausführung, Confirmation, Redaction, Outcomes |
| test_policy_regeneration_service.py | 542 | Regenerations-Retries, Limits, Drift |
| test_eval_service.py | 515 | Eval-Workflow (offline/live), Artefakte |
| test_runtime_evidence_report_service.py | 449 | Session-Berichte (JSON/Markdown) |
| test_ingest.py | 431 | Korpus-Ingest, Chunking-Edge-Cases |
| test_service_factories.py | 403 | Factory-/DI-Verdrahtung |
| test_nemo_guardrails_preflight_generator.py | 388 | Guardrails-Wrapper (Output-Rails) |
| test_health_service.py | 387 | Health-Checks (local/hosted) |
| test_beta_portal.py | 374 | Beta-Portal-Rendering (Landing/Dashboard) |
| test_nvidia_policy_compiler.py | 324 | Compiler-Provider |
| test_compiler_service.py | 303 | Compile-Service |
| test_router_service.py | 294 | Router, TaskProfile, Evidenztiefe |
| test_sqlite_vec.py | 287 | sqlite-vec-Index, KNN, Domänen-Budget |
| test_policy_evidence_trace_service.py | 285 | Trace-Materialisierung, CLI-Output |
| test_search_service.py | 264 | Such-Pipeline |
| test_runtime_paths.py | 281 | Pfad-Auflösung (standalone/checkout) |
| test_nvidia_policy_conformance.py | 280 | Conformance-Judge-Provider |
| test_nvidia_reranker.py | 274 | Rerank-Provider |
| test_policy_conformance_service.py | 272 | Conformance-Service |
| test_runtime_evidence_store.py | 272 | Evidence-SQLite-Schema, Concurrency |
| test_installer_contract.py | 258 | Installer-/Bundle-Verträge |
| test_nvidia_generator.py | 240 | Generator-Provider |
| test_auth_store.py | 232 | Beta-Auth-SQLite (Keys, Audit, Quota) |
| test_docs_runtime_workflows.py | 205 | Docs-Workflow-Contracts |
| test_beta_auth.py | 149 | BetaAuthService |
| test_ci_workflows.py | 142 | CI-Workflow-Contracts |
| test_docs_hosted_onboarding.py | 140 | Hosted-Onboarding-Docs |
| test_hosted_mcp_live.py | 130 | Live-Hosted-MCP (Marker `live`) |
| test_nvidia_embedder.py | 114 | Embedder-Provider |
| test_nemo_agent_toolkit_policy_conformance.py | 112 | NAT-Eval-Adapter-Gating |
| test_nemo_evaluator_policy_conformance.py | 110 | NeMo-Eval-Adapter-Gating |
| test_docker_build_live.py | 149 | Docker-Build (Marker `docker_live`) |
| test_dockerfile_contract.py | 68 | Dockerfile-Strukturverträge |
| test_package_release.py | 71 | Release-Dependency-Contracts |
| test_runtime_evidence_report_service.py | 449 | (siehe oben) |

## Testplan (tests/README.md, belegt)

- Abgedeckt: Parsing/Chunking-Edge-Cases (leere Sektionen, wiederholte verschachtelte Überschriften), isolierte Live-Eval-Index-Handhabung, Ingest, Search, Runtime, Eval, Preflight, Routing, Compilation, Conformance, Trace, Regeneration, NeMo-Gating, Guardrails, NVIDIA-Validierung, CLI-/MCP-/Hosted-/Docker-Contracts, Live-Smokes.
- Marker-Semantik: `live` (echte NVIDIA-Aufrufe, benötigt `NVIDIA_API_KEY`) und `docker_live` (Docker-Build, benötigt `POLICYNIM_RUN_DOCKER_TESTS`).
- Opt-in-Env-Variablen: `POLICYNIM_RUN_DOCKER_TESTS`, `POLICYNIM_BETA_MCP_URL`, `POLICYNIM_BETA_MCP_TOKEN`, `NVIDIA_API_KEY`.
- Docker-Erwartung: BuildKit-Secret statt Build-Arg für Secrets.

## Was wird getestet (Auswahl je Datei, belegt durch Dateiinhalte)

- **test_cli.py**: `init` (interaktives Setup, `NVIDIA_API_KEY`-Prompt, `--no-interactive` nicht vorhanden), `doctor` (JSON-Payload, Standalone ohne Provider-Aufrufe), `--version` (Fehlerpfad: Metadaten nicht verfügbar → Exit 1, stderr "Installed package metadata for PolicyNIM is unavailable." ohne Traceback), `mcp-config --client claude-code` (Checkout-JSON: `config.mcpServers.policynim` mit `{type: stdio, command: uv, args: [run, --directory, <checkout>, policynim, mcp, --transport, stdio], env: {NVIDIA_API_KEY: \${NVIDIA_API_KEY}}}`, `next_steps` mit "uv run policynim doctor"/"uv run policynim mcp-smoke"), Codex-local-stdio (via `configure_standalone_cli_environment`), `preflight`/`eval`/`ingest`-Output-Verträge, `beta-admin`, `runtime decide/execute`, `evidence report`.
- **test_mcp.py**: MCP-Tools `policy_preflight`/`policy_search` (top_k-Validierung, Domänen-Parameter, JSON-Shape), Hosted-HTTP-Runtime (streamable-http, Bearer-Auth-Status 401/403/429, Healthz 200/503).
- **test_preflight_service.py**: Mock-Komponenten (`MockEmbedder` mit Mapping "refresh token cleanup"→[1.0,0.0], "backend guidance"→[0.0,1.0], "missing citations"→[-1.0,-1.0]; `MockReranker(order=[...])`; `MockGenerator`; `MockPolicyCompiler` mit `insufficient_context`-Draft; `MockIndexStore(chunks, exists=True)`); Happy Path, Fail-closed bei Compiler-insufficient, `_MAX_CHUNKS_PER_POLICY`-Kappung (2 Chunks/Policy), unbekannte Zitations-IDs → insufficient, Dedupe + First-seen-Ordnung (SECURITY-1, BACKEND-1), Policy-Zitations-Disagree → invalid, Fallback auf Policy-level-Citations, fehlender Index → `MissingIndexError`, `close()`-Semantik.
- **test_settings_and_types.py**: mcp_port-Default 8123; `POLICYNIM_MCP_PORT` schlägt `PORT`; production + PORT → `0.0.0.0`; explizites `POLICYNIM_MCP_HOST` bleibt in production; User-Config ignoriert bei Plattform-Port; Env-/Datei-Loading; Modellvalidierungen.
- **test_nvidia_embedder/generator/reranker/policy_compiler/policy_conformance.py**: Provider-Verträge (Request-Aufbau, Response-Validierung, Fehlerklassen, closed-Semantik, Malformed-Payloads).
- **test_nemo_guardrails_preflight_generator.py**: Guardrails-Wrapper (Output-Rails passed/modified/blocked, Zitat-Validierung, Asset-Loading, fail-closed bei blocked).
- **test_nemo_evaluator/agent_toolkit_policy_conformance.py**: Optional-Package-Gating (`nemo-evaluator`/`nvidia-simple-evals` bzw. `nvidia-nat-eval`), Fehlerpfad bei fehlendem Paket, Evaluator-Delegation.
- **test_sqlite_vec.py**: Schema (policy_chunks/policy_vectors/index_metadata), Schema-Version, exists/count, KNN mit Domänen-Filter, Kandidaten-Budget (`_DOMAIN_CANDIDATE_MULTIPLIER`/`_MIN_DOMAIN_CANDIDATES`), Fehlerpfade (fehlende Extension).
- **test_runtime_evidence_store.py**: SQLite-Schema, Event-/Execution-Reihenfolge, Reopen, Concurrency, Session-Summaries.
- **test_runtime_decision_service.py**: Kind-Validierung, Regel-Matching, allow/confirm/block, Zitations-Verknüpfung, fail-closed.
- **test_runtime_execution_service.py**: Confirmer-Callback-Erfolg/Fehler, Redaction, durable Evidence, failure_class (non_zero_exit, confirmation_unavailable), Outcomes.
- **test_eval_service.py**: offline/live-Modi, Rerank-Vergleich, Conformance-Scoring, Artefakte (JSON/HTML), Phoenix-UI-Option.
- **test_service_factories.py**: alle `create_*`-Factories (Verdrahtung, close, Fehlerfälle).
- **test_health_service.py**: local/hosted Health-Checks, `HealthCheckResult`, Fallback-Reason.
- **test_beta_portal.py**: Landing/Dashboard-Rendering, Notices, Command-Cards, Usage-Bar.
- **test_auth_store.py**: Accounts, API-Key-Hashing, Audit-Events, Tagesquota.
- **test_beta_auth.py**: GitHub-OAuth-URLs, `authenticate_api_key`, `issue_api_key`, Token-Entscheidungen.
- **test_package_release.py**: `sqlite-vec==0.1.9` in Runtime-Dependencies; KEIN lancedb/-Namespace in Runtime/Optional; KEIN Extra `hosted-legacy-index`.
- **test_installer_contract.py**: Installer-/Standalone-Bundle-Verträge (Struktur, Pfade, SHA256SUMS).
- **test_dockerfile_contract.py** / **test_docker_build_live.py**: Dockerfile-Struktur (Staging, Non-Root, Secrets via BuildKit) und Live-Build (Marker `docker_live`).
- **test_ci_workflows.py**: CI-Workflow-Verträge (Jobs, Marker-Auswahl, Matrix).
- **test_docs_hosted_onboarding.py** / **test_docs_runtime_workflows.py**: Konsistenz der Docs-Workflows mit den tatsächlichen Kommandos.
- **test_hosted_mcp_live.py**: Live-Endpunkt (`POLICYNIM_BETA_MCP_URL` + `POLICYNIM_BETA_MCP_TOKEN`), Marker `live`.

## Annahmen und Edge-Cases (belegt durch Testinhalte)

1. **Fail-closed ist Kern-Invariante**: Bei Compiler-`insufficient_context` wird der Generator nie aufgerufen (`generator.last_request is None`), das Result meldet `insufficient_context=true` und `plan_steps == []` (test_preflight_service.py).
2. **Zitations-Grounding**: Unbekannte Zitations-IDs und leere Zitationslisten führen zu `insufficient_context`/invalid — nie zu ungrounded Antworten (test_preflight_service.py).
3. **Kappungs-Invariante**: Pro Policy werden maximal `_MAX_CHUNKS_PER_POLICY` Chunks im Kontext behalten (test_preflight_service.py).
4. **Konfigurations-Präzedenz**: Plattform-Port (`PORT`) wird von `POLICYNIM_MCP_PORT` überschrieben; User-Config wird bei Plattform-Port ignoriert (test_settings_and_types.py).
5. **Optional-Package-Gating**: Eval-/Guardrails-Backends sind nur mit den jeweiligen Extras verfügbar; fehlende Pakete ergeben `ConfigurationError` mit `missing_optional_dependency` (test_nemo_*).
6. **Runtime-Ausführung**: Confirmation ist Pflicht für `confirm`-Entscheidungen; Fehler im Confirmer → `confirmation_unavailable`; nicht-bereinigte/geblockte Aktionen werden nie ausgeführt (test_runtime_execution_service.py).

## Invarianten, die in Rust-Tests gefasst werden müssen

- Fail-closed-Matrix: keine Ausgabe ohne validierte Zitate/Kontext; `insufficient_context` ist Result-Zustand, kein Setup-Fehler.
- Kontext-Budget: `_MAX_CHUNKS_PER_POLICY`-Kappung + First-seen-Dedupe-Ordnung bleiben stabil.
- Schema-Versionierung: Index-Schema und Evidence-Schema sind versioniert; Reopen muss stabil sein.
- Config-Präzedenz: `POLICYNIM_*`-Env > `PORT` (Plattform) > User-Config > Defaults.
- Provider-Retry: max_retries+1 Versuche, Retry nur auf 429/5xx/Timeout/Connection; 401/403 → auth-Fehler.

## Benötigte Rust-Tests (für die spätere Neuentwicklung)

1. **Config-Präzedenz-Matrix** (test_settings_and_types.py-Analog).
2. **Fail-closed-Preflight-Matrix** (test_preflight_service.py-Analog: alle Zitations-/Kontext-/Compiler-Kombinationen, Kappung, Dedupe).
3. **Provider-Mock-Tests**: Embeddings (Batch, `input_type`, `truncate=NONE`, Dimension-Check), Rerank (Payload, Score-Extraktion aus List/Dict-Formaten), Chat (temperature=0/top_p=1, JSON-Parsing inkl. eingebettetem JSON, Leer-Response).
4. **sqlite-vec-Integration**: Schema, KNN, Domänen-Budget, Extension-Fehler.
5. **Runtime-Entscheidungs-Matrix**: Regel-Matching, Zitations-Verknüpfung, allow/confirm/block.
6. **Execution-Outcome-Matrix**: Outcomes, Confirmer-Vertrag, Redaction, durable Evidence.
7. **Guardrails-Vertrag**: RailStatus passed/modified/blocked, Zitat-Validierung nach Rail.
8. **Conformance-Validierung**: unsupported constraint/chunk ids → `invalid_response`.
9. **Regenerations-Limits**: max_regenerations/insufficient_context-Stops, Paketidentität (compile-once), Zitations-Drift-Ablehnung.
10. **Factory-/DI-Graph**: alle Konstruktoren aus Settings, close-Semantik.
