# nim-proxy — Rust-2024-Rewrite-Foundation

Capability-basierte Grundlage für die eigenständige Rust-2024-Neuentwicklung. Kein Python→Rust-Mapping, sondern: benötigte Capability → benötigte Rust Traits → Module → Services → Events → Storage → Async → Error Types → Crates. Alle Aussagen aus der Quellcode-Analyse (Belege in Klammern).

---

## 1. Kern-Capabilities (aus der Analyse)

| Capability | Beleg | Anmerkung |
|---|---|---|
| Fail-closed API-Key-Gate (Keyed-Default) | src/config.rs, CHANGELOG [0.3.0] | Kein Key → kein Upstream-Zugriff |
| Per-Key-Rate-Limit (Sliding Window, 40 rpm Default) | src/pool.rs, store.json-Seed | Null-Violations-Versprechen |
| Governor: Poll/Backoff/Grow/Dissolve | src/governor.rs | 250ms/2s/60s/30min |
| Sticky-Affinity mit Spillover | decisions/sticky-affinity-with-spillover.md | nimproxy_affinity_total |
| Slot-Dispatch mit Generation-Check | src/dispatch.rs | Shedding 503 bei Überlast |
| SSE-Streaming + Heartbeat + Idle-Timeout | src/proxy.rs, decisions/sse-heartbeats-for-rate-waits.md | 10s Heartbeat, 120s Idle |
| Usage-Injection mit Auto-Fallback | decisions/usage-injection-auto-fallback.md | strict_passthrough schaltet ab |
| Label-Sanitizing (max 64, Alphabet) | src/proxy.rs, fuzz/sanitize_label | Metrik-Injection-Schutz |
| Dashboard + History (300s-Sampling, 1 MB/100k, Retention) | src/history.rs, src/dashboard.html | v1-Legacy + v2-Marker |
| Multi-User-Auth (PBKDF2 600k, Sessions 12h) | src/auth.rs, CHANGELOG [0.6.0] | Scraper-Basic-Auth |
| Config-Store (v1, atomar, 0600, Commit-Pipeline) | src/config.rs, src/settings.rs | Revisionen, Pool-Rebuild-Carryover |
| Setup-Wizard (Einmal-Minting) | src/setup.html, e2e | Erst-Config + erster Key |
| Prometheus-Metriken | README-Metrik-Tabelle, src/*.rs | metrics-Stack |
| Supply-Chain/CI-Gates | ci.yml, release.yml, audit.yml | fmt/clippy/coverage/deny/gitleaks/zizmor |

## 2. Benötigte Rust Traits

- `trait ConfigStore { fn load() -> Result<StoredConfig>; fn save(&self) -> Result<()>; fn revision(&self) -> u64; }` — persistente Config mit Revisionen (Beleg: src/config.rs).
- `trait RateLimiter { fn try_reserve(&mut self, now: Instant) -> Result<Reservation>; }` — Sliding-Window-Logik (Beleg: src/pool.rs).
- `trait GovernorPolicy { fn next_action(&mut self, state: &PoolState) -> Option<PoolAction>; }` — Poll-Entscheidungen (Beleg: src/governor.rs).
- `trait Affinity { fn pick_lane<'a>(&self, model: &str, lanes: &'a [Lane]) -> Option<&'a Lane>; }` — sticky/spill-Routing (Beleg: src/pool.rs).
- `trait LabelSanitizer { fn sanitize(&self, label: &str) -> Cow<str>; }` — reine Funktion, fuzzbar (Beleg: src/proxy.rs).
- `trait SseParser { fn feed(&mut self, bytes: &[u8]) -> Vec<SseEvent>; }` — inkrementelles SSE-Parsing (Beleg: fuzz/sse_scan.rs).
- `trait UsageInjector { fn inject(&self, response: &mut Response, usage: &Usage); }` — Auto-Fallback-Injection (Beleg: decisions/usage-injection-auto-fallback.md).
- `trait SessionStore { fn create(&mut self, u: &User) -> Session; fn validate(&self, cookie: &str) -> Option<Session>; fn remove(&mut self, cookie: &str); }` (Beleg: src/auth.rs).
- `trait HistorySink { fn record(&mut self, event: HistoryEvent); fn prune(&mut self, before: Instant); fn query(&self, from: Instant, to: Instant, points: u32) -> Vec<Sample>; }` (Beleg: src/history.rs).
- `trait PasswordHasher { fn hash(&self, pw: &[u8], salt: &[u8], iters: u32) -> String; fn verify(&self, pw: &[u8], encoded: &str) -> bool; }` — PBKDF2-600k (Beleg: src/auth.rs).
- `trait JsonError { fn to_json_error(&self) -> Value; }` — `{error:{message,type:"proxy_error",code}}` (Beleg: src/settings.rs).

## 3. Benötigte Module

- `config` (StoredConfig, nim_mode, Limits, Pricing, NimKey, ClientKey, User)
- `auth` (Login/Logout, Session, Scraper-Basic, Key-Gate)
- `pool` (Lane, LaneSpec, Pool, Reservation, Affinity)
- `governor` (Poll-Cycle, Backoff, Grow, Dissolve, ModelPermit-RAII)
- `dispatch` (Slot, Generation, Waiter, Shedding)
- `proxy` (Chat-Forward, SSE, Usage-Injection, sanitize_label, Deadline)
- `history` (Sampling, JSONL-Writes, Retention, Revisionen)
- `dashboard` (API-Handler, Settings-PUT, Setup-Wizard)
- `metrics` (Prometheus-Export, Label-Const, Registry)
- `server` (axum-Router, State, Fallback, Healthcheck)

## 4. Benötigte Services

- `ConfigService` — Commit-Pipeline: validate → save → cfg-write → Pool-Rebuild (Rate-Carryover) → Retention → guard → revision++ (Beleg: src/settings.rs).
- `GovernorService` — periodische Poll-Task (250ms) mit Backoff/Grow/Dissolve (Beleg: src/governor.rs).
- `DispatcherService` — Slot-Vergabe mit GRANT_GAP 25ms, Waiter-Deadlines, Shedding (Beleg: src/dispatch.rs).
- `UpstreamService` — reqwest-Client mit rustls, Timeouts, Streaming (Beleg: src/proxy.rs, Cargo.toml).
- `HistoryService` — Event-Sink, Sampler (300s), JSONL-Writer (atomar, 1 MB/100k), Retention-Prune (Beleg: src/history.rs).
- `MetricsService` — Prometheus-Registry, Label-Sanitizing-Anbindung (Beleg: README-Metrik-Tabelle).
- `DashboardService` — Query-Handler (from/to/points=288, /now), Settings-Validation (Beleg: src/dashboard.html, src/history.rs).
- `SetupService` — Einmal-Init: Config + erster User + Client-Key-Minting (Beleg: src/setup.html, e2e).

## 5. Benötigte Events

- `ConfigChanged { revision: u64 }` — Pool-Rebuild, Retention-Prune, History-Reset-Erkennung (Beleg: src/settings.rs, decisions/reset-aware-dashboard-history.md).
- `RateLimitExceeded { lane: String, model: String }` — Governor-Anpassung + Metrik (Beleg: src/governor.rs).
- `SlotAcquired / SlotReleased { generation: u64 }` — Queue-Depth-Metriken (Beleg: src/dispatch.rs).
- `StreamStarted / StreamIdle / StreamTimeout` — History + Metriken (Beleg: src/proxy.rs).
- `UsageInjected { model: String, tokens: u64 }` — Token-Metriken (Beleg: decisions/usage-injection-auto-fallback.md).
- `LoginSucceeded / LoginFailed { user: String }` — Session + Throttle + Audit (Beleg: src/auth.rs).
- `HistoryReset { history_revision: u64, config_revision: u64 }` — Dashboard-Neu-Load (Beleg: src/history.rs, src/dashboard.html).
- `RetentionPruned { removed: u64, bytes_freed: u64 }` — Disk-Verwaltung (Beleg: e2e retention_change_prunes_queries_and_disk).

## 6. Benötigte Storage Layer

- `config.json` — StoredConfig v1, 0600, atomar via tmp+rename, korrupt = harter Boot-Fehler (Beleg: src/config.rs).
- `history.jsonl` — JSON Lines, v1-Legacy + v2-Boot-Marker, 1-MB/100k-Limits, atomare Writes (Beleg: src/history.rs).
- `DATA_DIR` — Env-gesteuert (Default data), unbeschreibbar = Boot-Fehler (Beleg: README, Dockerfile ENV DATA_DIR=/data).
- Keine Datenbank; kein weiterer Persistenz-Layer (Beleg: Repo-Inventar 113 Dateien).

## 7. Benötigte Async-Komponenten

- Tokio-Runtime (rt-multi-thread, macros, signal, time, sync) (Beleg: Cargo.toml).
- Governor-Poll-Loop (250ms Intervall) als `tokio::spawn`-Task (Beleg: src/governor.rs).
- SSE-Stream-Pipeline (reqwest-stream → Channel/StreamExt → Client) mit Heartbeat-Intervall (Beleg: src/proxy.rs).
- Waiter-Scheduler mit Deadlines (tokio::time::timeout/interval) (Beleg: src/dispatch.rs).
- Graceful Shutdown (SIGINT/SIGTERM) via axum-Server (Beleg: src/main.rs, docker-compose restart).
- Atomic Config-Swap für Pool-Rebuild ohne Blockierung (ArcSwap/RwLock) (Beleg: src/pool.rs, src/settings.rs).

## 8. Benötigte Error Types

- `ConfigError` (korrupte JSON, fehlende Datei, ungültige Werte, Persistenz-Fehler) → Boot-Abbruch (Beleg: src/config.rs).
- `AuthError` (401/403, Throttle, Session-Invalid) (Beleg: src/auth.rs).
- `RateLimitError` (Shedding 503, Timeout) (Beleg: src/dispatch.rs).
- `UpstreamError` (5xx, 429, Timeout, Payload-Too-Large) (Beleg: src/proxy.rs).
- `HistoryError` (Datei-Limits, Schreibfehler) (Beleg: src/history.rs).
- Einheitliches HTTP-Mapping → json_error-Shape (Beleg: src/settings.rs).
- Fehler-Enum als `enum ProxyError` mit `IntoResponse`-Implementierung (Rust-Konzept).

## 9. Benötigte Crates

- axum 0.8 (Router/Extractors), tokio 1 (full), reqwest 0.12 (rustls-tls, stream, gzip), serde/serde_json, metrics 0.24 + metrics-exporter-prometheus 0.18, bytes, futures-util, tokio-stream, tracing + tracing-subscriber (env-filter), dotenvy 0.15, hmac 0.12, sha2 0.10, subtle 2, getrandom 0.2 (Beleg: Cargo.toml).
- Dev/Test: cargo-llvm-cov, cargo-deny, cargo-fuzz (nightly), gitleaks, actionlint, zizmor (Beleg: ci.yml, fuzz.yml).
- Kein native-tls (nur rustls); keine Web-Framework-Erweiterungen (Beleg: Cargo.toml reqwest default-features=false).

## 10. Architekturentscheidung

- Monolithischer axum-Server mit klarer Modul-Trennung (kein Microservice-Split) (Beleg: src/lib.rs, ein Binary).
- In-Memory-Pool/Governor/Sessions; Persistenz nur für Config + History (Beleg: Repo-Struktur).
- Fuzz-Targets direkt an interne Funktionen über `#[cfg(fuzzing)]`-Re-Exporte (Beleg: src/lib.rs).
- Fail-closed als Default-Postur überall (Auth, Config, Gate) (Beleg: CHANGELOG [0.3.0]).
- Deterministische Testbarkeit: scriptbarer Mock-Upstream, tempdir-Isolation, Subprocess-Coverage (Beleg: tests/, ci.yml).

## Bewährte Architekturentscheidungen des Referenzprojekts (für das Rewrite zu übernehmen)

1. Sliding Window mit Jitter-Marge (61s) statt Token-Bucket — belegt durch decisions/sliding-window-not-token-bucket.md, decisions/window-jitter-margin.md.
2. Sticky-Affinity mit Spillover für Multi-Key-Pools — decisions/sticky-affinity-with-spillover.md.
3. Generator-Generation-Check für Slot-Validität (Pool-Rebuild-sicher) — src/dispatch.rs.
4. SSE-Heartbeats für Rate-Waits (Verbindungsstabilität) — decisions/sse-heartbeats-for-rate-waits.md.
5. Usage-Injection nur bei fehlendem usage, mit strict_passthrough-Schalter — decisions/usage-injection-auto-fallback.md.
6. Config-Store mit Revisionen statt Env-Config (ab 0.6.0) — decisions/ui-managed-config-store.md, CHANGELOG [0.6.0].
7. history_revision/config_revision für Reset-Erkennung im Dashboard — decisions/reset-aware-dashboard-history.md.
8. Rate-Carryover beim Pool-Rebuild (keine Violations bei Config-Änderung) — src/settings.rs, e2e.
9. PBKDF2 mit 600k Iterationen + Konstantzeit-Vergleich (subtle) für Auth — src/auth.rs.
10. distroless-scratch-Image mit non-root (10001), static musl, Selbst-Healthcheck — decisions/distroless-scratch-image.md, Dockerfile.
11. Release-Pipeline: cosign keyless sign + SLSA attest + SBOM am Manifest-Digest; gh-CLI statt Third-Party-Release-Action; Vertrags-Test — release.yml, scripts/test_release_contract.py.
12. Dependabot-Cooldown + Crypto-Stack-Major-Pins (hmac/sha2/getrandom) — .github/dependabot.yml, CHANGELOG [0.6.3].
13. Zero-Violations-Load-Harness als Pflicht-Gate für Rate-Limit-Änderungen — CONTRIBUTING.md, scripts/loadtest.py.
14. Wissensbasis (Open Knowledge Format, ADR-Decisions) im Repo — knowledge/, AGENTS.md.
