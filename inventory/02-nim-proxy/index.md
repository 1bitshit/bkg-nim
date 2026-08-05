# nim-proxy — Repo-Übersicht

Belegbasis: `/home/workspace/work/proxy/bkg-nim-studio/source/nim-proxy`

## Zweck

nim-proxy ist ein kleiner, rate-limit-bewusster, OpenAI-kompatibler Proxy für die NVIDIA-NIM-API. Er macht den NIM-Free-Tier (~40 RPM pro Key) für Agent-Harnesses nutzbar, indem er Requests auf das Per-Key-Rate-Limit paced, über mehrere Keys lastverteilt und Client-Verbindungen mit SSE-Heartbeats am Leben hält, während er auf einen Slot oder Upstream-Retry wartet (belegt durch `README.md`, `AGENTS.md`). Da der Proxy im Request-Pfad jeder Harness und jedes Modells sitzt, ist er zugleich ein Benchmarking- und Agent-Observability-Werkzeug: Er erfasst Tool-Intensität, Konversations-Tiefe, Sampling-Parameter, Truncation und Reasoning-Tokens — ausschließlich als Counts und Größen, nie als Nachrichteninhalt (belegt durch `README.md`, `knowledge/decisions/request-shape-metrics.md`).

Das Projekt ist eine Rust-Neuentwicklung (edition 2021, MSRV 1.87, Version 0.6.5, MIT), verteilt als signiertes Multi-Arch-Container-Image (`ghcr.io/miztertea/nim-proxy`) mit SBOM und Build-Provenance (belegt durch `Cargo.toml`, `LICENSE`, `.github/workflows/release.yml`).

## Verantwortlichkeit

Jede Datei hat eine klar abgegrenzte Verantwortlichkeit (Details in `files.md`):

- `src/main.rs` — Binary-Einstieg: Env-Parsing, Router-Aufbau, Graceful Shutdown, `--health`-Probe.
- `src/lib.rs` — Bibliotheks-Einstieg: Modulbaum, zentrale Typen (`Config`, `GovernorSettings`, `AppState`), Fuzz-Re-Exports.
- `src/proxy.rs` — HTTP-Proxy-Kern: `/v1/chat/completions` (Streaming + Buffered), `/v1/models`-Cache, `/health`, `/metrics`, SSE-Relay, Usage-Injection, Label-Sanitizing, API-Key-Gate.
- `src/pool.rs` — Key-Pool: ein Lane pro NIM-Key mit exaktem Sliding-Window-Limiter (40/60 s + 1 s Jitter-Marge), Affinity- und Spillover-Routing.
- `src/dispatch.rs` — globaler FIFO-Dispatcher: ein Warteschlangen-Slot pro Request, Fairness, Deadlines, `nimproxy_queue_*`-Metriken.
- `src/governor.rs` — Model-Pressure-Governor: adaptives per-Model-Concurrency-Limit gegen NIM-Worker-Exhaustion.
- `src/history.rs` — Metrik-History: 5-min-Snapshots, JSONL-Persistenz, Kompaktierung, Dashboard-API (`/api/dashboard`, `/api/dashboard/now`).
- `src/auth.rs` — Auth: Passwort-Hashing (PBKDF2-HMAC-SHA256, 600k Iterationen), Session-Cookies, Scraper-Auth, Login-Throttle, Konstantzeit-Vergleiche.
- `src/config.rs` — Konfig-Store: `StoredConfig` (Version 1), `NimKey`, `ClientKey`, `User`-Modelle, Boot-Ladung, atomare Writes.
- `src/settings.rs` — Settings-/Setup-API: Commit-Pipeline (validate → save → cfg-write → Pool-Rebuild → Retention), Wizard, rollenbasierte `/api/config`-Filterung.
- `src/dashboard.html` — eingebettetes Operator-Dashboard (eine HTML-Datei, kein Build-Schritt, 5 Tabs).
- `src/setup.html` — eingebetteter First-Run-Wizard.
- `tests/e2e.rs` + `tests/support/mod.rs` — End-to-End-Suite, die das echte Binary gegen ein scriptbares Mock-NIM bootet (97 Test-/Helfer-Funktionen).
- `knowledge/` — Open-Knowledge-Format-Bundle: Entscheidungen, validierte NIM-Forschung, Architektur-Notizen, Runbooks.
- `scripts/` — Load-Harness (`mock_nim.py` mit striktem Sliding-Window + Violations-Zähler, `loadtest.py`), Release-Vertrags-Test (`test_release_contract.py`).
- `fuzz/` — cargo-fuzz-Targets für die drei untrusted-byte-Parser (SSE-Scan, Label-Sanitizer, Config-Roundtrip) inkl. Seed-Corpus.
- `.github/` — 6 Workflows (CI, Release, Audit, Fuzz, CodeQL, Scorecard), Dependabot, Issue-/PR-Templates, CodeQL-Config.
- `Dockerfile` + `docker-compose*.yml` — 2-Stage-musl-Build `FROM scratch`, hardened Compose-Defaults (loopback-Publish, read_only, cap_drop ALL).
- `docs/assets/` — 7 Screenshots (Dashboard-Tabs, Setup-Wizard, Logo; Inhalte durch README-Bildreferenzen zugeordnet, visuelle Wiedergabe in dieser Analyse nicht möglich).

## Komponenten

1. **HTTP-Proxy-Kern** — OpenAI-kompatibler Passthrough mit genau einer Ausnahme (`stream_options.include_usage`-Injektion mit Auto-Fallback bei Modell-Ablehnung; `strict_passthrough`-Schalter deaktiviert die Injektion); Antwortet `/v1/models` aus Cache (10 min, Single-Flight) (belegt durch `src/proxy.rs`, `README.md`).
2. **Rate-Limit-Schicht (Pool + Dispatcher)** — ein Lane pro Key mit exaktem Sliding-Window-Limiter (40 Requests / rollende 60 s, 1 s Jitter-Marge); ein globaler FIFO-Dispatcher vergibt Slots streng in Ankunftsreihenfolge, kein Client kann einen anderen aushungern, ein Disconnect gibt den Slot zurück (belegt durch `src/pool.rs`, `src/dispatch.rs`, `knowledge/decisions/sliding-window-not-token-bucket.md`, `knowledge/decisions/global-fifo-dispatcher.md`).
3. **Affinity/Spillover** — jede Konversation bleibt pro Turn auf demselben Lane (Prefix-Cache warm), bei vollem Lane Spill auf den am wenigsten ausgelasteten bereiten Lane; gemessen als `nimproxy_affinity_total{result=sticky|spill|none}` (belegt durch `src/pool.rs`, `knowledge/decisions/sticky-affinity-with-spillover.md`).
4. **Streaming-Pipeline** — bei Streaming-Requests sofortiges `200 text/event-stream`-Commit, SSE-Kommentar-Heartbeats (`: heartbeat`) während Wartezeit/Retry, Ride-out von 429/5xx mit Retry-After und sofortigem Key-Failover, `stream_idle`-Cut für gestallte Streams, Terminal-SSE-Error-Events bei `deadline_exceeded` (belegt durch `src/proxy.rs`, `knowledge/decisions/sse-heartbeats-for-rate-waits.md`, `knowledge/decisions/explicit-request-deadline.md`).
5. **Model-Pressure-Governor** — klassifiziert NIMs per-Model-Worker-Concurrency-Exhaustion (`Worker local total request limit reached`) getrennt von 429s und bremst das Modell adaptiv (nie den Lane; Key-Failover hilft dort nicht); Metriken `nimproxy_worker_exhausted_total`, `nimproxy_model_inflight`, `nimproxy_model_limit` (belegt durch `src/governor.rs`, `knowledge/architecture/governor.md`).
6. **Auth-/Multi-User-Schicht** — fail-closed: vor Setup ist die Datenspur zu (`/v1` → 503 `setup_required`), danach verlangt jede Observability-Oberfläche eine Session; Rollen superuser/admin/user, Per-Key-Ownership, Client-Keys (`npk_…`, 128-Bit, genau einmal sichtbar, nur SHA-256-Digest + Last-4 gespeichert), Konstantzeit-Vergleich (belegt durch `src/auth.rs`, `src/config.rs`, `src/settings.rs`, `knowledge/decisions/auth-posture-and-dashboard-password.md`).
7. **Observability (History + Dashboard + Metrics)** — ~4-KB-Snapshot alle 5 Minuten, retained 30 Tage (konfigurierbar, 0 = unbegrenzt), kompaktiert bei Retention-Änderung; Prometheus-Exposition `/metrics`; Dashboard mit 5 persona-aligned Tabs (Overview, Models, Clients, Reliability, Capacity) und 3-s-Polling (belegt durch `src/history.rs`, `src/dashboard.html`, `README.md` Metrik-Referenztabelle).
8. **Setup-Wizard** — First-Visitor-claimt-Superuser, 3 Schritte (Superuser → ≥1 NIM-Key live validiert → Finish), mintet standardmäßig den ersten Client-Key mit einmaliger Anzeige (belegt durch `src/setup.html`, `src/settings.rs`, `tests/e2e.rs` `setup_can_mint_a_first_client_key`).

## Datenfluss

1. Harness sendet OpenAI-kompatiblen Request an `/v1/chat/completions` (Bearer-Key oder Open-Mode) — ggf. mit `X-Nim-Proxy-Deadline-Ms` (belegt durch `README.md`).
2. API-Key-Gate in `src/proxy.rs` (keyed: unbekannter Key → OpenAI-stil 401, Konstantzeit; open: unauthentisiert) und Label-Sanitizing von `model`/`path` (≤64 Zeichen, sicheres Charset, Kardinalitäts-Cap 256 → `other`) (belegt durch `src/proxy.rs`, `knowledge/decisions/input-sanitizing-and-xss.md`).
3. Governor prüft das per-Model-Concurrency-Limit; bei Überschreitung Backoff dieses Modells (belegt durch `src/governor.rs`).
4. Dispatcher stellt einen Slot (Deadline-abhängige Wartezeit, FIFO); während des Wartens sendet die Streaming-Strecke Heartbeats; `nimproxy_queue_wait_seconds` erfasst die Wartezeit (belegt durch `src/dispatch.rs`).
5. Pool reserviert einen Lane (Sliding-Window-Prüfung; Affinity-Sticky, sonst Spill; 429/5xx → Retry-After-Ride-out + Failover) (belegt durch `src/pool.rs`).
6. Request geht an Upstream (Default `https://integrate.api.nvidia.com/v1`); Streaming: SSE-Scan für Tokens/Usage/Finish; Usage-Injektion ergänzt `include_usage`, bei Ablehnung Retry ohne Injektion (belegt durch `src/proxy.rs`, `fuzz/seeds/sse_scan/stream-with-usage`).
7. Antwort-Relay an Client; Token-/Finish-/Tool-/Reasoning-Metriken werden erfasst (belegt durch `src/proxy.rs`, `README.md` Metrik-Tabelle).
8. History: Snapshot-Task sammelt Metriken und schreibt atomar an `history.jsonl`; Dashboard-API liefert typisierte Range-/Now-Verträge mit `history_revision`/`config_revision` (belegt durch `src/history.rs`, `tests/e2e.rs` `dashboard_now_contract_uses_current_pool_config_and_registry`).

## Persistenz

- **`DATA_DIR/config.json`** — Konfig-Store (Version 1, atomare Writes, Modus 0600): Upstream-Base-URL, NIM-Keys (rpm, owner, enabled), Client-Keys (nur SHA-256 + last4), Limits, Pricing, Dashboard-Window/Retention/SLO, Governor (enabled + Overrides), Users (username, password_hash, role). Ein korrupter/future-version-Store ist ein harter Boot-Fehler, nie ein stiller Fallback (belegt durch `src/config.rs`, `knowledge/decisions/ui-managed-config-store.md`, `fuzz/seeds/config_roundtrip/store.json`).
- **`DATA_DIR/history.jsonl`** — Metrik-Snapshots (~4 KB alle 5 min), `.tmp`-atomare Writes, v1-Legacy-/v2-Boot-Marker, Background-Kompaktierung bei Retention-Änderung; Größe workload-abhängig (real gemessen: 235.598.655 Bytes bei 7.316 Samples) (belegt durch `src/history.rs`, `CHANGELOG.md` [0.6.5]).
- **Sessions** — nur in-memory; der Cookie-Signing-Key ist pro Boot zufällig, daher loggt ein Restart alle Dashboard-Sessions aus (belegt durch `src/auth.rs`, `README.md` FAQ).
- **Rate-State** — in-memory; Windows resetten bei Restart (429-Burst wird von der Retry-Maschinerie absorbiert) (belegt durch `README.md` FAQ).
- **Fuzz-Corpus** — `fuzz/seeds/**` ist eingecheckter Seed-Corpus (per `.gitattributes` als binary markiert) (belegt durch `.gitattributes`, `.github/workflows/fuzz.yml`).

## Abhängigkeiten

- Rust stable; deklarierte MSRV 1.87 (`rust-version`, per CI-`msrv`-Job mit `cargo +1.87 check --locked --all-targets` erzwungen); `rust-toolchain.toml` pinnt Kanal stable + rustfmt/clippy für Contributors (belegt durch `Cargo.toml`, `rust-toolchain.toml`, `.github/workflows/ci.yml`).
- Crates (belegt durch `Cargo.toml`, `Cargo.lock`): axum 0.8 + hyper + tokio (Async-Webstack), reqwest mit rustls (Upstream-Client), metrics + metrics-exporter-prometheus (Metriken), hmac 0.12/sha2 0.10/getrandom 0.2 (Auth-Crypto, bewusst gepinnt), serde/serde_json (Modelle), dotenvy (Env), indexmap; transitiv u. a. quanta, foldhash, evmap/left-right (concurrent Maps), loom (in `Cargo.lock` vorhanden — Nutzung nicht direkt belegbar).
- Python 3 nur für die Test-/Load-Harness (`scripts/`, stdlib-only) und den Release-Vertragstest (belegt durch `CONTRIBUTING.md`, `scripts/mock_nim.py`).
- Docker: 2-Stage-Build, `rust:1-alpine@sha256:3c38f3f8…40900` → `FROM scratch` (statisches musl-Binary, TLS-Roots via rustls+webpki-roots kompiliert, keine CA-Dateien nötig) (belegt durch `Dockerfile`).
- Externe Dienste: NVIDIA NIM `https://integrate.api.nvidia.com/v1` (Default-Upstream), GHCR (Image-Push), GitHub Actions (CI/CD) (belegt durch `src/config.rs`, `docker-compose.yml`, `.github/workflows/*`).

## Eingaben / Ausgaben

**Eingaben:**
- Umgebungsvariablen (nur Container-Ebene, alles App-Level liegt im Store): `HOST` (Default 0.0.0.0), `PORT` (Default 8000), `DATA_DIR` (Default `data`; `/data` in Docker; unbeschreibbar = harter Boot-Fehler), `TRUST_PROXY` (Default false; markiert Session-Cookie `Secure`), `RUST_LOG` (Default `nim_proxy=info`); Compose liest zusätzlich `PUBLISH_HOST` (Default 127.0.0.1). Legacy-Env-Vars (NIM_API_KEYS, ADMIN_PASSWORD, …) werden ignoriert mit einzeiliger Boot-Warnung (belegt durch `.env.example`, `README.md`, `CHANGELOG.md` [0.6.0]).
- HTTP-Eingaben: OpenAI-kompatible Requests an `/v1/chat/completions` (stream + non-stream), `GET /v1/models`, optional `X-Nim-Proxy-Deadline-Ms` (absolutes Wall-Clock-Deadline; Heartbeats/Chunks resetten es nicht) (belegt durch `README.md`).
- Settings-/Wizard-API: POST `/setup`, `/login`, `/api/settings/*`, `/api/settings/validate-key`, `/api/config` (belegt durch `src/settings.rs`, `src/dashboard.html`).

**Ausgaben:**
- `/metrics` (Prometheus-Exposition; Scraper-Auth als `Bearer <user>:<password>` oder HTTP Basic) — vollständige Serienliste in der README-Metrik-Tabelle (26 Zeilen) (belegt durch `README.md`, `src/proxy.rs`).
- `/health` (öffentlich, exposiert nichts; dient als Container-Healthcheck `nim-proxy --health`) (belegt durch `Dockerfile`, `README.md`).
- Dashboard (GET `/`) mit `/api/dashboard`, `/api/dashboard/now` (typisierte Verträge; 401 ohne Session) (belegt durch `src/history.rs`, `src/dashboard.html`, `tests/e2e.rs`).
- Zugriffs-Log eine Zeile pro Request: `200 alice model /v1/chat/completions (3210 ms)` (belegt durch `README.md` Operations).
- `config.json` + `history.jsonl` unter `DATA_DIR` (belegt durch `src/config.rs`, `src/history.rs`).

## Zustände

- **API-Modus**: `open` | `keyed` (Default; keyed mit 0 Keys weist alles ab — fail closed) (belegt durch `src/config.rs`, `README.md`).
- **Setup-Status**: vor Setup ist `/v1` geschlossen (503 `setup_required`), Browser → `/setup`; der erste Visitor wird Superuser (belegt durch `README.md`, `tests/e2e.rs`).
- **Key-Zustände** (Dashboard-Chips, live via `/api/config`): `disabled`, `cooldown Ns`, `x / y in window`, `unassigned` (belegt durch `src/dashboard.html` `keyState`).
- **Rollen**: `superuser` (Admin, nie löschbar, immer Besitzer ≥1 enabled NIM-Key = Pool-Floor), `admin` (Server-Settings + User-Management), `user` (eigene Keys/Client-Keys, sieht alle Tabs, aber nur eigene Key-Zeilen) (belegt durch `README.md`, `src/settings.rs`).
- **Governor**: `0` = ungoverned; adaptiv: engages bei halber beobachteter Inflight, +1 pro stabile Minute, dissolves nach 30 sauberen Minuten; optional gepinnte Caps (belegt durch `src/governor.rs`, `CHANGELOG.md` [0.6.0]).
- **Request-Status** (Metrics-Label `status`): u. a. `disconnect`, `stall`, `stream_error`, `deadline` neben HTTP-Codes (belegt durch `README.md` Metrik-Tabelle).
- **History**: `history_revision`/`config_revision` in Now/Range-Verträgen; `compaction_pending`-Flag; Legacy-Buckets ohne Kapazitäts-Metadaten (belegt durch `src/history.rs`, `src/dashboard.html`, `tests/e2e.rs`).

## Sicherheitsrelevanz

- **Fail-closed-Postur**: vor Setup geschlossene Datenspur; nach Setup verlangen Dashboard und alle Observability-Surfaces eine Session; nur `/v1` kann `open` sein (belegt durch `README.md`, `SECURITY.md`, `CONTRIBUTING.md`).
- **Passwörter**: PBKDF2-HMAC-SHA256, 600k Iterationen; Session-Cookie signiert, HttpOnly, SameSite=Strict; Passwort-Änderung/Reset invalidiert andere Sessions sofort (Session trägt Username + Passwort-Hash-Fragment); Own-Password-Change gegen gleichzeitigen Admin-Reset abgesichert (409) (belegt durch `src/auth.rs`, `CHANGELOG.md` [0.6.0]).
- **Secrets**: Client-Key-Secret 128-Bit, genau einmal sichtbar, gespeichert nur als SHA-256-Digest + last4; NIM-Keys nie im Klartext in Logs/Metriken; Konstantzeit-Vergleich; abgelehnter API-Key mit Verzögerung (belegt durch `src/config.rs`, `src/auth.rs`, `README.md`).
- **Injection-Schutz**: `model`/`path`-Labels sanitized (sicheres Charset, Längen-Cap, Kardinalitäts-Cap); Dashboard `esc()`-escaped jedes dynamische `innerHTML`-Sink (textContent für Feedback); strikte CSP + Anti-Framing/-Sniffing-Header; Fuzz-Targets für die Parser (belegt durch `src/proxy.rs`, `src/dashboard.html`, `knowledge/decisions/input-sanitizing-and-xss.md`, `.github/workflows/fuzz.yml`).
- **DoS-Grenzen**: `max_inflight` (Default 64) mit 503-Shedding (`{"error":{"code":"overloaded"}}`), Login-Throttle mit saturierender Subtraktion, `X-Nim-Proxy-Deadline-Ms` als absolutes Deadline (belegt durch `src/proxy.rs`, `src/auth.rs`, `tests/e2e.rs` `overloaded_requests_are_shed_with_503`).
- **Supply Chain**: SHA-gepinnte Actions (Dependabot hält Pins frisch), harden-runner Egress-Monitoring, CodeQL, actionlint + zizmor (Gate bei high), Dependency-Review, cargo-deny (advisories pro PR + wöchentlich), Scorecard wöchentlich, Signing keyless cosign + SLSA-Provenance + SPDX-SBOM auf Manifest-Digest, `v*`-Tags durch Ruleset geschützt (belegt durch `.github/workflows/*`, `SECURITY.md`).
- **Kein eingebautes TLS** (by design; TLS-Terminierung am Reverse-Proxy; `TRUST_PROXY=true` für `Secure`-Cookie) — dokumentiert, keine Schwachstelle (belegt durch `README.md`, `SECURITY.md` Out of scope).
