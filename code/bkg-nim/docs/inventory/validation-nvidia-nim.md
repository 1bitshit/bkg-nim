# Validierung der Analyse-Inventardokumentation — nvidia-nim (nim-proxy)

## Validierungsprotokoll

| Feld | Wert |
|---|---|
| Validierung-ID | VAL-NIMPROXY-2026-08-05 |
| Repository | `nvidia-nim` (GitHub: `miztertea/nim-proxy`) |
| Repository-Lokation | `/workspaces/bkg-nim/repos/nvidia-nim` |
| Inventar-Ordner | `/workspaces/bkg-nim/inventory/02-nim-proxy/` |
| Commit-Basis | `content-copy` (lokaler Klon ohne `.git`, 113 Dateien) |
| Prüfsummen-Freeze | `docs/inventory/source-checksum.md`, Zeilen 163–426 (`./nvidia-nim/…`), 113/113 Hashs bei erneuter Berechnung identisch |
| Prüfzeitpunkt | 2026-08-05 |
| Prüfer | BKG-NIM Reverse-Engineering-Agent |

## Kriterien

1. Jede Datei des Repositories wurde mindestens einmal vollständig gelesen (Datei-Lese-Definition: Hash, Byte Size, Line Count, Encoding, Read Timestamp, Reader Result).
2. Jede Inventar-Aussage wird mit einer Quelle (Datei + Zeilenbereich) belegt.
3. Keine Inventar-Aussage enthält die verbotenen Statuswerte: `Nicht ermittelt`, `Unbekannt`, `Nicht analysiert`, `Nicht überprüft`, `Vermutlich`, `Wahrscheinlich`, `TODO`, `TBD`.
4. Negativaussagen müssen dokumentiert sein.
5. Prüfung der Querverweise (Importe, Aufrufer, Konfigurationen, Tests, Dokumentation).

## Ergebnis der Verbotsbegriff-Prüfung

`grep -rn -E "Nicht ermittelt|Unbekannt|Nicht analysiert|Nicht überprüft|Vermutlich|Wahrscheinlich|TODO|TBD" /workspaces/bkg-nim/inventory/02-nim-proxy/`

Ergebnis: **0 Treffer** (Exit-Code 1). Kein verbotener Begriff vorhanden.

## Ergebnis der Datei-Vollständigkeitsprüfung

- Dateien gesamt (ohne `.git`): **113** — entspricht der Index-Angabe (`files.md`, "Insgesamt 113 Dateien").
- 113 Read-Evidence-Einträge erzeugt; 113/113 Hashs gegen `source-checksum.md` verifiziert (Soll = Ist).
- **106 Dateien `FERTIG_ANALYSIERT`** (Textdateien, vollständig gelesen).
- **7 Dateien `ANALYSIS_BLOCKED`** (binäre PNG-Assets, nicht analysierbarer Binärinhalt; Nachweis: Hash, Typ, Byte Size, Verwendung in `src/dashboard.html`/`src/setup.html`).
- Gesamtzeilen (Textdateien, `wc -l`): **26.145**.

## Diskrepanzen (Claim → Befund)

1. **CLI-Fähigkeiten `--version` / `-V` / `--port`** — `files.md` (main.rs-Abschnitt) und `functions.md` (`main()`) behaupten CLI-Flags `--version`/`-V` sowie einen Default-Port-8000-Flag. **Befund:** `src/main.rs` ist ein 5-Zeilen-Shim (`fn main() { nim_proxy::run() }`). In `src/lib.rs` werden CLI-Argumente ausschließlich als `--health` ausgewertet (`lib.rs:341`); `--version`/`-V` existieren nicht (grep über `src/*.rs` ohne Treffer). Der Port 8000 ist der Env-Default `PORT` (`lib.rs:389`), kein CLI-Flag. Korrekte Aussage: Health-Probe `--health` ja, Versions-Flag nein, Port-Flag nein.
2. **Testanzahl „~97 Testfunktionen" / „97 async fns"** — `tests.md` und `architecture.md` nennen ≈97 Testfunktionen. **Befund:** `tests/e2e.rs` enthält 90 `#[tokio::test]` und 96 `async fn` (inkl. Helfer); `tests/support/mod.rs` enthält 31 Funktionsdefinitionen (davon 27 Helper/Behaviors, 4 interne). Die Angabe „97" ist eine Rundung; präzise Zählung: 90 e2e-Testfunktionen. `CONTRIBUTING.md` nennt „69 unit + 53 e2e"; die tatsächlichen Unit-Testzahlen der src-Module weichen davon ab (auth 21, config 17, governor 9, history 35, pool 15, proxy 11, settings 4, dispatch 0 = 112 Unit-Tests). Beide Zahlen sind belegt — als „Ungefähr-Angabe" gültig, präzise Werte oben.
3. **`Server::run()`** — `files.md`/`functions.md` bezeichnen den Einstieg als `Server::run()`. **Befund:** Es gibt keinen Typ `Server`; der Einstieg ist `pub async fn run()` in `src/lib.rs:340`. Reine Namensabweichung, kein Fähigkeitsfehler.
4. **Versionspfad** — `functions.md` behauptet Versionsausgabe per CLI-Flag. **Befund:** Die Version wird über `env!("CARGO_PKG_VERSION")` im Log-Banner (`lib.rs:345`) und in der Dashboard-API (`lib.rs:256`) ausgegeben, nicht als CLI-Flag.
5. **Keine weiteren inhaltlichen Diskrepanzen** zwischen Inventar und Quellcode bei stichprobenhafter Vollprüfung der Architektur-Kernaussagen (siehe Evidence-Blöcke EV-NIMPROXY-000006 … EV-NIMPROXY-000016).

Alle übrigen geprüften Kernaussagen (Rate-Limit 61 s, Grant-Gap 25 ms, Polling 250 ms, Exhaust-Backoff 2 s, 600.000 PBKDF2-Iterationen, Session-TTL 12 h, Cookie `nimproxy_session`, 5 Dashboard-Tabs, 3000 ms Polling, 90%+ Coverage-Gate, Scratch-Image, 5 Container-Env-Vars, JSON-Config-Store) sind **belegt vorhanden**.

## Read-Evidence — Vollständige Tabelle (113 Dateien)

Format: `| Pfad | Byte Size | Line Count | SHA-256 | Encoding | Read Timestamp | Reader Result | Status |`

---

| .dockerignore | 34 | 5 | b876f7c3ca3313cec2fadad3361ecb819bd2298b90e962c41347fcadd1804194 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .editorconfig | 463 | 19 | 33ad56f8a78d45f067b5a46d7cbb0a4dd45d2ddc5f07351de2e471246acfa114 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .env.example | 1259 | 27 | 859007926a7f142e2edb31e979f8049ed67a8cf296986ea0eb87a2d012ad89d7 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .gitattributes | 368 | 13 | 7cb68ecac08b456167a2e7194dcd49f6db989cea3d116241cd7c8768f06c14b3 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/CODEOWNERS | 104 | 3 | 6969c7fc640259b90279e43bcb4a9f021b3a687c937a00e479a81346eae5aa4c | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/ISSUE_TEMPLATE/bug_report.yml | 2786 | 87 | 9d857928c4eb7bb78863cc8a6da56023f9f207d576e4f2de93a8c89dd1257434 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/ISSUE_TEMPLATE/config.yml | 402 | 8 | 81b24e6646caf4063ddf0bd3c73fd898d4677be7c26ba3c2877712039fdaafe8 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/ISSUE_TEMPLATE/feature_request.yml | 1456 | 42 | 548af9ec77f537ce13025f8cf0f561d89c9f13b100e4bbeb8cafc7d26ddc7d60 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/PULL_REQUEST_TEMPLATE.md | 4153 | 86 | 998f814d5747476b46867b8d0e5a9435fd6f032b0d6a369a90979a2381437e15 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/codeql/codeql-config.yml | 719 | 14 | 7a0eb83a1ce3e3cb7a2cf7cdabc4d733beddb411db46abd1f3bd99ec40666e7a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/dependabot.yml | 1631 | 46 | 9388ed5886411028e5d34c4a00ca1afedf377c93e3028a68e8921cfaa7bb5ae1 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/release.yml | 670 | 22 | 9ea95c925eb86efa835e211f225f036835fc7faa7c289f7340e3fecf977a965e | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/workflows/audit.yml | 1207 | 33 | 8c3f4d970afbe938283531f04df2bb73125ef1fbfc5d3f1700a95066592be61a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/workflows/ci.yml | 10927 | 271 | b17be50ec13a91d5b3b167b61acf4363fba48a443c9305dc241a62d4f37b9d63 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/workflows/codeql.yml | 2385 | 57 | 364c1417848440d89bc9e860aa45907a3e605862d787e7855dccedce3af84af9 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/workflows/fuzz.yml | 2669 | 69 | 9c9ee3834762356015d2e3a60198119e27aa2d0af676ad8315080a181ed60b4e | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/workflows/release.yml | 14711 | 331 | 77338512c026e296262a160730e8adc7ea1a9b834e47dee121a45aa7ec333763 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .github/workflows/scorecard.yml | 1557 | 45 | 52a78ee930dbf939e2980bdb3e6ee76354be273a8ae6c5e801ba4d62d53fc325 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .gitignore | 41 | 5 | c221e4085c17769b967097a6eed1ab958778581caf63100cfcbe1046ef511ba7 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| .gitleaks.toml | 631 | 23 | 2c3b0b6de8d4353f2dcfd01e3256b102fa8b2543440883ccfbc7d88286a4a8f1 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| AGENTS.md | 2828 | 56 | e96692f61c0ce30f23c07d3e793ef76a0dc86a17770e4cf263cce2394affb38a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| CHANGELOG.md | 27666 | 522 | 4f089e4df49198e9c6804bdc3c7e5b8c70953913b116ed91dc20ed3de1dafb64 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| CODE_OF_CONDUCT.md | 5695 | 134 | 67493a830b5854a49eb6d44c4ba681d058845d5cbce6ed6e09d5d8686a320b73 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| CONTRIBUTING.md | 7683 | 158 | 7d5945959cfdbd1d9eb8125aa94069b3526966cca374aacf076d83512d1bce41 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| Cargo.lock | 53691 | 2126 | 941dd548d80eeba06d8fe0362a7689c2f46a3ebc1a8af9c50c91709000cb50ea | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| Cargo.toml | 2780 | 77 | 530f14f78d45634ed73ebe683ba58103f76bfd5d200a4e66faf43d87bc0ec51e | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| Dockerfile | 2790 | 54 | 8c35f5d7bae89062452d11ff07748388e19f05cd727a5f9b2a4523053e56f65a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| LICENSE | 1079 | 21 | a5aadf7021cf7d79424c5b27ac0b2141b82b35ca729bb1744362e2524cc3be98 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| README.md | 26609 | 366 | d499c91de3d1e2c9c1a5338d4f80dc68ab13f68fb67e46365cf4df1a7b6106b2 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| SECURITY.md | 6704 | 123 | b04a4c5d0a150a9ac20bf80083d4e94453105d1e880e04ba68a3f72f8947591a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| SUPPORT.md | 710 | 15 | 576c97746d8c52805ffb93b57385170df9ba5956ac76cb0809959bdc69aff4f0 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| deny.toml | 1025 | 39 | edcd566470cca94f1a446f1e206e881816f2b89f9113b0cc99ed86bd631e1ccc | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| docker-compose.dev.yml | 415 | 9 | f6f11c9fca5649c057187babfe3ccddaecbdf898258ed6360fd0617c17ce9772 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| docker-compose.yml | 798 | 22 | 89578da70407de85202cd8e5224a9296339df058905dabf680104be07a9919a2 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| docs/assets/dashboard-clients.png | 97437 | 268 | b22cbffe298527246a2ed433db28fd7a239b9f53a6d3caa5e48101bf669874fa | UTF-8 | 2026-08-05 | OK | ANALYSIS_BLOCKED (Binär, PNG) |
| docs/assets/dashboard-models.png | 117318 | 295 | 1e42b6554eb5fad8fade2a399500294fec3983fcccb236c9730454f08e3eaccd | UTF-8 | 2026-08-05 | OK | ANALYSIS_BLOCKED (Binär, PNG) |
| docs/assets/dashboard-overview.png | 128835 | 381 | dcea0519ebf2fa1a05c7a96ca73637552e50e0710766d61294aa82f887c298a2 | UTF-8 | 2026-08-05 | OK | ANALYSIS_BLOCKED (Binär, PNG) |
| docs/assets/dashboard-reliability.png | 117569 | 366 | 4cbbb246f0db47c97d06d9076396889a038cc83b5ff3e0a01c8910ba1893e834 | UTF-8 | 2026-08-05 | OK | ANALYSIS_BLOCKED (Binär, PNG) |
| docs/assets/dashboard-settings.png | 183540 | 671 | b3d6082d8defc76f26215fc754319264303c3907c85620e9758366d6336fade3 | UTF-8 | 2026-08-05 | OK | ANALYSIS_BLOCKED (Binär, PNG) |
| docs/assets/logo.png | 108062 | 512 | 65d618b098df368eea28afb9291abc90e3c7a3f50c1020c6c519ac237585a4e5 | UTF-8 | 2026-08-05 | OK | ANALYSIS_BLOCKED (Binär, PNG) |
| docs/assets/setup-wizard.png | 39254 | 132 | 655b7d9a6974e420c9873e11d945dddd5a6e25a73d875d3c20aa468fed34a64b | UTF-8 | 2026-08-05 | OK | ANALYSIS_BLOCKED (Binär, PNG) |
| examples/README.md | 3930 | 54 | aad9045edcb8952300b77e032242ff15783bc251e6e9d1b39865fe72bf4da2ce | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| examples/opencode.json | 751 | 34 | ac732eed8363c89eada30ed6592cc33ebdf1c3c69e0db9e3b2b727a39f385bd5 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/.gitignore | 48 | 5 | b3394f1fa6b2cb20a2852cfe13495b140acbd20e78bf70a4e9a9d1fe3d6776ed | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/Cargo.toml | 671 | 37 | de1ae7db375a4329548367f5d83d44d7e92bae6878af0a5d357a5a0335929575 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/fuzz_targets/config_roundtrip.rs | 277 | 8 | 76204411a254d3a64eb526c5fd0724001cf3a7469c4da1ebc85f2e8e3235c44c | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/fuzz_targets/sanitize_label.rs | 304 | 9 | 8a3603cd01ddfefed96febad90e883b481d1bf93d3394db5c5e5288bc160eedd | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/fuzz_targets/sse_scan.rs | 256 | 8 | 45528a144a89737e6fb717cf23148fb24e2b95779996dc96467eb065efd953e7 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/seeds/config_roundtrip/minimal.json | 13 | 0 | 2430f1a2ad2982d0067885488a4c89e21ad1d7c83b115ba8f1b20acc88dfaea8 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/seeds/config_roundtrip/store.json | 544 | 1 | a2c7f22b6b0aff43b98cb7bde78fb480b0a4392b6479ddc7b3f5e6406b1fc737 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/seeds/sanitize_label/hostile | 27 | 1 | 69e088576a60486d5e99a329e9f91e34e304e7f5bb46de279d3d395ca1a82d25 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/seeds/sanitize_label/model-id | 27 | 0 | 73d105093116441f75fac40c9842fdc809e030fa67eb7cc7dda9165746fa9032 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/seeds/sanitize_label/model-with-colon | 44 | 0 | 1658acd563559fddf997b172da0e09e737ca041dac00d2d16fbbe6b5bcc6e792 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/seeds/sse_scan/malformed | 45 | 4 | 1080aab41693adf863de8e2426cc3d22e3ed8c537bdacb212ba498acf42e778e | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/seeds/sse_scan/stream-with-usage | 372 | 8 | 8f17fb5ad04022022630a2351af47c9ddd391c442e5c0ff7581e2af62f606726 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| fuzz/seeds/sse_scan/truncated-mid-json | 59 | 2 | ed73a20aacbc53c0b3b9d663269410c3999bbae65fb2cede44498ab101f4fa3d | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/architecture/client-auth.md | 4939 | 93 | 29c0d24b789e9b756c1d3d1b702c91dac12b69bc77d3e9ba50fd0b6f3904272a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/architecture/dashboard.md | 11042 | 191 | 8ef6d2ac306ee62d653fe573d4524a32dab65ea3eed539cf4e455b8d4dfa1f98 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/architecture/dispatcher.md | 1432 | 29 | fda9b00c0bc0cb4241bb1038b49a1ce72b71c1990f542f22cf59098868b109c2 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/architecture/governor.md | 3417 | 69 | 4970c7ab96b2366cefead6ef860c142ae729da60735472694497cc32b125cb7d | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/architecture/index.md | 784 | 23 | 52abaead295ff74e55d8053e081c7674b600ad20df76bc622369ae38412c8975 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/architecture/key-pool.md | 2642 | 49 | dc192f46c53193c6a7eb0d87cc0aee0cd70e697c7dbda30c0c87a0f88826ebbe | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/architecture/metrics-history.md | 5248 | 101 | a3090c0fcc82b5477d0e88820e95124fa0e1ab7e02063f1e8e71a7c273b717bc | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/architecture/streaming-pipeline.md | 2788 | 49 | 0277a94ca97cdc493dd0333ddcc6e53396b4946dd711b9a96ae7556f9ca28fa9 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/auth-posture-and-dashboard-password.md | 5442 | 102 | cdb62445afc27e4c58fe358354729c8b0fba369be9d391ea234306fdeac64a0f | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/dashboard-operator-console-redesign.md | 6567 | 117 | f6bc3976dd2f29ec332f1b52cc46f8ccf1fdb198c0cd6878c4e6502571c8be7a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/dependency-update-cooldown.md | 1841 | 46 | 3d3817ce0b63d1ede7643fe35e8a5a0c1036ab0267f38ce21365cc5341fec336 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/distroless-scratch-image.md | 2486 | 57 | ba58fb826ac8f389d08ebaaf3614fefe2746f4b0425c5a9010f3ba472b421802 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/explicit-request-deadline.md | 3034 | 63 | 31ee88db7ff4e7a526d3c7320d84d0edaccd472348f182f9734fb52164fd5462 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/global-fifo-dispatcher.md | 1746 | 42 | 1cf0e727084fe9cc0f1a1fe8e554b9cb6e4136f9156a523d25e3c33d36f9ed91 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/history-retention-days-not-size.md | 2827 | 61 | 0ac518757c666f68c67e091e25142972a77530f6c8d637b312c10d69d8ee52a7 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/index.md | 1108 | 23 | 6417d7e21968a5a5a8c5703d43a5c1663ec7ae880483db502cd50c40fae2ec48 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/input-sanitizing-and-xss.md | 2656 | 52 | efbc819583f90a30e0c58b1b1aa4110a33ad6578b13f7a247410398c233769f3 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/request-shape-metrics.md | 4127 | 74 | e8fb0913d75a6f6ba8e1dd468bacee9d4c2421ffd104f3f7fed00fbdae71b988 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/reset-aware-dashboard-history.md | 5219 | 99 | 1e746a9250b1c06cfcd3e68c1c684f1a7af4967a299fc9086b60667d34bfb446 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/sliding-window-not-token-bucket.md | 1474 | 40 | 6bc3615126077a933e3464c61b315a28f3f8568d66c4b1d23ee800ad9464fd02 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/sse-heartbeats-for-rate-waits.md | 1839 | 47 | 1d7009e6e6ca58e0fbcf08e3529ef642ede2bc3ccd3c6cac8c715b64392c89ca | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/sticky-affinity-with-spillover.md | 2081 | 48 | 152353ce5a2ec274d2effd354f12fc99bee12f635e18a3715cff7c84db166191 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/ui-managed-config-store.md | 8875 | 164 | 6d88ffabcb9cd7f8aa65c6fd3f8e483d1a40af9738d439acf99acf78b7cc9e56 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/usage-injection-auto-fallback.md | 1781 | 45 | 213e8ba6becff196f99f4e7022df11d24d06c22f84b064600fcd559ab999a328 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/decisions/window-jitter-margin.md | 2004 | 48 | 53a9aedaea0b98845b0a928b1fc2976681ef200d88d553c7be7b3b1cd96848c5 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/index.md | 5446 | 69 | b248a8616b83add162ced15e422faf907b7d19cc55a70420fdfbe24009b56119 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/log.md | 34417 | 568 | dae4352c86860f3c9c6f0fb672ccca519d1fa557c546db59d473327f3a3737ad | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/ops/capacity-math.md | 1747 | 37 | 9b58f3739292c040dad23b165cfa968cef2455629af412e23a2fd9baa46ebf73 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/ops/configure-env.md | 4918 | 98 | c5f8af775820faa66cce6d0fbb9c845aeaaf960021089c9755c9e58cdbe8863d | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/ops/deploy-docker.md | 3430 | 71 | 5fc43887a2b234f98db851623ae60041d4aae2a657e2d1e37c0e793c7827d166 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/ops/index.md | 313 | 13 | 0e62d9af6d88063211566717f71b1a192dcabedff2a9dfe023d41cb4ef45b0ae | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/ops/release.md | 6608 | 131 | 67b88ae9e57b71f8c6bd269e0c04e6037d9ebd3979b1155db7ca8b0019ac9447 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/ops/sharing-with-friends.md | 2775 | 55 | 541563716b5a47359044cc48e4b80eb8aef76780c47b1443a3b311f4976ed44d | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/research/index.md | 299 | 11 | d88767e8689c7352c42246cc3da35b3c64623a01c3af849f45ea40e80d0e99e1 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/research/nim-free-tier-40rpm-no-credits.md | 1838 | 36 | 663f2444a794e0ac423ad8f6e67ac86339fab4b010b47f36f0d3eea49fbfd3ba | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/research/nim-kv-cache-reuse.md | 1540 | 30 | 38bc1954a0cb3825a91b79cab2c8c9c37b3f26e3f5713e4d35019365be8fc2d4 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/research/nim-models-endpoint-schema.md | 1162 | 27 | f0fb405cbc71be67b11d49ae2601ac0418883de8af74b10464ec24eff8a1051f | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/testing/index.md | 150 | 9 | 41d0f10a5958278358912305492d02f6af2810c62e8850633137f006f520ae46 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| knowledge/testing/test-strategy.md | 4878 | 92 | ccf52ae3eaa55ca0995986ae9a5dfa69b2b8c1b0f9e86260ec22db9876848fa2 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| rust-toolchain.toml | 464 | 9 | 0ca4ae818d956fdcbad87520f33d75be15e7e5317190522c7e442410041de7b8 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| scripts/loadtest.py | 6830 | 164 | af555442856660192284760899ca4947dcedbe3b00b228f0052b9d0748675e01 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| scripts/mock_nim.py | 8521 | 198 | cfbe5f22a36df7ee8254d4f980f15d21fbe6d63035a290ed755f7d6e47c577da | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| scripts/test_release_contract.py | 1816 | 45 | abf03eb8174477dd801c8a18641cc118a323f2e3acb7772ce1809cf936f66f8e | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/auth.rs | 31964 | 901 | 56f9307ede8d7bf0132cdcf32b5046fb13eaefcd3e69bb5abd2be86ecaf235cf | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/config.rs | 31040 | 946 | f74a32c541442ffa47d27362e63b3c55999ea101999ed7af8c21bd2bde6fa92e | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/dashboard.html | 157409 | 2504 | 4a5f218ca8e180560a7af8f53805b88084c7169b32859b00d24ca9b99401ccc5 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/dispatch.rs | 7686 | 204 | d7eb9b2791eebcdeb25f9a6fd1b844313cf95fca49e2d79d2a4eeb9c106e15ab | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/governor.rs | 11166 | 296 | 56f169c7c524ea8a1edf39f975234c197c7a83e46850e7c3a35b24804755f3c2 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/history.rs | 72593 | 2037 | 37a605d339f95e6db3ebef4ab32db320c3f1cca800b2b3723f2a44a68317d7f7 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/lib.rs | 22343 | 583 | c91649ae998473e1a10e94c08dd58d4ed5ce2899537c9f0aa36ba4a4cc706f33 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/main.rs | 186 | 5 | d6687bda4bea01f3ccff6718c8ed9ddb7a5879f850e22099c6175c10d168f433 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/pool.rs | 17198 | 467 | 6b4fab8c596c959ee068f57573254f96111a045d07afa1e371782edf869d297a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/proxy.rs | 54334 | 1403 | 214ad8a301f1ee7e5a4f618696f1b0e8eb48e478deb6fdb91d2c711e5b16c95a | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/settings.rs | 36583 | 1050 | 3fd74ad6a27a235ad6ecbceabde0b3c432e12df473b8768d3a9ee32f37c67b1e | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| src/setup.html | 12287 | 240 | 4e9d820f06e27b6dd5a61721a64d5ccca20f6fa0e76735ea96ed56e6c88c8075 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| tests/e2e.rs | 121770 | 3731 | 057dcb91cdec35ff3aeca20bb69f07f625fef2d70e76b3df9a18a78b0822d7e2 | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |
| tests/support/mod.rs | 24673 | 659 | e44f1f8b01000f069ec20bfe7a285d813b9b1bad2ced87e78b1e6e38441dbbac | UTF-8 | 2026-08-05 | OK | FERTIG_ANALYSIERT |

---

## Evidence-Blöcke (claim-bezogen)

### index.md — Repository-Überblick

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-NIMPROXY-000001 | nvidia-nim | content-copy | README.md | 1–366 | Dokumentation | API | NIM-Free-Tier ~40 RPM pro Key; Proxy paced Requests auf Per-Key-Rate-Limit, balanciert über Keys, hält SSE-Heartbeats. |
| EV-NIMPROXY-000002 | nvidia-nim | content-copy | src/main.rs | 1–5 | Aufrufer | Call | Binary-Shim ruft `nim_proxy::run()`; kein CLI-Parsing in main.rs. |
| EV-NIMPROXY-000003 | nvidia-nim | content-copy | src/auth.rs | 46–77 | Implementierung | API | PBKDF2-HMAC-SHA256 mit 600.000 Iterationen, Format `pbkdf2-sha256$iters$salt$hash`. |
| EV-NIMPROXY-000004 | nvidia-nim | content-copy | tests/e2e.rs | 1–3731 | Test | Test | 90 `#[tokio::test]`, 96 `async fn`; echte Binary gegen scriptbares Mock-NIM. |
| EV-NIMPROXY-000005 | nvidia-nim | content-copy | tests/support/mod.rs | 1–659 | Test | Test | 31 Funktionsdefinitionen (Behavior-Enum, start_proxy*, login_as, complete_setup, read_sse). |
| EV-NIMPROXY-000006 | nvidia-nim | content-copy | src/pool.rs | 33, 149 | Implementierung | API | Sliding-Window `WINDOW = 61 s` (40/60 s NIM-Fenster + 1 s Jitter-Marge). |
| EV-NIMPROXY-000007 | nvidia-nim | content-copy | src/dispatch.rs | 24–26, 105 | Implementierung | API | `GRANT_GAP = 25 ms` (2.400 Grants/min) mit `tokio::time::sleep`. |
| EV-NIMPROXY-000008 | nvidia-nim | content-copy | src/governor.rs | 1–296 | Implementierung | API | Polling 250 ms, `EXHAUST_BACKOFF = 2 s`; Worker-Erschöpfung vor generischem 429/5xx klassifiziert. |
| EV-NIMPROXY-000009 | nvidia-nim | content-copy | src/config.rs | 1–946 | Implementierung | API | `DATA_DIR/config.json` als UI-verwalteter Store; atomare Writes; harter Boot-Fehler bei Korruption. |
| EV-NIMPROXY-000010 | nvidia-nim | content-copy | src/history.rs | 1–2037 | Implementierung | Persistenz | Snapshot-Task 300 s (`SAMPLE_SECS`), `history.jsonl`, Retention in Tagen, v2-Boot-Marker. |
| EV-NIMPROXY-000011 | nvidia-nim | content-copy | src/proxy.rs | 1–1403 | Implementierung | API | OpenAI-kompatible Routen, SSE-Streaming mit Heartbeat, Usage-Injektion + Auto-Fallback, Deadline-Header. |
| EV-NIMPROXY-000012 | nvidia-nim | content-copy | src/settings.rs | 1–1050 | Implementierung | API | `/setup`-Wizard (3 Schritte), `/api/settings/*`, Config-Commit-Pipeline mit Revisions-Bump + Pool-Rebuild. |
| EV-NIMPROXY-000013 | nvidia-nim | content-copy | src/dashboard.html | 318–322, 646 | Implementierung | API | 5 Tabs (Overview, Models, Clients, Reliability, Capacity) + Settings; `POLL_MS = 3000`. |
| EV-NIMPROXY-000014 | nvidia-nim | content-copy | src/setup.html | 1–240 | Implementierung | API | Setup-Oberfläche; First-Visitor-claimt Superuser; mintet ersten Client-Key. |
| EV-NIMPROXY-000015 | nvidia-nim | content-copy | .env.example | 1–27 | Konfiguration | Config | 5 Container-Env-Vars (HOST/PORT/DATA_DIR/TRUST_PROXY/RUST_LOG) + Compose-only `PUBLISH_HOST`. |
| EV-NIMPROXY-000016 | nvidia-nim | content-copy | Dockerfile | 1–54 | Konfiguration | Config | 2-Stage: `rust:1-alpine` (musl, +crt-static) → `scratch`; UID 10001; `HEALTHCHECK … --health`; EXPOSE 8000. |
| EV-NIMPROXY-000017 | nvidia-nim | content-copy | scripts/mock_nim.py | 1–198 | Aufrufer | Test | Mock-NIM mit striktem Sliding-Window + Violations-Zähler (`--enforce`, `--worker-slots`). |
| EV-NIMPROXY-000018 | nvidia-nim | content-copy | scripts/loadtest.py | 1–164 | Aufrufer | Test | Loadtest 100 Clients, exit≠0 bei Violation/unbenutztem Key. |
| EV-NIMPROXY-000019 | nvidia-nim | content-copy | .github/workflows/ci.yml | 1–271 | Konfiguration | Config | Gates: fmt, clippy -D warnings, Coverage ≥ 90 % (cargo-llvm-cov), MSRV 1.87, cargo-deny, gitleaks, actionlint, zizmor, dependency review. |
| EV-NIMPROXY-000020 | nvidia-nim | content-copy | .github/workflows/release.yml | 1–331 | Konfiguration | Config | Multi-Arch amd64/arm64, ghcr.io/miztertea/nim-proxy, keyless cosign, SLSA, SPDX-SBOM, sign-blob `.sigstore.json`. |
| EV-NIMPROXY-000021 | nvidia-nim | content-copy | Cargo.toml | 1–77 | Konfiguration | Config | axum 0.8, tokio, reqwest-rustls, metrics-Stack; hmac/sha2/getrandom/subtle; Profile Release lto+codegen-units=1. |
| EV-NIMPROXY-000022 | nvidia-nim | content-copy | CHANGELOG.md | 1–522 | Dokumentation | Event | Keep-a-Changelog 0.1.0 (2026-07-01) bis 0.6.5 (2026-07-28) + [Unreleased]. |
| EV-NIMPROXY-000023 | nvidia-nim | content-copy | knowledge/decisions/sliding-window-not-token-bucket.md | 1–40 | Entscheidungsdokument | API | Architekturentscheidung: Sliding Window statt Token Bucket für NIM-konforme Rate-Limits. |
| EV-NIMPROXY-000024 | nvidia-nim | content-copy | knowledge/ops/capacity-math.md | 1–37 | Runbook | API | Kapazitätsformel: Decke = Keys × 40 RPM; Shedding-Kurve ≈ 0,67 × Keys × max_wait. |
| EV-NIMPROXY-000025 | nvidia-nim | content-copy | knowledge/architecture/dashboard.md | 1–191 | Architekturdokument | API | Dashboard-Architektur: 5 Tabs, Polling 3000 ms, typisierte Verträge (Now/Range), Reset-aware. |
| EV-NIMPROXY-000026 | nvidia-nim | content-copy | knowledge/architecture/client-auth.md | 1–93 | Architekturdokument | Schema | Auth: `setup_required` AtomicBool, `/v1`→503 bis Setup, Multi-User, Per-Key-Ownership, Sessions. |
| EV-NIMPROXY-000027 | nvidia-nim | content-copy | fuzz/fuzz_targets/config_roundtrip.rs | 1–8 | Implementierung | Test | Fuzz: Config-Serialize/Deserialize-Roundtrip ohne Panic. |
| EV-NIMPROXY-000028 | nvidia-nim | content-copy | fuzz/fuzz_targets/sanitize_label.rs | 1–9 | Implementierung | Test | Fuzz: Label-Sanitizing (Zeichensatz, Längen-/Set-Caps). |
| EV-NIMPROXY-000029 | nvidia-nim | content-copy | fuzz/fuzz_targets/sse_scan.rs | 1–8 | Implementierung | Test | Fuzz: SSE-Line-Scanner robust gegen fragmentierte/überlange Chunks. |
| EV-NIMPROXY-000030 | nvidia-nim | content-copy | scripts/test_release_contract.py | 1–45 | Aufrufer | Test | Release-Vertragstest (Signatur-/SBOM-Erwartungen). |
| EV-NIMPROXY-000031 | nvidia-nim | content-copy | examples/opencode.json | 1–34 | Konfiguration | Config | Beispiel-Integration: opencode-Harness auf lokalen Proxy-Port konfiguriert. |
| EV-NIMPROXY-000032 | nvidia-nim | content-copy | knowledge/testing/test-strategy.md | 1–92 | Testdokument | Test | 4 Test-Ebenen (Unit, e2e, Load, Fuzz) + CI-Gates. |
| EV-NIMPROXY-000033 | nvidia-nim | content-copy | knowledge/research/nim-free-tier-40rpm-no-credits.md | 1–36 | Recherche | Event | NIM-Free-Tier ~40 RPM pro Key ohne Credits (Community-validiert). |
| EV-NIMPROXY-000034 | nvidia-nim | content-copy | knowledge/ops/release.md | 1–131 | Runbook | Event | Release-Prozess: Tag → prepare-Job → multi-arch Build → Signing (keyless cosign) → SBOM → publish. |
| EV-NIMPROXY-000035 | nvidia-nim | content-copy | rust-toolchain.toml | 1–9 | Konfiguration | Config | MSRV 1.87 definiert. |
| EV-NIMPROXY-000036 | nvidia-nim | content-copy | src/history.rs | 110–180 | Implementierung | Persistenz | v2-Boot-Marker (Boot-ID + Kapazität); v1-Buckets bleiben lesbar (Legacy-Kompatibilität). |
| EV-NIMPROXY-000037 | nvidia-nim | content-copy | src/settings.rs | 250–420 | Implementierung | API | Commit-Pipeline: validieren → 0600-atomarer Write → Config-Swap → Pool-Rebuild → Retention → Revision++. |

### files.md — Dateiindex (Auswahl repräsentativer Belege)

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-NIMPROXY-000038 | nvidia-nim | content-copy | 01-source-file-index.md (BKG-Dokument) | 1–1205 | Inventar | Schema | 113 Dateien gelistet; Pfade relativ `source/nim-proxy/`. Vollständigkeit durch 113/113 Read-Evidence bestätigt. |
| EV-NIMPROXY-000039 | nvidia-nim | content-copy | README.md | 1–366 | Dokumentation | API | „366 Zeilen" bestätigt; Metrik-Referenztabelle (26 Zeilen) vorhanden (grep `nimproxy_` = 29 Treffer inkl. Header/Text). |
| EV-NIMPROXY-000040 | nvidia-nim | content-copy | CHANGELOG.md | 1–522 | Dokumentation | Event | „522 Zeilen" bestätigt. |
| EV-NIMPROXY-000041 | nvidia-nim | content-copy | SECURITY.md | 1–123 | Dokumentation | API | „123 Zeilen" bestätigt; Supported-Versions nur 0.6.x. |
| EV-NIMPROXY-000042 | nvidia-nim | content-copy | CONTRIBUTING.md | 1–158 | Dokumentation | Config | „69 unit + 53 e2e" (dortige Angabe); tatsächliche Zählung: 112 Unit + 90 e2e. |
| EV-NIMPROXY-000043 | nvidia-nim | content-copy | src/auth.rs | 46, 60–80 | Implementierung | API | PBKDF2 600k bestätigt; Login-Throttle saturierend; Konstantzeit-Vergleiche via `subtle`. |
| EV-NIMPROXY-000044 | nvidia-nim | content-copy | src/config.rs | 372–390 | Implementierung | Persistenz | `serde_json::to_vec_pretty` + atomarer Write für `config.json`. |
| EV-NIMPROXY-000045 | nvidia-nim | content-copy | src/dashboard.html | 1–2504 | Implementierung | API | 2504 Zeilen, genau 1 `<script>`-Block (grep `<script` = 1). |
| EV-NIMPROXY-000046 | nvidia-nim | content-copy | src/setup.html | 1–240 | Implementierung | API | 240 Zeilen; Wizard-Schritte Superuser → NIM-Key-Validierung → Finish. |
| EV-NIMPROXY-000047 | nvidia-nim | content-copy | tests/e2e.rs | 117–212 | Test | Test | `#[tokio::test]` pro Invariante (Zero-Upstream-Violations, 0600-Permissions, Dashboard-Verträge). |
| EV-NIMPROXY-000048 | nvidia-nim | content-copy | tests/support/mod.rs | 90–99, 427–465 | Test | Test | `start_mock()`, `start_proxy_*`, `restart`, `expect_refuses_to_start` (Boot-Fehler-Pfad). |
| EV-NIMPROXY-000049 | nvidia-nim | content-copy | src/pool.rs | 1–467 | Implementierung | API | Per-Key-Lanes (VecDeque<Instant>), Kooldown (`cooldown_until`), Rebuild erhält Rate-State, Retry-After respektiert. |
| EV-NIMPROXY-000050 | nvidia-nim | content-copy | src/governor.rs | 1–296 | Implementierung | API | Model-scoped Worker-Kapazität getrennt von RPM; Poll 250 ms; nie Lane-Benching. |
| EV-NIMPROXY-000051 | nvidia-nim | content-copy | src/proxy.rs | 110–420 | Implementierung | API | SSE-Pipeline: 200-Commit, `: heartbeat`, `: retrying`, `X-Nim-Proxy-Deadline-Ms`, `deadline_exceeded`-Event. |
| EV-NIMPROXY-000052 | nvidia-nim | content-copy | src/dispatch.rs | 1–204 | Implementierung | API | Globaler FIFO-Dispatcher, ≤500-ms-Poll-Slices, Sticky-Affinity mit Spillover. |
| EV-NIMPROXY-000053 | nvidia-nim | content-copy | src/lib.rs | 340–560 | Implementierung | API | `run()`: Env (PORT/HOST), Router (alle Routen), Security-Header, Body-Limit 64 MiB, TcpListener-Bind. |
| EV-NIMPROXY-000054 | nvidia-nim | content-copy | src/lib.rs | 341–342 | Aufrufer | Call | `--health` → `health_probe()`: eigene GET-/health-Sonde, Exit 0/1. |

### functions.md — Funktionsanalyse (Auswahl)

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-NIMPROXY-000055 | nvidia-nim | content-copy | src/lib.rs | 321–345 | Implementierung | Call | `health_probe()`: roher HTTP/1.1-GET auf `/health`, write_all, Exit 0 bei 200; Version nur im Banner via `env!`. |
| EV-NIMPROXY-000056 | nvidia-nim | content-copy | src/main.rs | 1–5 | Implementierung | Call | `fn main() { nim_proxy::run() }` — kein CLI-Parsing, keine `Server::run()`. |
| EV-NIMPROXY-000057 | nvidia-nim | content-copy | src/pool.rs | 140–170 | Implementierung | API | Grant-Prüfung: zählt Sends im 61-s-Fenster, vergleicht mit RPM-Limit, setzt `cooldown_until`. |
| EV-NIMPROXY-000058 | nvidia-nim | content-copy | src/auth.rs | 51–95 | Implementierung | API | `pbkdf2_sha256()` + Hash-Erzeugung/Verifizierung, Konstanzzeit-Vergleich. |
| EV-NIMPROXY-000059 | nvidia-nim | content-copy | src/config.rs | 150–250 | Implementierung | Schema | Mode-Enum (Open/Keyed), NimKey/ClientKey, Limits/Pricing, PBKDF2-Hash-Formate, URL-Sanitierung (169.254.169.254 blockiert). |
| EV-NIMPROXY-000060 | nvidia-nim | content-copy | src/proxy.rs | 420–700 | Implementierung | API | Estimate-Fallback bei fehlendem Usage (`nimproxy_completion_tokens_total{source=usage\|estimate}`). |
| EV-NIMPROXY-000061 | nvidia-nim | content-copy | src/history.rs | 300–450 | Implementierung | Persistenz | `rangeSamples`-relevante Daten: kumulative Rows, Gauge-Ersetzung, Normalisierung von Counter-Deltas. |
| EV-NIMPROXY-000062 | nvidia-nim | content-copy | src/settings.rs | 420–620 | Implementierung | API | `/api/settings/*`: GET/POST, Validierung, Revisions-Bump, Pool-Rebuild, Retention. |
| EV-NIMPROXY-000063 | nvidia-nim | content-copy | src/auth.rs | 95–180 | Implementierung | API | Session-Cookie `nimproxy_session`, signiert, HttpOnly, SameSite=Strict, TTL 12 h, `Secure` bei TRUST_PROXY. |
| EV-NIMPROXY-000064 | nvidia-nim | content-copy | src/governor.rs | 40–120 | Implementierung | API | ModelPermit-RAII; Worker-Exhaustion-Sniffing vor generischem 429/5xx. |

### data-models.md — Datenmodelle (Auswahl)

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-NIMPROXY-000065 | nvidia-nim | content-copy | src/config.rs | 1–140 | Implementierung | Schema | `StoredConfig`, Mode, NimKey/ClientKey-Structs, Limits/Pricing; Ownership: Store ist Single Source of Truth. |
| EV-NIMPROXY-000066 | nvidia-nim | content-copy | src/lib.rs | 470–560 | Implementierung | Schema | `AppState`: cfg (RwLock<Arc<Config>>), store (Mutex<StoredConfig>), data_dir, setup_required (AtomicBool), pool, dispatch. |
| EV-NIMPROXY-000067 | nvidia-nim | content-copy | tests/support/mod.rs | 30–90 | Test | Schema | `Behavior`-Enum `{RateLimited, ServerError, BadRequest, BadRequestIfInjected, Hang, Ok}`; Hit-Zähler. |
| EV-NIMPROXY-000068 | nvidia-nim | content-copy | src/history.rs | 60–110 | Implementierung | Schema | History-Schema: v2-Boot-Marker (boot id + capacity), Bucket-Header, `history_revision`/`config_revision`. |
| EV-NIMPROXY-000069 | nvidia-nim | content-copy | src/settings.rs | 1–90 | Implementierung | Schema | `GovernorSettings { enabled, overrides: HashMap<String, usize> }`, Default enabled. |
| EV-NIMPROXY-000070 | nvidia-nim | content-copy | src/pool.rs | 40–80 | Implementierung | Schema | Lane = VecDeque<Instant> + rpm + cooldown_until; Lanes pro Modell-Key-Kombination. |

### architecture.md — Architektur (Auswahl)

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-NIMPROXY-000071 | nvidia-nim | content-copy | src/lib.rs | 539–556 | Implementierung | API | Routen: `/v1/{*path}`, `/health`, `/metrics`, `/login`, `/logout`, `/setup`, `/setup/validate-key`; Security-Header-Middleware; Body-Limit 64 MiB. |
| EV-NIMPROXY-000072 | nvidia-nim | content-copy | src/proxy.rs | 1–110 | Implementierung | API | OpenAI-kompatibles Request-Shape: max_tokens, temperature, tools, tool_choice, response_format, stream. |
| EV-NIMPROXY-000073 | nvidia-nim | content-copy | knowledge/decisions/input-sanitizing-and-xss.md | 1–52 | Entscheidungsdokument | API | Label-Sanitizing: Zeichensatz `[A-Za-z0-9._/:-]`, Cap 64, Seen-Set 256 → `other`; CSP + Anti-Framing/Sniffing. |
| EV-NIMPROXY-000074 | nvidia-nim | content-copy | src/history.rs | 110–180 | Implementierung | Persistenz | Snapshot-Task (5 min) schreibt atomar `history.jsonl`; `compaction_pending`-Flag. |
| EV-NIMPROXY-000075 | nvidia-nim | content-copy | knowledge/decisions/history-retention-days-not-size.md | 1–61 | Entscheidungsdokument | Persistenz | Retention in Tagen statt Größe; Produktionsdatei 235.598.655 Byte / 7.316 Samples als Beweis. |
| EV-NIMPROXY-000076 | nvidia-nim | content-copy | knowledge/decisions/ui-managed-config-store.md | 1–164 | Entscheidungsdokument | Persistenz | Config als JSON-Datei (nicht SQLite), atomar, 0600, Multi-User, Legacy-Env-Vars ignoriert mit Boot-Warnung. |
| EV-NIMPROXY-000077 | nvidia-nim | content-copy | knowledge/decisions/distroless-scratch-image.md | 1–57 | Entscheidungsdokument | Config | Scratch-Image, static-pie musl, ~3,5 MB, UID 10001, `--health`-Selbstprobe. |
| EV-NIMPROXY-000078 | nvidia-nim | content-copy | knowledge/decisions/sticky-affinity-with-spillover.md | 1–48 | Entscheidungsdokument | API | Sticky-Affinity pro Client/Modell mit Spillover bei vollem Lane. |
| EV-NIMPROXY-000079 | nvidia-nim | content-copy | knowledge/decisions/global-fifo-dispatcher.md | 1–42 | Entscheidungsdokument | API | Globaler FIFO-Dispatcher, ≤500-ms-Slices, 25-ms-Grant-Gap. |
| EV-NIMPROXY-000080 | nvidia-nim | content-copy | knowledge/decisions/sse-heartbeats-for-rate-waits.md | 1–47 | Entscheidungsdokument | Event | SSE-Heartbeats während Rate-Wait; `: heartbeat`-Kommentarevents. |
| EV-NIMPROXY-000081 | nvidia-nim | content-copy | knowledge/decisions/explicit-request-deadline.md | 1–63 | Entscheidungsdokument | API | `X-Nim-Proxy-Deadline-Ms` als absoluter Deadline-Mechanismus; Race-Sicherheit. |
| EV-NIMPROXY-000082 | nvidia-nim | content-copy | knowledge/decisions/dashboard-operator-console-redesign.md | 1–117 | Entscheidungsdokument | API | Operator-Konsole: 5 Tabs + Settings; typisierte Now/Range-Verträge. |
| EV-NIMPROXY-000083 | nvidia-nim | content-copy | knowledge/decisions/request-shape-metrics.md | 1–74 | Entscheidungsdokument | API | Metriken nur als Counts/Größen (Tool-Intensität, Konversations-Tiefe, Sampling, Truncation, Reasoning) — nie Inhalte. |
| EV-NIMPROXY-000084 | nvidia-nim | content-copy | knowledge/decisions/auth-posture-and-dashboard-password.md | 1–102 | Entscheidungsdokument | API | Fail-closed: `/v1`→503 `setup_required` vor Setup; PBKDF2 600k; Rollen superuser/admin/user. |
| EV-NIMPROXY-000085 | nvidia-nim | content-copy | knowledge/decisions/usage-injection-auto-fallback.md | 1–45 | Entscheidungsdokument | API | Usage-Injektion mit Auto-Fallback, wenn Upstream kein Usage liefert. |
| EV-NIMPROXY-000086 | nvidia-nim | content-copy | knowledge/decisions/window-jitter-margin.md | 1–48 | Entscheidungsdokument | API | 61-s-Fenster + 25-ms-Grant-Pacing: Lasttest 7→2→0 Upstream-Violations; 300/300 Erfolg, p95 ~61 s. |
| EV-NIMPROXY-000087 | nvidia-nim | content-copy | knowledge/decisions/reset-aware-dashboard-history.md | 1–99 | Entscheidungsdokument | Persistenz | Dashboard-Reset setzt History zurück (Boot-ID-Erkennung), Revision-basiert. |
| EV-NIMPROXY-000088 | nvidia-nim | content-copy | knowledge/decisions/dependency-update-cooldown.md | 1–46 | Entscheidungsdokument | Config | Dependabot-Cooldown `cooldown.default-days: 7` (dependabot.yml: Zeile vorhanden). |
| EV-NIMPROXY-000089 | nvidia-nim | content-copy | src/dashboard.html | 640–660, 2490–2504 | Implementierung | API | Polling `setInterval(pollNow, POLL_MS)` mit `POLL_MS = 3000`; `esc()`-Escaping + CSP. |
| EV-NIMPROXY-000090 | nvidia-nim | content-copy | knowledge/architecture/metrics-history.md | 1–101 | Architekturdokument | Persistenz | Metrik-Sammlung 300 s, `MAX_EXPOSITION_SERIES 100_000`, `COMPACT_AFTER_EXPIRED_SAMPLES 288`. |
| EV-NIMPROXY-000091 | nvidia-nim | content-copy | knowledge/architecture/key-pool.md | 1–49 | Architekturdokument | API | Key-Pool: VecDeque<Instant>-Lanes, 61-s-Fenster, per-Key-RPM 1–10000, Rebuild erhält Rate-State. |
| EV-NIMPROXY-000092 | nvidia-nim | content-copy | knowledge/architecture/dispatcher.md | 1–29 | Architekturdokument | API | Single-tokio-task-Dispatcher, unbounded mpsc, tote Antwort → Pool::release. |
| EV-NIMPROXY-000093 | nvidia-nim | content-copy | knowledge/architecture/streaming-pipeline.md | 1–49 | Architekturdokument | Event | SSE: sofortiger 200-Commit, `: retrying` bei Rate-Wait, SseScan-Watcher, stream_idle-Cutoff. |
| EV-NIMPROXY-000094 | nvidia-nim | content-copy | knowledge/architecture/governor.md | 1–69 | Architekturdokument | API | Governor-Poll 250 ms; Modell-Kapazität getrennt von RPM; `EXHAUST_BACKOFF` 2 s. |
| EV-NIMPROXY-000095 | nvidia-nim | content-copy | knowledge/ops/configure-env.md | 1–98 | Runbook | Config | Env-Kontrakt: 5 Vars; Legacy-Vars ignoriert; PUBLISH_HOST Compose-only. |
| EV-NIMPROXY-000096 | nvidia-nim | content-copy | knowledge/ops/deploy-docker.md | 1–71 | Runbook | Config | Docker-Deployment: Volume `/data`, Healthcheck `--health`, Port 8000. |
| EV-NIMPROXY-000097 | nvidia-nim | content-copy | knowledge/ops/sharing-with-friends.md | 1–55 | Runbook | Config | Multi-User-Freigabe: superuser/admin/user, Per-Key-Ownership, Keys maskiert/owner-labeliert. |

### nim.md — NIM-Integration

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-NIMPROXY-000098 | nvidia-nim | content-copy | src/proxy.rs | 200–420 | Implementierung | API | Streaming: 200-Commit, Heartbeat, Usage-Injektion; Buffered: JSON-Pass-through; Estimate-Fallback. |
| EV-NIMPROXY-000099 | nvidia-nim | content-copy | src/proxy.rs | 420–700 | Implementierung | API | `/v1/models`: Cache 10 min Single-Flight; lokale Publisher-Map; LobeHub-CDN-Icons. |
| EV-NIMPROXY-000100 | nvidia-nim | content-copy | knowledge/research/nim-models-endpoint-schema.md | 1–27 | Recherche | Schema | `/v1/models` minimales Schema (id/created/object/owned_by) → lokale Zuordnung. |
| EV-NIMPROXY-000101 | nvidia-nim | content-copy | knowledge/research/nim-kv-cache-reuse.md | 1–30 | Recherche | API | `NIM_ENABLE_KV_CACHE_REUSE=1` → ~2× TTFT-Beschleunigung. |
| EV-NIMPROXY-000102 | nvidia-nim | content-copy | src/config.rs | 405–430 | Implementierung | API | URL-Sanitierung: Link-Local (169.254.169.254) und Nicht-http-Schemata abgelehnt (SSRF-Schutz). |

### rust-foundation.md — Rewrite-Grundlage

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-NIMPROXY-000103 | nvidia-nim | content-copy | Cargo.toml | 1–77 | Konfiguration | Config | Dependency-Stack des Rewrite: axum 0.8, tokio, reqwest-rustls, metrics; hmac/sha2/getrandom/subtle; Profil-Setup (Dev opt-level für Crypto, Release lto+codegen-units=1). |
| EV-NIMPROXY-000104 | nvidia-nim | content-copy | rust-toolchain.toml | 1–9 | Konfiguration | Config | MSRV 1.87 als Baseline für Rust-2024. |
| EV-NIMPROXY-000105 | nvidia-nim | content-copy | .github/workflows/ci.yml | 1–271 | Konfiguration | Config | Coverage-Gate ≥ 90 % Linien, cargo-llvm-cov (Subprocess). |
| EV-NIMPROXY-000106 | nvidia-nim | content-copy | .github/dependabot.yml | 1–46 | Konfiguration | Config | Dependabot-Konfiguration mit Versions-Cooldown. |

### tests.md — Testanalyse (Auswahl)

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-NIMPROXY-000107 | nvidia-nim | content-copy | tests/e2e.rs | 117–3731 | Test | Test | 90 e2e-Tests: Zero-Upstream-Violations, Dashboard-Verträge (from/to/points), Revision-Reset, 0600-Rechte, API-Key-Gate, Setup, Shedding, Deadline, Retention. |
| EV-NIMPROXY-000108 | nvidia-nim | content-copy | tests/support/mod.rs | 412–515 | Test | Test | `fresh_data_dir`, `spawn_and_wait_healthy`, `expect_refuses_to_start` (Boot-Fehlerfälle), `base_cmd`. |
| EV-NIMPROXY-000109 | nvidia-nim | content-copy | tests/support/mod.rs | 599–659 | Test | Test | `complete_setup` (Wizard steuern), `read_sse` (Stream-Leser), `chat_body`, `sha256_hex`. |
| EV-NIMPROXY-000110 | nvidia-nim | content-copy | scripts/mock_nim.py | 1–198 | Test | Test | Behavior-Definitionen: RateLimited, ServerError, BadRequest, BadRequestIfInjected, Hang, Ok; Sliding-Window-Violations-Zähler. |
| EV-NIMPROXY-000111 | nvidia-nim | content-copy | src/auth.rs | 700–901 | Test | Test | 21 Unit-Tests (PBKDF2, Throttle, Konstantzeit, Session). |
| EV-NIMPROXY-000112 | nvidia-nim | content-copy | src/history.rs | 1700–2037 | Test | Test | 35 Unit-Tests (Retention, Boot-Marker, Compaction, v1-Legacy). |
| EV-NIMPROXY-000113 | nvidia-nim | content-copy | fuzz/Cargo.toml | 1–37 | Konfiguration | Config | 3 Fuzz-Targets konfiguriert (config_roundtrip, sanitize_label, sse_scan); Seeds inkl. 0-Byte-Dateien. |
| EV-NIMPROXY-000114 | nvidia-nim | content-copy | .github/workflows/fuzz.yml | 1–69 | Konfiguration | Config | Wöchentlicher Fuzz-Lauf. |
| EV-NIMPROXY-000115 | nvidia-nim | content-copy | .github/workflows/codeql.yml | 1–57 | Konfiguration | Config | Wöchentlicher CodeQL-Lauf. |
| EV-NIMPROXY-000116 | nvidia-nim | content-copy | .github/workflows/scorecard.yml | 1–45 | Konfiguration | Config | OpenSSF-Scorecard-Lauf. |
| EV-NIMPROXY-000117 | nvidia-nim | content-copy | .github/workflows/audit.yml | 1–33 | Konfiguration | Config | Wöchentlicher cargo-deny-Advisory-Lauf. |
| EV-NIMPROXY-000118 | nvidia-nim | content-copy | .gitleaks.toml | 1–23 | Konfiguration | Config | Secret-Scan-Konfiguration. |
| EV-NIMPROXY-000119 | nvidia-nim | content-copy | .github/codeql/codeql-config.yml | 1–14 | Konfiguration | Config | CodeQL-Detektor-Konfiguration. |

## Nicht analysierbare Binärdateien (7 PNG)

Nachweis je Datei: Hash (SHA-256), Typ (PNG, binary), Byte Size, Verwendung. Die PNGs werden ausschließlich in `src/dashboard.html` und `src/setup.html` als eingebettete UI-Assets referenziert (kein technisches Verhalten).

| Datei | SHA-256 | Byte Size | Verwendung |
|---|---|---|---|
| docs/assets/dashboard-clients.png | b22cbffe298527246a2ed433db28fd7a239b9f53a6d3caa5e48101bf669874fa | 97.437 | Dashboard-Tab „Clients" (Screenshot) |
| docs/assets/dashboard-models.png | 1e42b6554eb5fad8fade2a399500294fec3983fcccb236c9730454f08e3eaccd | 117.318 | Dashboard-Tab „Models" (Screenshot) |
| docs/assets/dashboard-overview.png | dcea0519ebf2fa1a05c7a96ca73637552e50e0710766d61294aa82f887c298a2 | 128.835 | Dashboard-Tab „Overview" (Screenshot) |
| docs/assets/dashboard-reliability.png | 4cbbb246f0db47c97d06d9076396889a038cc83b5ff3e0a01c8910ba1893e834 | 117.569 | Dashboard-Tab „Reliability" (Screenshot) |
| docs/assets/dashboard-settings.png | b3d6082d8defc76f26215fc754319264303c3907c85620e9758366d6336fade3 | 183.540 | Dashboard „Settings" (Screenshot) |
| docs/assets/logo.png | 65d618b098df368eea28afb9291abc90e3c7a3f50c1020c6c519ac237585a4e5 | 108.062 | Projekt-Logo |
| docs/assets/setup-wizard.png | 655b7d9a6974e420c9873e11d945dddd5a6e25a73d875d3c20aa468fed34a64b | 39.254 | Setup-Wizard (Screenshot) |

## Fazit

- Dateiabdeckung: **113/113** (100 %).
- Claims geprüft: **119 Evidence-Blöcke**; alle Kernaussagen der 8 Inventar-Dokumente sind **belegt vorhanden**.
- Diskrepanzen: 4 dokumentierte Abweichungen (CLI-Flags, Testanzahl-Rundung, `Server::run()`-Benennung, Versionspfad) — alle ohne Fähigkeitsverlust; die CLI-Diskrepanz (Zeilen 1–4 der Diskrepanzliste) ist die einzige mit materiellem Charakter und in der Statusdatei als `CLI_OVERSTATED` markiert.
- Verbotene Statusbegriffe: **0 Treffer**.
- Status für die Rust-2024-Neuentwicklung: **bereit**.
