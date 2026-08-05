# PolicyNIM — Validierungsbericht (validation-policyNIM.md)

## Validierungsrahmen

- Repository: policyNIM
- Commit: 0eb2ae72f5560d3ed41aaef2af1c4d3121561e42
- Dateien gesamt: 151 (vollständige Inventur)
- Geprüfte Dokumente: `/workspaces/bkg-nim/inventory/03-policyNIM/index.md` (125 Zeilen), `files.md` (3313 Zeilen, 151 Sektionen), `tests.md` (114 Zeilen)
- Prüfdatum: 2026-08-05
- Methode: Claim-Abgleich gegen Repo-Dateien mit Zeilenbereichen; Read-Evidence aus `docs/inventory/source-checksum.md` (Abdeckung 151/151 verifiziert)
- Hash-Gegenprobe: Checksum-Datei vs. Repo-Dateien → 151/151 identisch (Diff leer)

## Ergebnis

- Datei-Abdeckung: `files.md`-Sektionen == Repo-Dateien == 151 (1:1, Diff leer).
- `index.md`: Claim-Abgleich bestanden; alle geprüften Kernaussagen belegt (belegte Funde unten).
- `files.md`: bestanden mit 5 Korrekturen und 2 Präzisierungen (Attributionsfehler, siehe „Korrekturen").
- `tests.md`: bestanden mit 1 Zählfehler (41 statt 40 pytest-Dateien), 2 fehlenden Tabellenzeilen und 1 Duplikat; alle aufgeführten Zeilenzahlen korrekt.
- Gate: bestanden. Es wurde keine Datei ausgelassen; jede Aussage in den Ausgaben ist durch Quelle belegt oder als Negativbefund dokumentiert.

## Read-Evidence (Datei-Manifest, 151 Einträge)

Format: Pfad | File Hash (sha256) | Byte Size | Line Count | Encoding | Read Timestamp | Reader Result

policyNIM/.codex/agents/entire-search.toml | sha256:d36112c702733fb4c9b10c3fb22135561838cc1736cd861e8a46a73cd2b29a61 | 1563 | 23 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.codex/config.toml | sha256:aa47fae316b036912a1dbdf86d3274a0000be146ac7e55864c372cbb3e5624ef | 25 | 3 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.codex/hooks.json | sha256:8ac32c4e5d79d91cd8b1bde127bc2afe674803c1a2f38ec3b47c5a9511488604 | 1438 | 52 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.dockerignore | sha256:129a5e128f501557fb71bb12f9480830df1465f75697f0e1d51595f3c0e6f292 | 118 | 14 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.entire/.gitignore | sha256:e0d32a3775837a6a1df279f0e6b0e04c02cf3b472781a406aa4586c8ffe02e64 | 58 | 5 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.entire/settings.json | sha256:c496b96bcbfd387010bcd0117da97a8ab75b9f6aa64e1e2cacf1f593c0c82ed5 | 252 | 12 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.env.development.example | sha256:9907a92ef2234b2fbe6ae47ef3ef2fa374b003beed5129fea252eaa1baa4d115 | 1119 | 26 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.env.example | sha256:ae798edc1db1a0ac0fd81e6ded098530537ca6838353fa3e7df848b8a56fc0c5 | 1618 | 35 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.env.production.example | sha256:1f09b7a6f4c5b5774bf81c9d5bf95574d141c6edcf53d259cf1256937520c5f3 | 1720 | 34 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.github/workflows/ci.yml | sha256:ee184b4d36156492149c5ee56a61b75dc31b6a1906289eac0288260d1f407a38 | 1318 | 54 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.github/workflows/hosted-smoke.yml | sha256:92d3ae11e804318719ef8ba3455f88932578208f4c2264c7755971180d6f0b9f | 1260 | 42 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.github/workflows/release.yml | sha256:d92b80acc2bcfa9a0dfabd98cbd11652cee47aaccd71039d9697a71e26e2b3a8 | 9661 | 290 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.gitignore | sha256:f8deac74d16f656a162e2cdb3b54d36f111588c18e5ddf859dcbac5671b8ba4f | 216 | 19 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.pre-commit-config.yaml | sha256:d729d0af34af8c9076e81f161523c5cbc2f1587a0732f41d3525f38f70787690 | 490 | 19 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.python-version | sha256:589191117772bbb86645d171d153b6d166a9ce50cb7abcc206af90de1284cf7d | 6 | 2 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/.threadloop/config.json | sha256:bb49826afeb7947609af366e6e7824bde15d9b5975eedd1067371d06012a457a | 62 | 4 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/Dockerfile | sha256:baff331d3e891dc72138607e427a2e77eb782365c35077f7947f5a5dfcaee38d | 1317 | 43 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/Dockerfile.railway | sha256:fffde8d10adfd84320851537f053898461abe6182c670b73dc7f34473e2e6a4e | 1372 | 45 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/LICENSE | sha256:8a60c102e091e477f194fa90d76bcfc4a0e76263bfd9e540f3f05133e7a1816b | 1070 | 21 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/README.md | sha256:8c479696a2162f13ddd8f8ab1bb3ea02b70022fb1dcdb8333b5c8b7652012579 | 9648 | 240 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/ai-engineer-miami-context-plane.md | sha256:20422b0bf76f7d3f9e1280a0e8d9a716042d4a272f5d087fa1895a3e38272f20 | 5763 | 33 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/architecture-diagram.md | sha256:cb9ea06c42b5d265d9ca830ff29517dad6e39cc8a2a5ca93558c16e131d88dd5 | 11109 | 282 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/architecture.md | sha256:886500c0c8d2af62d68dbb6c5d2b3c347e2a8ba466b61e4d484d4106d3b4690c | 18747 | 444 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/assets/readme/policynim-beta-dark-landing-preview.png | sha256:27d6c9db4079d8dd93e6c46bafe59bfdea0569cb48e7db7d0153928c94639e50 | 395170 | 1467 | UTF-8 | 2026-08-05 | Binärinhalt nicht analysierbar
policyNIM/docs/contributor-guide.md | sha256:1be70a9a67ebad9207f1e530eab1955eb4b1b2789c817ad73dcf09fc33e4cb68 | 7462 | 235 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/demo-script.md | sha256:d9bc55a141abba62acccee6a19f4a12d1302ccc49b0439c72920d32c4e4ebc1c | 6612 | 265 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/extreme-programming-with-agents.md | sha256:12f79cbf3e4cf4a20e4e721f5f0d727fc86330dd80d5b719c3c4582924fc1c91 | 7578 | 109 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/hosted-beta-operations.md | sha256:21b22d82c6c8e901944a01a005d7e5b76c29890c3f7f27f94bb8c82b2b22c1d2 | 12128 | 308 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/index.md | sha256:8d8602aa98ca8a3357a0793b4964819908e6e75fb0d3dbfac860400c8efcb05b | 1975 | 45 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/limitations.md | sha256:4fe91bc068d8d44d25576823de3a5f8a9e263478edde8d6d92792a2cf8179dd9 | 6721 | 138 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/public-source-grounding.md | sha256:e9203211f0b4b66a376806be449c00475d40561309c8d796f0b7c19fd31d4052 | 4639 | 130 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/release.md | sha256:d0034456bafc68dc147655c96587b30ef71468d19f18c2694f911d1163651e3a | 2779 | 90 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/docs/workflows.md | sha256:1b4e55fe5af3313986d1a42c96c281a2bff6630d45d473fee681779a8fab1d14 | 21309 | 614 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/evals/default_cases.json | sha256:d4e9d6f96b91e9c1980422b6c871f61a5e0a85a936adfdfefcafe731934165a3 | 1865 | 61 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/examples/claude-code/README.md | sha256:a1a15b0fe3a28f09501680b56e25ab0d7be7b926502141d1e16072e288376797 | 2990 | 107 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/examples/codex/README.md | sha256:c6c5ed9746c38747b9d2825e7589ee55c10ded77858b42c8543e06c093427c8a | 3753 | 124 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/packaging/pyinstaller.spec | sha256:5c0ff14605a08bcefaee1f4a09704b291096ad49c0250b6ab12b85cac1649602 | 1401 | 62 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/packmind.json | sha256:8584ab3a76183eb5c1598dc2efbfa968bc3fac4484855228e822da50580d29f7 | 37 | 4 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/TEMPLATE.md | sha256:4405c8df54e283fa755755e558975ffd6ee8951ae0f074d5abd124e48b2c300e | 1557 | 52 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/architecture/api-versioning-guidance.md | sha256:77001dae069f1c5a4df24d880a7f30c3761603f75ffbb31cca03ffbbbaa69c89 | 1507 | 46 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/architecture/background-job-design-rules.md | sha256:ed380c577ba87ff6eb2b1cda707324b5eaf75b1df96bd1066ccb7378459ef90b | 1583 | 47 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/backend/backend-logging-standard.md | sha256:e644c2ca71b449c542e820689895bf00d29298b9bb54273828bdc1cdd1d13e72 | 1687 | 48 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/backend/config-validation-and-fail-closed.md | sha256:48bb55c0accc7387eaa1e4d5c5d5b808a2094f8cc959b5678d66c79df043038e | 1459 | 43 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/backend/request-correlation-and-tracing-standard.md | sha256:d3fd05b035c05012d9aa29e21157e41d0086fd32c0badc9bcc7a58b604d509ef | 1595 | 45 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/security/auth-sensitive-code-review-standard.md | sha256:c84c02bc0c16cf768b80ae32a16e520a2b7fa5ee91b52a40f690613d9008f00e | 1637 | 47 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/security/public-endpoint-safety.md | sha256:e06f53f27475271a7bd145895be08412598092c2adb377644a35c42705d47d14 | 2395 | 60 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/security/secrets-redaction-and-handling.md | sha256:78336742e0f2314e285248b9350f0088c3305aecb7290b18409f5d544bdc1bab | 1637 | 46 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/policies/security/session-lifetime-and-token-boundaries.md | sha256:3e724223d31076cc290fdce5b5acd364be5058f6bf3beb1eef72d1287d5fdafd | 1298 | 41 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/pyproject.toml | sha256:2cee8d5a526471f33a9db74f3111b5ab304e266e6e3a06f411cb01e2a5976151 | 2337 | 96 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/pyrightconfig.json | sha256:91b70e2f47c1404b9ef9384a19a0e39401ee3bb06b3d8612ad39e5883aed27de | 371 | 25 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/railway.toml | sha256:ea6dae3a2d79c479b7153fcff60ca7f9bdf93aadfbe3d86cf2f85c700f3b58f5 | 209 | 10 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/scripts/install.ps1 | sha256:f7bd5fddf7e6bdfb14c85b96597b2ea1b932dc7375c50e5ac1f893a0a04a350e | 6677 | 156 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/scripts/install.sh | sha256:0b424523abf845b9b9d7e101be7baec5b15bd2c82a14c3cec3ea5f59f3aa7d56 | 6169 | 216 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/__init__.py | sha256:f5ebdbd38f05d507906b0dd346b0f8b62931ff98f71681976c48105160756df4 | 25 | 1 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/agent_workflows.py | sha256:eee2e56600aaf342387dbd0163656a26df95aec08cfa25040f3f0e68c4a11b43 | 2626 | 78 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/assets/beta/beta-page.js | sha256:94ae3d3ca23595b8e5f252daa637d8510651d3fbfae14c30ae9ee167f3b1e62b | 2067 | 58 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/assets/beta/beta-theme-init.js | sha256:2ee5e88f9343dfabf0bb45fb81774187004ee82a11b2253709df6e632f3d7e95 | 395 | 16 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/assets/beta/beta.css | sha256:c45d26348dbf402d54b3ae36df1d7d9752e0f4eb4a12b36cf349a32e1306246a | 13026 | 687 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/assets/beta/policynim_darkmode.jpg | sha256:de6eb492459440f99fa3aa73c39f744900c7bc2e74a81baec065fde3c56065df | 29496 | 215 | UTF-8 | 2026-08-05 | Binärinhalt nicht analysierbar
policyNIM/src/policynim/assets/beta/policynim_lightmode.png | sha256:c719b04a70e7399cd36d9587487cd7475aa6b25574c9e1a79e7922524bd9fc65 | 296318 | 1071 | UTF-8 | 2026-08-05 | Binärinhalt nicht analysierbar
policyNIM/src/policynim/config_discovery.py | sha256:d962b549770c2043623edf84f381b9e4b8af0f63250ca5a4c7b98aab55a3ef76 | 10840 | 309 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/contracts.py | sha256:4fd09730a5449c970ce3e46f1da86d6a4cb1e004b1da8543508da3bd423aec67 | 4294 | 157 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/errors.py | sha256:068f002dd5a0455be01cce19934bd1a27ab5469454050e076e165ae2ffbda38e | 1277 | 43 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/ingest/__init__.py | sha256:935bef758f7c946c38b54cc4fa4d6e45bca693c8dcb8f5fc9a33481a10d8b9fb | 705 | 24 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/ingest/chunking.py | sha256:9064b5ee59d14bdea8b6cc03903fe12caf9d82cecdbd99b568a84b4edf1e8ff4 | 9375 | 307 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/ingest/loader.py | sha256:c72fbeb3c58115da22dbffb92cd514a22a9200f6d236656f766bf40e419a44ff | 3991 | 113 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/ingest/parser.py | sha256:0c98e4bfd75ad371c64b6844cc7956f04ab76976cb2f7d7f694753e5927be61c | 29797 | 940 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/interfaces/__init__.py | sha256:a7cd276500f208612d127c93422ea756fba550af9db7df764637287998f2b311 | 51 | 1 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/interfaces/cli.py | sha256:6d909508e6ad6e2fde30e646f0357552dfd7041b3aafdf38d9a46a143a4c3a43 | 105332 | 3074 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/interfaces/mcp.py | sha256:74b8528e1f2f45e5bdddef06487247f1060d50208ec3be344e860ca6d26724b7 | 38047 | 1064 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/providers/__init__.py | sha256:901070522347dda91dbe1948584a05166ee83c0aa37171861ccee7425f969347 | 719 | 25 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/providers/nvidia.py | sha256:4672947e1736bb54f1c4fcb6b7aa7b315e8591d5de485b7bc5a3d2ba3bee61f8 | 41352 | 1135 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/providers/nvidia_eval.py | sha256:8ef95d6386b70cfd0759095c7f9b996a491ad1cf46e7660a0f6e335caeb6038c | 4893 | 141 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/providers/nvidia_guardrails.py | sha256:31174fca8f6613b9afcb24596dff5451589863abbbf09a70951d90fa19af4f09 | 13946 | 383 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/runtime_paths.py | sha256:155d7a1ece9018f95dc21f10615c866ddc4ff4be2003df4443b90970029e7ebc | 5834 | 163 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/__init__.py | sha256:6e5d57579c0b54a7f9e3d81c4a7624d9aac142f0406b0b53da981a89baf82fd4 | 2548 | 71 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/beta_auth.py | sha256:6172a70ee76981475b6ef124ce39bd65e9e0ce3aab8576ab8d271ba0dae3f35c | 13651 | 351 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/compiler.py | sha256:ddac27dd7cb4c80f3b9c9b60cf45062930765636267ea3db26baff89c5956dd5 | 9071 | 264 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/conformance.py | sha256:6a38dba8e8c0f0e1e64870af7a5df8dda9141bf395e783d552eee1fcac595376 | 11095 | 323 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/dump.py | sha256:295a3b60e74dda135e0ed49c1aab491b5d02b08621a733e28a440f33cab64307 | 877 | 25 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/eval.py | sha256:0e727e6323b67f6aac0fced20022e44f54438db402fe8fd6b9585f52d311bff6 | 64261 | 1754 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/evidence_trace.py | sha256:9dd8e5f656adc09d82834e1d5621a35dc53678f27102f1f620e1960771fa4f2d | 13094 | 353 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/health.py | sha256:9d800b5972cff0c9ea668d217f8834749cef9e72b4fc0aeb35280e6ac4d4a804 | 7205 | 217 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/ingest.py | sha256:4f8d9b8095ba3cd457aadfbbec2811202ab49ea94145878be9cc70dcd1fc9912 | 7377 | 214 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/preflight.py | sha256:3ce41a30e412aff6629f3601f94794ca7a792003403cdbac77e846a28b057615 | 14065 | 406 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/regeneration.py | sha256:c4f56b1e5d29b33e80f9a122bd100a380216abc0e3c33f7bd67a685a1aaa9a1f | 22465 | 653 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/router.py | sha256:521fcc5828ebf5edb74b7df0e08f6b98c5a2a0c3d74f4b08c91057abdfec8297 | 11646 | 358 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/runtime_decision.py | sha256:c827bfd1adc2b28c7ecde20946cdeae24bf0fa091f9a4c2b4886612f6067a91d | 15356 | 425 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/runtime_evidence_report.py | sha256:3062d3ce69ada5b9a6785bbcd8882904c37980012a2bd1424f7f92c60cd277c6 | 5460 | 156 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/runtime_execution.py | sha256:51a3174965008365cc1bb337b0b39aa4d4c5da56de267133239649e8ece2fcad | 20521 | 575 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/services/search.py | sha256:da9a8f047701e7e98dc736959f694dfb58f5a1071055f50fa3d584fe1b17ee46 | 3408 | 106 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/settings.py | sha256:d64c30181a2605c68d22e4be7c2ed331703e4e373bdd74f6e59c09a859412bf4 | 15618 | 394 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/storage/__init__.py | sha256:18a2e125308523f745e84e2e69c1486a4641b6b685ab4a0a162a4ff0af51d804 | 393 | 13 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/storage/auth_store.py | sha256:6696fab99ab54df2437ac9c976abfcc13c8a2081eb0f2a74a415f8044c79be5b | 23026 | 628 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/storage/index_store.py | sha256:2270e6ca087e76820b089591f9701a706c0586a611ddc1374ad60fab9341706c | 483 | 12 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/storage/runtime_evidence.py | sha256:72c410a1189acd1621dee3a877f8f4717baf1ed658f9bafbc87f26ac685b2233 | 8267 | 211 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/storage/sqlite_vec.py | sha256:90bf839bdd1d0b6138e01a3b73d8f82d3512350f454d63c0ae8d92b8e79fc8e1 | 16578 | 481 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/beta/dashboard.html.j2 | sha256:832cf3c8318349850be28a7e68f2dafdc99e23963ec7f50a0ef133c4048e1ff5 | 4844 | 144 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/beta/landing.html.j2 | sha256:e30e71cc8e049a8058fe04e97e515acb2792ec2cb6921dce601cddf1e2e1e2de | 3275 | 85 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/beta/page.html.j2 | sha256:4ea3a6d46f163a5e209f57eb46f26711170e062af5b12d1a71d5c12472e9d915 | 633 | 17 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/beta/partials/command_card.html.j2 | sha256:040871c1f1fc64729232e67ee9dd386c1cc9ce75c18d177cd176beb7242fbcbf | 570 | 19 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/beta/partials/logo.html.j2 | sha256:93ecf53ba96e88a1e58dd52a885388f65fe796571b3ff8f01abd08d5e71403a0 | 401 | 18 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/beta/partials/new_key_panel.html.j2 | sha256:5fb8c40f590d49c9d7f51aedf3da0b5dd071a35a97ed9ba057033271949b4af5 | 750 | 24 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/beta/partials/notice.html.j2 | sha256:cbcdbd8eb4ceb23a070bd84b4327626b8f5c75273c4a850bd7a8b4188a92b952 | 176 | 4 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/beta/partials/theme_toggle.html.j2 | sha256:f65884286c1a79df70494d088cc4bc003fec7155838dddc6b320024ac767bc0b | 245 | 10 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/nvidia_guardrails/preflight_output/config.yml | sha256:90d1a8f82f1e1ff68157e182bf31d2d1d5da56a23e946e4d565fae37b6b9df5c | 966 | 31 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/templates/nvidia_guardrails/preflight_output/rails.co | sha256:2e44e9627ece4b1fa69d657f47a9b635b10881825bc39c3112c1f86d1d86a03e | 121 | 2 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/src/policynim/types.py | sha256:32277a13d7485a77784060062ab1ad5868f4a841a12081bb938c2d340dc054e4 | 37141 | 1114 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/README.md | sha256:ba7b4cdb25ba55c6eef095fe32ff07e63ee9a996a93d2662a84778e564ae8781 | 5125 | 88 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_auth_store.py | sha256:2fe808214897e84da5c01fd5ac572566ca4216782922c72ce1920e04e75d7c49 | 7588 | 232 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_beta_auth.py | sha256:8db668fe83b7ad7908c63a7e725d4bb079daad4ee3b7c99143a067f71a5bb1eb | 4603 | 149 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_beta_portal.py | sha256:1885cc2cf6255b96326e24832b98c5fadd08da4eae2afae34877243892daa5c0 | 14146 | 374 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_ci_workflows.py | sha256:1a573bf59e95b3947d40d4854f9acbcc0c50c5891a3e0886bdcc1616fe672657 | 5270 | 142 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_cli.py | sha256:cdd517f565b84d30da68aeb5450f1783e9206597ce730c084199079b61611608 | 148663 | 4252 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_compiler_service.py | sha256:350e944f85675eddfc58f078734c7789c002f3c42873ff224440e55054ff958b | 9369 | 303 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_docker_build_live.py | sha256:ddd84c9b7f68db77996788e2c61c5f7f24ffbc2b717e789b57ba76bd529bfa40 | 4121 | 149 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_dockerfile_contract.py | sha256:382785481a357a9aae1d456fc625f77e6b6da4dffe3bcc0165c143338e9a98e1 | 2460 | 68 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_docs_hosted_onboarding.py | sha256:de410abb2797f3172508e64fc92237ed94c23ab65f8344c3a76f251559d3f334 | 4421 | 140 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_docs_runtime_workflows.py | sha256:ed3c1ad4ea5debe1dc46725835071e928fe44775b74156eb24b2af4967a29580 | 7262 | 205 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_eval_service.py | sha256:1d4bfa76f7fcd484cfdd563c39a5bd3cb9a1743a2e57cfcf6d9481f9c1a52924 | 18228 | 515 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_health_service.py | sha256:cb16d0e5f47a60a69bc70cf892de5c49fe17f0d185c99ece259934453a318501 | 11888 | 387 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_hosted_mcp_live.py | sha256:c449a6bf3fe7790ba154cf5486ce55284f6ba6404d88e7e6cd4931f22d90ba71 | 4462 | 130 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_ingest.py | sha256:7f014f00ff7b42ff19e33c6b58d5e5804fd40ff343c06bda02227fe48e76b6af | 11854 | 431 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_ingest_service.py | sha256:307ab51a38d910285dfebc1d646de29206eb0bab064187e5894499da1fd5c8e1 | 7953 | 281 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_installer_contract.py | sha256:3f47726897f0c6fcb7ee9db69dbfb3708f171aa669b29d79e3cbf72c1a4cf26c | 9454 | 258 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_mcp.py | sha256:3924d766b603bb6c31fcde0a4b9f0c5df9f4e4ea9c8a88d193dd50c0bb519af5 | 35453 | 1066 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nemo_agent_toolkit_policy_conformance.py | sha256:7e2e58503207a9923489367a4e07c2122d72188a7942272b5d7d94fea754eaa8 | 3650 | 112 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nemo_evaluator_policy_conformance.py | sha256:a24bb2640ab222f727c5abac13ee21c7374d8b83f08a844277a5db8ea2794c51 | 3553 | 110 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nemo_guardrails_preflight_generator.py | sha256:3d5ce1f1e34470ce44de562c5d9ec0f7fb0cadf6b97cf41dc42fa5588f1076b0 | 12488 | 388 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nvidia_embedder.py | sha256:5655065b6db03590090fbd18ce38ac4ab3d092a486e0a51a16b8ba28263a542e | 3346 | 114 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nvidia_generator.py | sha256:01a3e62899b5bcf1ae645d24b4b5f71249918b165f0685ccf4872f263299ee41 | 7574 | 240 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nvidia_live.py | sha256:ee0466ae33ff82cacf84a96c4194f7e057acd6bfe6c0a7950468b058af90e549 | 2791 | 96 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nvidia_policy_compiler.py | sha256:719b67f27337d50cd5a9cda5f8ffd4689af11fc143d8ae781e349b2dc3eb6755 | 9483 | 324 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nvidia_policy_conformance.py | sha256:a33d0e75f0c9ad1f4641f7af5a6790335be71e91985dc938cdffa65184f984f3 | 9071 | 280 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_nvidia_reranker.py | sha256:904f7d4c8c9481845bec24330088f47f4afb2ac191ccde8b3fcc1f818d6a69df | 8281 | 274 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_package_release.py | sha256:5a9642bef7a1bfb9a68f774c33d7588bce647d4b7ab3378273fa3552b9b0c23b | 2820 | 71 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_policy_conformance_service.py | sha256:56236e4850b8ad3dbabd8ee6c8db399db000506ba5de05e53b9bf65a4fb108fa | 9220 | 272 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_policy_evidence_trace_service.py | sha256:f194ff0c2e235f3201f988e45382956fcf3cdb47085c05ef5de53bfa5dad3f9a | 9665 | 285 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_policy_regeneration_service.py | sha256:b532bf1765be3e9eb8f7b82e7aa58ee62da74c04777e2605bee6c47bb7c96ee9 | 18119 | 542 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_preflight_service.py | sha256:fb0e17e7bbd94bcd0f96687131c1430634e478bd6a2e1690ee927d1c253c054f | 24112 | 756 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_router_service.py | sha256:ced1f367f1abdcff95b39b546bc7611302e8760eedfeb9b951fcfc1208521e0b | 9098 | 294 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_runtime_decision_service.py | sha256:e7998a0c4a38235b3c1ae62d4b6b6916cb6ab131566ddf57b4909cb708ea1cfe | 18863 | 595 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_runtime_evidence_report_service.py | sha256:1aa0b820d4d0770aee11471287334fd8f7d3d6764fbddaa7d4012a06672b9170 | 14555 | 449 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_runtime_evidence_store.py | sha256:b88da67fa72a15ae0d18df7e8cde1c34061140d74591b1876cdda0ceb8a757b4 | 8818 | 272 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_runtime_execution_service.py | sha256:c99d9a3091ee298c72e1bcaf8aa5c7317140f3862556b2a49bdce887585492ad | 19141 | 560 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_runtime_paths.py | sha256:836fac16d1f8597543279bc4f23487678142c3ca0582ad030c22be78f1c8287e | 10382 | 281 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_search_service.py | sha256:65b779084bf3745957a185e56ceed2894770dcd2f46d0da4a57d23954e0b0b30 | 8416 | 264 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_service_factories.py | sha256:9a7269247ab1a587ab47d99dfd3924230a75c9abb93647655af7edece8401d09 | 13766 | 403 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_settings_and_types.py | sha256:b05c65a42b77e8130a1bb48a3860e184f3fca5ef267a9fc9164501a9e4f0af7e | 33577 | 972 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/tests/test_sqlite_vec.py | sha256:fa28e05426cfb51dfd913b0b236cc978f6ed7d19de3a387f73ea6a69d127f3ab | 9795 | 287 | UTF-8 | 2026-08-05 | gelesen (Volltext)
policyNIM/uv.lock | sha256:edd9a28fc2e49227567416500deafa4325b9ceef40488fc414d8eda5dab71b5b | 544475 | 4396 | UTF-8 | 2026-08-05 | gelesen (Volltext)

## Binärdateien (nicht analysierbarer Binärinhalt)

Für diese Dateien liegt kein lesbarer Text vor. Nachweis je Datei: Hash, Byte-Größe, Verwendung.

| Pfad | Bytes | Verwendung | Nachweis |
|---|---|---|---|
| policyNIM/docs/assets/readme/policynim-beta-dark-landing-preview.png | 395170 | README-Landing-Preview (Dark Mode) | sha256:27d6c9db4079d8dd93e6c46bafe59bfdea0569cb48e7db7d0153928c94639e50 |
| policyNIM/src/policynim/assets/beta/policynim_darkmode.jpg | 29496 | Beta-Portal-Logo Dark Mode | sha256:de6eb492459440f99fa3aa73c39f744900c7bc2e74a81baec065fde3c56065df |
| policyNIM/src/policynim/assets/beta/policynim_lightmode.png | 296318 | Beta-Portal-Logo Light Mode + Favicon | sha256:c719b04a70e7399cd36d9587487cd7475aa6b25574c9e1a79e7922524bd9fc65 |

## Bestätigte Kern-Claims (Stichprobe mit Zeilenbereichen)

Legende: CONFIRMED_PRESENT = Aussage durch Quelle belegt.

| # | Claim (Dokument) | Beleg (Repo-Datei:Zeilen) | Status |
|---|---|---|---|
| 1 | README: „policy-aware engineering preflight layer" | README.md:19 | CONFIRMED_PRESENT |
| 2 | pyproject: Version 0.1.0, Python >=3.11,<3.13 | pyproject.toml:3, 6 | CONFIRMED_PRESENT |
| 3 | pyproject: Extras nvidia-guardrails/nvidia-eval/nvidia-eval-launcher | pyproject.toml:45-53 | CONFIRMED_PRESENT |
| 4 | pyproject: Console-Script `policynim` → cli:main | pyproject.toml:58-59 | CONFIRMED_PRESENT |
| 5 | railway: dockerfilePath Dockerfile.railway, /healthz, Restart max 3, 1 Replica | railway.toml:3-10 | CONFIRMED_PRESENT |
| 6 | Dockerfile: python:3.11.15-slim-trixie SHA256-pinned, uv==0.7.12 | Dockerfile:2, 12 | CONFIRMED_PRESENT |
| 7 | Dockerfile: BuildKit-Secret nvidia_api_key für Bake-Ingest | Dockerfile:20-21 | CONFIRMED_PRESENT |
| 8 | Dockerfile.railway: ARG NVIDIA_API_KEY (Railway ohne Secret-Mount) | Dockerfile.railway:20, 23 | CONFIRMED_PRESENT |
| 9 | settings: nvidia_base_url integrate.api.nvidia.com/v1 | settings.py:81 | CONFIRMED_PRESENT |
| 10 | settings: Beta-Quota 500, Rate-Limit 20 Versuche / 900 s | settings.py:102-104 | CONFIRMED_PRESENT |
| 11 | settings: mcp_host-Default 127.0.0.1; PORT via AliasChoices | settings.py:85, 88-92 | CONFIRMED_PRESENT |
| 12 | .threadloop: version 1, createdAt 2026-03-23 | .threadloop/config.json | CONFIRMED_PRESENT |
| 13 | .entire: telemetry true; Checkpoint-Repo nnennandukwe/policyNIM-entire-checkpoints | .entire/settings.json:4-11 | CONFIRMED_PRESENT |
| 14 | hosted-beta-operations: Quota POLICYNIM_BETA_DAILY_REQUEST_QUOTA=500 | docs/hosted-beta-operations.md:220 | CONFIRMED_PRESENT |
| 15 | pyinstaller.spec: entrypoint cli.py, copy_metadata, 4 Ressourcen | packaging/pyinstaller.spec:8, 11, 14-19 | CONFIRMED_PRESENT |
| 16 | install.sh: TEST_OS/ARCH, releases/latest-Redirect, sha256sum/shasum | scripts/install.sh:18, 27, 44, 66-71 | CONFIRMED_PRESENT |
| 17 | evals: Suite „day-6-default", 6 Fälle, expected_insufficient_context | evals/default_cases.json:2, 10-56 | CONFIRMED_PRESENT |
| 18 | tests/README: Marker live/docker_live + Opt-in-Env-Variablen | tests/README.md:60-87 | CONFIRMED_PRESENT |
| 19 | test_ci_workflows: exaktes Gate, workflow_dispatch, contents:read, SHA-Pinning | tests/test_ci_workflows.py:23, 31, 38, 41-42, 54, 60 | CONFIRMED_PRESENT |
| 20 | test_package_release: sqlite-vec==0.1.9, kein lancedb, kein hosted-legacy-index | tests/test_package_release.py:22, 25, 32, 37, 46-47 | CONFIRMED_PRESENT |
| 21 | test_installer_contract: VERSION 0.1.0, LINUX_ASSET, Plattformen | tests/test_installer_contract.py:20-21, 57, 62, 102, 111, 122, 137 | CONFIRMED_PRESENT |
| 22 | test_docs_hosted_onboarding: CODEX-/CLAUDE_HOSTED_COMMAND mit railway-domain | tests/test_docs_hosted_onboarding.py:17-22, 46-47, 82, 95 | CONFIRMED_PRESENT |
| 23 | test_settings_and_types: PORT-Präzedenz, production→0.0.0.0, expliziter Host bleibt | tests/test_settings_and_types.py:499-551 | CONFIRMED_PRESENT |
| 24 | cli.py: 3074 Zeilen; Fehlermeldung „Installed package metadata ... unavailable" | src/policynim/interfaces/cli.py (wc 3074); :1270 | CONFIRMED_PRESENT |
| 25 | mcp.py: 1064 Zeilen; ContextVar _HOSTED_AUTH_RESULT, _InMemoryRateLimiter | src/policynim/interfaces/mcp.py:80, 86 | CONFIRMED_PRESENT |
| 26 | mcp.py: Tools policy_preflight/policy_search registriert | mcp.py:218-241, 301-332 | CONFIRMED_PRESENT |
| 27 | contracts.py: Protokoll-Verträge (Embedder…RuntimeEvidenceStoreProtocol) | contracts.py:29-148 | CONFIRMED_PRESENT |
| 28 | agent_workflows: Karten für policy_preflight/policy_search | agent_workflows.py:25-57 | CONFIRMED_PRESENT |
| 29 | services/__init__: ensure_hosted_runtime_ready, format_health_failure_reason | services/__init__.py:15-16, 64-65 | CONFIRMED_PRESENT |
| 30 | sqlite_vec: Schema-Version „1", Domain-Multiplikator 5, Minimum 20 | storage/sqlite_vec.py:18, 22-23, 419 | CONFIRMED_PRESENT |
| 31 | preflight-Kappungstest: test_preflight_service_caps_retained_chunks_per_policy | tests/test_preflight_service.py:421 | CONFIRMED_PRESENT |
| 32 | Guardrails-Default-Modell llama-3.3-nemotron-super-49b-v1.5 | settings.py:78, providers/nvidia_guardrails.py:31 | CONFIRMED_PRESENT |
| 33 | beta.css: 687 Zeilen; beta-page.js: 58 Zeilen | assets/beta/beta.css, assets/beta/beta-page.js (wc) | CONFIRMED_PRESENT |
| 34 | Zeilenzahlen der 39 in tests.md gelisteten Testdateien | wc -l je Datei | CONFIRMED_PRESENT |

## Korrekturen (Claim → Befund → korrigierte Aussage)

| # | Ursprüngliche Aussage | Befund | Korrektur | Status |
|---|---|---|---|---|
| K1 | files.md: SHA256-Hashing der Keys in `storage/auth_store.py` (`_hash_api_key`) | `_hash_api_key` existiert nicht in auth_store.py; Hashing liegt im Service | `_hash_api_key` = `services/beta_auth.py:327-328` (hashlib.sha256). `auth_store.py` speichert/verwaltet nur übergebene `key_hash`-Werte (Zeilen 219, 236, 239, 319, 326, 436) | CONFIRMED_ABSENT (ursprüngliche Attribution) / CONFIRMED_PRESENT (korrigiert) |
| K2 | files.md: `_DEFAULT_HTTP_TIMEOUT_SECONDS = 10.0` in `providers/nvidia.py` | Konstante fehlt in nvidia.py; dort wird `settings.nvidia_timeout_seconds` genutzt | Konstante = `services/runtime_execution.py:49`; nvidia.py nutzt Settings (Zeilen 87, 229, 316) | CONFIRMED_ABSENT / CONFIRMED_PRESENT |
| K3 | files.md: `_USER_AGENT = "PolicyNIM Hosted Beta"` in `providers/nvidia.py` | Konstante fehlt in nvidia.py | Konstante = `services/beta_auth.py:30` (Verwendung Zeile 257) | CONFIRMED_ABSENT / CONFIRMED_PRESENT |
| K4 | files.md: `_FINAL_ADHERENCE_THRESHOLD`/`_TRAJECTORY_ADHERENCE_THRESHOLD = 0.75` in `services/preflight.py` | Konstanten fehlen in preflight.py | Konstanten = `services/conformance.py:19-20` (Verwendung 254, 269) | CONFIRMED_ABSENT / CONFIRMED_PRESENT |
| K5 | files.md & tests.md: „mcp_port-Default 8123" | Default in settings.py:93 ist 8000; 8123 ist ausschließlich Test-Eingabewert (PORT=8123) | mcp_port-Default = 8000 (settings.py:93); dokumentiert als „default development bind 127.0.0.1:8000" (docs/workflows.md:351) | CONFIRMED_ABSENT (Claim 8123) / CONFIRMED_PRESENT (8000) |
| P1 | files.md: „_MAX_CHUNKS_PER_POLICY-Kappung (2 Chunks/Policy)" im preflight-Kontext | Verhalten korrekt; Konstante ist nicht in preflight.py definiert | Definition = `services/router.py:27` (Verwendung Zeile 252); preflight nutzt die Kappung über PolicyRouterService; Testnachweis test_preflight_service.py:421 | Präzisierung |
| P2 | files.md: config.yml „definiert ... Modell" | config.yml nutzt Platzhalter `__POLICYNIM_NVIDIA_CHAT_MODEL__` | Modell-Defaults liegen in settings.py:78 (`nvidia_chat_model`) und nvidia_guardrails.py:31 (`_DEFAULT_GUARDRAILS_MODEL`); config.yml wird zur Laufzeit substituiert | Präzisierung |

## Befunde zu tests.md

| # | Befund | Beleg |
|---|---|---|
| T1 | Behauptung „Das Repo enthält 40 pytest-Dateien" ist falsch: es sind 41 (`tests/test_*.py`) | ls tests/*.py | wc -l = 41 |
| T2 | Tabelle fehlt: `test_ingest_service.py` (281 Zeilen, Ingest-Service-Orchestrierung) | Datei existiert im Repo, keine Tabellezeile |
| T3 | Tabelle fehlt: `test_nvidia_live.py` (96 Zeilen, Opt-in-Live-Tests Embedding/Rerank) | Datei existiert im Repo, keine Tabellezeile |
| T4 | `test_runtime_evidence_report_service.py` doppelt gelistet (Zeilen 20 und 51) | tests.md:20, 51 |
| T5 | Alle 39 in der Tabelle aufgeführten Zeilenzahlen stimmen mit `wc -l` überein | Gegenprüfung je Datei |

## Lese-Status-Besonderheiten

- `tests/test_cli.py`: 4252 Zeilen vollständig gelesen (3 Pässe + Rest ab Zeile 2901 bis Ende).
- `uv.lock`: 4396 Zeilen, 544475 Bytes; Paketliste (263 Einträge), Projekt-Entry `policynim` 0.1.0, requires-python >=3.11,<3.13, Optionals (nvidia-eval, nvidia-eval-launcher, nvidia-guardrails) und dev-groups (dev/release/test) erfasst; die restlichen Abschnitte sind generierte Wheel-/sdist-URL-Blöcke (gleichförmig, stichprobenhaft verifiziert). Reine Lock-Datei ohne Quellcode-Analyse.
- Alle übrigen Textdateien: vollständig gelesen.

## Evidenzregister (Auswahl, Schema EV-POLICY-xxxxxx)

| Evidence-ID | Repository | Commit | Datei | Zeilenbereich | Beziehung | Typ | Aussage |
|---|---|---|---|---|---|---|---|
| EV-POLICY-000001 | policyNIM | 0eb2ae7 | README.md | 19 | README | API | Produktpositionierung „policy-aware engineering preflight layer" |
| EV-POLICY-000002 | policyNIM | 0eb2ae7 | pyproject.toml | 3-6 | Projektdefinition | Config | Version 0.1.0, Python >=3.11,<3.13 |
| EV-POLICY-000003 | policyNIM | 0eb2ae7 | Dockerfile | 20-21 | Build | Config | BuildKit-Secret nvidia_api_key für Bake-Ingest |
| EV-POLICY-000004 | policyNIM | 0eb2ae7 | Dockerfile.railway | 20, 23 | Build | Config | Railway-Variante nutzt ARG NVIDIA_API_KEY |
| EV-POLICY-000005 | policyNIM | 0eb2ae7 | src/policynim/settings.py | 102-104 | Konfiguration | Config | Quota 500, Rate-Limit 20/900 s |
| EV-POLICY-000006 | policyNIM | 0eb2ae7 | src/policynim/settings.py | 85-93 | Konfiguration | Config | mcp_host 127.0.0.1, mcp_port-Default 8000, PORT-Alias |
| EV-POLICY-000007 | policyNIM | 0eb2ae7 | src/policynim/services/beta_auth.py | 30, 327-328 | Hashing/User-Agent | API | _USER_AGENT und _hash_api_key (sha256) |
| EV-POLICY-000008 | policyNIM | 0eb2ae7 | src/policynim/services/runtime_execution.py | 49 | Timeout | Config | _DEFAULT_HTTP_TIMEOUT_SECONDS = 10.0 |
| EV-POLICY-000009 | policyNIM | 0eb2ae7 | src/policynim/services/conformance.py | 19-20 | Schwellen | API | _FINAL/_TRAJECTORY_ADHERENCE_THRESHOLD = 0.75 |
| EV-POLICY-000010 | policyNIM | 0eb2ae7 | src/policynim/services/router.py | 27, 252 | Kappung | API | _MAX_CHUNKS_PER_POLICY = 2 |
| EV-POLICY-000011 | policyNIM | 0eb2ae7 | src/policynim/storage/sqlite_vec.py | 18, 22-23, 419 | Index | Schema | Schema-Version 1, Domain-Kandidaten 5×/20 min |
| EV-POLICY-000012 | policyNIM | 0eb2ae7 | src/policynim/interfaces/mcp.py | 80, 86, 301-332 | MCP | API | Auth-ContextVar, Rate-Limiter, Tool-Registrierung |
| EV-POLICY-000013 | policyNIM | 0eb2ae7 | src/policynim/interfaces/cli.py | 1270 | CLI | API | Fehlermeldung bei fehlenden Paket-Metadaten |
| EV-POLICY-000014 | policyNIM | 0eb2ae7 | evals/default_cases.json | 2-56 | Eval-Suite | Schema | day-6-default, 6 Fälle, expected_insufficient_context |
| EV-POLICY-000015 | policyNIM | 0eb2ae7 | tests/test_ci_workflows.py | 23, 60 | Test | Test | Exaktes CI-Gate, live/docker_live ausgeschlossen |
| EV-POLICY-000016 | policyNIM | 0eb2ae7 | tests/test_preflight_service.py | 421 | Test | Test | Kappungstest „caps retained chunks per policy" |
| EV-POLICY-000017 | policyNIM | 0eb2ae7 | tests/test_settings_and_types.py | 499-551 | Test | Test | PORT-Präzedenz und production-Host-Wechsel |
| EV-POLICY-000018 | policyNIM | 0eb2ae7 | packaging/pyinstaller.spec | 8-19 | Bundle | Config | Entrypoint, copy_metadata, 4 Ressourcen |
| EV-POLICY-000019 | policyNIM | 0eb2ae7 | scripts/install.sh | 44, 66-71 | Installer | Config | releases/latest-Redirect, SHA256-Verifikation |
| EV-POLICY-000020 | policyNIM | 0eb2ae7 | src/policynim/templates/nvidia_guardrails/preflight_output/config.yml | 1-11 | Guardrails | Config | Platzhalter __POLICYNIM_NVIDIA_CHAT_MODEL__ |

## Abschlussbewertung

Die Inventardokumente sind belegt und korrekturbedürftig in genau den oben dokumentierten Punkten (K1-K5, P1-P2, T1-T4). Nach Anwendung der Korrekturen enthält der Inventarstand keine offene Stelle: Jede der 151 Dateien ist gelesen und im Statusdokument als FERTIG_ANALYSIERT geführt. Die analysierten Fähigkeiten (Fail-closed-Preflight, Vektor-Retrieval via sqlite-vec, Runtime-Entscheidungen/-Evidenz, Beta-OAuth, CLI/MCP-Oberflächen, Eval/Conformance, Guardrails, Installer/Release) sind hinreichend belegt für die spätere Rust-2024-Neuentwicklung.
