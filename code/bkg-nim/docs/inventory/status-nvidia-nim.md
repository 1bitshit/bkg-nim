# Status-Analyse — nvidia-nim (nim-proxy)

## Übersicht

| Feld | Wert |
|---|---|
| Repository | `nvidia-nim` (GitHub: `miztertea/nim-proxy`) |
| Pfad | `/workspaces/bkg-nim/repos/nvidia-nim` |
| Commit-Basis | `content-copy` (Klon ohne `.git`) |
| Prüfsummen-Freeze | `source-checksum.md` Zeilen 163–426; 113/113 Hashs identisch |
| Dateien gesamt | 113 |
| FERTIG_ANALYSIERT | 106 |
| ANALYSIS_BLOCKED (Binär) | 7 |
| DISCOVERED / READING | 0 / 0 |
| Gesamtzeilen (Text) | 26.145 |
| Evidence-Blöcke | 119 (EV-NIMPROXY-000001 … EV-NIMPROXY-000119) |
| Verbotsbegriffe | 0 Treffer |
| Ergebnis | FERTIG_ANALYSIERT (Repository vollständig analysiert) |

## Status je Datei (113 Einträge)

Statuswerte: `FERTIG_ANALYSIERT` = vollständig gelesen und verifiziert; `ANALYSIS_BLOCKED` = binäres Asset, nicht analysierbar (Nachweis vorhanden).

| # | Datei | Status | Beleg |
|---|---|---|---|
| 1 | .dockerignore | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Aussage: Docker-Ignore; Byte 34, 5 Zeilen) |
| 2 | .editorconfig | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Byte 463, 19 Zeilen) |
| 3 | .env.example | FERTIG_ANALYSIERT | EV-NIMPROXY-000015 (5 Env-Vars + PUBLISH_HOST) |
| 4 | .gitattributes | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Byte 368, 13 Zeilen) |
| 5 | .github/CODEOWNERS | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Owner-Zuordnung) |
| 6 | .github/ISSUE_TEMPLATE/bug_report.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Bug-Report-Formular) |
| 7 | .github/ISSUE_TEMPLATE/config.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Issue-Formular-Routing) |
| 8 | .github/ISSUE_TEMPLATE/feature_request.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Feature-Request-Formular) |
| 9 | .github/PULL_REQUEST_TEMPLATE.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (PR-Vorlage) |
| 10 | .github/codeql/codeql-config.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000119 |
| 11 | .github/dependabot.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000106 (Cooldown default-days 7) |
| 12 | .github/release.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Release-Dispatch-Konfig) |
| 13 | .github/workflows/audit.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000117 (wöchentlicher Advisory-Lauf) |
| 14 | .github/workflows/ci.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000019 / EV-NIMPROXY-000105 (Gates, Coverage ≥ 90 %) |
| 15 | .github/workflows/codeql.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000115 |
| 16 | .github/workflows/fuzz.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000114 |
| 17 | .github/workflows/release.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000020 (multi-arch, cosign, SLSA, SBOM) |
| 18 | .github/workflows/scorecard.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000116 |
| 19 | .gitignore | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 |
| 20 | .gitleaks.toml | FERTIG_ANALYSIERT | EV-NIMPROXY-000118 |
| 21 | AGENTS.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Agent-Verhaltensregeln) |
| 22 | CHANGELOG.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000022 / EV-NIMPROXY-000040 |
| 23 | CODE_OF_CONDUCT.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Contributor Covenant 2.1, 134 Zeilen) |
| 24 | CONTRIBUTING.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000042 (Test-Kontrakt 69+53) |
| 25 | Cargo.lock | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (2126 Zeilen, 53.691 Byte) |
| 26 | Cargo.toml | FERTIG_ANALYSIERT | EV-NIMPROXY-000021 / EV-NIMPROXY-000103 |
| 27 | Dockerfile | FERTIG_ANALYSIERT | EV-NIMPROXY-000016 (scratch, UID 10001, --health) |
| 28 | LICENSE | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (MIT, 21 Zeilen) |
| 29 | README.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 / EV-NIMPROXY-000039 (366 Zeilen) |
| 30 | SECURITY.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000041 (123 Zeilen) |
| 31 | SUPPORT.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (15 Zeilen) |
| 32 | deny.toml | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (39 Zeilen) |
| 33 | docker-compose.dev.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Dev-Compose) |
| 34 | docker-compose.yml | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (PUBLISH_HOST, Volume /data) |
| 35 | docs/assets/dashboard-clients.png | ANALYSIS_BLOCKED | Binär (PNG, 97.437 Byte); Nachweis: Hash b22cbffe…, Verwendung Dashboard-Tab |
| 36 | docs/assets/dashboard-models.png | ANALYSIS_BLOCKED | Binär (PNG, 117.318 Byte); Hash 1e42b655… |
| 37 | docs/assets/dashboard-overview.png | ANALYSIS_BLOCKED | Binär (PNG, 128.835 Byte); Hash dcea0519… |
| 38 | docs/assets/dashboard-reliability.png | ANALYSIS_BLOCKED | Binär (PNG, 117.569 Byte); Hash 4cbbb246… |
| 39 | docs/assets/dashboard-settings.png | ANALYSIS_BLOCKED | Binär (PNG, 183.540 Byte); Hash b3d6082d… |
| 40 | docs/assets/logo.png | ANALYSIS_BLOCKED | Binär (PNG, 108.062 Byte); Hash 65d618b0… |
| 41 | docs/assets/setup-wizard.png | ANALYSIS_BLOCKED | Binär (PNG, 39.254 Byte); Hash 655b7d9a… |
| 42 | examples/README.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (54 Zeilen) |
| 43 | examples/opencode.json | FERTIG_ANALYSIERT | EV-NIMPROXY-000031 |
| 44 | fuzz/.gitignore | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (5 Zeilen) |
| 45 | fuzz/Cargo.toml | FERTIG_ANALYSIERT | EV-NIMPROXY-000113 |
| 46 | fuzz/fuzz_targets/config_roundtrip.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000027 |
| 47 | fuzz/fuzz_targets/sanitize_label.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000028 |
| 48 | fuzz/fuzz_targets/sse_scan.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000029 |
| 49 | fuzz/seeds/config_roundtrip/minimal.json | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (0 Byte, Seed) |
| 50 | fuzz/seeds/config_roundtrip/store.json | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (1 Zeile, Seed) |
| 51 | fuzz/seeds/sanitize_label/hostile | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Seed) |
| 52 | fuzz/seeds/sanitize_label/model-id | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (0 Byte, Seed) |
| 53 | fuzz/seeds/sanitize_label/model-with-colon | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (0 Byte, Seed) |
| 54 | fuzz/seeds/sse_scan/malformed | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Seed) |
| 55 | fuzz/seeds/sse_scan/stream-with-usage | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Seed) |
| 56 | fuzz/seeds/sse_scan/truncated-mid-json | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Seed) |
| 57 | knowledge/architecture/client-auth.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000026 |
| 58 | knowledge/architecture/dashboard.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000025 |
| 59 | knowledge/architecture/dispatcher.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000092 |
| 60 | knowledge/architecture/governor.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000094 |
| 61 | knowledge/architecture/index.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Wissensbasis-Index) |
| 62 | knowledge/architecture/key-pool.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000091 |
| 63 | knowledge/architecture/metrics-history.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000090 |
| 64 | knowledge/architecture/streaming-pipeline.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000093 |
| 65 | knowledge/decisions/auth-posture-and-dashboard-password.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000084 |
| 66 | knowledge/decisions/dashboard-operator-console-redesign.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000082 |
| 67 | knowledge/decisions/dependency-update-cooldown.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000088 |
| 68 | knowledge/decisions/distroless-scratch-image.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000077 |
| 69 | knowledge/decisions/explicit-request-deadline.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000081 |
| 70 | knowledge/decisions/global-fifo-dispatcher.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000079 |
| 71 | knowledge/decisions/history-retention-days-not-size.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000075 |
| 72 | knowledge/decisions/index.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (ADR-Index, 15 Entscheidungen) |
| 73 | knowledge/decisions/input-sanitizing-and-xss.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000073 |
| 74 | knowledge/decisions/request-shape-metrics.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000083 |
| 75 | knowledge/decisions/reset-aware-dashboard-history.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000087 |
| 76 | knowledge/decisions/sliding-window-not-token-bucket.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000023 |
| 77 | knowledge/decisions/sse-heartbeats-for-rate-waits.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000080 |
| 78 | knowledge/decisions/sticky-affinity-with-spillover.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000078 |
| 79 | knowledge/decisions/ui-managed-config-store.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000076 |
| 80 | knowledge/decisions/usage-injection-auto-fallback.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000085 |
| 81 | knowledge/decisions/window-jitter-margin.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000086 |
| 82 | knowledge/index.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Wissensbasis-Übersicht) |
| 83 | knowledge/log.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (568 Zeilen, Projektlog) |
| 84 | knowledge/ops/capacity-math.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000024 |
| 85 | knowledge/ops/configure-env.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000095 |
| 86 | knowledge/ops/deploy-docker.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000096 |
| 87 | knowledge/ops/index.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Ops-Index) |
| 88 | knowledge/ops/release.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000034 |
| 89 | knowledge/ops/sharing-with-friends.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000097 |
| 90 | knowledge/research/index.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Recherche-Index) |
| 91 | knowledge/research/nim-free-tier-40rpm-no-credits.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000033 |
| 92 | knowledge/research/nim-kv-cache-reuse.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000101 |
| 93 | knowledge/research/nim-models-endpoint-schema.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000100 |
| 94 | knowledge/testing/index.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000001 (Test-Index) |
| 95 | knowledge/testing/test-strategy.md | FERTIG_ANALYSIERT | EV-NIMPROXY-000032 |
| 96 | rust-toolchain.toml | FERTIG_ANALYSIERT | EV-NIMPROXY-000035 / EV-NIMPROXY-000104 (MSRV 1.87) |
| 97 | scripts/loadtest.py | FERTIG_ANALYSIERT | EV-NIMPROXY-000018 |
| 98 | scripts/mock_nim.py | FERTIG_ANALYSIERT | EV-NIMPROXY-000017 / EV-NIMPROXY-000110 |
| 99 | scripts/test_release_contract.py | FERTIG_ANALYSIERT | EV-NIMPROXY-000030 |
| 100 | src/auth.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000003 / EV-NIMPROXY-000043 / EV-NIMPROXY-000058 / EV-NIMPROXY-000063 / EV-NIMPROXY-000111 |
| 101 | src/config.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000009 / EV-NIMPROXY-000044 / EV-NIMPROXY-000059 / EV-NIMPROXY-000065 / EV-NIMPROXY-000102 |
| 102 | src/dashboard.html | FERTIG_ANALYSIERT | EV-NIMPROXY-000013 / EV-NIMPROXY-000045 / EV-NIMPROXY-000089 |
| 103 | src/dispatch.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000007 / EV-NIMPROXY-000052 |
| 104 | src/governor.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000008 / EV-NIMPROXY-000050 / EV-NIMPROXY-000064 |
| 105 | src/history.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000010 / EV-NIMPROXY-000036 / EV-NIMPROXY-000061 / EV-NIMPROXY-000068 / EV-NIMPROXY-000074 / EV-NIMPROXY-000112 |
| 106 | src/lib.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000053 / EV-NIMPROXY-000054 / EV-NIMPROXY-000055 / EV-NIMPROXY-000066 / EV-NIMPROXY-000071 |
| 107 | src/main.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000002 / EV-NIMPROXY-000056 (Shim; CLI-Claims der Inventardoku teils überzeichnet) |
| 108 | src/pool.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000006 / EV-NIMPROXY-000049 / EV-NIMPROXY-000057 / EV-NIMPROXY-000070 |
| 109 | src/proxy.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000011 / EV-NIMPROXY-000051 / EV-NIMPROXY-000060 / EV-NIMPROXY-000072 / EV-NIMPROXY-000098 / EV-NIMPROXY-000099 |
| 110 | src/settings.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000012 / EV-NIMPROXY-000037 / EV-NIMPROXY-000062 / EV-NIMPROXY-000069 |
| 111 | src/setup.html | FERTIG_ANALYSIERT | EV-NIMPROXY-000014 / EV-NIMPROXY-000046 |
| 112 | tests/e2e.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000004 / EV-NIMPROXY-000047 / EV-NIMPROXY-000107 |
| 113 | tests/support/mod.rs | FERTIG_ANALYSIERT | EV-NIMPROXY-000005 / EV-NIMPROXY-000048 / EV-NIMPROXY-000067 / EV-NIMPROXY-000108 / EV-NIMPROXY-000109 |

## Qualitätsprüfung (GOAL-CHECK)

| Prüfung | Ergebnis |
|---|---|
| Enthält eine Markdown-Datei „Nicht ermittelt"? | Nein (0 Treffer) |
| Enthält eine Markdown-Datei „Unbekannt"? | Nein (0 Treffer) |
| Enthält eine Markdown-Datei „Nicht analysiert"? | Nein (0 Treffer) |
| Enthält eine Markdown-Datei „TODO"? | Nein (0 Treffer) |
| Enthält eine Markdown-Datei „TBD"? | Nein (0 Treffer) |
| Datei mit Status DISCOVERED? | Nein (0) |
| Datei mit Status READING? | Nein (0) |
| Ungelesene Datei? | Nein (113/113 gelesen bzw. als Binär belegt) |

## Gesamtergebnis

**FERTIG_ANALYSIERT** — Das Repository `nvidia-nim` ist vollständig analysiert und validiert. Grundlage für die eigenständige Rust-2024-Neuentwicklung ist vollständig vorhanden und evidence-basiert dokumentiert (siehe `validation-nvidia-nim.md`).
