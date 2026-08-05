# nim-proxy — Datenmodelle

Alle Datenmodelle aus `src/*.rs` und `tests/`. Bedeutung, Lebenszyklus, Ownership, Beziehungen, Persistenz, Validierung, Mutationen, Thread-Sicherheit, benötigte Rust-Konzepte. Belege in Klammern.

---

## StoredConfig (v1)

- Bedeutung: Die vollständige persistierte Konfiguration des Proxys (Mode, Upstream, Keys, Limits, Pricing, Users) (belegt durch src/config.rs, fuzz/seeds/config_roundtrip/store.json).
- Lebenszyklus: Wird beim Erststart via Setup erzeugt (src/setup.html, e2e setup_can_mint_a_first_client_key), bei jeder Settings-Änderung via commit()-Pipeline neu geschrieben (src/settings.rs), beim Boot geladen (src/config.rs).
- Ownership: Single-Owner: der Config-Layer (src/config.rs); AppState hält den Zugriff.
- Beziehungen: Basis für `Limits`, `Pricing`, `NimKey`, `ClientKey`, `User`, `nim_mode`; bestimmt Pool-Lane-Konfiguration (src/pool.rs) und History-Retention (src/history.rs).
- Persistenz: config.json im DATA_DIR, 0600, atomar via tmp+rename (belegt durch src/config.rs, README "config.json").
- Validierung: Serde-Schema v1; korrupte Datei = harter Boot-Fehler; Mode-Validierung (Open/Keyed); RPM-Bereiche; Price-Werte; User-Passwort-Policy (belegt durch src/config.rs, e2e).
- Mutationen: Nur durch commit()-Pipeline (validate → save → cfg-write → Pool-Rebuild → Retention → guard → config_revision++) (belegt durch src/settings.rs).
- Thread-Sicherheit: Geteilt via Arc/RwLock-artige Guards im AppState; Mutationen serialisiert durch die Commit-Pipeline (belegt durch src/lib.rs, src/settings.rs).
- benötigte Rust-Konzepte: serde-Derive mit `#[serde(default)]`, Version-Enum, Arc<RwLock<T>> oder ArcSwap für den aktuellen Config-Stand, Revision-Counter (u64) für Invalidation.

## nim_mode

- Bedeutung: Zugriffsmodus: Open (kein Key-Gate) vs. Keyed (API-Key-Gate, Fail-closed-Default) (belegt durch src/config.rs, CHANGELOG [0.3.0] fail-closed).
- Lebenszyklus: Statisch pro Config; Änderung via Settings-UI.
- Ownership: Teil von StoredConfig.
- Beziehungen: Bestimmt das Verhalten des API-Key-Gates in src/proxy.rs und der Auth-Schicht (src/auth.rs).
- Persistenz: In config.json als client_auth.mode (belegt durch store.json-Seed: `"client_auth":{"mode":"keyed",...}`).
- Validierung: Enum-Serialisierung; unbekannter Mode → Fehler.
- Mutationen: Nur via Commit-Pipeline.
- Thread-Sicherheit: Immutabel während eines Config-Stands (Swap bei Änderung).
- benötigte Rust-Konzepte: `enum NimMode { Open, Keyed }` mit serde, Pattern-Matching im Gate-Pfad.

## Limits

- Bedeutung: Operative Limits: max_wait_secs (30), heartbeat_secs (10), stream_idle_secs (120), request_timeout_secs (300), max_inflight (64), strict_passthrough (false) (belegt durch store.json-Seed, src/proxy.rs).
- Lebenszyklus: Teil von StoredConfig; Defaults beim Erststart, änderbar via Settings-UI.
- Ownership: Teil von StoredConfig (limits).
- Beziehungen: Steuert Proxy-Timeout-Verhalten, SSE-Heartbeat-Intervall, Shedding-Schwelle (max_inflight) und Injection-Verhalten (strict_passthrough).
- Persistenz: config.json.
- Validierung: Numerische Bereiche (z. B. max_wait_secs > 0), Boolean-Typen.
- Mutationen: Commit-Pipeline.
- Thread-Sicherheit: Immutabel pro Config-Stand.
- benötigte Rust-Konzepte: `struct Limits` mit serde-Defaults, u64/i64-Felder, Duration-Konvertierung.

## Pricing

- Bedeutung: Preis-Modell fürs Dashboard: price_in (Default 0.5), price_out (Default 2.0) (belegt durch src/dashboard.html cfg-Defaults).
- Lebenszyklus: Teil von StoredConfig; änderbar via Settings-UI (dashboard.html Settings-Tab).
- Ownership: Teil von StoredConfig (pricing).
- Beziehungen: Basis für Kosten-Metriken im Dashboard (Overview-Tab).
- Persistenz: config.json.
- Validierung: Nicht-negative Dezimalwerte.
- Mutationen: Commit-Pipeline.
- Thread-Sicherheit: Immutabel pro Config-Stand.
- benötigte Rust-Konzepte: `struct Pricing { price_in: f64, price_out: f64 }`.

## NimKey

- Bedeutung: Upstream-API-Key für NVIDIA NIM: key, owner, enabled, rpm (Default 40) (belegt durch store.json-Seed, src/config.rs).
- Lebenszyklus: Angelegt im Setup, verwaltet via Settings-UI; Grundlage der Lane-Konfiguration (belegt durch e2e dashboard_now_contract: lanes=3, rpms 40).
- Ownership: Teil von StoredConfig (upstream.nim_keys); Pool-Lanes referenzieren sie.
- Beziehungen: Je Key eine Lane (oder mehrere bei Multi-Key-Pool); enabled → Lane aktiv.
- Persistenz: config.json (Klartext-Key im Store; 0600-Datei als Schutzmaßnahme).
- Validierung: Nicht-leer, rpm-Bereich, enabled-Bool.
- Mutationen: Commit-Pipeline; Pool-Rebuild bei Änderung.
- Thread-Sicherheit: Immutabel pro Config-Stand; Pool-Lanes tragen den laufenden Zählerstand.
- benötigte Rust-Konzepte: `struct NimKey { key: String, owner: String, enabled: bool, rpm: u32 }`, Geheimnis-Handling (kein Logging).

## ClientKey

- Bedeutung: Client-API-Key für den Proxy-Zugang: name, secret_sha256, owner (belegt durch store.json-Seed).
- Lebenszyklus: Wird im Setup erzeugt (einmalige Ausgabe), bei Key-Änderungen via Settings-UI; Secret wird nur als SHA-256 gespeichert (belegt durch src/auth.rs, README "single-view secret").
- Ownership: Teil von StoredConfig (client_auth.keys).
- Beziehungen: Gate in src/proxy.rs prüft Client-Requests gegen diese Keys (Keyed-Modus).
- Persistenz: config.json (nur Hash, nie das Klartext-Secret).
- Validierung: Nicht-leer, eindeutiger Name, Hash-Format.
- Mutationen: Commit-Pipeline.
- Thread-Sicherheit: Immutabel pro Config-Stand.
- benötigte Rust-Konzepte: `struct ClientKey { name: String, secret_sha256: String, owner: String }`, SHA-256-Vergleich mit Konstantzeit.

## User

- Bedeutung: Dashboard-Benutzer: username, password_hash (PBKDF2-Format `pbkdf2-sha256$iters$salt$hash`), role (superuser) (belegt durch store.json-Seed, src/auth.rs).
- Lebenszyklus: Erster User im Setup; weitere via Settings-UI (Multi-User seit 0.6.0, belegt durch CHANGELOG).
- Ownership: Teil von StoredConfig (users).
- Beziehungen: Login-Authentifizierung (src/auth.rs), Session-Erzeugung.
- Persistenz: config.json (Hash, nie Klartext-Passwort).
- Validierung: PBKDF2-Format, Rollen-Enum, Passwort-Policy.
- Mutationen: Commit-Pipeline.
- Thread-Sicherheit: Immutabel pro Config-Stand.
- benötigte Rust-Konzepte: `struct User { username: String, password_hash: String, role: Role }`, PBKDF2-Verifikation.

## AppState

- Bedeutung: Geteilter Zustand der Anwendung (Config-Zugriff, Pool, History, Governor-Steuerung) (belegt durch src/lib.rs).
- Lebenszyklus: Erzeugt in app(), lebt für die Server-Laufzeit.
- Ownership: Arc-basiert, geteilt über alle Handler.
- Beziehungen: Bündelt Config, PoolHandle, History, Task-Spawner.
- Persistenz: Keine (nur In-Memory).
- Validierung: Keine.
- Mutationen: Durch Handler (Settings-PUT), Governor (Pool), History (Events).
- Thread-Sicherheit: Zentral — Arc<T> für Lesen, interne Mutexe/RwLocks für Mutation; config_revision als Invalidation-Signal.
- benötigte Rust-Konzepte: `struct AppState { ... }` + `Arc<AppState>` oder Aufteilung in Arc-Felder, `#[derive(Clone)]` für axum-Extractor.

## Lane / LaneSpec / Pool

- Bedeutung: Rate-Limit-Lane (eine pro Key/Modell): rate_remaining, capacity, enabled; LaneSpec als statische Konfiguration; Pool als Gesamtheit mit WINDOW 61s (belegt durch src/pool.rs).
- Lebenszyklus: Beim Pool-Rebuild (Config-Änderung) neu erzeugt; Governor mutiert Laufzeit-Zustand.
- Ownership: PoolHandle im AppState; Lanes intern.
- Beziehungen: Lane ↔ NimKey; Pool ↔ Governor (poll_cycle); Dispatch → Slot ↔ Lane.
- Persistenz: Keine (In-Memory).
- Validierung: Rate/Capacity-Werte aus Config.
- Mutationen: Governor (Grow/Shrink/Backoff), Dispatch (Reservation/Slot), Pool-Rebuild.
- Thread-Sicherheit: Mutex/RwLock-geschützte Lanes; Pool-Rebuild via Arc-Swap; Generation-Zähler gegen Stale-Slots.
- benötigte Rust-Konzepte: Sliding-Window-Zustand (Instant + Zähler), ArcSwap/RwLock für Pool-Tausch, u64-Generation.

## Session

- Bedeutung: Dashboard-Session: Expiry (12h), Username, PWHash-Fragment (belegt durch src/auth.rs).
- Lebenszyklus: Login → erzeugt; Logout/Expiry → entfernt.
- Ownership: Session-Store im AppState (In-Memory).
- Beziehungen: Cookie nimproxy_session ↔ Session-Entry.
- Persistenz: Keine (Restart = neue Sessions).
- Validierung: Cookie-Hash-Validierung, Expiry-Check.
- Mutationen: Login (Insert), Logout (Remove), Expiry (Delete).
- Thread-Sicherheit: Mutex-geschützter Store.
- benötigte Rust-Konzepte: HashMap + Mutex, Instant-Expiry, Cookie-Hash-Vergleich.

## History-Einträge (v1-Legacy / v2)

- Bedeutung: Persistierte Abfrage-/Rate-Limit-Events in history.jsonl; v2 mit Boot-Marker, v1 als Legacy-Format (belegt durch src/history.rs).
- Lebenszyklus: Events → Sampling (300s) → Datei-Anhang; Retention-Prune entfernt alte Samples.
- Ownership: History-Modul; Datei im DATA_DIR.
- Beziehungen: Samples → Dashboard-API (from/to/points).
- Persistenz: history.jsonl (JSON Lines), 1-MB/100k-Limits, atomare Writes (belegt durch src/history.rs).
- Validierung: Zeilen-Parsing; korrupte Zeilen werden übersprungen (Robustheit; Details nicht vollständig belegt).
- Mutationen: Anhang (Events), Prune (Retention), Boot-Marker (v2).
- Thread-Sicherheit: Mutex-geschützter Writer; Revisionen (history_revision, config_revision) für Reset-Erkennung.
- benötigte Rust-Konzepte: JSONL-Streaming (serde_json Deserializer), tempfile-Rotation, u64-Revisionen.

## Dashboard-Samples (API-Response)

- Bedeutung: Zeitreihen-Aggregate fürs Dashboard: Werte über Fenster, Punkte (default 288) (belegt durch src/history.rs, src/dashboard.html).
- Lebenszyklus: Bei API-Call aus History berechnet (kein eigener Persistenz-Zyklus).
- Ownership: Antwort-JSON.
- Beziehungen: from/to/points-Parameter ↔ Samples; /api/dashboard/now für Live-Werte.
- Persistenz: Keine.
- Validierung: Parameter-Bereiche.
- Mutationen: Keine (Lesepfad).
- Thread-Sicherheit: Keine geteilten Zustände.
- benötigte Rust-Konzepte: serde-Serialisierung, Zeitfenster-Aggregation, Downsampling (Counter-Deltas + Gauge-Ersetzung laut dashboard.html rangeSamples).

## GovernorSettings (fuzzing-relevanter Typ)

- Bedeutung: Parameter für den Governor (POLL, Backoff, Grow-Interval, Dissolve) (belegt durch src/lib.rs, src/governor.rs).
- Lebenszyklus: Erzeugt beim AppState-Aufbau; während der Laufzeit unverändert.
- Ownership: AppState.
- Beziehungen: Steuert do_poll_cycle-Verhalten.
- Persistenz: Keine.
- Validierung: Zeitwerte > 0.
- Mutationen: Keine (immutable zur Laufzeit).
- Thread-Sicherheit: Immutabel, frei teilbar.
- benötigte Rust-Konzepte: `struct GovernorSettings { poll: Duration, ... }`, Copy/Clone.

## config_revision / history_revision

- Bedeutung: Monotone Revisions-Zähler zur Erkennung von Config-/History-Reset (belegt durch src/config.rs, src/history.rs, decisions/reset-aware-dashboard-history.md).
- Lebenszyklus: Inkrement bei jeder Config-Speicherung bzw. History-Mutation.
- Ownership: Config-Layer bzw. History-Modul.
- Beziehungen: Dashboard-API liefert Revisionen; UI erkennt Reset und lädt neu (belegt durch src/dashboard.html).
- Persistenz: In-Memory (Zähler); History-Revision auch in der Datei sichtbar.
- Validierung: Keine.
- Mutationen: commit()-Pipeline (config_revision++), History-Writes (history_revision++).
- Thread-Sicherheit: Atomare Zähler (AtomicU64) oder unter Mutex.
- benötigte Rust-Konzepte: AtomicU64, Vergleichslogik im UI.

## Metrics (Prometheus)

- Bedeutung: Laufzeit-Metriken (nimproxy_queue_depth, nimproxy_affinity_total{result}, Tokens, Latency, Rate-Limit-Events) (belegt durch README-Metrik-Tabelle, src/dispatch.rs, src/pool.rs).
- Lebenszyklus: Prozess-lebenslang; Reset bei Neustart; History-Samples halten Verdichtung.
- Ownership: metrics-Registry (metrics 0.24 + metrics-exporter-prometheus 0.18).
- Beziehungen: Dashboard-Zeitreihen basieren auf History-Events, die aus Metrik-Daten gespeist werden (belegt durch src/history.rs).
- Persistenz: Keine direkt; indirekt via History.
- Validierung: Label-Sanitizing (sanitize_label, max 64, Alphabet) (belegt durch src/proxy.rs).
- Mutationen: Governor/Dispatcher/Proxy-Handler.
- Thread-Sicherheit: metrics-Crates sind intern synchronisiert.
- benötigte Rust-Konzepte: metrics-Crate-API, Label-Const für Stabilität.

## Request-Deadline

- Bedeutung: Explizite Deadline-Metadaten pro Request: x-nim-proxy-deadline-ms-Header (belegt durch CHANGELOG [0.6.2] explicit-request-deadline, decisions/explicit-request-deadline.md).
- Lebenszyklus: Pro Request; vom Client gesetzt oder Proxy-Default.
- Ownership: Request-Kontext.
- Beziehungen: Steuert Timeout/Shedding-Entscheidungen im Dispatch/Proxy.
- Persistenz: Keine.
- Validierung: Zeitstempel-Format, Zukunfts-/Vergangenheitswerte.
- Mutationen: Keine.
- Thread-Sicherheit: Pro-Request (kein Teilen).
- benötigte Rust-Konzepte: Header-Parsing, Instant-Berechnung, Deadlines in Waiters.
