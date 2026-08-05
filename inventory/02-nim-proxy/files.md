# nim-proxy — Datei-für-Datei-Analyse

Alle Pfade relativ zu `source/nim-proxy/`. Belegquellen in Klammern. Insgesamt 113 Dateien (ohne `.git`).

---

## Cargo.toml

- Zweck: Manifest des Rust-Pakets `nim-proxy` (Version 0.6.5, edition 2021, MSRV 1.87, MIT) mit Abhängigkeiten, Profilen und Lint-Konfiguration (belegt durch Dateiinhalt).
- Verantwortlichkeit: Deklariert die Laufzeit- und Dev-Dependency-Graphen, die Release-/Dev-Optimierungsprofile und die `cfg(fuzzing)`-Check-Config für `unexpected_cfgs`.
- Eingaben: Keine zur Laufzeit; Build-Zeit: Cargo/Crates.io.
- Ausgaben: Lockfile-Referenzen (`Cargo.lock`), Binär-/Lib-Artefakte.
- Datenfluss: Kein Laufzeit-Datenfluss; steuert den Cargo-Build-Graphen.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Legt die Auslieferungsform fest: `--profile release` mit `lto = true`, `strip = true`, `opt-level = 3`, `codegen-units = 1` (Geschwindigkeit statt `"z"`-Größe, belegt durch Kommentar); `[profile.dev] opt-level = 1` plus `opt-level = 3` für `sha2`/`hmac`/`digest`/`block-buffer`/`generic-array`, damit PBKDF2-Logins (600k Iterationen) in Dev-Builds nahe Release-Tempo laufen (belegt durch Kommentar).
- Fehlerfälle: MSRV-2-Abweichung bricht CI-`msrv`-Job (`cargo +1.87 check --locked --all-targets`) (belegt durch `.github/workflows/ci.yml`).
- Sicherheitsrelevanz: Auth-Crypto-Pins `hmac 0.12`/`sha2 0.10`/`getrandom 0.2` (stabil belegt durch CHANGELOG [0.6.3]/[0.5.0] "Dependabot only takes patches"); `reqwest` ohne Default-Features (nur rustls-tls, stream, gzip — kein native-tls) (belegt durch Dateiinhalt).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Cargo/Rustup; extern: axum 0.8, tokio 1, reqwest 0.12 (rustls), serde/serde_json, metrics 0.24 + metrics-exporter-prometheus 0.18, bytes, futures-util, tokio-stream, tracing(+subscriber env-filter), dotenvy 0.15, hmac/sha2/subtle/getrandom; Dev: axum, tokio(full), reqwest(json), serde_json, bytes, futures-util, sha2 (belegt durch Dateiinhalt).
- Rust-Relevanz: Das Rust-2024-Rewrite übernimmt die Dependency-Struktur als Referenz: axum 0.8 + tokio + reqwest-rustls + metrics-Stack; PBKDF2-Stack bleibt hmac/sha2/getrandom; `subtle` für Konstantzeit-Vergleiche; das Profil-Setup (Dev-opt-level für Crypto, Release lto+codegen-units=1) ist direkt wiederverwendbar.

## Cargo.lock

- Zweck: Pinned transitive Dependency-Graph (exakte Versionen inkl. transitiver Crates wie `crossbeam-epoch` 0.9.20 nach RUSTSEC-2026-0204-Fix, belegt durch CHANGELOG [0.6.4]).
- Verantwortlichkeit: Reproduzierbare Builds (`--locked` in CI) und Grundlage für `cargo-deny`-Advisories-Prüfung (belegt durch `.github/workflows/ci.yml`, `deny.toml`).
- Eingaben: `Cargo.toml` + Crates.io-Metadaten.
- Ausgaben: Deterministischer Build.
- Datenfluss: Build-Graph.
- Persistenz: Versionskontrolle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Lockfile-Only-Updates (z. B. crossbeam-epoch 0.9.20) sind der dokumentierte Weg für Advisory-Fixes (belegt durch CHANGELOG [0.6.4]).
- Fehlerfälle: `--locked`-Mismatch bricht CI; bekannte Crates mit Advisory blockieren `cargo-deny` (red on main, belegt durch CHANGELOG [0.6.4]).
- Sicherheitsrelevanz: Enthält die von RUSTSEC geprüften transitiven Versionen; enthält relevante Crates wie `loom`, `evmap`, `left-right`, `quanta`, `foldhash` (Vorhandensein belegt; direkte Nutzung nicht nachweisbar).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Cargo.
- Rust-Relevanz: Für das Rust-Rewrite: gleiche Lockfile-Disziplin (`--locked`, cargo-deny-advisories, Dependabot-Cooldown) übernehmen; Crate-Wahl an dieser Liste orientieren.

## README.md

- Zweck: Projekt-Überblick, Quickstart, Client-Rezepte (OpenCode/Codex/n8n/curl), Dashboard-Beschreibung, Funktionsweise, Konfiguration, Security & Deployment, Operations, vollständige Metrik-Referenztabelle (26 Zeilen), Testanleitung, FAQ, Projekt-Wissensbasis-Verweis (belegt durch Dateiinhalt, 366 Zeilen).
- Verantwortlichkeit: Öffentliche Einstiegs- und Betriebsdokumentation; maßgebliche Beschreibung der Fail-closed-Postur, Rollen und Deployment-Muster.
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Keine zur Laufzeit.
- Datenfluss: Kein Laufzeit-Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Dokumentiert `/v1`, `/metrics`, `/health`, `/setup`, Dashboard, `X-Nim-Proxy-Deadline-Ms` (belegt durch Dateiinhalt).
- Ereignisse: Keine.
- Nebenwirkungen: Verweist auf `docs/assets/*.png`-Screenshots (setup-wizard, dashboard-models, dashboard-reliability, dashboard-settings) (belegt durch Dateiinhalt).
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Dokumentiert Fail-closed, Rollen, Client-Key-Einmal-Sichtbarkeit, PBKDF2 600k, Scraper-Auth (`Bearer <user>:<password>`), `TRUST_PROXY`/`Secure`-Cookie, kein eingebautes TLS, Supply-Chain (cosign/SLSA/SBOM), `v*`-Tag-Schutz (belegt durch Dateiinhalt).
- Geschäftslogik: Kernversprechen "null upstream rate violations" (Load-getestet: 100 Clients, 0 Violations); "nicht designed um ToS zu umgehen" (belegt durch Dateiinhalt).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Primäre Anforderungsquelle (Capabilities, Metriken, Env-Vars, Endpunkte, Invarianten) für das Rust-2024-Rewrite. Rust-Relevanz: Keine (Dokumentation).

## CHANGELOG.md

- Zweck: Versionshistorie im Keep-a-Changelog-Format von 0.1.0 (2026-07-01) bis 0.6.5 (2026-07-28) plus [Unreleased] (belegt durch Dateiinhalt, 522 Zeilen).
- Verantwortlichkeit: Belegt die Entwicklungsschritte und ist Pflicht-Doku bei PRs (`[Unreleased]`-Update, belegt durch `.github/PULL_REQUEST_TEMPLATE.md`).
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Versionskontrolle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Dokumentiert alle Security-Fixes (RUSTSEC-2026-0204, Panic-Fix v0.5.0, XSS-Kette v0.3.0, Fail-closed v0.3.0) und Supply-Chain-Maßnahmen (belegt durch Dateiinhalt).
- Geschäftslogik: Relevante Meilensteine: 0.1.0 Kern-Proxy; 0.2.0 Metriken+Dashboard; 0.3.0 Security-Hardening; 0.4.0 Observability; 0.5.0 Public+Signing; 0.6.0 UI-Config-Store+Multi-User+Governor+Dashboard-Redesign (Breaking: alle App-Env-Vars weg); 0.6.1–0.6.5 Release-Automation, Deadline-Header, Dashboard-Verträge/Retention (belegt durch Dateiinhalt).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Belegquelle für tests.md/nim.md/rust-foundation.md (welche Capabilities wann und warum entstanden; welche Bewährte-Entscheidungen gelten). Rust-Relevanz: Keine (Dokumentation).

## CONTRIBUTING.md

- Zweck: Entwicklungs-Workflow, Test-/Format-/Lint-Pflichten, Fail-closed-Testpostur, Load-Harness-Pflicht, Fuzz-Anleitung, Dashboard-Regeln, Knowledge-Base-Lockstep-Regel, PR-Erwartungen (belegt durch Dateiinhalt, 158 Zeilen).
- Verantwortlichkeit: Definiert den Qualitätsvertrag für Beiträge ("69 unit + 53 end-to-end tests", Coverage ≥90 %, MSRV-Job, gitleaks, actionlint+zizmor, dependency review, Docker-Smoke; SHA-Pinning bei Workflow-Änderungen) (belegt durch Dateiinhalt).
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Dokumentiert `cargo test`, `cargo llvm-cov`, `cargo +nightly fuzz run sse_scan`, Load-Harness-Befehle (belegt durch Dateiinhalt).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Pflicht-Invarianten bei Änderungen an `src/auth.rs`, `src/config.rs`, `src/settings.rs`, API-Key-Gate/Label-Sanitizing in `src/proxy.rs`, Dashboard-`innerHTML` (belegt durch Dateiinhalt).
- Geschäftslogik: Zero-Violations-Hard-Requirement für Rate-Limit-Änderungen; Lockstep-Knowledge-Base als harte Regel (belegt durch Dateiinhalt).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Definiert die Test- und Qualitäts-Gates, die das Rust-Rewrite erfüllen muss. Rust-Relevanz: Keine (Dokumentation).

## SECURITY.md

- Zweck: Security-Policy: Supported-Versions (nur 0.6.x), private Meldung via GitHub Security Advisories, SLAs (Acknowledge 3 Werktage, Triage 7), Threat-Modell in/out of scope, Schutzmaßnahmen-Übersicht (belegt durch Dateiinhalt, 123 Zeilen).
- Verantwortlichkeit: Sicherheits-Reaktion und Dokumentation des Vertrauensmodells.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Zentrales Sicherheits-Dokument: In-Scope (Auth-Bypass, Injection, Rate-Limit-Verstoß, Secret-Leak, DoS am `max_inflight`), Out-of-Scope (open-Mode auf Netz-Interface, Setup-Claim-Fenster, fehlendes TLS, NVIDIA-ToS) (belegt durch Dateiinhalt).
- Geschäftslogik: Definiert, welche Verhaltensweisen als Schwachstellen gelten (inkl. "Ways to make the proxy exceed a key's upstream rate limit" = Kerninvariante) (belegt durch Dateiinhalt).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Anforderungsliste der Sicherheitsinvarianten für das Rewrite. Rust-Relevanz: Keine (Dokumentation).

## SUPPORT.md

- Zweck: Support-Routing: Fragen → GitHub Discussions, Bugs → Issues, private Security-Meldungen → Advisories; kein SLA (belegt durch Dateiinhalt).
- Verantwortlichkeit: Community-Support-Zuordnung.
- Eingaben/Ausgaben: Keine.
- Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle: Keine.
- Sicherheitsrelevanz: Verweist auf private Meldepfade.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (reines Support-Dokument). Rust-Relevanz: Keine.

## LICENSE

- Zweck: MIT-Lizenz, Copyright (c) 2026 nim-proxy contributors (belegt durch Dateiinhalt).
- Verantwortlichkeit: Rechtliche Nutzungsbedingungen (Verwendung, Kopie, Modifikation, Verkauf, Unterlizenzierung; "AS IS"-Haftungsausschluss).
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine direkte; definiert Haftungsausschluss.
- Abhängigkeiten: Keine.
- Rust-Relevanz: MIT ist lizenzkompatibel mit einer Neuentwicklung; das Rust-Projekt benötigt eine eigene Lizenzdatei. Rust-Relevanz: Keine.

## CODE_OF_CONDUCT.md

- Zweck: Contributor Covenant 2.1 (Pledge, Standards, Enforcement-Verantwortung, Scope, Enforcement-Guidelines, Attribution) (belegt durch Dateiinhalt, 134 Zeilen).
- Verantwortlichkeit: Community-Verhaltensregeln; Kontakt: Maintainer @miztertea, private Security-Advisories für vertrauliche Meldungen.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Verweist auf den privaten Meldepfad für Sicherheitsvorfälle.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (Community-Dokument). Rust-Relevanz: Keine.

## AGENTS.md

- Zweck: Agenten-Leitfaden: Modul-Überblick, Working-on-the-code-Befehle, Knowledge-Base-Schema (Open Knowledge Format: type-Pflicht-Frontmatter, ADR-Form für Decisions, Wartungs-Workflow Ingest/Log/Lint) (belegt durch Dateiinhalt).
- Verantwortlichkeit: Steuert, wie Agenten (und Contributors) das Repo pflegen und die Wissensbasis aktuell halten.
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Keine.
- Datenfluss: Kein Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Dokumentiert `cargo test`, `cargo fmt`, `cargo clippy --all-targets -- -D warnings`, Load-Harness-Pflicht (belegt durch Dateiinhalt).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Verweist auf Auth-/Sanitizing-Invarianten vor Änderungen (belegt durch Dateiinhalt).
- Geschäftslogik: Code ist Quelle für *what*, Wiki für *why*; Widerspruch → Seite fixen (belegt durch Dateiinhalt).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Dokumentiert die Wartungs-Konventionen, die auch im Rewrite-Projekt gelten können. Rust-Relevanz: Keine (Dokumentation).

## rust-toolchain.toml

- Zweck: Contributor-Komfort: rustup installiert automatisch Kanal stable + Komponenten rustfmt/clippy (belegt durch Dateiinhalt, 9 Zeilen).
- Verantwortlichkeit: Sorgt dafür, dass `cargo fmt`/`cargo clippy` out-of-the-box verfügbar sind; CI nutzt denselben Kanal.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine; Hinweis: CI-`msrv`-Job entfernt die Datei gezielt (`rm rust-toolchain.toml`), weil sie Kanal stable pinnt und den MSRV-Test verfälschen würde (belegt durch `.github/workflows/ci.yml`).
- Abhängigkeiten: Rustup.
- Rust-Relevanz: Für das Rewrite: `rust-version` in Cargo.toml bleibt die MSRV-Quelle; rust-toolchain.toml nur als Contributor-Komfort.

---

## deny.toml

- Zweck: cargo-deny-Konfiguration: yanked="deny", Lizenz-Allowlist (MIT, Apache-2.0, Apache-2.0 WITH LLVM-exception, BSD-2/3, ISC, Zlib, Unicode-3.0, CDLA-Permissive-2.0), wildcards="deny", unknown-registry/git="deny", allow-registry crates.io (belegt durch Dateiinhalt).
- Verantwortlichkeit: Lizenz- und Quellen-Policy für den Dependency-Graphen; CI-Job `deny` führt `cargo deny check` aus; wöchentlicher Advisories-Audit (belegt durch `.github/workflows/ci.yml`, `audit.yml`).
- Eingaben: Cargo.lock.
- Ausgaben: CI-Gate-Ergebnis.
- Datenfluss: Kein Laufzeit-Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Blockiert PRs mit neuen yanked/unknown Crates.
- Fehlerfälle: RUSTSEC-Advisory → Red-Gate (belegt durch CHANGELOG [0.6.4]).
- Sicherheitsrelevanz: Hohe — Quellen-/Lizenz-Policy und Advisories-Frühwarnung.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: cargo-deny.
- Rust-Relevanz: Dieselbe Policy ist für das Rewrite direkt übernehmbar (Allowlist, yanked=deny, wildcards=deny).

## .env.example

- Zweck: Dokumentiert die 5 Container-Ebene-Env-Vars: `HOST=0.0.0.0`, `PORT=8000`, `DATA_DIR=data` (Image: `/data`), `TRUST_PROXY=false`, `RUST_LOG=nim_proxy=info` (belegt durch Dateiinhalt).
- Verantwortlichkeit: Env-Kontrakt; `PUBLISH_HOST` ist Compose-only (Default 127.0.0.1); Legacy-Env-Vars werden ignoriert mit einzeiliger Boot-Warnung (belegt durch Dateiinhalt, README).
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen: Keine.
- Fehlerfälle: Unbeschreibbares `DATA_DIR` = harter Boot-Fehler (belegt durch README).
- Sicherheitsrelevanz: Enthält nur Platzhalter, keine echten Secrets; gitleaks-Allowlist (belegt durch `.gitleaks.toml`).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Docker Compose.
- Rust-Relevanz: Env-Kontrakt des Rewrite bleibt gleich (HOST/PORT/DATA_DIR/TRUST_PROXY/RUST_LOG); App-Level-Config liegt im Store, nicht in Env.

## Dockerfile

- Zweck: 2-Stage-Build: Stage `build` auf `rust:1-alpine@sha256:3c38f3f8…40900` (musl, `RUSTFLAGS="-C target-feature=+crt-static"`, TARGETARCH-Mapping amd64→x86_64-unknown-linux-musl / arm64→aarch64-unknown-linux-musl, Dependency-Cache via leerem main.rs + `touch src/main.rs`), Stage `final` = `scratch`; OCI-Labels (title, description, source, url, licenses MIT, version aus ARG VERSION=dev); `COPY --chown=10001:10001 /app/data /data`; `ENV DATA_DIR=/data`; `USER 10001:10001`; `EXPOSE 8000`; `HEALTHCHECK --interval=30s --timeout=3s --start-period=2s CMD ["/nim-proxy", "--health"]`; `ENTRYPOINT ["/nim-proxy"]` (belegt durch Dateiinhalt, 54 Zeilen).
- Verantwortlichkeit: Erzeugt das ~5-MB-statische-musl-Image ohne Shell/libc/CA-Bundle (TLS-Roots via rustls+webpki-roots kompiliert) (belegt durch Dateiinhalt, README).
- Eingaben: Build-Arg `VERSION` (Release-Workflow übergibt die git-Tag-Version) (belegt durch `.github/workflows/release.yml`).
- Ausgaben: Container-Image.
- Datenfluss: Build → Image.
- Persistenz: Named Volume `/data` (leeres, dem Runtime-User gehörendes Verzeichnis für Erst-Mount-Ownership) (belegt durch Dateiinhalt).
- Zustände: Keine.
- APIs: Healthcheck-Schnittstelle `nim-proxy --health`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Unbeschreibbares /data = Boot-Fehler.
- Sicherheitsrelevanz: Hoch — scratch-Basis, non-root (UID 10001), read_only/cap_drop in Compose; kein Debug-Tooling im Image (belegt durch Dateiinhalt, docker-compose.yml, README).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `rust:1-alpine` (Digest-pinned), Buildx.
- Rust-Relevanz: Das Rewrite-Image übernimmt das Muster: statisches musl-Binary, scratch, non-root, selbst-Probe-Healthcheck, Digest-Pinning.

## .dockerignore

- Zweck: Schließt `target`, `.git`, `.env`, `README.md` und `design` vom Build-Kontext aus (belegt durch Dateiinhalt).
- Verantwortlichkeit: Build-Kontext-Hygiene.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Hält `.env`/Secrets aus dem Build-Kontext.
- Abhängigkeiten: Docker.
- Rust-Relevanz: Konzept übertragbar (target/, .env ausschließen). Rust-Relevanz: Keine.

## .editorconfig

- Zweck: Editor-Defaults: utf-8, LF, Final-Newline, 2 Spaces (4 für `*.rs`), Markdown behält Trailing-Whitespace (belegt durch Dateiinhalt).
- Verantwortlichkeit: Cross-Editor-Konsistenz.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine.
- Abhängigkeiten: Editor-Unterstützung.
- Rust-Relevanz: Rust-Formatierung regelt rustfmt; Datei bleibt Konvention. Rust-Relevanz: Keine.

## .gitattributes

- Zweck: `* text=auto eol=lf`; PNG/JPG/JPEG/ICO als binary; `fuzz/seeds/**` als binary (belegt durch Dateiinhalt).
- Verantwortlichkeit: LF-Normalisierung + Language-Stats-Korrektur (Repo soll als Rust, nicht HTML zählen) (belegt durch CHANGELOG [0.6.3]).
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine.
- Abhängigkeiten: Git.
- Rust-Relevanz: Konvention übertragbar. Rust-Relevanz: Keine.

## .gitignore

- Zweck: Ignoriert `/target`, `.env`, `/data`, `__pycache__/`, `.claude/` (belegt durch Dateiinhalt).
- Verantwortlichkeit: Hält Build-Artefakte, Secrets und Runtime-Daten aus Git.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Schließt `.env` (potenzielle Secrets) und `/data` (config.json + history.jsonl, credentialhaltig) aus (belegt durch Dateiinhalt, README "Volume backups contain credentials").
- Abhängigkeiten: Git.
- Rust-Relevanz: Als Vorlage für das Rust-Repo übernehmbar (target/, data/). Rust-Relevanz: Keine.

## .gitleaks.toml

- Zweck: gitleaks-Scanner-Konfiguration: `extend` nutzt Default-Regeln; Allowlist für Pfade (`.env.example`, `examples/.*`, `README.md`) und Regexes (`nvapi-xxx`, `nvapi-yyy`, `nvapi-…`, `your-proxy-secret`, `a-long-random-string`, `[a-z]+:[0-9a-f]{4}\.\.\.`) (belegt durch Dateiinhalt).
- Verantwortlichkeit: Verhindert Fehlalarme für dokumentierte Beispiele; CI-Job `gitleaks` scannt mit voller History (fetch-depth: 0) (belegt durch `.github/workflows/ci.yml`).
- Eingaben: Git-Repo.
- Ausgaben: Secret-Scan-Ergebnis.
- Datenfluss: Kein Laufzeit-Datenfluss.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Leak → Gate-Fail.
- Sicherheitsrelevanz: Hoch — erkennt gepushte Secrets.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: gitleaks.
- Rust-Relevanz: Konzept für das Rewrite-Repo übernehmen (Allowlist für Beispiel-Secrets). Rust-Relevanz: Keine.

## docker-compose.yml

- Zweck: Produktions-Compose: Image `ghcr.io/miztertea/nim-proxy:latest`, `restart: unless-stopped`, Port `${PUBLISH_HOST:-127.0.0.1}:8000:8000`, `env_file: .env`, `read_only: true`, `cap_drop: [ALL]`, `no-new-privileges: true`, Named Volume `history:/data` (belegt durch Dateiinhalt).
- Verantwortlichkeit: Sichere Standard-Bereitstellung (loopback-Publish, rootless-kompatibel).
- Eingaben: `.env` (PUBLISH_HOST, HOST/PORT etc.).
- Ausgaben: Laufender Container.
- Datenfluss: Volume `/data`.
- Persistenz: `history`-Volume.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Hoch — Default-Loopback, read_only, cap_drop ALL, no-new-privileges (belegt durch Dateiinhalt, README).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Docker Compose.
- Rust-Relevanz: Deployment-Vorlage für das Rewrite übernehmbar. Rust-Relevanz: Keine (Config).

## docker-compose.dev.yml

- Zweck: Dev-Override: baut lokales Image `nim-proxy:dev` aus `.` (belegt durch Dateiinhalt).
- Verantwortlichkeit: Source-Builds als expliziter Dev-Pfad (seit 0.6.0; `docker compose up` nutzt das Published-Image) (belegt durch CHANGELOG [0.6.0]).
- Eingaben: Quellbaum.
- Ausgaben: Dev-Image.
- Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine besonderen.
- Abhängigkeiten: Docker Compose.
- Rust-Relevanz: Entwicklungs-Workflow-Vorlage. Rust-Relevanz: Keine (Config).

---

## src/main.rs

- Zweck: Binär-Einstieg: CLI-Parsing, Env-Bootstrap, Tokio-Runtime, `Server::run()`, Panic-Hook für Startup-Panics (belegt durch Dateiinhalt).
- Verantwortlichkeit: Prozess-Lifecycle (Boot → Serve → Shutdown); Wiret `tracing_subscriber` mit `NIM_PROXY_LOG_LEVEL`/`RUST_LOG`, `app` (Docker-Labels) und `port`.
- Eingaben: Env-Vars (`PORT`, `HOST` via lib), CLI-Flags `--health` (Exit 0 bei erreichbarem Server), `--version`/`-V` (Version aus Cargo), Default `--port 8000` (belegt durch Dateiinhalt, README).
- Ausgaben: Exit-Codes; Healthcheck-Kanal.
- Datenfluss: CLI/Env → Config-Layer → Server.
- Persistenz: Keine.
- Zustände: Einfache CLI-Zustände (health-check/version/serve).
- APIs: Keine HTTP-APIs; `--health`-Probe als Prozess-API.
- Ereignisse: SIGINT/SIGTERM → Graceful Shutdown (via Server).
- Nebenwirkungen: Startet Tokio-Runtime und Netzwerk-Listener.
- Fehlerfälle: Fehlende/fehlerhafte Umgebung → Panik-Hook mit Kontext; Healthcheck-Timeout → Exit 1.
- Sicherheitsrelevanz: Keine eigene; bindet nur den vom Config-Layer geprüften Zustand.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: lib (Server), tokio, tracing, dotenvy (Env-Loading).
- Rust-Relevanz: Rust-2024-Pendant: `#[tokio::main]`-Einstieg, CLI via clap oder std-arg-Parsing (minimal: `--health`/`--version`), Panic-Hook, Healthcheck-Probe als eigener Netzwerk-Check.

## src/lib.rs

- Zweck: Library-Wurzel: deklariert Module, re-exportiert Config/GovernorSettings/AppState, `#[cfg(fuzzing)]`-Re-Exporte (u. a. `dispatch::schedule`, `governor::do_poll_cycle`, `proxy::sanitize_label`), `app()`-Setup mit Routes und Fallback, AppState-Aufbau (belegt durch Dateiinhalt).
- Verantwortlichkeit: Gesamtkomposition (Config → Pool → Governor → Proxy → Dashboard → History); Single-Point-of-Testbarkeit (fuzzing-Re-Exporte nur unter cfg).
- Eingaben: Konfigurationsdaten (StoredConfig), Server-Start-Anweisung.
- Ausgaben: Vollständiger axum-Server (Router + State).
- Datenfluss: Config → AppState → Handler.
- Persistenz: Delegate an Config/History.
- Zustände: AppState (Thread-Safe-Arc-Strukturen).
- APIs: Definierte Routen (nicht vollständig im E2E-Test aufgeführt, aber belegt durch src/setup.html, README).
- Ereignisse: Keine.
- Nebenwirkungen: Setzt Task-Spawner/Governor-Loop auf.
- Fehlerfälle: Config-Fehler → Boot-Abbruch.
- Sicherheitsrelevanz: Zentraler Zusammenschluss aller Sicherheits-Komponenten.
- Geschäftslogik: Gesamtsystem-Orchestrierung.
- Algorithmen: Keine.
- verwendete Datenmodelle: Config, AppState, Pool-State.
- Abhängigkeiten: Alle internen Module, axum.
- Rust-Relevanz: `cfg(fuzzing)`-Re-Exporte sind das Muster, um Fuzz-Targets gegen interne Funktionen ohne Feature-Flags laufen zu lassen; `app()` als testbare Factory.

## src/auth.rs

- Zweck: Auth-Schicht: Login/Logout/Session-Handling, API-Key-Authentifizierung (Header/Bearer/Basic), Cookie-Management, PBKDF2-Hashing, Session-TTL, Scraper-Auth, Throttling (belegt durch Dateiinhalt, CHANGELOG).
- Verantwortlichkeit: Zugriffskontrolle auf das Dashboard und die API; Einmal-Key-Ausgabe; Session-Mapping.
- Eingaben: HTTP-Request (Cookies, Authorization-Header), Benutzername/Passwort, Client-Keys, Scraper-Credentials (belegt durch Dateiinhalt).
- Ausgaben: Session-Responses (Set-Cookie), Login/Logout-Status, Auth-Errors (401/403), API-Key-Verifikation (belegt durch Dateiinhalt).
- Datenfluss: Request → Cookie/Header-Validierung → Session-Lookup → Zugriff.
- Persistenz: Session-Store in-memory (TTL 12h), Keys aus StoredConfig; keine Session-Persistenz.
- Zustände: Session (Expiry, Username, PWHash-Fragment), API-Key-Zustand (enabled).
- APIs: `POST /login`, `POST /logout`; Auth-Extractors für Dashboard/API.
- Ereignisse: Login/Logout; Session-Expiry (12h).
- Nebenwirkungen: Setzt Cookies, erzeugt Sessions, aktualisiert Login-Throttle.
- Fehlerfälle: Falsche Credentials → 401/403, Login-Throttle, ungültige Cookies.
- Sicherheitsrelevanz: Hoch — PBKDF2 (600k Iterationen), Session-Hashing, CSRF-Schutz, Scraper-Auth, keine Credentials im Klartext (belegt durch Dateiinhalt, README).
- Geschäftslogik: Rollen (Admin/User), Key-Enablement, Throttling von Logins.
- Algorithmen: PBKDF2-SHA256 (600k Iterationen), Konstantzeit-Vergleich (subtle).
- verwendete Datenmodelle: Session, ClientKey, StoredConfig.
- Abhängigkeiten: config, hmac, sha2, subtle, getrandom.
- Rust-Relevanz: auth-Modul bleibt zentral; Rust-2024: `tokio::time` für Expiry, hmac/sha2/subtle unverändert, Session-Store evtl. als struct mit Mutex.

## src/config.rs

- Zweck: Konfigurations-Layer: Laden/Speichern/Validieren von `config.json` (StoredConfig), Mode-Enum (Open/Keyed), NimKey/ClientKey, Limits/Pricing, PBKDF2-Hash-Verarbeitung, Datenpfad-Layout (belegt durch Dateiinhalt).
- Verantwortlichkeit: Single Source of Truth für Konfiguration; Fehler-Handling bei korrupten Dateien (Boot-Abbruch); Schema-Versionierung.
- Eingaben: Dateisystem (config.json), Env-Vars (DATA_DIR), Schreib-/Lesekontext.
- Ausgaben: Geparste/validierte Config, Fehlerberichte.
- Datenfluss: Datei → Parse → Validieren → In-Memory-Config → Speichern.
- Persistenz: JSON-Datei im DATA_DIR (0600, atomar via tmp+rename) (belegt durch Dateiinhalt).
- Zustände: Config-Zustand (Lädt/Verschoben/Speichert); `config_revision`-Inkrement.
- APIs: Config-Load/Save-API; `#[serde(default)]`-Kompatibilität.
- Ereignisse: Config-Änderung (durch settings.rs).
- Nebenwirkungen: Schreibt config.json, setzt Permissions, inkrementiert Revision.
- Fehlerfälle: Korrupte JSON → harter Boot-Fehler; Schreibfehler → Fehler-Propagation.
- Sicherheitsrelevanz: Hoch — Credentials (NimKeys, ClientKeys, PBKDF2) liegen in config.json; Datei-0600; `Keyed` als Default (fail-closed) (belegt durch Dateiinhalt, CHANGELOG).
- Geschäftslogik: Modus-Auswahl (Open vs. Keyed), Key-Enforcements.
- Algorithmen: PBKDF2-Verifizierung/Erzeugung.
- verwendete Datenmodelle: StoredConfig (v1), NimKey, ClientKey, Limits, Pricing, nim_mode.
- Abhängigkeiten: serde, serde_json, sha2, getrandom, std::fs.
- Rust-Relevanz: Unverändertes Kernkonzept; Rust-2024: `serde`-Strukturen mit `#[serde(rename_all = "camelCase")]`, atomare Writes via `tempfile`/eigene tmp-Rotation.

## src/dispatch.rs

- Zweck: Anfrage-Dispatch auf Pool-Slots: Slot-Zuordnung mit Generation-Check, Waiter-Mit-Waiter, Slot-Acquire-Releases (belegt durch Dateiinhalt).
- Verantwortlichkeit: Koordinierung zwischen Client-Anfragen und Pool-Ressourcen; Timeout- und GAP-Steuerung.
- Eingaben: Slot-Anfrage (Anzahl Slots, Pool-Generation), Waiter-Status.
- Ausgaben: Slot-Permits, Timeout-Fehler, Waiter-Queue.
- Datenfluss: Anfrage → Slot-Zuteilung → Release; Generation-Wechsel invalidates alte Slots.
- Persistenz: Keine.
- Zustände: Slot (Generation, in-use), Waiter (Deadline, prefer).
- APIs: Slot-Acquire/Release; Waiter-Warteschlange.
- Ereignisse: Slot-Releases, Waiter-Timeout.
- Nebenwirkungen: NimProxy-Queue-Depth-Metriken, Generation-Wechsel (via Pool-Rebuild).
- Fehlerfälle: Timeout, Generation-Mismatch, Pool-Leer (Shedding).
- Sicherheitsrelevanz: Keine direkte; beugt Überlast-Situationen vor (DoS-Schutz via max_inflight/Timeout).
- Geschäftslogik: GRANT_GAP 25ms (kein ständiges Polling), Waiters mit prefer-Präferenz, Shedding bei Überlast (belegt durch Dateiinhalt).
- Algorithmen: Generation-Check (ABA-Schutz), Deadline-basiertes Timeout, prefer-Queue.
- verwendete Datenmodelle: Waiter, Slot, Pool-Generation.
- Abhängigkeiten: governor (Pool-Konfiguration), pool (Reservations), tokio.
- Rust-Relevanz: Rust-2024: `tokio::sync`-Primitive (Mutex/Notify) statt eigenem Polling; Generation-Check als `u64`-Token bleibt relevant.

## src/governor.rs

- Zweck: Rate-Governor: periodischer Poll von Pool-Zuständen, Exhaustion-Backoff, Adaptive-Burst-Growth, Pool-Dissolve bei Inaktivität (belegt durch Dateiinhalt).
- Verantwortlichkeit: Zentrales Rate-Limit-Brain; orchestriert Lane-Auslastung und Pool-Zustand.
- Eingaben: Pool-State, Lane-Last, Reservations.
- Ausgaben: Lane-Anpassungen (Grow/Shrink), Pool-Umbau-Trigger, Exhaust-Markierungen.
- Datenfluss: Poll-Zyklus (250ms) → Zustandsauswertung → Anpassungen → Pool.
- Persistenz: Keine (nur In-Memory).
- Zustände: Exhaustion, Backoff, Growth-Counter, Dissolve.
- APIs: `do_poll_cycle` (via lib.rs fuzzing-Re-Export).
- Ereignisse: Exhaustion → Backoff; Inaktivität → Dissolve (30min); Rate-Limit-Erhöhung → Grow.
- Nebenwirkungen: Mutiert Pool-Lanes, setzt Metriken (z. B. rate-limit-Events), kann neue Lanes anlegen.
- Fehlerfälle: Keine harten Fehler; alle Zustandsübergänge intern.
- Sicherheitsrezeption: Kern der DoS-/Rate-Protection.
- Geschäftslogik: POLL 250ms, EXHAUST_BACKOFF 2s, GROW_INTERVAL 60s, DISSOLVE_AFTER 30min; ModelPermit-RAII (belegt durch Dateiinhalt).
- Algorithmen: Exponential Backoff, Adaptive Growth, Pool-Zustandsmaschine (Enabled→Exhausted→Backoff→Grow).
- verwendete Datenmodelle: Lane, Pool, ModelPermit.
- Abhängigkeiten: pool, dispatch, tokio.
- Rust-Relevanz: Rust-2024: `tokio::time`-Loop statt Poll-Thread; Zustandsmaschine als enum; RAII-Permit via `Drop`.

## src/history.rs

- Zweck: Historie/Dashboard-Daten: Sampling (SAMPLE_SECS 300), 1-MB/100k-Limits, v1-Legacy+v2-Boot-Marker, atomare Writes, `history_revision`/`config_revision`, `/api/dashboard`-Abfragen (belegt durch Dateiinhalt).
- Verantwortlichkeit: Persistente Zeitreihen-Aggregation (Abfrage-Profil, Rate-Limit-Events) für Dashboard-Zeitreihen.
- Eingaben: Abfrage-Events, Config-Änderungen, Retention-Werte, `from`/`to`-Zeitfenster, `points`-Sampling (belegt durch Dateiinhalt, e2e).
- Ausgaben: Aggregierte Zeitreihen-Samples, Revisionen, gespeicherte History-Dateien.
- Datenfluss: Events → Sammeln → Sampling → Datei (atomar) → Dashboard-Abfragen.
- Persistenz: history.jsonl (JSON Lines, mit v2-Marker), 1-MB-Limit (belegt durch Dateiinhalt).
- Zustände: History-Revision, Config-Revision, Boot-Marker.
- APIs: `/api/dashboard?from&to&points=288`, `/api/dashboard/now`, `/api/dashboard`-Events (belegt durch README).
- Ereignisse: Config-Änderung, Retention-Prune, Boot-Marker-Set.
- Nebenwirkungen: Schreibvorgänge auf Disk; Revision-Inkremente.
- Fehlerfälle: Disk voll, korrupte JSON-Lines (Robustheit via Zeilen-Skip? belegt nicht vollständig — als nicht nachweisbar markieren).
- Sicherheitsrelevanz: Keine Credentials in History (nur Aggregat-Daten).
- Geschäftslogik: 300s-Sampling, 1-MB-Cap, 100k-Events, Retention-Prune, Sampling-Rate (belegt durch Dateiinhalt).
- Algorithmen: Zeitfenster-Aggregation, Revisions-Zähler.
- verwendete Datenmodelle: Sample, History-Eintrag (v1/v2), TimeSeries-Daten.
- Abhängigkeiten: config (Limits), serde, serde_json, std::fs.
- Rust-Relevanz: Rust-2024: JSONL-Streaming mit `serde_json::Deserializer` (line-delimited), atomare Writes via `tempfile`+rename, Retention als `tokio::task`.

## src/pool.rs

- Zweck: Modell-Pool: PoolHandle, Lane (Rate-Limit), Pool-Zustand, Reservations (Ready/Wait), Affinity (sticky/spill), Lane-Preferences (belegt durch Dateiinhalt).
- Verantwortlichkeit: Logische Modell-Zuordnung + physikalische Lane-Instanzen; Rate-Limit-Verfolgung pro Lane.
- Eingaben: Modell-Anfragen, Lane-Konfiguration (rpm, capacity), Reservations (Anzahl Slots).
- Ausgaben: Reservations (Ready/Wait), Lane-Auswahl, Affinity-Ergebnis (sticky vs. spill).
- Datenfluss: Anfrage → Lane-Auswahl (Preferenzen/Enabled) → Reservation → Ready/Wait → Slot-Grant.
- Persistenz: Keine (Lanes in-memory).
- Zustände: Lane (rate_remaining, capacity, enabled), Pool (WINDOW 61s Jitter-Marge), Reservation (Ready/Wait).
- APIs: Pool-Lookup, Lane-Reservation, Slot-Acquire.
- Ereignisse: Lane-Refresh, Pool-Rebuild (bei Config-Änderung).
- Nebenwirkungen: Rate-Limit-Zähler-Updates, Affinity-Metriken (`nimproxy_affinity_total{result}`).
- Fehlerfälle: Keine Lane gefunden → keine Reservierung; Reservations-Timeout.
- Sicherheitsrelevanz: Keine direkte.
- Geschäftslogik: Enabled-Lanes zuerst, Affinity sticky→spill, WINDOW 61s (Jitter-Marge), Lane-Preferenzen, Capacity-basiertes RPM (belegt durch Dateiinhalt, e2e).
- Algorithmen: Sliding-Window-Rate-Limit, Affinity-Matching, Lane-Sortierung.
- verwendete Datenmodelle: LaneSpec, Lane, Pool, Reservation.
- Abhängigkeiten: config (LaneSpec), dispatch (Slot), governor.
- Rust-Relevanz: Rust-2024: Sliding-Window via `std::time::Instant`+Ringpuffer, Affinity als HashMap-Ranking, Pool-Rebuild als `ArcSwap`/RwLock.

## src/proxy.rs

- Zweck: Kern-Proxy: Anfrage-Weiterleitung an NVIDIA NIM, Streaming (SSE), Tool-Calling, Usage-Injection, sanitize_label (max 64, `[A-Za-z0-9._/:-]`), 1-MiB-Guard, Timeouts (deadline), Heartbeat (belegt durch Dateiinhalt).
- Verantwortlichkeit: Ausführung des eigentlichen Proxy-Vorgangs inkl. Fehler-/Timeout-Handling und Metrik-Injection.
- Eingaben: Client-Request (Stream, Messages, Tools), Upstream-Response, Deadline-Header.
- Ausgaben: Upstream-Response an Client (ggf. mit injizierten usage-Feldern), Fehler-Responses (503/504/429).
- Datenfluss: Client → (sanitize_label) → Upstream → SSE-Passthrough/Streaming → Client; Usage-Injection (bei fehlendem usage) (belegt durch Dateiinhalt).
- Persistenz: Keine.
- Zustände: Streaming (Idle/Active), Timeout (stream_idle 120s), Deadline-Ablauf.
- APIs: `/v1/chat/completions`, `/v1/models` (Aufrufe, nicht vollständig definiert — belegt durch README).
- Ereignisse: SSE-Heartbeat, Stream-Ende, Timeout, Deadline.
- Nebenwirkungen: Metriken (Tokens, Latency, Rate-Limit-Events), History-Events.
- Fehlerfälle: Upstream-Fehler (5xx), 429 (Rate-Limit), Timeout, Invalid-Label, Payload-Too-Large (1 MiB), Strict-Passthrough-Modus.
- Sicherheitsrelevanz: Keine Credentials im Response; sanitize_label verhindert Metrik-Injection; Timeouts verhindern DoS.
- Geschäftslogik: strict_passthrough (kein Injection), Deadline-Header, Tool-Calling-Propagation, Stream-Idle-Timeout, Heartbeat-Intervall (10s), Retry-Policy (nicht nachweisbar im Detail).
- Algorithmen: SSE-Parsing (sse_scan), Token-Counting, Deadline-Berechnung, Label-Sanitizing.
- verwendete Datenmodelle: Stream-Segment, Request-Body, Usage, ToolCall.
- Abhängigkeiten: config, history, reqwest, serde_json, tokio.
- Rust-Relevanz: Rust-2024: `futures_util::StreamExt` für SSE, `reqwest::stream`-Feature, Heartbeat via `tokio::time`, Deadline als `Instant`-Vergleich.

## src/settings.rs

- Zweck: Einstellungs-Persistenz: Commit-Pipeline (validate → save → cfg-write → Pool-Rebuild → Retention → Guard → config_revision++), json_error-Shape `{error:{message,type:"proxy_error",code}}` (belegt durch Dateiinhalt).
- Verantwortlichkeit: Verwaltung der schreibenden Config-Operationen mit Rollback-fähiger Pipeline.
- Eingaben: Neuer Config-Stand (aus UI/API), StoredConfig-Felder.
- Ausgaben: Bestätigung/Fehler; persistierte Config.
- Datenfluss: UI → commit() → Validation → Persistenz → Pool/Retention-Anpassung.
- Persistenz: config.json (atomar, 0600).
- Zustände: Config-Valid/Invalid, Revisionen.
- APIs: Settings-Endpunkte (PUT/GET) (belegt durch dashboard.html, README).
- Ereignisse: Config-Change (History, Revision, Pool-Rebuild).
- Nebenwirkungen: Pool-Neubau (Rate-Carryover!), Retention-Prune, config_revision-Inkrement.
- Fehlerfälle: Validierungsfehler (z. B. rpm-Bereiche), Persistenzfehler (Rollback).
- Sicherheitsrelevanz: Falsche Eingaben können Konfiguration destabilisieren; Fehler-Shape ist einheitlich.
- Geschäftslogik: Commit-Pipeline, Rate-Carryover beim Pool-Rebuild (belegt durch Dateiinhalt, e2e).
- Algorithmen: Pipeline-Validierung, Rollback.
- verwendete Datenmodelle: StoredConfig, Pricing, Limits.
- Abhängigkeiten: config, pool, history, serde_json.
- Rust-Relevanz: Rust-2024: Validierungs-Pipeline als `Result`-Kette, Rollback via Transaction-Pattern (z. B. `SavedConfig`+`CommitError`).

## src/setup.html

- Zweck: Erstkonfigurations-Seite (Setup-Wizard): Initial-Config-Erstellung (Upstream, NIM-Keys, Pricing, Limits, User), served beim ersten Boot (belegt durch Dateiinhalt, README).
- Verantwortlichkeit: Erste-Schritte-Bootstrap ohne CLI.
- Eingaben: Formular-Daten (Upstream-URL, NIM-Key, Limits, Pricing, User-Name/Passwort).
- Ausgaben: POST an Setup-Endpunkt → Config-Store + UI-Wizard.
- Datenfluss: UI → Setup-API → config.json.
- Persistenz: config.json (beim Setup-Abschluss).
- Zustände: Setup-Pending → Complete (danach Setup nicht mehr erreichbar) (belegt durch README "setup claim window").
- APIs: Setup-Endpunkte (nicht vollständig definiert; belegt durch README/e2e `setup_can_mint_a_first_client_key`).
- Ereignisse: Setup-Abschluss (Config-Store-Init).
- Nebenwirkungen: Initial-Config + User + Key-Ausgabe.
- Fehlerfälle: Validierungsfehler, doppelter Setup-Versuch.
- Sicherheitsrelevanz: Einmal-Setup-Fenster (Claim) ist out-of-scope per SECURITY.md.
- Geschäftslogik: Bootstrap-Fluss.
- Algorithmen: Keine.
- verwendete Datenmodelle: InitialConfig, StoredConfig.
- Abhängigkeiten: JS (innerHTML-basiert), Config-API.
- Rust-Relevanz: Rust-2024: statisches HTML-Asset als `include_str!`, Setup-API als eigener Router-Branch, Validierung serverseitig.

## src/dashboard.html

- Zweck: Dashboard-UI: 5 Tabs (Overview, Models, Clients, Reliability, Settings), 3000ms-Polling, Zeitfenster-Steuerung, Modus-Umschalter (belegt durch Dateiinhalt, 2504 Zeilen, vollständig gelesen).
- Verantwortlichkeit: Visualisierung von Metriken, Config-Edition, History-Verträge.
- Eingaben: Poll-Daten (JSON), Nutzer-Interaktion (Tab, Fenster, Mode, Config-Formulare).
- Ausgaben: UI-Render, API-Aufrufe (Config-Änderungen, History).
- Datenfluss: History-API → Poll → Rendering; Settings-Formulare → PUT.
- Persistenz: Keine (nur Browser-State).
- Zustände: mode{kind,preset,paused,from,to}, cfg-Defaults (price_in 0.5, price_out 2.0, default_window_days 30, retention_days 30, slo 99.9) (belegt durch Dateiinhalt).
- APIs: `/api/dashboard/now`, `/api/dashboard?from&to&points=288`, Settings-PUT (belegt durch Dateiinhalt, README).
- Ereignisse: Poll-Intervall (3000ms), User-Aktionen, Config-Save.
- Nebenwirkungen: Keine Server-Nebenwirkungen außer Settings-API-Aufrufen.
- Fehlerfälle: API-Fehler → Fehleranzeige im UI; Konfig-Invalid → Fehlermeldung.
- Sicherheitsrelevanz: `innerHTML`-Nutzung (XSS-relevante Verarbeitung wird in CONTRIBUTING.md als Invariante genannt; 0.3.0-Fix: XSS-Kette durch Output-Escaping/HTML-Encoding) (belegt durch CHANGELOG, CONTRIBUTING).
- Geschäftslogik: Modus-Defaults, Fenster-Berechnung, Metrik-Anzeige (Deltas/Gauges).
- Algorithmen: rangeSamples: Counter-Deltas + Gauge-Ersetzung (belegt durch Dateiinhalt).
- verwendete Datenmodelle: Dashboard-Samples, Config-JSON, History-API-Response.
- Abhängigkeiten: Keine externen (vanilla JS, keine Frameworks).
- Rust-Relevanz: Rust-2024: statisches Asset einbetten, Polling-Verträge beibehalten; Frontend-Framework-frei bleibt möglich (Zeichenbudget).

## src/dashboard.js

- Zweck: (nicht im Repo vorhanden — nur dashboard.html enthält das Script; kein separates JS-File belegt).
- Verantwortlichkeit: Nicht vorhanden; entfällt.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle/Abhängigkeiten: Entfällt.
- Sicherheitsrelevanz: Keine.
- Rust-Relevanz: Keine (Datei existiert nicht; Dashboard-Logik liegt in dashboard.html).

---

## tests/e2e.rs

- Zweck: End-to-End-Testsuite (53 e2e-Tests, belegt durch CONTRIBUTING.md): startet Server-Stack mit Mock-NIM, prüft Proxy-Verhalten, Auth, Dashboard-Verträge, Retention, Shedding, Setup, Security-Header (belegt durch Dateiinhalt, 3731 Zeilen, 97+ Testfunktionen).
- Verantwortlichkeit: Verifiziert alle Kerninvarianten (Rate-Limits, Auth, History, Dashboard) gegen einen scriptbaren Mock.
- Eingaben: Test-Setup (Config, Env), HTTP-Requests, Mock-Steuerung.
- Ausgaben: Test-Ergebnisse (pass/fail), Coverage-Messungen.
- Datenfluss: Test-Helper → Server-Start (tempdir DATA_DIR) → HTTP → Assertion.
- Persistenz: Tempdir (data), wird nach Test gelöscht.
- Zustände: Test-Isolation pro Funktion; Server-Lifecycle je Suite.
- APIs: GET/POST auf alle Routen inkl. `/api/dashboard`, `/api/dashboard/now`, `/setup`, `/metrics`, `/health`.
- Ereignisse: Config-Änderung (Retention), Shedding, Stream-Ende.
- Nebenwirkungen: Schreibt tempdir-Dateien, startet Mock-NIM-Port.
- Fehlerfälle: Erwartete Fehlercodes (503, 429, 401, 403, 404), Timeouts.
- Sicherheitsrelevanz: Testet Security-Header, CSP, Key-Gating, Label-Sanitizing, Fail-closed.
- Geschäftslogik: Deckt die Invarianten aus CONTRIBUTING.md ab (69 Unit + 53 e2e; Dashboard-Verträge; Retention; Shedding).
- Algorithmen: Keine eigenen; nutzt Test-Helper (spawn, request).
- verwendete Datenmodelle: Response-JSON, Dashboard-Samples.
- Abhängigkeiten: tokio (Test), reqwest (Test), serde_json, sha2 (Test-Helfer).
- Rust-Relevanz: Vorlage für Rust-2024-Testsuite: gleiche Vertrags-Tests (dashboard_now_contract, retention_change, overloaded_requests_shed, setup_can_mint_key), Mock-Server als eigenes Test-Modul.

## tests/support/mod.rs

- Zweck: Test-Support: Shared Helpers (spawn Server, Mock-NIM, Request-Helfer, tempdir-Setup, Assertions).
- Verantwortlichkeit: Reduziert Test-Duplikation, hält Setup konsistent.
- Eingaben: Konfiguration (Config-Overrides, Env).
- Ausgaben: Laufende Test-Instanz + Helpers.
- Datenfluss: Setup → Helper-Funktionen → Test-Körper.
- Persistenz: tempdir (data).
- Zustände: Server-Lifecycle.
- APIs: Interne Test-API (spawn, request, assert).
- Ereignisse: Keine.
- Nebenwirkungen: Startet Server/Mock auf Ports, legt tempdir an.
- Fehlerfälle: Setup-Fehler propagieren als Test-Failure.
- Sicherheitsrelevanz: Stellt sicher, dass Tests isoliert und mit Fail-closed-Config laufen.
- Geschäftslogik: Keine eigene.
- Algorithmen: Keine.
- verwendete Datenmodelle: TestConfig, TestServer.
- Abhängigkeiten: tokio, reqwest, serde_json, sha2.
- Rust-Relevanz: Wiederverwendbares Test-Harness-Muster für das Rewrite.

---

## knowledge/index.md

- Zweck: Einstiegs-Verzeichnis der Wissensbasis: Links zu allen Kategorien (Decisions, Architecture, Research, Ops, Testing) und Lese-Empfehlung (belegt durch Dateiinhalt).
- Verantwortlichkeit: Navigation; AGENTS.md verlangt das Lesen von knowledge/index.md vor nicht-trivialen Änderungen.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (Dokumentation).

## knowledge/decisions/ (17 Dateien)

- Zweck: Architektur-Entscheidungen im ADR-Format (16 ADRs + index.md): u. a. fail-closed-Auth, Config-Versionierung, Pool-Governor, Dashboard-Verträge, Sanitizing, Scraper-Auth, Release-Signing, MSRV-Policy (belegt durch Dateiinhalt, CHANGELOG-Verweise).
- Verantwortlichkeit: Begründete Entscheidungen (Context/Options/Choice/Consequences) als Wissensquelle für das Rewrite.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Markdown-Dateien.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Dokumentiert Sicherheits-Entscheidungen (Keyed-Default, 0600-Permissions, PBKDF2-Iterationen).
- Geschäftslogik: Enthält die Design-Begründungen hinter den Kerninvarianten.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Wichtigste Design-Quelle für rust-foundation.md (Begründungen statt Code). Rust-Relevanz: Keine (Dokumentation).

## knowledge/architecture/ (8 Dateien)

- Zweck: Architektur-Dokumente: Modul-Überblick, Datenfluss, Governor-Design, Pool-Lanes, History-Format, Dashboard-Verträge, Streaming (belegt durch Dateiinhalt).
- Verantwortlichkeit: Erklärt die Systemarchitektur (Was/Warum) unabhängig vom Code.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle: Keine zur Laufzeit.
- Sicherheitsrelevanz: Dokumentiert Sicherheitsgrenzen und Vertrauensmodelle.
- Geschäftslogik: Architektur-Begründungen (z. B. warum Lane-basiert statt Token-Bucket global).
- Algorithmen: Beschreibt Algorithmen (z. B. Sliding Window, Pool-Zustandsmaschine) konzeptionell.
- verwendete Datenmodelle: Beschreibt StoredConfig, Lane, Sample etc. konzeptionell.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Primärquelle für architecture.md/rust-foundation.md. Rust-Relevanz: Keine (Dokumentation).

## knowledge/research/ (4 Dateien)

- Zweck: Forschung/Themen: NIM-Free-Tier 40rpm ohne Credits, NIM-KV-Cache-Reuse, NIM-Models-Endpoint-Schema + index.md (belegt durch Dateiinhalt).
- Verantwortlichkeit: Belegt externe Faktoren (z. B. NIM-Fehlerformen, Header), die die Implementierung beeinflussen.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle: Keine.
- Sicherheitsrelevanz: Ggf. relevante externe Sicherheits-Erkenntnisse.
- Geschäftslogik: Externes Wissen.
- Algorithmen: Extern dokumentierte Algorithmen.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Hintergrundwissen für NIM-Integration. Rust-Relevanz: Keine (Dokumentation).

## knowledge/ops/ (6 Dateien)

- Zweck: Betriebs-Dokumentation: Deployment, Upgrade, Backup/Restore, Fehlerbehebung, Metriken, Logs (belegt durch Dateiinhalt).
- Verantwortlichkeit: Betriebshandbuch.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle: Keine zur Laufzeit.
- Sicherheitsrelevanz: Dokumentiert Secrets-Handling und Backup/Recovery.
- Geschäftslogik: Betriebsabläufe (z. B. Upgrade-Pfad 0.6.x → 0.7).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Betriebsanforderungen für das Rewrite (Upgrade-Pfad, Backup). Rust-Relevanz: Keine (Dokumentation).

## knowledge/testing/ (2 Dateien)

- Zweck: Test-Strategie & Load-Harness: Testkategorien, Invarianten, Load-Tests (belegt durch Dateiinhalt).
- Verantwortlichkeit: Belegt Test-Anforderungen und -Methodik.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle: Keine zur Laufzeit.
- Sicherheitsrelevanz: Testet Sicherheitsinvarianten (Fail-closed, Sanitizing).
- Geschäftslogik: Test-Strategie (Unit vs. e2e, Coverage ≥ 90 %).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Test-Anforderungen für das Rewrite (Vertrags-Tests, Load-Harness). Rust-Relevanz: Keine (Dokumentation).

## knowledge/log.md

- Zweck: Änderungslog der Wissensbasis (Wartungs-Workflow) (belegt durch Dateiinhalt).
- Verantwortlichkeit: Nachvollziehbarkeit der Wissenspflege.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (Wartungslog).

---

## .github/workflows/ci.yml

- Zweck: CI-Hauptpipeline mit 8 Jobs: `check` (fmt, clippy -D warnings, `cargo test`, Dashboard-JS-Syntaxcheck via Python-Extraktion + `node --check`), `coverage` (llvm-cov, Gate ≥ 90 % lines, Subprocess-Coverage für e2e-Kinderprozesse), `msrv` (entfernt rust-toolchain.toml, `cargo check --locked --all-targets` mit Rust 1.87), `deny` (cargo-deny check), `gitleaks` (fetch-depth 0), `lint-workflows` (actionlint v1.7.12 checksum-verifiziert + release-contract-Test + cosign-Bundle-Smoke-Test + zizmor SARIF/Action + CLI-Gate `--min-severity high`), `dependency-review` (nur PRs, fail-on-severity low, license-check false, Kommentar bei Failure), `docker` (Buildx-Image mit gha-Cache, Smoke-Test: `/health` erreichbar + Container-HEALTHCHECK healthy) (belegt durch Dateiinhalt, 271 Zeilen).
- Verantwortlichkeit: Hauptqualitäts-Gate bei PRs/Main; Release-Jobs hängen an main-CI (Kommandozeilen-Kommentar).
- Eingaben: Push (main) / Pull-Request-Ereignisse.
- Ausgaben: Status-Checks, Step-Summary mit Coverage-Block.
- Datenfluss: Code → Jobs → Status.
- Persistenz: Keine.
- Zustände: Job-State (pass/fail).
- APIs: Keine.
- Ereignisse: push (main), pull_request.
- Nebenwirkungen: Keine außer CI-Status und Pull-Request-Kommentaren (dependency-review).
- Fehlerfälle: Fail bei Lint/Coverage/Deny/Secrets/Workflow-Lint.
- Sicherheitsrelevanz: Hoch — Least-Privilege-`permissions: contents: read` (Jobs optieren einzeln), harden-runner (egress audit) in jedem Job, persist-credentials: false, SHA-gepinnte Actions mit Versionskommentar, gitleaks mit voller History, zizmor-Gate, Dependency-Review, Docker-Smoke mit `--health` (belegt durch Dateiinhalt).
- Geschäftslogik: MSRV-Job entfernt rust-toolchain.toml (Kanal stable würde MSRV-Test verfälschen); Coverage-Job reicht das Coverage-Profil an den e2e-Kinderprozess durch (Subprocess-Coverage); zizmor-CLI gated nur high severity; dependency-review nur auf PRs (API-Abhängigkeit).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: GitHub Actions, cargo, llvm-cov (taiki-e/install-action), cargo-deny-action, gitleaks-action, actionlint (curl+sha256), zizmor, docker/buildx, node (Dashboard-JS).
- Rust-Relevanz: CI-Vorlage für das Rewrite (Jobs/Gates 1:1 übertragbar; insbesondere Subprocess-Coverage-Trick und JS-Syntaxcheck des eingebetteten HTML).

## .github/workflows/release.yml

- Zweck: Release-Pipeline: Version aus Cargo.toml minten, Tag verweigern falls existent, cosign sign, SLSA attest, SBOM, GitHub-Release, Docker-Push (belegt durch Dateiinhalt).
- Verantwortlichkeit: Automatisierte, signierte Releases; `# zizmor: ignore[artipacked]` für checkout-credentials (belegt durch Dateiinhalt).
- Eingaben: `workflow_dispatch` (tag), Push auf `v*`.
- Ausgaben: Tag, Release, Container-Images (ghcr.io).
- Datenfluss: Cargo.toml-Version → Tag → Build → Sign → Publish.
- Persistenz: GitHub-Release, Container-Registry.
- Zustände: Release-State (draft → published).
- APIs: GitHub-API, Container-Registry.
- Ereignisse: workflow_dispatch, tag-push.
- Nebenwirkungen: Veröffentlichte Artefakte.
- Fehlerfälle: Tag existiert → Abbruch.
- Sicherheitsrelevanz: cosign-Signaturen, SLSA-Provenance, SBOM, `v*`-Tag-Schutz.
- Geschäftslogik: Tag-Existenzprüfung; Version-Quelle ist Cargo.toml.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: GitHub Actions, cosign, slsa-github-generator, docker/buildx, docker-push.
- Rust-Relevanz: Release-/Signing-Pipeline als Vorlage.

## .github/workflows/audit.yml

- Zweck: Wöchentlicher Security-Audit: cargo-deny-advisories, gitleaks, OSV-Scanner (belegt durch Dateiinhalt).
- Verantwortlichkeit: Periodische Abhängigkeits- und Secret-Überwachung.
- Eingaben: Cargo.lock, Repo.
- Ausgaben: Audit-Report (Issue-Erstellung bei Findings).
- Datenfluss: Lockfile → Scanner → Report.
- Persistenz: GitHub-Issues.
- Zustände: Findings-Offen/Geschlossen.
- APIs: Keine.
- Ereignisse: Cron (wöchentlich).
- Nebenwirkungen: Erstellt Issues bei Schwachstellen.
- Fehlerfälle: Findings → Issue.
- Sicherheitsrelevanz: Primärer Security-Audit-Takt.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: GitHub Actions, cargo-deny, gitleaks, OSV-Scanner.
- Rust-Relevanz: Audit-Pipeline als Vorlage.

## .github/workflows/fuzz.yml

- Zweck: Fuzz-Pipeline: Targets sse_scan, sanitize_label, config_roundtrip; 60s je Target; `--target x86_64-unknown-linux-gnu` (load-bearing wegen musl/ASan-Problem); upload-artifact bei failure (belegt durch Dateiinhalt).
- Verantwortlichkeit: Regelmäßige Fuzzing-Absicherung der Parse-/Sanitize-/Config-Pfade.
- Eingaben: Targets, Seeds, Korpra.
- Ausgaben: Fuzz-Report/Artefakte.
- Datenfluss: Seeds → Corpus → Target → Findings.
- Persistenz: Artefakte (bei Failure).
- Zustände: Finding/Kein Finding.
- APIs: Keine.
- Ereignisse: Cron/PR.
- Nebenwirkungen: Corpus-Erweiterung.
- Fehlerfälle: Panic/Timeout → Finding.
- Sicherheitsrelevanz: Fängt Parsing-/Sanitizing-Bugs.
- Geschäftslogik: 60s je Target; Architecture-Pin.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: GitHub Actions, cargo-fuzz, nightly.
- Rust-Relevanz: Fuzz-Targets (sse_scan, sanitize_label, config_roundtrip) bleiben als Integrationstests relevant.

## .github/workflows/codeql.yml

- Zweck: CodeQL-SAST (Rust): `build-mode: none` (Extractor parst Quelltext ohne cargo-Build, dadurch günstig), Config-File `codeql/codeql-config.yml` (paths-ignore tests/** und fuzz/** wegen Fixture-Credentials), läuft bei push auf main, pull_request und wöchentlich (Dienstag 06:31 UTC) (belegt durch Dateiinhalt, 57 Zeilen).
- Verantwortlichkeit: Statische Sicherheitsanalyse (Security → Code scanning); Findings inkl. wöchentlichem Rescan mit neuen Queries.
- Eingaben: Quellcode, CodeQL-Queries.
- Ausgaben: SARIF-Upload.
- Datenfluss: Code → Extractor → Queries → SARIF.
- Persistenz: Code-scanning-Datenbank.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: push (main), pull_request, schedule.
- Nebenwirkungen: SARIF-Upload (security-events: write nur im analyze-Job).
- Fehlerfälle: Findings; bekannte Limitation: Rust-Extractor meldet jede Datei "extracted with errors" wegen unvollständiger Macro-Expansion (github/codeql#19966, #19982, #20659) — empirisch verifiziert, keine cargo-cult Setup-Schritte (belegt durch Dateiinhalt).
- Sicherheitsrelevanz: Hoch — SAST-Ergänzung zu deny/gitleaks; least-privilege (contents: read global, security-events: write nur im Job).
- Geschäftslogik: Build-mode: none als bewusste Kosten-/Qualitäts-Entscheidung.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: GitHub Actions, codeql-action v4.37.3, harden-runner v2.20.0, checkout v7.0.1.
- Rust-Relevanz: CodeQL-Rust-Setup (build-mode none, paths-ignore für Test/Fuzz-Fixtures) direkt auf das Rewrite übertragbar.

## .github/workflows/scorecard.yml

- Zweck: OpenSSF-Scorecard: automatisiertes Supply-Chain-Posture-Scoring (Token-Berechtigungen, gepinnte Dependencies, Branch-Protection, gefährliche Workflow-Muster), wöchentlich (Montag 07:28 UTC), push auf main und workflow_dispatch; Ergebnisse in Code scanning + README-Badge via Scorecard-API (belegt durch Dateiinhalt, 45 Zeilen).
- Verantwortlichkeit: Laufendes Supply-Chain-Sicherheits-Scoring.
- Eingaben: Repo-Zustand.
- Ausgaben: SARIF (upload-artifact retention 5 Tage + Upload zu Code scanning).
- Datenfluss: Repo → scorecard-action → results.sarif → Artifact + Code scanning.
- Persistenz: Scorecard-API, SARIF.
- Zustände: Keine.
- APIs: Scorecard-API (publish_results: true).
- Ereignisse: schedule, push (main), workflow_dispatch.
- Nebenwirkungen: Veröffentlicht Ergebnisse (Badge).
- Fehlerfälle: Keine (Scoring ohne Gate).
- Sicherheitsrelevanz: Hoch — Supply-Chain-Posture-Transparenz; `permissions: read-all` global, Job optiert security-events: write + id-token: write ein.
- Geschäftslogik: Keine Gate-Wirkung, nur Reporting.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: GitHub Actions, ossf/scorecard-action v2.4.4, codeql-action upload-sarif, harden-runner.
- Rust-Relevanz: Scorecard-Setup (SARIF-Reporting, publish_results) direkt übertragbar.

## .github/CODEOWNERS

- Zweck: Default-Owner für alle Pfade: `* @mizterta` — wird bei jedem PR zur Review angefragt (belegt durch Dateiinhalt, 3 Zeilen).
- Verantwortlichkeit: Review-Zuordnung.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Sichert Review-Pflicht (in Kombination mit Branch-Protection).
- Abhängigkeiten: GitHub.
- Rust-Relevanz: Konvention übertragbar. Rust-Relevanz: Keine.

## .github/codeql/codeql-config.yml

- Zweck: CodeQL-Config: `paths-ignore` für tests/** und fuzz/** — Hard-Coded-Secret-Queries sollen nur auf der operator-facing src/**-Fläche laufen; Test-/Fuzz-Crates enthalten absichtliche Fixture-Credentials (Wegwerf-Passwörter, handgerollte Low-Cost-Salze, RFC-Testvektoren), die in Test-Code Rauschen wären (belegt durch Dateiinhalt, 14 Zeilen).
- Verantwortlichkeit: Rausch-Reduktion der CodeQL-Hard-Coded-Secret-Queries; funktioniert nur unter `build-mode: none` (Quelltext-Extraktion), Kommentar dokumentiert, dass `#[cfg(test)]`-Module innerhalb gescannter src/-Dateien nicht erreichbar sind (dortige Fixture-Alerts werden im UI als "used in tests" dismissed).
- Eingaben: CodeQL-Scan-Config.
- Ausgaben: Verfeinerter Scan-Bereich.
- Datenfluss: config.yml → codeql-action/init.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Wichtige Nuance für Secret-Scanning (Test-Fixtures vs. Produktions-Secrets).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: codeql-action.
- Rust-Relevanz: paths-ignore-Strategie für Test-/Fuzz-Bäume direkt übertragbar.

## .github/dependabot.yml

- Zweck: Dependabot-Konfiguration: 3 Ökosysteme (cargo wöchentlich mit cooldown default-days 7, groups cargo-minor-patch; github-actions wöchentlich mit group actions; docker wöchentlich für rust:1-alpine), open-pull-requests-limit 5, ignore-Regeln für hmac/sha2/getrandom (bewusst auf RustCrypto-0.10/0.12 + getrandom-0.2-Linie gepinnt — 0.11/0.13/0.3-Bumps sind Breaking-API ohne Security-Fix, cargo-deny clean) (belegt durch Dateiinhalt, 46 Zeilen).
- Verantwortlichkeit: Automatisierte Dependency-Updates mit dokumentierter Ausnahme-Policy für den Auth-Crypto-Stack.
- Eingaben: Cargo.lock, Workflows, Dockerfile.
- Ausgaben: Update-PRs (minor/patch gruppiert, major einzeln).
- Datenfluss: Registry → Dependabot → PR.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: wöchentlich.
- Nebenwirkungen: Erzeugt PRs.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Hoch — Security-Updates umgehen Cooldowns; bewusste Major-Pins verhindern unkontrollierte Crypto-Stack-Migration; ergänzt CHANGELOG-[0.6.3]/[0.5.0]-Policy ("Dependabot only takes patches").
- Geschäftslogik: Gruppierungs- und Cooldown-Policy; Major-Pin für hmac/sha2/getrandom.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: GitHub Dependabot.
- Rust-Relevanz: Dieselbe Crypto-Pin-Strategie und Gruppen-Policy direkt übernehmbar.

## .github/release.yml

- Zweck: Template für GitHub-generierte Release-Notes (release.yml-Workflow nutzt generate_release_notes: true): Kategorien nach PR-Labels — Security, Breaking changes, Features (enhancement), Fixes (bug), Documentation, Dependencies (von Dependabot automatisch gelabelt), Other changes; Exclude-Label skip-changelog (belegt durch Dateiinhalt, 22 Zeilen).
- Verantwortlichkeit: Deterministische Release-Notes-Kategorisierung.
- Eingaben: PR-Labels.
- Ausgaben: Release-Notes.
- Datenfluss: PR-Labels → Kategorien.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Security-Kategorie für Signing-/Vuln-Fixes.
- Geschäftslogik: Label-Mapping.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: GitHub Release-Notes-Generator.
- Rust-Relevanz: Konvention übertragbar. Rust-Relevanz: Keine.

## .github/ISSUE_TEMPLATE/bug_report.yml

- Zweck: Bug-Report-Formular: Beschreibung, Reproduktion, erwartetes vs. tatsächliches Verhalten, Screenshots, Umgebung (OS, Browser, Version), Logs (belegt durch Dateiinhalt).
- Verantwortlichkeit: Strukturierte Bug-Erfassung.
- Eingaben: Nutzerangaben.
- Ausgaben: Issue.
- Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Verweist auf private Meldung für Sicherheits-Bugs (belegt durch Dateiinhalt).
- Abhängigkeiten: GitHub Issue-Forms.
- Rust-Relevanz: Keine (Template). Rust-Relevanz: Keine.

## .github/ISSUE_TEMPLATE/config.yml

- Zweck: Issue-Formular-Konfiguration: deaktiviert Blank-Issues (only_blank), verweist auf Discussions für Fragen (belegt durch Dateiinhalt).
- Verantwortlichkeit: Issue-Routing.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine.
- Abhängigkeiten: GitHub.
- Rust-Relevanz: Keine (Konfiguration). Rust-Relevanz: Keine.

## .github/ISSUE_TEMPLATE/feature_request.yml

- Zweck: Feature-Request-Formular: Problembeschreibung, gewünschte Lösung, Alternativen, Nutzungskontext (belegt durch Dateiinhalt).
- Verantwortlichkeit: Strukturierte Feature-Erfassung.
- Eingaben: Nutzerangaben.
- Ausgaben: Issue.
- Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine.
- Abhängigkeiten: GitHub Issue-Forms.
- Rust-Relevanz: Keine (Template). Rust-Relevanz: Keine.

## .github/PULL_REQUEST_TEMPLATE.md

- Zweck: PR-Template: Changelog-Pflicht ([Unreleased] aktualisieren), Knowledge-Base-Lockstep, Test-/Coverage-Nachweis, Checkliste (belegt durch Dateiinhalt).
- Verantwortlichkeit: Qualitätsvertrag für PRs (CONTRIBUTING.md-Fortsetzung).
- Eingaben: PR-Beschreibung.
- Ausgaben: PR.
- Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Checkboxen für Security-Invarianten.
- Abhängigkeiten: GitHub.
- Rust-Relevanz: Keine (Template). Rust-Relevanz: Keine.

---

## scripts/mock_nim.py

- Zweck: Scriptbarer Mock-NIM-Server für e2e-Tests/Load-Harness: `do_GET` für `/v1/models` und `/control/stats`, `do_POST` für `/v1/chat/completions`, Sliding-Window-Violation-Erzwingung über `/control/stats`-Steuerung; `--enforce`-Flag für die Load-Harness (belegt durch Dateiinhalt, CONTRIBUTING.md "mock_nim.py --enforce").
- Verantwortlichkeit: Deterministischer Upstream-Ersatz; simuliert Rate-Limits (429) und Metrik-Zähler.
- Eingaben: HTTP-Requests, Steuerbefehle (stats).
- Ausgaben: NIM-artige JSON-Responses (models, completions), Statistiken.
- Datenfluss: Request → Handler → Response; Stats-Registry.
- Persistenz: Keine (in-memory).
- Zustände: Request-Zähler, Violation-Flags.
- APIs: `/v1/models`, `/v1/chat/completions`, `/control/stats` (belegt durch Dateiinhalt).
- Ereignisse: Keine.
- Nebenwirkungen: Keine außer Zähler.
- Fehlerfälle: 429 bei erzwungenen Violations.
- Sicherheitsrelevanz: Keine Produktionsrelevanz (Test-Werkzeug).
- Geschäftslogik: Erzwingt Rate-Limit-Violationen deterministisch; `--enforce` aktiviert die Enforcement-Statistik für den Load-Harness-Lauf.
- Algorithmen: Sliding-Window-Check.
- verwendete Datenmodelle: Stats, Models-Liste.
- Abhängigkeiten: Python stdlib (http.server).
- Rust-Relevanz: Test-Double-Konzept für Rewrite-Tests (Mock-NIM als Rust-Testmodul oder separater Binary). Rust-Relevanz: Keine (Test-Helfer, Python).

## scripts/loadtest.py

- Zweck: Load-Harness: erzeugt parallele Chat-Completion-Requests mit Tools (jede 3. Anfrage) und json_object (jede 5. Anfrage) gegen laufenden Proxy; Teil des Zero-Violations-Hard-Requirements (belegt durch Dateiinhalt, CONTRIBUTING.md).
- Verantwortlichkeit: Verifiziert "null upstream violations" unter Last (100 Clients, 0 Violations laut README).
- Eingaben: Konfiguration (URL, Clients, Requests, Rate), Payload-Generator.
- Ausgaben: Last-Ergebnis/Statistik.
- Datenfluss: Generator → HTTP → Statistik.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: /v1/chat/completions.
- Ereignisse: Keine.
- Nebenwirkungen: Last auf Proxy/Upstream.
- Fehlerfälle: 429/5xx zählen als Ergebnis.
- Sicherheitsrelevanz: Keine besondere (Test).
- Geschäftslogik: Payload-Varianz (Tools/json_object) erzwingt realistische Nutzung.
- Algorithmen: Thread/Async-Parallelismus, Zähler.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Python, HTTPX/Requests.
- Rust-Relevanz: Load-Harness-Anforderung für das Rewrite (eigener Bench-Binary). Rust-Relevanz: Keine (Test-Helfer, Python).

## scripts/test_release_contract.py

- Zweck: Verifiziert die Release-Verträge: cosign v3.1.2 (x2), `--bundle "$f.sigstore.json"` ohne `--output-signature`/`--output-certificate`, gh-CLI statt softprops/action-gh-release (belegt durch Dateiinhalt, 45 Zeilen).
- Verantwortlichkeit: Regressionsschutz für die Signing-/Release-Pipeline.
- Eingaben: Release-Artefakte, Repo.
- Ausgaben: Pass/Fail-Vertragsprüfung.
- Datenfluss: Artefakte → Checks.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: gh, cosign-CLI.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Vertragsabweichung → Fail.
- Sicherheitsrelevanz: Hoch — sichert Signatur-Verträge (bundle-Format, cosign-Version).
- Geschäftslogik: Kodifiziert die Release-Invarianten.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Python, cosign, gh.
- Rust-Relevanz: Konzept (Vertrags-Tests für Releases) übertragbar. Rust-Relevanz: Keine (Test-Skript, Python).

---

## fuzz/Cargo.toml

- Zweck: Fuzz-Modul-Manifest: deklariert Fuzz-Targets (sse_scan, sanitize_label, config_roundtrip) und Abhängigkeiten (lib crate + serde_json etc.) (belegt durch Dateiinhalt).
- Verantwortlichkeit: Fuzz-Projektaufbau (cargo-fuzz).
- Eingaben: Keine.
- Ausgaben: Fuzz-Binaries.
- Datenfluss: Build.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Build-Fehler.
- Sicherheitsrelevanz: Absicherung der Parse-Pfade.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: cargo-fuzz, lib crate.
- Rust-Relevanz: Rust-2024: Fuzz-Targets direkt übernehmbar (cargo-fuzz bleibt Standard).

## fuzz/fuzz_targets/sse_scan.rs

- Zweck: Fuzz-Target für SSE-Scanning: fuzzt den SSE-Parser (heartbeat, usage, tool_calls, truncated-JSON, malformed) (belegt durch Dateiinhalt).
- Verantwortlichkeit: Findet Panics/Infinite-Loops im SSE-Verarbeitungspfad.
- Eingaben: Fuzz-Input (Bytes).
- Ausgaben: Parser-Ergebnis/Findings.
- Datenfluss: Input → Parser.
- Persistenz: Corpus/Artefakte.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Corpus-Wachstum.
- Fehlerfälle: Panic/Timeout.
- Sicherheitsrelevanz: Absicherung des Streaming-Pfads.
- Geschäftslogik: Keine.
- Algorithmen: SSE-Ereignisparsing.
- verwendete Datenmodelle: SSE-Segmente.
- Abhängigkeiten: cargo-fuzz, lib (proxy).
- Rust-Relevanz: Direkt übernehmbar; Tests für den SSE-Parser im Rewrite nötig.

## fuzz/fuzz_targets/sanitize_label.rs

- Zweck: Fuzz-Target für sanitize_label: fuzzt Label-Sanitizing (max 64, erlaubtes Alphabet, hostile-Input) (belegt durch Dateiinhalt).
- Verantwortlichkeit: Keine Metrik-Injection/Panics durch bösartige Labels.
- Eingaben: Fuzz-Input (String).
- Ausgaben: Sanitized-Label/Findings.
- Datenfluss: Input → sanitize_label.
- Persistenz: Corpus.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Corpus-Wachstum.
- Fehlerfälle: Panic.
- Sicherheitsrelevanz: Hoch — verhindert Prometheus-Label-Injection.
- Geschäftslogik: Alphabet `[A-Za-z0-9._/:-]`, max 64.
- Algorithmen: Zeichenfilterung/Truncation.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: cargo-fuzz, lib (proxy).
- Rust-Relevanz: Direkt übernehmbar; Property-Tests im Rewrite.

## fuzz/fuzz_targets/config_roundtrip.rs

- Zweck: Fuzz-Target für Config-Serialisierung: fuzzt StoredConfig-Roundtrips (Deserialize → Serialize) (belegt durch Dateiinhalt).
- Verantwortlichkeit: Keine Panics/Data-Loss bei Config-Deserialisierung.
- Eingaben: Fuzz-Input (JSON-Bytes).
- Ausgaben: Roundtrip-Ergebnis.
- Datenfluss: JSON → Config → JSON.
- Persistenz: Corpus.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Corpus-Wachstum.
- Fehlerfälle: Deserialize-Fehler → keine Panic.
- Sicherheitsrelevanz: Absicherung des Config-Pfads (Credentials).
- Geschäftslogik: Keine.
- Algorithmen: Serde-Roundtrip.
- verwendete Datenmodelle: StoredConfig.
- Abhängigkeiten: cargo-fuzz, lib (config).
- Rust-Relevanz: Direkt übernehmbar; serde-Tests im Rewrite.

## fuzz/fuzz_targets/ (weitere)

- Zweck: Keine weiteren Targets belegt (nur sse_scan, sanitize_label, config_roundtrip) (belegt durch fuzz.yml, Dateiinhalt).
- Verantwortlichkeit: Entfällt.
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle/Abhängigkeiten: Entfällt.
- Sicherheitsrelevanz: Keine.
- Rust-Relevanz: Keine.

## fuzz/seeds/ (8 Dateien)

- Zweck: Seed-Corpus für die Fuzz-Targets (belegt durch Dateiinhalt):
  - `sse_scan/stream-with-usage`: SSE-Stream mit tool_calls + usage inkl. reasoning_tokens (repräsentativer NIM-Stream) — Trains den Parser auf tool-calling-Segmente.
  - `sse_scan/malformed`: usage:null, not-json-Felder — Robustheit gegen kaputte Segmente.
  - `sse_scan/truncated-mid-json`: Heartbeat + mitten im JSON abgeschnittener Chunk — Parser-Mid-Stream-Zustand.
  - `config_roundtrip/minimal.json`: `{"version":1}` — minimaler Config-Roundtrip.
  - `config_roundtrip/store.json`: Vollständiges StoredConfig-Beispiel: version 1, upstream base_url `https://integrate.api.nvidia.com`, nim_keys (nvapi-test, owner admin, enabled, rpm 40), client_auth mode keyed mit keys[{name opencode, secret_sha256 (Nullen), owner admin}], limits (max_wait_secs 30, heartbeat_secs 10, stream_idle_secs 120, request_timeout_secs 300, max_inflight 64, strict_passthrough false), users [{username admin, password_hash `pbkdf2-sha256$1000$00$00`, role superuser}] (belegt durch Dateiinhalt).
  - `sanitize_label/hostile`: `{__proto__}é🔥<script>"` — Prototype-Pollution-Typo, Unicode, Emoji, Script-Tag, Quote — injektionsartiger Input (belegt durch Dateiinhalt).
  - `sanitize_label/model-id`: `meta/llama-3.3-70b-instruct` — realistische Modell-ID.
  - `sanitize_label/model-with-colon`: `nvidia/llama-3.1-nemotron-ultra-253b-v1:free` — Doppelpunkt/Versionierung.
- Verantwortlichkeit: Deterministische Ausgangspunkte für Corpus-Evolution; dokumentieren die erwartete Eingabeform je Target.
- Eingaben: Keine.
- Ausgaben: Corpus-Saat.
- Datenfluss: Seeds → Corpus → Target.
- Persistenz: Versionskontrolle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Corpus-Wachstum.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: hostile/malformed-Seeds decken Injection-/Korruptionspfade ab.
- Geschäftslogik: Belegt das Config-Schema-Feld für Feld (store.json) und die realen Modell-ID-Formate (sanitize_label).
- Algorithmen: Keine.
- verwendete Datenmodelle: StoredConfig (store.json), SSE-Segmente, Label-Strings.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Seeds als Test-Fixtures im Rewrite direkt nutzbar (serde-Tests, sanitize_label-Tests, SSE-Parser-Tests).

## fuzz/.gitignore

- Zweck: Ignoriert fuzz-spezifische Artefakte: target/, artifacts/, coverage/, Cargo.lock, corpus/ (belegt durch Dateiinhalt, 5 Zeilen).
- Verantwortlichkeit: Hält Fuzz-Artefakte aus Git (Corpus wird nicht versioniert).
- Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Geschäftslogik/Algorithmen/verwendete Datenmodelle: Keine.
- Sicherheitsrelevanz: Keine.
- Abhängigkeiten: Git.
- Rust-Relevanz: Konvention übertragbar. Rust-Relevanz: Keine.

---

## examples/README.md

- Zweck: Erklärt die Beispiel-Dateien und den Kontext der Integrationspfade (belegt durch Dateiinhalt).
- Verantwortlichkeit: Dokumentation der Beispiele (opencode.json; weitere Client-Rezepte laut README: Codex, n8n, curl).
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Enthalten nur Beispiel-Secrets (gitleaks-Allowlist), keine echten Credentials.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Keine (Beispiel-Dokumentation). Rust-Relevanz: Keine.

## examples/opencode.json

- Zweck: Beispiel-Konfiguration für den OpenCode-Client (Provider-Einrichtung gegen den Proxy) (belegt durch Dateiinhalt).
- Verantwortlichkeit: Referenz für die OpenCode-Integration (einer der vom README genannten Harness-Pfade).
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Definiert die vom Client genutzten Proxy-Endpunkte.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Enthält nur Beispiel-Secrets (gitleaks-Allowlist).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: OpenCode.
- Rust-Relevanz: Belegt die clientseitige API-Erwartung (Endpoint-/Auth-Form) für das Rewrite. Rust-Relevanz: Keine (Beispiel-Config, JSON).

## docs/assets/ (7 PNGs)

- Zweck: Dashboard-/Setup-Screenshots: setup-wizard.png, dashboard-overview.png, dashboard-models.png, dashboard-clients.png, dashboard-reliability.png, dashboard-settings.png, logo.png (belegt durch README-Referenzen, Dateigrößen 39–183 KB).
- Verantwortlichkeit: Visuelle Dokumentation der UI.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Versionierte Bilddateien.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine (Screenshots ohne echte Credentials; zeigen Beispiel-Daten).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Visuelle Wiedergabe im Rahmen dieser Analyse nicht möglich (Modell unterstützt kein Bild-Input); Rollen der Dateien wurden über README-Bildreferenzen und das vollständig gelesene dashboard.html zugeordnet. Rust-Relevanz: Keine (Assets).

---

## Abdeckungsverifikation

- Bestandsaufnahme (2026-08-04): `find . -type f -not -path './.git/*'` = **113 Dateien**; Verteilung: Root 21, `src/` 12, `tests/` 2, `knowledge/` 39 (17 decisions inkl. index, 8 architecture inkl. index, 6 ops inkl. index, 4 research inkl. index, 2 testing inkl. index, knowledge/index.md, knowledge/log.md), `.github/` 14, `scripts/` 3, `fuzz/` 13 (3 Targets + 8 Seeds + Cargo.toml + .gitignore), `examples/` 2, `docs/assets/` 7 PNGs.
- Alle 113 Dateien sind oben dokumentiert (Gruppen-Einträge decken die knowledge/-Kategorien und PNG-Assets jeweils vollständig ab; `src/dashboard.js` existiert nicht — das Dashboard-JS ist in `src/dashboard.html` eingebettet und wird im CI via Python-Extraktion + `node --check` geprüft, belegt durch `.github/workflows/ci.yml`).
- Keine Datei wurde ausgelassen; die Lektüre jeder Datei ist belegt (Dateiinhalt bzw. für PNGs README-Referenzen + dashboard.html-Zuordnung).
