# MindFoundry — Rust-Rewrite-Foundation (Neuentwicklung, keine Migration)

## Benötigte Capability

- Deterministische Simulationsgenerierung (Seed 42) mit exakten Zähl-Invarianten (250/500/8/500/3150/500, Wiederholungsverteilung 300/100/50).
- Zweisprachige (en/zh-TW) Nachrichten- und Vorlagen-Expansion mit Platzhaltern.
- SQLite-Speicher mit FK und Indizes; read-only-Konsumenten.
- Deterministisches Retrieval: Tokenisierung (CJK), Scoring (2-Punkt-exakt/1-Punkt-Präfix, Intent-Boni), Limits, Routing-Vorschlag, Guardrail-Summary.
- Replicant-Profile (Pro-Person-Wissen) mit Slice-Limits (case ≤4, discord ≤8).
- Policy-Redaktion: 6 Klassen, Rollen-Matrix, Audit-JSONL, `[KIND REDACTED]`.
- NIM-Synthese mit Zitatpflicht, Fallback-Renderer, Retry, Feature-Flag.
- Discord-API-v10-Client: Token, Kanäle, Chunking (1900), Rate-Limit (0,7 s), Gate vor Post.
- Agenten-Loops (Bridge-Watcher, Drip) mit State-Dedup (posted/answered/seen), cursor_created_at.
- Wissensextraktion: Kanal→Rolle, Staff-Prefix, Memory-Würdigkeit, SHA1-ID (12 Hex), Append-only.
- Google-Workspace-Provisionierung (Users, Groups, Drives, Docs) idempotent mit Status created/exists/added/skipped.
- HTTP-JSON-API (localhost) + statisches Dashboard.
- Evaluations-Harness (4 Kriterien) und Report-Schreiben.

## Benötigte Rust Traits

- `Serialize`/`Deserialize` (serde) für alle Datenmodelle, State- und Report-Formate.
- `FromStr`/`Display` für Enums (IncidentType, Severity, Status, Lang, Loyalty, RoomType, Role, Clearance, PolicyKind).
- `Error` + `std::error::Error`-Implementierung für den Fehler-Enum (with source chaining).
- `AsyncRead`/`AsyncWrite`-basiertes HTTP: `Service`/`ServiceFactory` (tower) für Testbarkeit der API.
- `trait Nimbus`-Client-Schnittstelle mit `MockNimbus` (Fallback-/Errorpfade testbar).
- `trait DiscordApi`/`trait GoogleApi` für Test-Doubles.
- `Iterator`-basiertes Event-Streaming (serde_json `StreamDeserializer`).
- `Hash`/`Eq` für Dedup-Keys (posted/answered/seen, SHA1-Hex).

## Benötigte Module

- `sim::generate` (Seed, Weighted-Sampling, Template-Expansion), `sim::schema`, `sim::types`.
- `db` (Connection-Pool, FK, Indizes, Read-Models, Aggregationen).
- `rag::retrieve` (tokenize, score_text, classify_query, live-injection), `rag::replicants` (Profile), `rag::live_ops` (Summary).
- `policy::gate` (Klassen-Regexe, Rollen-Matrix, Audit), `policy::docs` (Snippets).
- `nim` (Client, Payload, Fallback-Renderer, Retry, Feature-Flag).
- `discord` (API-Client, Chunking, Kanal-Mapping, Token-Loader, Utils).
- `agents::bridge` (Watcher-Loop, State, Dedup), `agents::drip`, `agents::seed`, `agents::updater`.
- `eval` (DeterministicAgent, Score, Report).
- `workspace` (Import-Prep, Provisioning-Client, RBAC-Mappings).
- `api` (Router, Handler, Static-Assets, JSON-Middleware).
- `state` (Atomic-JSON-State, Safe-Writes).

## Benötigte Services

- `SimulationService` (generate, verifizieren: Zähl-Invarianten, Verteilungen).
- `RetrievalService` (query → Citations + Route + Guardrail).
- `PolicyService` (gate_text, gate_payload, Audit-Abhängigkeit).
- `SynthesisService` (NIM oder Fallback; Quellenliste).
- `DiscordService` (send, read, audit-Post, Chunking, Gate).
- `ReplicantService` (build, retrieve, summary).
- `LiveOpsService` (Aggregate, Owner-Queues, Urgent).
- `ProvisioningService` (User/Gruppen/Drives/Docs, Status-Enum).
- `ApiService` (Router + Assets).

## Benötigte Events

- `MessageReceived { channel, author, content }` (Bridge/Updater-Eingang).
- `QuestionAnswered { id, used_nim, redaction_count, error? }` (Audit).
- `IncidentPosted { incident_id, channel, mirror? }` (Drip/Seed-Ausgang).
- `IncidentFollowupPosted { incident_id, staff, severity }`.
- `MemoryAppended { person, memory_id }` (Updater-Ausgang).
- `GateViolation { kind, destination, sample }` (Redaktions-Audit).
- `ProvisionStatus { resource, status }` (created/exists/added/skipped).
- `EvalCompleted { report_path, score_distribution }`.
- `LiveSummaryRefreshed { summary, counts }`.

## Benötigte Storage Layer

- SQLite (rusqlite/sqlx): Tables/Indizes/FK; read-only Konsumenten; Pooling für API.
- Append-only JSONL (Memories, Event-Stream, Audit): StreamWriter mit fsync-Konfiguration.
- Atomic State JSON (temp+rename) für Agenten-Dedup-Zustände.
- Markdown-Policy-Store (Snippet-Index mit doc-id/§-Anker).
- Report-Store (eval-JSON, live-ops-txt, provision-JSON).
- Secrets-Store (NVIDIA-Key, Discord-Token, Workspace-Credentials; 0600-Permissions; `~/.openclaw/secure/`-Äquivalent).

## Benötigte Async-Komponenten

- Async HTTP-Server (axum) mit State-Share (Arc<AppState>) und Read-Models.
- Async NIM-Client (reqwest) mit Retry/Timeout (tokio::time).
- Async Discord-Client (reqwest) mit Rate-Limit-Sleeper.
- Watcher-Loops (bridge/drip) als tokio-Tasks mit Graceful-Shutdown (CancellationToken/abort_on_drop).
- Async Google-API-Client (oauth2 + google-apis).
- Kein paralleler Schreibzugriff auf JSONL/State nötig — Serialisierung über dedizierte Tasks/Channels.

## Benötigte Error Types

- `SimError` (Seed-Mismatch, Verteilungs-Assertion).
- `DbError` (rusqlite::Error, Constraint-Violation).
- `RetrievalError` (leerer Corpus, Fehl-Config).
- `GateError` (ungültige Destination → Fallback public_discord).
- `NimError` (MissingKey, InvalidKeyEncoding, Http{status,body}, Network{attempts}, Decode, Payload).
- `DiscordError` (Http{status}, Token, RateLimit, Audit-Fehler).
- `StateError` (fehlende/korrupte State-JSON → Init).
- `ProvisionError` (Credential, Api{status}, NotFound, Exists, Skipped).
- `ApiError` (400/404/500 mit serde-Responses).
- Zentrale Verwendung: `thiserror` für Definition, `anyhow` nur an Prozess-Grenzen (main).

## Benötigte Crates

- tokio (Runtime, Tasks, time), axum + tower + hyper (API), reqwest (HTTP-Client), serde/serde_json (Modelle), rusqlite (SQLite), chrono (Zeitstempel, +8), rand_chacha/rand (Seed-Stabilität), sha1 (Memory-ID), regex (Klassen-Redaktion, Scoring), pulldown-cmark (Policy-Snippets), csv (Importe), thiserror/anyhow (Fehler), tracing + tracing-subscriber (Logging), uuid oder Sluggish-IDs (falls nötig), futures (Streams), tempfile (Atomic-Writes), mockall (Traits-Test-Doubles), proptest (Generator-/Regex-Eigenschaften).

## Architekturentscheidung

- Schichten: domäne-agnostische Core-Crates (`mindfoundry-sim`, `mindfoundry-rag`, `mindfoundry-policy`) getrennt von Integrations-Crates (`mindfoundry-discord`, `mindfoundry-nim`, `mindfoundry-workspace`), API/UI (`mindfoundry-api`), Agenten (`mindfoundry-agents`), Zustand/Storage (`mindfoundry-state`).
- Konfiguration über Typed Config (env + Datei) statt Umgebungsvariablen-Zerstreuung; Feature-Flags (nim, discord, workspace) analog NIM_DISABLE.
- Retrieval bleibt deterministisch (kein Vector-Store-Pflicht): Keyword-Scoring in Rust mit Trait-Erweiterung für spätere Vektor-Retrieval-Provider.
- State-Dedup einheitlich über Atomic-JSON-Store-Trait (Bridge, Drip, Seed, Updater nutzen dieselbe Abstraktion).
- Testbarkeit: alle externen Systeme (NIM, Discord, Google) hinter Traits; SQLite-Fixtures statt Seed-42-Produktivdaten in Tests; Invarianten als Consts + Tests.
- Determinismus: Seed-Reproduktion über rand_chacha mit striktem Generator-Ersatz; Zähl-Invarianten als Test-Consts.
