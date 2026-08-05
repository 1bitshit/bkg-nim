# nim-proxy — NVIDIA-NIM-Integration

Die Proxy-Seite der NIM-Integration: Chat, Embeddings, Streaming, Tool Calling, Request-/Response-Aufbau, Retry, Fehlerbehandlung, Rate Limits, Model Routing. Belege in Klammern; "Nicht nachweisbar." wo der Quellcode keine Aussage erlaubt.

---

## Chat (POST /v1/chat/completions)

- Request-Aufbau: Client-Request (Stream, Messages, Tools) wird an den Upstream weitergeleitet; Upstream-Basis-URL aus config (store.json-Seed: `base_url: "https://integrate.api.nvidia.com"`) (belegt durch src/proxy.rs, fuzz/seeds/config_roundtrip/store.json).
- Response-Aufbau: Upstream-Response wird an den Client durchgereicht; bei fehlendem usage-Feld injiziert der Proxy usage (Usage-Injection mit Auto-Fallback, belegt durch decisions/usage-injection-auto-fallback.md).
- strict_passthrough: Wenn aktiv (Default false), keinerlei Response-Injection (belegt durch store.json-Seed `"strict_passthrough":false`, src/proxy.rs).
- Payload-Guard: Requests über 1 MiB werden abgelehnt (belegt durch src/proxy.rs).
- Request-Timeout: request_timeout_secs (Default 300) begrenzt den Upstream-Call (belegt durch store.json-Seed).
- Rust-Relevanz: reqwest-POST mit Body-Streaming, serde_json-Request-/Response-Typen, Timeout via tokio-time.

## Embeddings

- Nicht nachweisbar. Es gibt keinen eigenen Embeddings-Endpunkt-Beleg im Quellcode; die NIM-API bietet embeddings (README nennt /v1-Chat-Kompatibilität, aber keine Embeddings-Route). Der Proxy fokussiert Chat-Completions und Models (belegt durch src/proxy.rs, README).

## Streaming (SSE)

- SSE-Forwarding: Der Proxy reicht den SSE-Stream des Upstreams an den Client durch (belegt durch src/proxy.rs, fuzz-Target sse_scan).
- SSE-Heartbeat: Bei Rate-Waits sendet der Proxy Heartbeat-Segmente, damit Client-Verbindungen nicht abreißen (belegt durch decisions/sse-heartbeats-for-rate-waits.md, src/proxy.rs; Intervall heartbeat_secs 10).
- Stream-Idle-Timeout: stream_idle_secs (Default 120) beendet inaktive Streams (belegt durch store.json-Seed).
- SSE-Parser: Der Proxy scannt SSE-Ereignisse (usage, tool_calls, Heartbeats) — fuzz-gesichert (fuzz/fuzz_targets/sse_scan.rs).
- Rust-Relevanz: futures-util StreamExt, reqwest-stream, SSE-Chunk-Parsing, Heartbeat-Intervall.

## Tool Calling

- Weiterleitung: Tool-Calls aus Client-Payloads werden unverändert an den Upstream durchgereicht (belegt durch src/proxy.rs, fuzz-Seed stream-with-usage mit tool_calls, scripts/loadtest.py: Tools jede 3. Anfrage).
- Response: tool_calls-Felder im SSE-Stream werden erkannt/verarbeitet (belegt durch fuzz-Seed sse_scan/stream-with-usage).
- Kein eigenes Tool-Routing im Proxy — nur Durchleitung (belegt durch src/proxy.rs).
- Rust-Relevanz: serde-Typen für tool_calls (Optional-Felder), Durchleitungs-Logik.

## Request-/Response-Aufbau (Detail)

- Header: x-nim-proxy-deadline-ms-Header für explizite Deadlines (belegt durch CHANGELOG [0.6.2], decisions/explicit-request-deadline.md).
- JSON-Shape: OpenAI-kompatible chat/completions-Struktur (messages, model, stream, tools) (belegt durch scripts/loadtest.py, examples/opencode.json).
- Fehler-Shape: Proxy-Fehler als `{error:{message,type:"proxy_error",code}}` (belegt durch src/settings.rs).
- Rust-Relevanz: Request-/Response-DTOs als serde-Typen, Fehler-Enum → JSON.

## Retry

- Nicht nachweisbar. Es gibt keinen expliziten Retry-Mechanismus-Beleg im Quellcode (keine Retry-Schleife in src/proxy.rs erkennbar); Timeouts und Shedding-Entscheidungen werden nicht retried (belegt durch src/proxy.rs, dispatch-Shedding).
- Hinweis: Der Proxy wartet (Waiter mit Deadline) statt zu retrien, wenn das Rate-Limit das zulässt (belegt durch src/dispatch.rs, decisions/sse-heartbeats-for-rate-waits.md).

## Fehlerbehandlung

- Upstream-Fehler: 5xx/4xx vom Upstream werden an den Client durchgereicht (belegt durch src/proxy.rs).
- 429 (Rate-Limit): Beim Upstream-Rate-Limit wartet der Proxy (Waiter/Queue), statt sofort zu failen; Shedding greift bei Überlast (503) (belegt durch src/dispatch.rs, e2e overloaded_requests_are_shed_with_503).
- Timeouts: Request-Timeout, Stream-Idle-Timeout, Deadline-Ablauf → Fehlerantwort (belegt durch src/proxy.rs, decisions/explicit-request-deadline.md).
- Payload-Too-Large: >1 MiB → Ablehnung (belegt durch src/proxy.rs).
- Rust-Relevanz: Fehler-Enum mit HTTP-Mapping, Timeout-Fehler-Differenzierung, Shedding-Codes.

## Rate Limits

- Per-Key-Limit: Jede Lane hat ein RPM-Limit (Default 40, belegt durch store.json-Seed `"rpm":40`, knowledge/research/nim-free-tier-40rpm-no-credits.md).
- Sliding Window: Rate-Limit-Zählung als Sliding Window mit WINDOW 61s (Jitter-Marge) statt Token-Bucket (belegt durch decisions/sliding-window-not-token-bucket.md, decisions/window-jitter-margin.md, src/pool.rs).
- Governor: adaptives Wachstum (GROW_INTERVAL 60s), Exhaustion-Backoff (2s), Dissolve bei Inaktivität (30min), Poll alle 250ms (belegt durch src/governor.rs).
- Zero-Violations: Hard-Requirement — kein Request darf das per-Key-Limit überschreiten (belegt durch CONTRIBUTING.md, README).
- Rust-Relevanz: Sliding-Window-Implementierung, Governor-Loop (tokio), Metrik-Zähler.

## Model Routing

- Affinity: sticky mit Spillover — ein Modell bleibt bevorzugt auf seiner Lane, weicht bei Auslastung auf andere Lanes aus (belegt durch decisions/sticky-affinity-with-spillover.md, src/pool.rs, Metrik nimproxy_affinity_total{result}).
- Enabled-Lanes zuerst: Nur aktive Keys werden zuerst genutzt (belegt durch src/pool.rs).
- Modell-Liste: /v1/models wird vom Upstream durchgereicht (belegt durch README, tests/support Mock: /v1/models).
- Rust-Relevanz: Routing-Entscheidung als reine Funktion (testbar), Affinity-Ranking.
