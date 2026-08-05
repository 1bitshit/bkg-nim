# nim-proxy — Funktionsanalyse

Funktionen aus `src/*.rs` (10 Dateien). Jede Fähigkeit beschrieben nach Zweck, Problem, Regeln, Datenveränderung, Zuständen, Algorithmen, Fehlern, Seiteneffekten. Belegquellen in Klammern (Datei + Zeile, wo ermittelbar).

---

## src/main.rs

### main()

- Warum existiert sie: Prozess-Einstiegspunkt; startet Logging, parse CLI, ruft `Server::run()` auf (belegt durch Dateiinhalt).
- Welches Problem löst sie: Bootstrapping von Env/CLI/Tokio-Runtime mit sauberen Exit-Codes.
- Welche Regeln implementiert sie: CLI-Flags `--health` (Exit 0 bei erreichbarem Server), `--version`/`-V` (Version aus Cargo), Default-Port 8000 (belegt durch Dateiinhalt, README).
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Serve-/Healthcheck-/Version-Modus.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Fehlende/fehlerhafte Env, Healthcheck-Timeout → Exit 1.
- Welche Seiteneffekte besitzt sie: Startet Netzwerk-Listener, konfiguriert Tracing.
- Rust-Relevanz: `#[tokio::main]`, clap-ähnliches CLI oder manuelles Parsing, Panic-Hook.

## src/lib.rs

### app()

- Warum existiert sie: Baut den vollständigen axum-Router mit State und Fallback (belegt durch Dateiinhalt).
- Welches Problem löst sie: Komposition aller Handler zu einem testbaren Server.
- Welche Regeln implementiert sie: Routen-Zuordnung, Fallback-Handling, State-Threading.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: AppState wird erzeugt.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Fallback-Routen (404).
- Welche Seiteneffekte besitzt sie: Setzt die Governor-Loop auf.
- Rust-Relevanz: axum-Router-Builder mit `Router::new()`, `with_state()`.

### (cfg(fuzzing)-Re-Exporte)

- Warum existieren sie: Machen interne Funktionen (`dispatch::schedule`, `governor::do_poll_cycle`, `proxy::sanitize_label`) nur unter Fuzz-Cfg erreichbar (belegt durch Dateiinhalt).
- Welches Problem lösen sie: Fuzz-Targets können interne Pfade ohne Feature-Flags ansprechen.
- Welche Regeln implementiert sie: Nur-Fuzz-Sichtbarkeit (kein Produktions-API-Leak).
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Keine.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Keine.
- Welche Seiteneffekte besitzt sie: Keine.
- Rust-Relevanz: `#[cfg(fuzzing)]`-Pattern für Fuzz-Re-Exporte ist direkt übernehmbar.

## src/auth.rs

### Login-Handler (POST /login)

- Warum existiert sie: Authentifiziert Dashboard-Nutzer; erzeugt Session-Cookie (belegt durch Dateiinhalt, CHANGELOG).
- Welches Problem löst sie: Zugriffskontrolle auf Dashboard/API; Session-Lebenszyklus (12h TTL).
- Welche Regeln implementiert sie: PBKDF2-600k-Hash-Verifikation, Konstantzeit-Vergleich, Login-Throttling, Session-Format `pbkdf2-sha256$iters$salt$hash`.
- Welche Daten verändert sie: Session-Store (neu), Throttle-Zähler.
- Welche Zustände entstehen: Session aktiv/expired; Login-Throttle.
- Welche Algorithmen werden verwendet: PBKDF2-SHA256 (600k), Konstantzeit-Vergleich (subtle).
- Welche Fehler behandelt sie: Falsche Credentials, unbekannter User, Throttle-Block.
- Welche Seiteneffekte besitzt sie: Setzt `Set-Cookie` (nimproxy_session), protokolliert Login-Events.
- Rust-Relevanz: hmac/sha2/subtle/getrandom-Stack, Session-TTL via tokio-time, Throttling via Zähler + Zeitfenster.

### Logout-Handler (POST /logout)

- Warum existiert sie: Beendet Session, invalidiert Cookie (belegt durch Dateiinhalt).
- Welches Problem löst sie: Sauberes Session-Ende.
- Welche Regeln implementiert sie: Cookie-Löschung, Session-Entfernung.
- Welche Daten verändert sie: Session-Store (Entfernung).
- Welche Zustände entstehen: Session beendet.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Ungültige Session.
- Welche Seiteneffekte besitzt sie: Set-Cookie-Löschung.
- Rust-Relevanz: Session-Store-Mutation, Cookie-Header-Handling.

### Scraper-Auth (Basic)

- Warum existiert sie: Erlaubt Programmatic Access für Scraper (Basic-Auth `Bearer <user>:<password>` laut README) (belegt durch Dateiinhalt, README).
- Welches Problem löst sie: CLI/Harness-Zugriff ohne Browser-Login.
- Welche Regeln implementiert sie: Basic-Auth-Parsing, Credential-Prüfung.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Keine.
- Welche Algorithmen werden verwendet: Konstantzeit-Vergleich.
- Welche Fehler behandelt sie: Fehlende/falsche Credentials (401).
- Welche Seiteneffekte besitzt sie: Keine.
- Rust-Relevanz: Authorization-Header-Parsing, Konstantzeit-Vergleich.

## src/config.rs

### Config-Load

- Warum existiert sie: Liest und validiert `config.json` aus DATA_DIR (belegt durch Dateiinhalt).
- Welches Problem löst sie: Persistente Konfiguration mit Schema-Versionierung.
- Welche Regeln implementiert sie: Mode Open/Keyed (Default Keyed = fail-closed), Datei 0600, atomare Writes, korrupte Datei = harter Boot-Fehler.
- Welche Daten verändert sie: In-Memory-Config, `config_revision`.
- Welche Zustände entstehen: Config geladen/fehlerhaft.
- Welche Algorithmen werden verwendet: Serde-Deserialisierung, PBKDF2-Hash-Verarbeitung.
- Welche Fehler behandelt sie: Korrupte JSON, fehlende Datei (erste Boot), unzulässige Werte.
- Welche Seiteneffekte besitzt sie: Schreibt ggf. Default-Config beim Erststart.
- Rust-Relevanz: serde-Derive, Fehler-Enum, atomare Datei-IO, Revision-Counter.

### Config-Save

- Warum existiert sie: Persistiert Config-Änderungen (belegt durch Dateiinhalt).
- Welches Problem löst sie: Dauerhaftigkeit der UI-Config.
- Welche Regeln implementiert sie: 0600, atomar (tmp+rename), Revision-Inkrement.
- Welche Daten verändert sie: config.json, In-Memory-Config.
- Welche Zustände entstehen: Gespeichert/Fehler.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Schreibfehler, fehlende Rechte.
- Welche Seiteneffekte besitzt sie: Dateiänderung, Revision.
- Rust-Relevanz: tempfile-Rotation, Permissions-Set.

## src/dispatch.rs

### schedule() / Slot-Acquire

- Warum existiert sie: Koordiniert Anfragen auf Pool-Slots mit Generation-Check (belegt durch Dateiinhalt, lib.rs fuzzing-Re-Export).
- Welches Problem löst sie: Verhindert ABA-/Stale-Slot-Probleme bei Pool-Rebuilds; erzwingt fair/priorisiertes Warten.
- Welche Regeln implementiert sie: GRANT_GAP 25ms, Waiter mit Deadline und prefer, Shedding bei Überlast, Generation-Validierung.
- Welche Daten verändert sie: Slot-Zustand (in-use), Waiter-Queue, Metriken (nimproxy_queue_depth).
- Welche Zustände entstehen: Slot frei/belegt/abgelaufen; Waiter wartend/zeitabgelaufen.
- Welche Algorithmen werden verwendet: Generation-Check, Deadline-basierte Wartezeit, prefer-Priorität.
- Welche Fehler behandelt sie: Timeout, Generation-Mismatch, Pool-leer (Shedding → 503).
- Welche Seiteneffekte besitzt sie: Metrik-Updates, Waiter-Aktivierung.
- Rust-Relevanz: tokio-sync-Primitive (Mutex/Notify), Instant-Deadlines, u64-Generation-Token.

### Slot-Release

- Warum existiert sie: Gibt Slots nach Anfrageende frei (belegt durch Dateiinhalt).
- Welches Problem löst sie: Kapazität-Rückgabe.
- Welche Regeln implementiert sie: Generation-konsistentes Freigeben, Waiter-Benachrichtigung.
- Welche Daten verändert sie: Slot (frei), Metriken.
- Welche Zustände entstehen: Slot verfügbar.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Doppelte Freigabe (Generation).
- Welche Seiteneffekte besitzt sie: Weckt Waiter.
- Rust-Relevanz: Notify-Wakeup, Drop-basierte Freigabe.

## src/governor.rs

### do_poll_cycle()

- Warum existiert sie: Periodische Governor-Entscheidung: Exhaustion-Backoff, adaptive Growth, Dissolve (belegt durch Dateiinhalt, lib.rs fuzzing-Re-Export).
- Welches Problem löst sie: Erhält das Null-Violations-Versprechen dynamisch (Rate-Limit-Anpassung).
- Welche Regeln implementiert sie: POLL 250ms, EXHAUST_BACKOFF 2s, GROW_INTERVAL 60s, DISSOLVE_AFTER 30min.
- Welche Daten verändert sie: Lane-Zustände, Pool-Zustand, Metriken.
- Welche Zustände entstehen: Enabled → Exhausted → Backoff → Grow; Dissolve.
- Welche Algorithmen werden verwendet: Exponential Backoff, adaptive Growth-Entscheidung.
- Welche Fehler behandelt sie: Keine harten Fehler (alle Zustandsübergänge intern).
- Welche Seiteneffekte besitzt sie: Pool-Mutation, Metrik-Events.
- Rust-Relevanz: tokio-time-Loop, Zustandsmaschine als enum, RAII-Permit.

### ModelPermit (RAII)

- Warum existiert sie: Garantiert Permit-Freigabe beim Drop (belegt durch Dateiinhalt).
- Welches Problem löst sie: Leak-freie Slot-Bewirtschaftung auch bei Panics/Early-Returns.
- Welche Regeln implementiert sie: Drop → Release.
- Welche Daten verändert sie: Slot-Zustand.
- Welche Zustände entstehen: Permit aktiv/freigegeben.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Keine.
- Welche Seiteneffekte besitzt sie: Slot-Release beim Scope-Ende.
- Rust-Relevanz: Drop-Trait-Muster.

## src/pool.rs

### Pool-Lookup / Lane-Auswahl

- Warum existiert sie: Findet passende Lane für ein Modell mit Affinity (belegt durch Dateiinhalt).
- Welches Problem löst sie: Modell → Key/Lane-Zuordnung unter Capacity-Regeln.
- Welche Regeln implementiert sie: Enabled-Lanes zuerst, Affinity sticky → spill, WINDOW 61s (Jitter-Marge).
- Welche Daten verändert sie: Keine (Lesepfad).
- Welche Zustände entstehen: Lane gefunden/nicht gefunden.
- Welche Algorithmen werden verwendet: Sliding-Window-Zählung, Affinity-Ranking.
- Welche Fehler behandelt sie: Keine Lane → keine Reservierung (Shedding).
- Welche Seiteneffekte besitzt sie: Affinity-Metriken (nimproxy_affinity_total{result}).
- Rust-Relevanz: HashMap-Lookup, Ranking-Sortierung, Window-Timeout.

### Reservation (Ready/Wait)

- Warum existiert sie: Reserviert Kapazität einer Lane für eine Anfrage (belegt durch Dateiinhalt).
- Welches Problem löst sie: Deterministische Kapazitätszusage vor Upstream-Call.
- Welche Regeln implementiert sie: Rate-remaining-Check, Ready/Wait-Semantik, Capacity-basiertes RPM.
- Welche Daten verändert sie: Lane-Zähler (rate_remaining), Metriken.
- Welche Zustände entstehen: Ready (sofort), Wait (bis Window), Timeout.
- Welche Algorithmen werden verwendet: Sliding-Window-Reservierung.
- Welche Fehler behandelt sie: Auslastung → Wait, Kapazität erschöpft → Fehler.
- Welche Seiteneffekte besitzt sie: Zähler-Dekrement, Metriken.
- Rust-Relevanz: Mutex-geschützte Lane-State, Instant-Window.

## src/proxy.rs

### sanitize_label()

- Warum existiert sie: Bereinigt Prometheus-Labels (belegt durch Dateiinhalt, fuzz-Target).
- Welches Problem löst sie: Metrik-Injection via Modell-IDs/Parametern.
- Welche Regeln implementiert sie: Max 64 Zeichen, Alphabet `[A-Za-z0-9._/:-]`, Label-Cap 256 → other.
- Welche Daten verändert sie: Keine (reine Funktion).
- Welche Zustände entstehen: Keine.
- Welche Algorithmen werden verwendet: Zeichenfilter/Ersetzung, Truncation.
- Welche Fehler behandelt sie: Ungültige Zeichen, Überlänge.
- Welche Seiteneffekte besitzt sie: Keine.
- Rust-Relevanz: Reine Funktion; Property-Tests; fuzz-Target.

### Chat-Completion-Forward (POST /v1/chat/completions)

- Warum existiert sie: Proxied Anfragen an NVIDIA NIM mit Pacing (belegt durch Dateiinhalt, README).
- Welches Problem löst sie: Hält das per-Key-Rate-Limit unter Last.
- Welche Regeln implementiert sie: Request-Timeout, Stream-Idle-Timeout (120s), Heartbeat (10s), 1-MiB-Payload-Guard, Deadline-Header (x-nim-proxy-deadline-ms), strict_passthrough-Modus.
- Welche Daten verändert sie: Pool-Reservierungen, History-Events, Metriken.
- Welche Zustände entstehen: Streaming aktiv/idle/beendet; Deadline abgelaufen.
- Welche Algorithmen werden verwendet: SSE-Parsing, Token-Counting, Deadline-Berechnung.
- Welche Fehler behandelt sie: Upstream 5xx, 429 (Rate-Limit), Timeouts, Payload-Too-Large, Invalid-Label.
- Welche Seiteneffekte besitzt sie: Upstream-Request, Metrik-Injection, History-Writes.
- Rust-Relevanz: reqwest-stream, futures-util StreamExt, tokio-timeouts.

### Usage-Injection (mit Auto-Fallback)

- Warum existiert sie: Ergänzt fehlende usage-Felder für Metrik-Konsistenz (belegt durch Dateiinhalt, decisions/usage-injection-auto-fallback.md).
- Welches Problem löst sie: Dashboard/Token-Metriken ohne zuverlässige usage-Angaben.
- Welche Regeln implementiert sie: Nur injizieren wenn usage fehlt (Auto-Fallback); strict_passthrough deaktiviert Injection.
- Welche Daten verändert sie: Response-Body (usage).
- Welche Zustände entstehen: Injectiert/Nicht-injiziert.
- Welche Algorithmen werden verwendet: Token-Schätzung/Counting.
- Welche Fehler behandelt sie: Fehlendes usage.
- Welche Seiteneffekte besitzt sie: Response-Mutation.
- Rust-Relevanz: serde_json-Mutation im Stream, Feature-Flag-Logik.

### SSE-Heartbeat

- Warum existiert sie: Hält Client-Verbindung während Rate-Waits offen (belegt durch Dateiinhalt, decisions/sse-heartbeats-for-rate-waits.md).
- Welches Problem löst sie: Timeouts/Disconnects bei langen Wartezeiten.
- Welche Regeln implementiert sie: Heartbeat-Intervall (10s, aus limits), Stream-Idle-Timeout.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Heartbeat-Timer aktiv.
- Welche Algorithmen werden verwendet: Timer-basierte Heartbeats.
- Welche Fehler behandelt sie: Idle-Verbindungen.
- Welche Seiteneffekte besitzt sie: Sendet SSE-Kommentarzeilen.
- Rust-Relevanz: tokio-time-Intervall im Stream.

### Models-Forward (GET /v1/models)

- Warum existiert sie: Leitet die Modell-Liste an Clients weiter (belegt durch README, e2e).
- Welches Problem löst sie: Client-Erkennung verfügbarer Modelle.
- Welche Regeln implementiert sie: Upstream-Weiterleitung, ggf. Filterung (Details nicht vollständig belegt).
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Keine.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Upstream-Fehler.
- Welche Seiteneffekte besitzt sie: Upstream-Request.
- Rust-Relevanz: einfacher Proxy-Handler.

## src/settings.rs

### commit()-Pipeline

- Warum existiert sie: Persistiert und aktiviert Settings-Änderungen atomar (belegt durch Dateiinhalt).
- Welches Problem löst sie: Konsistente Config-Mutation mit Rollback-Möglichkeit.
- Welche Regeln implementiert sie: validate → save → cfg-write → Pool-Rebuild (Rate-Carryover) → Retention → guard → config_revision++.
- Welche Daten verändert sie: config.json, Pool, History, Revision.
- Welche Zustände entstehen: Commit abgeschlossen/fehlgeschlagen (Rollback).
- Welche Algorithmen werden verwendet: Pipeline-Validierung, Rate-Carryover beim Pool-Rebuild.
- Welche Fehler behandelt sie: Validierungsfehler, Persistenzfehler.
- Welche Seiteneffekte besitzt sie: Pool-Neubau, Retention-Prune, Revision-Inkrement.
- Rust-Relevanz: Transaction-Pattern, Result-Ketten, Revision-Counter.

### json_error-Shape

- Warum existiert sie: Einheitliches Fehlerformat `{error:{message,type:"proxy_error",code}}` (belegt durch Dateiinhalt).
- Welches Problem löst sie: Maschinenlesbare, konsistente Fehlerantworten.
- Welche Regeln implementiert sie: Feste Shape, Typ- und Code-Felder.
- Welche Daten verändert sie: Keine.
- Welche Zustände entstehen: Keine.
- Welche Algorithmen werden verwendet: Keine.
- Welche Fehler behandelt sie: Alle API-Fehler.
- Welche Seiteneffekte besitzt sie: Keine.
- Rust-Relevanz: Fehler-Enum → JSON-Mapping.

## src/history.rs

### Sampling (SAMPLE_SECS 300)

- Warum existiert sie: Aggregiert Abfragen/Rate-Limit-Events in 5-Minuten-Samples (belegt durch Dateiinhalt).
- Welches Problem löst sie: Dashboard-Zeitreihen ohne Event-Flut.
- Welche Regeln implementiert sie: SAMPLE_SECS 300, 1-MB/100k-Limits, v1-Legacy + v2-Boot-Marker.
- Welche Daten verändert sie: history.jsonl, history_revision.
- Welche Zustände entstehen: Sample aktuell/abgeschlossen.
- Welche Algorithmen werden verwendet: Zeitfenster-Aggregation.
- Welche Fehler behandelt sie: Datei-Limits, korrupte Zeilen.
- Welche Seiteneffekte besitzt sie: Atomare Datei-Writes.
- Rust-Relevanz: JSONL-Streaming, tempfile-Rotation, Revisionen.

### Retention-Prune

- Warum existiert sie: Löscht alte Samples nach retention_days (belegt durch e2e retention_change_prunes_queries_and_disk).
- Welches Problem löst sie: Disk/Datei-Begrenzung.
- Welche Regeln implementiert sie: retention_days-Config, Prune bei Config-Änderung.
- Welche Daten verändert sie: history.jsonl (Verkürzung).
- Welche Zustände entstehen: Prune gelaufen.
- Welche Algorithmen werden verwendet: Zeitfilter.
- Welche Fehler behandelt sie: Leere/fehlende Datei.
- Welche Seiteneffekte besitzt sie: Disk-Schreibvorgang.
- Rust-Relevanz: Zeitbasierte Filterung, Datei-Rewrite.

### Dashboard-Queries (/api/dashboard)

- Warum existiert sie: Liefert Zeitreihen fürs Dashboard (belegt durch Dateiinhalt, e2e).
- Welches Problem löst sie: History-Visualisierung mit Fenster-/Punkt-Steuerung.
- Welche Regeln implementiert sie: `from`/`to`-Fenster, `points=288` (Tagesauflösung), `/api/dashboard/now` für Live-Daten.
- Welche Daten verändert sie: Keine (Lesepfad).
- Welche Zustände entstehen: Keine.
- Welche Algorithmen werden verwendet: Zeitfenster-Aggregation, Downsampling.
- Welche Fehler behandelt sie: Ungültige Parameter.
- Welche Seiteneffekte besitzt sie: Keine.
- Rust-Relevanz: Query-Parameter-Parsing, Sampling-Logik.
