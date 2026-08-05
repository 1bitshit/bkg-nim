# nim-proxy — Architektur-Analyse

## Komponenten

### 1. HTTP-Proxy-Kern (`src/proxy.rs`)
- `serve`/Routing: `/v1/chat/completions` (POST), `/v1/models` (GET, Cache 10 min Single-Flight), `/health`, `/metrics`, `/login`, `/logout`, `/setup`, Dashboard + API-Routen (belegt durch `src/proxy.rs`, `src/settings.rs`, `src/history.rs`).
- API-Key-Gate (keyed/open), Label-Sanitizing (`sanitize_label`: ≤64 Zeichen, Charset `[A-Za-z0-9._/:-]`, Cap 256 → `other`), Konstantzeit-Key-Vergleich (belegt durch `src/proxy.rs`, `knowledge/decisions/input-sanitizing-and-xss.md`).
- Streaming-Relay: sofortiges 200-Commit, SSE-Scan (Heartbeats `: heartbeat`, Chunks, Tool-Calls, Usage, Finish-Reason, `[DONE]`), `stream_idle`-Stall-Cut, 1-MiB-SSE-Guard, `X-Nim-Proxy-Deadline-Ms`-Abbruch (belegt durch `src/proxy.rs`, `tests/e2e.rs`, `fuzz/seeds/sse_scan/*`).
- Usage-Injektion: `stream_options.include_usage` ergänzt, bei Modell-Ablehnung Retry ohne Injektion + Never-Inject-Gedächtnis; `strict_passthrough` deaktiviert (belegt durch `src/proxy.rs`, `README.md`).
- Request-Shape-/Quality-Metriken: conversation depth, tools, temperature, max_tokens, streaming-Mix, JSON-Mode, finish reason, tool calls, reasoning tokens, TPOT (belegt durch `src/proxy.rs`, `knowledge/decisions/request-shape-metrics.md`).

### 2. Rate-Limit-Schicht (`src/pool.rs` + `src/dispatch.rs`)
- **Pool**: `PoolHandle` (Clone-able), `Pool` mit `Lane`s; ein Lane pro Key; Sliding-Window-Limiter (40/60 s + 1 s Jitter-Marge, WINDOW 61 s), `LaneSpec`/`Lane`-Trennung; enabled-Lanes zuerst; Reservation `Ready`/`Wait` (belegt durch `src/pool.rs`, `knowledge/decisions/window-jitter-margin.md`).
- **Affinity**: Konversations-Hash → bevorzugter Lane; bei vollem Lane Spill auf den am wenigsten ausgelasteten bereiten Lane; `nimproxy_affinity_total{result}` (belegt durch `src/pool.rs`, `knowledge/decisions/sticky-affinity-with-spillover.md`).
- **Dispatcher**: globaler FIFO-Waiter-Queue, Slots mit Pool-Generation, Deadline + `prefer`-Priorisierung, GRANT_GAP 25 ms Pacing zwischen Grants, `nimproxy_queue_depth`-Gauge, Slot-Rückgabe bei Disconnect (belegt durch `src/dispatch.rs`, `knowledge/decisions/global-fifo-dispatcher.md`).

### 3. Model-Pressure-Governor (`src/governor.rs`)
- Erkennt NIM-Worker-Exhaustion (`Worker local total request limit reached`, 429 mit `detail`-Marker) getrennt von normalen 429s; hält per-Model-Limit (`ModelState{limit, inflight, blocked_until, …}`); POLL 250 ms, EXHAUST_BACKOFF 2 s, GROW_INTERVAL 60 s, DISSOLVE_AFTER 30 min; `ModelPermit` RAII gibt Slots zurück; Metriken `nimproxy_worker_exhausted_total`/`model_inflight`/`model_limit` (belegt durch `src/governor.rs`, `knowledge/architecture/governor.md`, `CHANGELOG.md` [0.6.0]).

### 4. Auth-/Identity-Schicht (`src/auth.rs` + `src/config.rs` + `src/settings.rs`)
- PBKDF2-HMAC-SHA256 (600k), Format `pbkdf2-sha256$iters$salt$hash`; Session-Cookie `nimproxy_session` (signiert, HttpOnly, SameSite=Strict, TTL 12 h, `Secure` bei TRUST_PROXY); Session = Expiry + Username + Passwort-Hash-Fragment; Login-Throttle (saturierend); Basic-Auth-Branch für Scraper (belegt durch `src/auth.rs`, `knowledge/decisions/auth-posture-and-dashboard-password.md`).
- Config-Store: `StoredConfig` Version 1, atomare Writes, 0600; `Mode::Open`/`Keyed` (Default Keyed, fail-closed); `NimKey{key, owner, enabled, rpm:40}`, `ClientKey{name, secret_sha256, owner}` (belegt durch `src/config.rs`, `fuzz/seeds/config_roundtrip/store.json`).
- Settings-Commit-Pipeline (exakt): validate → save → cfg-write → Pool-Rebuild mit Rate-Carryover → Retention-Trim → guard → `config_revision++` (belegt durch `src/settings.rs`).

### 5. Observability (`src/history.rs` + `src/dashboard.html`)
- History: SAMPLE_SECS 300, JSONL-Snapshots, `.tmp`-atomare Writes, 1-MB-/100k-Eintrags-Guards, v1-Legacy-Import + v2-Boot-Marker, Background-Kompaktierung bei Retention, `history_revision`/`config_revision`; `/api/dashboard?from&to&points=288`, `/api/dashboard/now` (belegt durch `src/history.rs`, `tests/e2e.rs`).
- Dashboard: eine eingebettete HTML-Datei (2504 Zeilen, genau ein `<script>`-Block), 5 Tabs (Overview, Models, Clients, Reliability, Capacity) + Settings; Polling POLL_MS 3000; typisierte Verträge statt Roh-Metriken; `rangeSamples` rekonstruiert kumulative Rows aus normalisierten Counter-Deltas, Gauge-Ersetzung (belegt durch `src/dashboard.html`).

### 6. Setup-Wizard (`src/setup.html` + `src/settings.rs`)
- 3 Schritte (Superuser → ≥1 NIM-Key live-validiert → Finish); mints ersten Client-Key atomar mit Claim (Secret genau einmal); opt-out mit Warnung (belegt durch `src/setup.html`, `tests/e2e.rs` `setup_can_mint_a_first_client_key`).

### 7. Test-/Verifikations-Schicht (`tests/`, `scripts/`, `fuzz/`)
- e2e.rs (3731 Zeilen, 97 async fns) bootet das echte Binary gegen scriptbares Mock-NIM in tempdir-DATA_DIR; Coverage-Gate 90 % Linien via `cargo llvm-cov` (Subprocess-Coverage) (belegt durch `tests/e2e.rs`, `tests/support/mod.rs`, `.github/workflows/ci.yml`).
- Load-Harness: `mock_nim.py --enforce` (striktes Sliding-Window, Violations-Zähler, `--worker-slots` für Governor-Übung) + `loadtest.py` (100 Clients, exit≠0 bei Fehlern/Violation/unbenutztem Key) (belegt durch `scripts/*.py`, `CONTRIBUTING.md`).
- Fuzz: 3 Target-Bins (`sse_scan`, `sanitize_label`, `config_roundtrip`) mit Seed-Corpus; wöchentlich + bei PRs auf fuzzbare Pfade (belegt durch `.github/workflows/fuzz.yml`, `fuzz/`).

## Module & Layer

- **Layer-0 (Binary)**: `main.rs` — Env, Router, Shutdown, Healthcheck.
- **Layer-1 (Library)**: `lib.rs` — Typen/AppState; `config.rs` (Datenmodelle), `proxy.rs` (HTTP), `pool.rs` + `dispatch.rs` + `governor.rs` (Rate-/Concurrency-Steuerung), `history.rs` (Persistenz), `auth.rs` + `settings.rs` (Identity/Settings).
- **Layer-2 (UI)**: `dashboard.html` + `setup.html` (eingebettete Assets, via `include_str!`-artiger Einbindung ausgeliefert).
- **Layer-3 (Test-/Betrieb)**: `tests/`, `scripts/`, `fuzz/`, `.github/workflows/`, `Dockerfile`/Compose.
- **Querschnitt**: Metriken (jeder Request-Pfad), Label-Sanitizing (jede untrusted Eingabe), Fail-closed-Auth (jede Surface), `X-Nim-Proxy-Deadline-Ms` (jeder Request-Owner).

## Datenfluss

1. Harness → `POST /v1/chat/completions` → Auth-Gate (keyed: Client-Key-Konstantzeit-Check; open: frei) → Body-Deserialisierung (Shape-Metriken) → `max_inflight`-Guard (Shed 503 `overloaded` bei Cap) (belegt durch `src/proxy.rs`, `tests/e2e.rs`).
2. Governor-Admission (per-Model-Inflight gegen Limit; `ModelPermit`) (belegt durch `src/governor.rs`).
3. Dispatcher-Slot (FIFO-Waiter; Deadline-Setup; Queue-Wait-Metrik) (belegt durch `src/dispatch.rs`).
4. Pool-Reservation (Sliding-Window; sticky/spill; `LaneReservation`) → Upstream-Send (belegt durch `src/pool.rs`).
5. Upstream-Response: Streaming-Scan (Tokens/Usage/Tool-Calls/Finish) → Relay an Client; bei 429/5xx Retry-After-Ride-out + Key-Failover; bei `stream_idle`-Stall Cut; bei Deadline Abbruch + Terminal-Event/504 (belegt durch `src/proxy.rs`).
6. Metrik-Erfassung (jeder Pfad) → History-Snapshot-Task (5 min) → `history.jsonl`; Dashboard liest via typisierte API (belegt durch `src/history.rs`).

## Eventfluss

- **HTTP-Events**: Request an `/v1` (Streaming: sofortiges 200-Commit, danach SSE-Events; Buffered: JSON); `/login`/`/logout`/`/setup`/`/api/settings/*` (POST → Store-Commit → Config-Revision-Bump → Pool-Rebuild); Dashboard-Polling `/api/dashboard/now` alle 3 s (belegt durch `src/proxy.rs`, `src/settings.rs`, `src/dashboard.html`).
- **Kein Push/Pub-Sub**: Es existieren keine asynchronen Event-Kanäle zwischen Komponenten; Kommunikation erfolgt über Funktionen, Metrik-Registry und den Konfig-/Pool-Zustand (belegt durch `src/lib.rs`, `src/history.rs`).

## APIs

- **Intern (HTTP)**:
  - `/v1/chat/completions` (POST, OpenAI-kompatibel, stream + non-stream), `/v1/models` (GET, Cache).
  - `/health` (GET, öffentlich), `/metrics` (GET, Scraper-Auth).
  - `/setup` (GET Seite / POST Wizard), `/login`/`/logout` (POST), `/` (Dashboard).
  - `/api/config` (GET, rollen-gefiltert), `/api/settings/*` (POST: nim-keys, clients, upstream, limits, governor, pricing, history, users, account), `/api/settings/validate-key` (POST, Live-Probe), `/api/dashboard` + `/api/dashboard/now` (GET) (belegt durch `src/settings.rs`, `src/history.rs`, `src/dashboard.html`, `tests/e2e.rs`).
- **Extern**: NVIDIA NIM `https://integrate.api.nvidia.com/v1` (Default; Chat-Completions + Models), resp. konfigurierte Upstream-URL (belegt durch `src/config.rs`, `fuzz/seeds/config_roundtrip/store.json`).

## State Machines

- **Setup**: fresh → wizard (Superuser → Key → Finish) → konfiguriert; pre-setup: `/v1` 503 `setup_required`, Browser → `/setup`; Doppel-Claim abgewehrt (belegt durch `README.md`, `tests/e2e.rs`).
- **Key**: `enabled` ↔ `disabled` (Rate-Window bleibt warm), `cooldown` (nach Exhaustion), `in_window` (x/y), `unassigned` (belegt durch `src/dashboard.html` `keyState`).
- **Governor (per Modell)**: ungoverned (`limit=0`) → engages bei Exhaustion → climb (+1/60 s stabil) → dissolve (30 min sauber); Override-Caps setzen fest (belegt durch `src/governor.rs`, `CHANGELOG.md` [0.6.0]).
- **Request**: accepted → gated → queued → dispatched → in-flight → done / shed / deadline / disconnect / stall / stream_error (Status-Label) (belegt durch `README.md` Metrik-Tabelle).
- **Session**: gültig ↔ abgelaufen (TTL 12 h) / invalidiert (Passwort-Änderung, User-Löschung) / Reset (Boot-Signing-Key) (belegt durch `src/auth.rs`, `README.md` FAQ).

## Scheduler / Worker / Pipelines

- **History-Snapshot-Task**: periodisch (5 min) Sammeln der Metrik-Registry → atomarer JSONL-Write; Retention-Kompaktierung als Hintergrund-Task mit `.tmp`-Datei (belegt durch `src/history.rs`).
- **Keine expliziten Worker-Pools/Queues in der Anwendung**; der Dispatcher ist eine Warteschlangen-Struktur (siehe Queues), keine Thread-Pool-Scheduler (belegt durch `src/dispatch.rs`).
- **Load-Harness (Betrieb)**: `loadtest.py` startet 100 Threads × N Requests (belegt durch `scripts/loadtest.py`).
- **CI/CD-Pipelines**: siehe `.github/workflows/` (CI: fmt/clippy/test/coverage/msrv/deny/gitleaks/lint/dependency-review/docker; Release: prepare→build×2→merge→release; Audit/Fuzz/CodeQL/Scorecard wöchentlich) (belegt durch `.github/workflows/*`).

## Queues

- **Dispatcher-Waiter-Queue** (global, FIFO): ein Slot pro wartendem Request, Vergabe in Ankunftsreihenfolge, kein Client verhungert, Slot-Rückgabe bei Disconnect; GRANT_GAP 25 ms; Deadline-basiert (`x-nim-proxy-deadline-ms`) (belegt durch `src/dispatch.rs`, `knowledge/decisions/global-fifo-dispatcher.md`).
- **Lane-Rate-Windows**: je Key eine Deque vergangener Request-Zeitpunkte (rollendes 60-s-Fenster, 61-s-Jitter-Marge) — keine klassische Produktions-Queue (belegt durch `src/pool.rs`, `knowledge/decisions/window-jitter-margin.md`).
- **Keine Message-Broker/Event-Bus**: Nicht nachweisbar.

## Memory-Systeme

- **Short Memory**: Nicht nachweisbar (keine Konversations-Inhalts-Speicherung).
- **Long Memory**: `history.jsonl` (Metrik-Snapshots, keine Inhalte) + `config.json` (belegt durch `src/history.rs`, `src/config.rs`).
- **Episodic/Semantic/Facts/Knowledge Graph**: Nicht nachweisbar — das Projekt speichert keine Gesprächsinhalte oder Wissensgraphen; die einzige konversationsbezogene Zustandsgröße ist die Lane-Affinität (Hash-basiert, inhaltsfrei) (belegt durch `src/pool.rs`, `knowledge/decisions/sticky-affinity-with-spillover.md`).
- **Consolidation/Ranking/Retrieval**: Nicht nachweisbar (keine Memory-Konsolidierung).
- **Wissensbasis `knowledge/`**: statisches, cross-verlinktes Markdown (Entscheidungen/Forschung/Runbooks), kein Retrieval-System (belegt durch `knowledge/index.md`).

## RAG

Nicht nachweisbar. Das Repo enthält keine RAG-/Embedding-/Vektor-Komponente; der Proxy arbeitet rein request-/metrikbasiert (belegt durch `src/`-Modulbaum in `lib.rs`).

## Policy

- Keine Policy-Engine im Sinne von Regel-Interpretern; die Policy-äquivalenten Mechanismen sind: Auth-/Rollenmodell (superuser/admin/user, Per-Key-Ownership), Fail-closed-Postur, Label-Sanitizing, `strict_passthrough`, `max_inflight`-Shedding, Governor-Caps, Pricing-Referenz (belegt durch `src/auth.rs`, `src/settings.rs`, `src/proxy.rs`, `README.md`).
- Audit: Zugriffs-Log (eine Zeile pro Request), `nimproxy_unauthorized_total`, `nimproxy_login_failures_total`, `nimproxy_shed_total` (belegt durch `README.md` Metrik-Tabelle).
- Keine PII-Verarbeitung durch den Proxy (nur Counts/Sizes; kein Message-Content-Logging) (belegt durch `knowledge/decisions/request-shape-metrics.md`).

## NVIDIA NIM

- **Chat**: `POST {base}/v1/chat/completions` — Bodies passieren bis auf `include_usage`-Injektion unverändert; Streaming bevorzugt (Heartbeat-Kompatibilität), Buffered ohne Herzschlag (wartet still bis `max_wait`) (belegt durch `src/proxy.rs`, `README.md`).
- **Embeddings**: Nicht nachweisbar (kein Embeddings-Endpunkt implementiert; nur chat/completions + models) (belegt durch `src/proxy.rs`).
- **Streaming**: SSE-Scan (Chunks, Tool-Calls, Usage, Finish-Reason, `[DONE]`), 1-MiB-Guard, `stream_idle`-Cut, Terminal-Error-Events, Heartbeats `: heartbeat` (belegt durch `src/proxy.rs`, `fuzz/seeds/sse_scan/*`).
- **Tool Calling**: pass-through (keine eigene Tool-Verarbeitung); Request-Tool-Offering und Antwort-Tool-Calls werden gemessen (`nimproxy_request_tools`, `nimproxy_tool_calls_total`, `nimproxy_tool_choice_total{mode=auto|none|required|named}`) (belegt durch `src/proxy.rs`, `README.md`).
- **Request-Aufbau**: OpenAI-kompatibel; `max_tokens`, `temperature`, `tools`, `tool_choice`, `response_format` (JSON-Mode), `stream` werden durchgereicht; Usage-Injektion ergänzt `stream_options.include_usage` mit Auto-Fallback (belegt durch `src/proxy.rs`, `scripts/loadtest.py`).
- **Response-Aufbau**: OpenAI-kompatibel (chat.completion / chat.completion.chunk); `usage` aus Upstream übernommen; bei fehlendem Usage Estimate-Fallback (pro-SSE-Event) (`nimproxy_completion_tokens_total{source=usage|estimate}`) (belegt durch `README.md`).
- **Retry/Fehlerbehandlung**: 429 + Retry-After-Ride-out (Retry-After exakt befolgt), 500/502/503/504-Failover auf nächsten Key, Exhaustion-429-Sonderfall an Governor, `stream_idle`-Stall, Deadline-Abbruch, Modell-Ablehnung der Injektion → Retry ohne Injektion (belegt durch `src/proxy.rs`, `tests/e2e.rs`, `knowledge/architecture/governor.md`).
- **Rate Limits**: Per-Key-Sliding-Window 40/60 s (NIM-konform) + 1 s Jitter-Marge; globale Kapazität = Σ Keys × rpm; Shedding-Kurve laut capacity-math-Runbook; Mock erzwingt das Fenster strikt und zählt Violations (belegt durch `src/pool.rs`, `knowledge/ops/capacity-math.md`, `scripts/mock_nim.py`).
- **Model Routing**: Konversations-Affinität (Sticky) + Least-Loaded-Spillover; Modelle passieren Verbatim (`GET /v1/models` spiegelt Upstream-Katalog, 10-min-Cache) (belegt durch `src/pool.rs`, `src/proxy.rs`, `README.md`).

## Tool Calling / Streaming

- **Tool Calling**: pass-through (nicht interpretiert); gemessen: angebotene Tools je Request (Histogramm je Client), Tool-Calls je Modell, Tool-Choice-Modus (belegt durch `src/proxy.rs`, `README.md`, `scripts/loadtest.py` Tool-Zweig).
- **Streaming**: voll unterstützt mit Heartbeats, Usage-Injektion, Stall-Detection, Deadline; Buffered ohne Herzschlag (belegt durch `src/proxy.rs`, `README.md` FAQ).

## Sicherheit

- Fail-closed; PBKDF2 600k; Session-Cookie signiert/HttpOnly/Strict/12-h-TTL; Passwort-Reset invalidiert Sessions; Konstantzeit-Vergleiche; Client-Key-Secrets genau einmal; Label-Sanitizing; CSP + Anti-Framing/-Sniffing; `max_inflight`-Shedding; Login-Throttle; 0600-Store mit atomaren Writes; kein TLS eingebaut; SCRATCH-Image ohne Shell/libc (belegt durch `src/auth.rs`, `src/proxy.rs`, `src/config.rs`, `Dockerfile`, `SECURITY.md`, `README.md`).

## Fehlerbehandlung

- **Setup-Store**: korrupt/unlesbar/future-version → harter Boot-Fehler, nie Fallback zu Setup (belegt durch `src/config.rs`, `tests/e2e.rs`).
- **Upstream**: 429 (Retry-After), 5xx (Failover), Exhaustion (Governor), Stall (`stream_idle`), Deadlines (Terminal-SSE-Event / 504 `deadline_exceeded`), Usage-Injektions-Ablehnung (Retry ohne), `stream_options`-Fehlschlag → 400-Fallback-Pfad (belegt durch `src/proxy.rs`, `tests/e2e.rs`).
- **Client**: Disconnect → Slot-/Permit-Rückgabe sofort (Race gegen blockierte Upstream-Reads), Shed 503 `overloaded` bei `max_inflight` (belegt durch `src/proxy.rs`, `tests/e2e.rs`).
- **Settings**: Validate-before-commit (Fehlermeldungen als `{error:{message,type:"proxy_error",code}}`-JSON), Own-Password-Change vs. Admin-Reset → 409, Key-Probe-Fehler → "Add anyway"-Pfad (belegt durch `src/settings.rs`, `src/dashboard.html`).
- **Metrics/History**: fehlerhafte Samples werden nicht korrupt geschrieben (atomare Writes, Guards); Legacy-v1-Daten werden gemappt (belegt durch `src/history.rs`).

## Komponenten-/Modul-Graph (Zusammenfassung)

```
Harness → /v1 (proxy.rs: Gate+Sanitize+max_inflight) → governor.rs (ModelPermit)
       → dispatch.rs (FIFO-Slot, Deadline) → pool.rs (Lane: Sliding-Window+Affinity)
       → NIM Upstream → Relay (SSE/JSON, Heartbeats, Retry/Failover) → Client
       → Metriken → history.rs (5-min-Snapshots → history.jsonl)
                            ↑ /api/dashboard, /api/dashboard/now
                            └ dashboard.html (5 Tabs, 3-s-Polling)
auth.rs/settings.rs/config.rs → config.json (0600) → Pool-Rebuild (Rate-Carryover)
scripts/mock_nim.py + loadtest.py → Load-Verifikation (Violations=0)
fuzz/ → sse_scan · sanitize_label · config_roundtrip (untrusted-byte-Parser)
.github/workflows/ → CI-Gates + Release (Signing/Provenance/SBOM)
```
