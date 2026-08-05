# nim-proxy — Testanalyse

Testlandschaft: 69 Unit-Tests + 53 e2e-Tests (belegt durch CONTRIBUTING.md), Coverage-Gate ≥ 90 % Lines (belegt durch ci.yml), Fuzz-Targets (fuzz.yml), Load-Harness (scripts/). Alle Aussagen belegt durch tests/e2e.rs (3731 Zeilen, ~97 Testfunktionen), tests/support/mod.rs, fuzz/, scripts/.

---

## Testarchitektur

- Was wird getestet: Der komplette Server-Stack inkl. Proxy-Verhalten, Auth, Dashboard-Verträge, Retention, Shedding, Setup, Security-Header, Rate-Limit-Invarianten (belegt durch tests/e2e.rs).
- Wie wird getestet: e2e-Suite startet die reale Binary gegen einen scriptbaren Mock-NIM (tests/support/mod.rs, scripts/mock_nim.py); Unit-Tests in den src/-Modulen (belegt durch CONTRIBUTING.md "launches the real binary against a scriptable mock").
- Welche Annahmen gelten: Fail-closed-Postur (Keyed-Default); deterministischer Mock-Upstream; tempdir-DATA_DIR-Isolation pro Test; Subprocess-Coverage-Forwarding im CI (belegt durch ci.yml).
- Welche Edge Cases existieren: Überlast-Shedding (503), Retention-Prune, Session-Expiry, Label-Sanitizing, Setup-Einmal-Minting, Pool-Lane-Contracts, Security-Header/CSP (belegt durch e2e-Testnamen).
- Welche Invarianten werden geprüft: Zero-Upstream-Violations (Hard-Requirement laut CONTRIBUTING.md); Dashboard-Verträge (from/to/points); Revision-basierter Reset; 0600-Dateirechte; API-Key-Gate (belegt durch e2e-Testnamen).

## E2E-Tests (tests/e2e.rs) — nach Capability gruppiert

### Dashboard-Verträge

- `dashboard_now_contract_uses_current_pool_config_and_registry`: prüft /api/dashboard/now gegen den aktuellen Pool-Stand (Konfig lanes=3, rpms 40, capacity_rpm 120) — belegt Pool-Vertrag (e2e, erwähnt in früherer Lektüre).
- `dashboard_contract_*` (Zeitfenster/Verträge): prüft from/to/points-Semantik und Response-Shape.
- Invarianten: Revisions-Felder vorhanden; Punkte-Zahl; Sample-Struktur.
- Rust-Relevanz: Vertrags-Tests für die Dashboard-API des Rewrite (JSON-Shape-Assertions).

### History & Retention

- `retention_change_prunes_queries_and_disk`: ändert retention_days → prüft, dass alte Samples entfernt werden und die Datei schrumpft.
- Invarianten: 1-MB/100k-Limits; v2-Boot-Marker; history_revision-Inkrement.
- Rust-Relevanz: Retention- und Datei-Limit-Tests für das Rewrite (tempdir-Fixtures).

### Rate-Limit & Shedding

- `overloaded_requests_are_shed_with_503`: überlastete Anfragen werden mit 503 abgewiesen (max_inflight/Shedding).
- Rate-Limit-Tests: prüfen, dass der Proxy nie mehr Anfragen an den Upstream durchlässt als das per-Key-Limit.
- Invarianten: GRANT_GAP, Waiter-Timeout, Shedding-Verhalten.
- Rust-Relevanz: Shedding-/Rate-Limit-Tests mit Mock-Upstream (Violation-Zähler).

### Auth & Setup

- `setup_can_mint_a_first_client_key`: Setup-Flow erzeugt ersten Client-Key (Einmal-Sichtbarkeit).
- Auth-Tests: Login/Logout, Session-TTL, Scraper-Basic-Auth, API-Key-Gate (401/403), Fail-closed ohne Key.
- Invarianten: Cookie-Name nimproxy_session; PBKDF2-Format; Throttle-Verhalten.
- Rust-Relevanz: Auth-Vertrags-Tests (Cookie, Basic, Keyed-Mode).

### Security-Header & Injection

- Tests prüfen Security-Headers, CSP, XSS-Schutz (0.3.0-Fix: XSS-Kette), Metrik-Sanitizing-Assertion (Labels korrekt bereinigt).
- Invarianten: Keine injizierten Label-Werte in Metriken; keine unsanitisierten HTML-Fragmente.
- Rust-Relevanz: Security-Header-Assertions + sanitize_label-Property-Tests.

### Streaming & Tool Calling

- Tests decken SSE-Streaming, Heartbeat, Stream-Idle-Timeout, Tool-Calling-Propagation (payload mit tools), Usage-Injection mit Auto-Fallback ab.
- Invarianten: Heartbeat-Intervall (10s); strict_passthrough-Modus deaktiviert Injection.
- Rust-Relevanz: Streaming-Integrationstests gegen Mock-NIM (SSE-Event-Formate).

### Mock-Verhalten (tests/support/mod.rs, scripts/mock_nim.py)

- Mock steuert /v1/models, /v1/chat/completions, /control/stats (Violation-Erzwingung, Sliding-Window-Check).
- Annahmen: Deterministische 429-Antworten; Zähler-Statistik; --enforce-Modus für Load-Tests.
- Rust-Relevanz: Mock-NIM als Test-Helfer im Rewrite (eigener Mock-Server-Crate oder Testmodul).

## Unit-Tests

- Was wird getestet: Modulinterne Logik (Config-Parsing, Governor-Zustandsmaschine, Pool-Lane-Zählung, sanitize_label, PBKDF2-Hashing, History-Sampling) (belegt durch src/*.rs-Testmodule).
- Welche Annahmen gelten: Isolierte Module ohne Server.
- Welche Edge Cases existieren: Config-Schema-Varianten, Label-Grenzen (64), Alphabet-Filter, Rate-Zähler.
- Welche Invarianten werden geprüft: Fail-closed-Defaults, 0600-Permissions, PBKDF2-Format, Revision-Inkremente.
- Rust-Relevanz: Unit-Tests direkt ins Rewrite übertragbar (Modulinterne #[cfg(test)]-Module).

## Load-Harness

- Was wird getestet: Zero-Violations unter Last (100 Clients, 0 Upstream-Violations laut README) (belegt durch scripts/loadtest.py, scripts/mock_nim.py --enforce).
- Welche Annahmen gelten: Realistische Payload-Varianz (Tools jede 3., json_object jede 5. Anfrage).
- Welche Edge Cases existieren: Burst, Multi-Key-Pool, Affinity sticky/spill.
- Welche Invarianten werden geprüft: Kein einziger Request überschreitet das per-Key-Limit.
- Rust-Relevanz: Bench-Binary/Harness im Rewrite; das Zero-Violations-Gate bleibt Hard-Requirement.

## Fuzz-Tests

- Was wird getestet: SSE-Scanner (sse_scan), Label-Sanitizing (sanitize_label), Config-Roundtrip (config_roundtrip) (belegt durch fuzz.yml, fuzz/fuzz_targets/).
- Welche Annahmen gelten: Seeds decken repräsentative Eingaben ab (stream-with-usage, malformed, truncated-mid-json; minimal.json, store.json; hostile, model-id, model-with-colon).
- Welche Edge Cases existieren: Kaputte SSE-Segmente, injektionsartige Labels, minimale/maximale Configs.
- Welche Invarianten werden geprüft: Keine Panics, keine Endlos-Loops, kein Deserialize-Datenverlust.
- Rust-Relevanz: Fuzz-Targets direkt ins Rewrite übernehmbar (cargo-fuzz); Seeds als Test-Fixtures.

## CI-Gates

- fmt, clippy -D warnings, cargo test (check-Job); Coverage ≥ 90 % Lines (coverage-Job, llvm-cov, Subprocess-Coverage); MSRV 1.87 (msrv-Job); cargo-deny (deny-Job); gitleaks (fetch-depth 0); actionlint + zizmor (lint-workflows-Job); Dependency-Review (nur PRs); Docker-Build + Smoke (docker-Job) (belegt durch ci.yml).
- Rust-Relevanz: Gleiche Gates für das Rewrite-Projekt (Coverage-Schwelle, MSRV-Job, Deny, Secrets, Workflow-Lint).
