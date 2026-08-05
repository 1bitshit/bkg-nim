# PolicyNIM — Datei-Inventar (files.md)

Für jede Datei des Repos werden die Pflichtfelder dokumentiert. Belegquellen sind die jeweiligen Dateien selbst und ihr Zusammenspiel (belegt z. B. durch `src/policynim/settings.py`).

Feldabkürzungen: Z=Zweck, V=Verantwortlichkeit, E=Eingaben, A=Ausgaben, DF=Datenfluss, P=Persistenz, ZU=Zustände, API=APIs, ER=Ereignisse, NW=Nebenwirkungen, F=Fehlerfälle, S=Sicherheitsrelevanz, G=Geschäftslogik, AL=Algorithmen, DM=verwendete Datenmodelle, AB=Abhängigkeiten, R=Rust-Relevanz.

---

## README.md

- Zweck: Projektübersicht, Installationsanleitungen (pipx/uv tool/GitHub-Installer), Quickstart, Docs-Map, Limits.
- Verantwortlichkeit: Kurzdokumentation für Erstnutzer; verweist auf die detaillierten Docs.
- Eingaben: Keine programmatischen Eingaben; Markdown-Datei als Installations-/Orientierungsquelle.
- Ausgaben: Keine; dokumentiert Kommandos wie `policynim init`, `ingest`, `search`, `route`, `compile`, `preflight`, `eval`, `mcp`.
- Datenfluss: Dokumentation → Nutzeraktion (Copy-Paste-Kommandos).
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine; referenziert CLI- und MCP-Oberflächen.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine; verweist auf `docs/limitations.md` für bewusste Produktgrenzen.
- Sicherheitsrelevanz: Erwähnt Checksummen-Verifikation der Installer; `NVIDIA_API_KEY`-Handling.
- Geschäftslogik: Keine; Produktpositionierung als "policy-aware engineering preflight layer".
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Verweist auf `pyproject.toml`, `docs/*`, `tests/README.md`, `examples/*`.
- Rust-Relevanz: Nicht erkennbar.

---

## LICENSE

- Zweck: MIT-Lizenztext, Copyright (c) 2026 Nnenna Ndukwe.
- Verantwortlichkeit: Rechtliche Freigabe der Nutzung.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine (Rechtstext).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## pyproject.toml

- Zweck: Build- und Paketdefinition (hatchling), Abhängigkeiten, Dependency-Groups, Ruff-/Pytest-Konfiguration, CLI-Entrypoint.
- Verantwortlichkeit: Definierter Projektvertrag: Python `>=3.11,<3.13`, Version `0.1.0`, Package `src/policynim`.
- Eingaben: Keine zur Laufzeit; Build-Input für uv/hatchling.
- Ausgaben: Wheel, sdist; Console-Script `policynim` → `policynim.interfaces.cli:main`.
- Datenfluss: Build-System → installiertes Paket; `force-include` packt `policies` und `evals` ins Wheel.
- Persistenz: Keine zur Laufzeit.
- Zustände: Keine.
- APIs: Console-Script-Definition.
- Ereignisse: Keine.
- Nebenwirkungen: Wheel enthält `policynim/policies` und `policynim/evals` (gebündelte Ressourcen).
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine; deklariert optionale Extras (`nvidia-guardrails`, `nvidia-eval`, `nvidia-eval-launcher`).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: arize-phoenix, arize-phoenix-evals, httpx, itsdangerous, jinja2, markdown-it-py, mcp[cli], openai, pandas, platformdirs, pydantic, pydantic-settings, sqlite-vec, typer; Dev/Test: pytest, pre-commit, pyright, ruff; Release: pyinstaller.
- Rust-Relevanz: Nicht erkennbar.

---

## uv.lock

- Zweck: Deterministischer Lockfile für uv; exakt gepinnte transitive Abhängigkeiten.
- Verantwortlichkeit: Reproduzierbare Umgebungen (CI, Docker, local).
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Keine zur Laufzeit.
- Datenfluss: `uv sync --frozen` in CI/Docker nutzt den Lockfile.
- Persistenz: Lock-Datei im Repo.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Vollständiger Abhängigkeitsgraph; Top-Level siehe `pyproject.toml`.
- Rust-Relevanz: Nicht erkennbar. (Hinweis: reine Lock-Datei, keine Quellcode-Analyse; Zusammenfassung erlaubt laut Auftrag.)

---

## pyrightconfig.json

- Zweck: Pyright-Typchecker-Konfiguration: `include` src+tests, Python 3.11, venv `.venv`, standard type checking.
- Verantwortlichkeit: Statische Typprüfung als Quality Gate (CI und pre-commit).
- Eingaben: Python-Dateien unter `src/` und `tests/`.
- Ausgaben: Pyright-Bericht/Exit-Code.
- Datenfluss: CI (`uv run pyright`) → Typprüfungs-Ergebnis.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pyright (dev group).
- Rust-Relevanz: Nicht erkennbar.

---

## railway.toml

- Zweck: Railway-Deploy-Vertrag: Dockerfile-Builder mit `Dockerfile.railway`, `/healthz`-Healthcheck, Restart-Policy, 1 Replica.
- Verantwortlichkeit: Plattform-Konfiguration für das gehostete Beta-Deployment.
- Eingaben: Railway-Deploy-Ereignis.
- Ausgaben: Deployment-Konfiguration.
- Datenfluss: Railway → Dockerfile.railway → Container → `/healthz`-Probe.
- Persistenz: Keine.
- Zustände: Healthcheck-Timeout 60s, Restart on failure max. 3, numReplicas 1.
- APIs: Keine.
- Ereignisse: Healthcheck-Ereignisse.
- Nebenwirkungen: Keine.
- Fehlerfälle: Nicht nachweisbar.
- Sicherheitsrelevanz: Keine direkt; Healthcheck-Endpunkt öffentlich.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `Dockerfile.railway`.
- Rust-Relevanz: Nicht erkennbar.

---

## Dockerfile

- Zweck: Produktions-Image für gehostetes HTTP (streamable-http); bäckt den SQLite-Index in das Image.
- Verantwortlichkeit: Zwei-Stufen-Build (builder/runtime); Bake-Time-Ingest mit BuildKit-Secret.
- Eingaben: Repo-Inhalte (pyproject, uv.lock, src, policies, evals); BuildKit-Secret `nvidia_api_key`.
- Ausgaben: Image mit `/app/data/index.sqlite3`, `.venv`, `policynim mcp --transport streamable-http`.
- Datenfluss: Build → `uv sync --frozen` → `policynim ingest` (bake) → Runtime-Kopie.
- Persistenz: `/app/data/index.sqlite3` (baked); `/app/state` erwartet als Volume (via Env, siehe `.env.production.example`).
- Zustände: Keine im Dockerfile; Runtime-Zustand via Env gesteuert.
- APIs: CMD `policynim mcp --transport streamable-http`.
- Ereignisse: Build-Ereignisse.
- Nebenwirkungen: Secret wird im Builder gemountet, nicht im finalen Image gespeichert (belegt durch hosted-beta-operations.md).
- Fehlerfälle: Fehlendes/leeres Secret → Build-Fehler während `policynim ingest`.
- Sicherheitsrelevanz: `NVIDIA_API_KEY` nur als Build-Secret, nie im finalen Image; `PYTHONDONTWRITEBYTECODE`, `PYTHONUNBUFFERED`.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `python:3.11.15-slim-trixie` (SHA256-pinned), uv 0.7.12.
- Rust-Relevanz: Nicht erkennbar.

---

## Dockerfile.railway

- Zweck: Railway-kompatible Variante des Dockerfiles (Railway unterstützt keine Secret-Mounts außer Cache-Mounts).
- Verantwortlichkeit: Gleiche Struktur wie `Dockerfile`, aber Bake-Time-Key via `ARG NVIDIA_API_KEY`.
- Eingaben: Repo-Inhalte; Build-Arg `NVIDIA_API_KEY`.
- Ausgaben: Image wie Dockerfile; CMD `policynim mcp --transport streamable-http`.
- Datenfluss: Build → `uv sync --frozen` → `NVIDIA_API_KEY=... uv run policynim ingest` → Runtime-Kopie.
- Persistenz: `/app/data/index.sqlite3` (baked).
- Zustände: Keine.
- APIs: Wie Dockerfile.
- Ereignisse: Build-Ereignisse.
- Nebenwirkungen: Key bleibt als Build-Arg im Build-Kontext; nicht im finalen Image (belegt durch hosted-beta-operations.md).
- Fehlerfälle: Fehlender `NVIDIA_API_KEY` → Bake-Ingest schlägt fehl.
- Sicherheitsrelevanz: Build-Arg statt Secret-Mount; Runtime-Key separat erforderlich.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Wie Dockerfile.
- Rust-Relevanz: Nicht erkennbar.

---

## .env.example

- Zweck: Rückwärtskompatibles lokales Entwicklungs-Env-Template (Alias für Development-Defaults, belegt durch docs/contributor-guide.md).
- Verantwortlichkeit: Referenz für lokale CLI-/Eval-/stdio-MCP-Konfiguration.
- Eingaben: Nutzerkopie → `.env`.
- Ausgaben: Env-Werte für `settings.py`.
- Datenfluss: Datei → pydantic-settings-DotEnv-Source.
- Persistenz: `.env`-Datei.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: `NVIDIA_API_KEY=` leer; MCP-Auth optional auskommentiert.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `settings.py`.
- Rust-Relevanz: Nicht erkennbar.

---

## .env.development.example

- Zweck: Bevorzugtes lokales Entwicklungs-Template (belegt durch docs/contributor-guide.md).
- Verantwortlichkeit: Definition der Development-Defaults (Pfade unter `data/`, `127.0.0.1:6000` MCP, Port 8001 Eval-UI).
- Eingaben: Kopie → `.env`.
- Ausgaben: Env-Werte.
- Datenfluss: Wie `.env.example`.
- Persistenz: `.env`.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine besonderen; Auth-Settings auskommentiert.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `settings.py`.
- Rust-Relevanz: Nicht erkennbar.

---

## .env.production.example

- Zweck: Gehostetes Produktions-Template für Docker/Railway.
- Verantwortlichkeit: Produktions-Defaults: `/app/data/index.sqlite3`, `0.0.0.0`-Host, Beta-Auth-Settings, Rate-Limit-Settings.
- Eingaben: Railway-Service-Variablen-Übersetzung.
- Ausgaben: Env-Werte.
- Datenfluss: Railway-Env → `settings.py`.
- Persistenz: Runtime-Datenpfade (`/app/state` für Evidence und Auth-DB).
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: `BETA_SESSION_SECRET`, GitHub-OAuth-Credentials als optional auskommentierte Secrets.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `settings.py`.
- Rust-Relevanz: Nicht erkennbar.

---

## .gitignore

- Zweck: Ignoriert lokale Artefakte (`.env`, data/, dist/, Caches, `.threadloop/state/`).
- Verantwortlichkeit: Repository-Hygiene.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: `.env` (mit Secrets) bleibt unversioniert; Templates ausgenommen.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## .dockerignore

- Zweck: Hält lokale Zustände und Tools aus dem Docker-Build-Kontext.
- Verantwortlichkeit: Build-Kontext-Hygiene (`.git`, `.env`, data/, dist/, Caches, `.threadloop`, `.voice`).
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Build-Kontext-Filter.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: `.env` mit `NVIDIA_API_KEY` wird nicht in den Build-Kontext kopiert.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## .python-version

- Zweck: Pins die Python-Version auf 3.11 für uv/Pyenv.
- Verantwortlichkeit: Runtime-Versionsvertrag.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: uv → Python-Interpreter-Auswahl.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## .pre-commit-config.yaml

- Zweck: Lokale pre-commit-Hooks: ruff check --fix, ruff format, pyright.
- Verantwortlichkeit: Automatische Qualitätsgates vor Commit.
- Eingaben: Gestagte Python-Dateien.
- Ausgaben: Hook-Erfolg/Fehlschlag.
- Datenfluss: pre-commit → uv run --group dev.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: `ruff check --fix` verändert Dateien automatisch.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pre-commit, ruff, pyright, uv.
- Rust-Relevanz: Nicht erkennbar.

---

## .github/workflows/ci.yml

- Zweck: Standard-CI: Lint (ruff, pyright) und offline pytest (`-m "not live and not docker_live"`).
- Verantwortlichkeit: Automatische Verifikation bei PR und Push auf main.
- Eingaben: GitHub-Ereignisse (pull_request, push main).
- Ausgaben: CI-Status.
- Datenfluss: Checkout → uv setup → sync → Gates.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: PR/Push.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine Secrets; nur `contents: read`.
- Geschäftslogik: CI bleibt offline (keine Live-NVIDIA-Checks).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: actions/checkout, astral-sh/setup-uv, actions/setup-python.
- Rust-Relevanz: Nicht erkennbar.

---

## .github/workflows/hosted-smoke.yml

- Zweck: Manueller "Hosted Beta Smoke"-Workflow (workflow_dispatch) gegen deployed Railway-Service.
- Verantwortlichkeit: Deployment-Promotion-Check mit echten NVIDIA-Backend-Calls.
- Eingaben: Secrets `POLICYNIM_BETA_MCP_URL`, `POLICYNIM_BETA_MCP_TOKEN`.
- Ausgaben: pytest-Ergebnis von `tests/test_hosted_mcp_live.py` (`-m live`).
- Datenfluss: Secrets → Env → Live-Test.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Manuelles Dispatch.
- Nebenwirkungen: Echte MCP-Calls gegen deployed Service.
- Fehlerfälle: Fehlende Secrets → expliziter Abbruch.
- Sicherheitsrelevanz: Gehostete Secrets; keine Ausgabe der Werte.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Wie ci.yml.
- Rust-Relevanz: Nicht erkennbar.

---

## .github/workflows/release.yml

- Zweck: Release-Workflow: verify (lock, ruff, pyright, pytest, build), wheel-smoke, standalone-build (PyInstaller-Matrix linux/darwin-arm64/darwin-amd64/windows), GitHub-Release-Erstellung (draft), PyPI-Publish via Trusted Publishing.
- Verantwortlichkeit: Vollständiger Release-Pipeline-Vertrag.
- Eingaben: Tag `v*.*.*` oder workflow_dispatch mit `version`/`publish_pypi`.
- Ausgaben: Wheel, sdist, Standalone-Archive, `SHA256SUMS`, Draft-Release, PyPI-Paket.
- Datenfluss: Verify → wheel-smoke → standalone-build (Matrix) → publish-github-release → publish-pypi (nur bei Tag oder explizitem Flag).
- Persistenz: GitHub-Release-Assets.
- Zustände: Release-Status (draft).
- APIs: GitHub Release API (via gh), PyPI Trusted Publishing (OIDC).
- Ereignisse: Tag-Push, manuelles Dispatch.
- Nebenwirkungen: PyPI-Veröffentlichung ist unveränderlich; Reihenfolge erzwingt Release zuerst.
- Fehlerfälle: Versionsmismatch zwischen Tag und pyproject → Abbruch.
- Sicherheitsrelevanz: PyPI-Job mit `id-token: write`; GitHub-Token mit `contents: write` nur im Release-Job.
- Geschäftslogik: Versionsauflösung aus Tag/pyproject; Versionen müssen übereinstimmen.
- Algorithmen: Version-Normalisierung (`v`-Präfix).
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: uv, pyinstaller, gh, pypa/gh-action-pypi-publish.
- Rust-Relevanz: Nicht erkennbar.

---

## packaging/pyinstaller.spec

- Zweck: PyInstaller-Spec für Standalone-Bundles.
- Verantwortlichkeit: Bündelt `policynim.interfaces.cli` als `policynim`-Binary samt `policies`, `evals`, `assets`, `templates` als Data.
- Eingaben: CLI-Entrypoint + Paketdaten.
- Ausgaben: Standalone-Bundle-Verzeichnis `dist/policynim/`.
- Datenfluss: Release-Workflow → pyinstaller → Bundle.
- Persistenz: Bundle.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: PyInstaller 6.19.0.
- Rust-Relevanz: Nicht erkennbar.

---

## packmind.json

- Zweck: Leere PackMind-Konfiguration (`packages: {}`, `agents: []`).
- Verantwortlichkeit: Platzhalter/Initialzustand für PackMind-Tooling.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## scripts/install.sh

- Zweck: Unix-Installer für Standalone-Bundles; lädt Release-Asset + `SHA256SUMS`, verifiziert Checksumme, extrahiert, installiert mit Backup/Rollback, schreibt Launcher.
- Verantwortlichkeit: Deterministische, prüfsummengesicherte Installation für darwin-amd64, darwin-arm64, linux-amd64.
- Eingaben: `POLICYNIM_VERSION`/Argument, `POLICYNIM_INSTALLER_TEST_OS/ARCH` (Test), `POLICYNIM_RELEASE_BASE_URL`, `POLICYNIM_REPOSITORY_URL`.
- Ausgaben: Installationsverzeichnis `~/.local/share/policynim/<version>`, Launcher `~/.local/bin/policynim`, PATH-Hinweis.
- Datenfluss: curl → SHA256-Verifikation → tar-Extraktion → Staging → atomarer Ersatz mit Backup.
- Persistenz: Installierte Binärdateien.
- Zustände: Backup-/Staging-Zustände während Installation.
- APIs: GitHub Release Download.
- Ereignisse: Installationserfolg/Fehlschlag.
- Nebenwirkungen: Ersetzt bestehende Installation (mit Backup-Rollback); `rm -rf` in Work-Verzeichnis.
- Fehlerfälle: Fehlende Kommandos, unsupported platform, Checksum-Mismatch, fehlender Checksum-Eintrag, fehlendes Binary → `fail` mit Exit 1; Rollback bei fehlgeschlagenem Ersatz.
- Sicherheitsrelevanz: Checksummen-Verifikation vor Installation; keine Prompt-/Secret-Erfassung.
- Geschäftslogik: Keine.
- Algorithmen: SHA256-Vergleich, Version-Auflösung via `releases/latest`-Redirect.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: curl, tar, awk, sha256sum/shasum.
- Rust-Relevanz: Nicht erkennbar.

---

## scripts/install.ps1

- Zweck: Windows-Installer (PowerShell) für `windows-amd64`-Bundle; analog zu install.sh mit SHA256-Verifikation und Backup/Rollback.
- Verantwortlichkeit: Installationspfad für Windows.
- Eingaben: `$Version`, `POLICYNIM_INSTALLER_TEST_ARCH`, Env-Overrides.
- Ausgaben: `%LocalAppData%\PolicyNIM\<version>`, Launcher `policynim.cmd`.
- Datenfluss: Invoke-WebRequest → Get-FileHash → Expand-Archive → Staging → Ersatz.
- Persistenz: Installierte Dateien.
- Zustände: Backup-/Staging-Zustände.
- APIs: GitHub Release Download.
- Ereignisse: Installationserfolg/Fehlschlag.
- Nebenwirkungen: Ersetzt Installationen mit Rollback; Work-Dir wird im finally entfernt.
- Fehlerfälle: Checksum-Mismatch, fehlende Assets, unerwartete Architektur → `Stop-Install`.
- Sicherheitsrelevanz: SHA256-Verifikation.
- Geschäftslogik: Keine.
- Algorithmen: SHA256-Vergleich.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: PowerShell-Cmdlets.
- Rust-Relevanz: Nicht erkennbar.

---

## .codex/agents/entire-search.toml

- Zweck: Codex-Subagent-Definition "entire-search" für historische Suche über Entire-Checkpoints/Transcripts.
- Verantwortlichkeit: Fokussierte Suche ausschließlich via `entire search --json`.
- Eingaben: Nutzerfragen zu früherer Arbeit.
- Ausgaben: Evidenzbasierte Zusammenfassungen.
- Datenfluss: Agent → `entire search --json` → Ergebnisse.
- Persistenz: Keine.
- Zustände: Read-only-Sandbox.
- APIs: `entire search --json`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine (read-only).
- Fehlerfälle: Fehlende Auth/Setup → Prerequisite-Meldung, keine Repo-Änderungen.
- Sicherheitsrelevanz: Prompt-Injection-Schutz: Nutzertext als Daten behandeln.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Entire CLI.
- Rust-Relevanz: Nicht erkennbar.

---

## .codex/config.toml

- Zweck: Codex-Konfiguration; aktiviert Hooks-Feature.
- Verantwortlichkeit: Feature-Schalter.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Codex → hooks.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Codex.
- Rust-Relevanz: Nicht erkennbar.

---

## .codex/hooks.json

- Zweck: Codex-Hook-Registrierung für Entire (PostToolUse, SessionStart, Stop, UserPromptSubmit).
- Verantwortlichkeit: Session-/Tool-Lebenszyklus-Events an Entire CLI weiterreichen.
- Eingaben: Codex-Events.
- Ausgaben: `entire hooks codex <event>`-Aufrufe.
- Datenfluss: Codex → sh → entire.
- Persistenz: Keine (Entire verwaltet Checkpoints extern).
- Zustände: Session-Events.
- APIs: Entire CLI.
- Ereignisse: PostToolUse, SessionStart, Stop, UserPromptSubmit.
- Nebenwirkungen: Checkpoint-/Transcript-Aufzeichnung durch Entire; SystemMessage-Hinweis bei fehlender Installation.
- Fehlerfälle: Entire fehlt → Hook bricht mit Exit 0 ab.
- Sicherheitsrelevanz: Keine direkte; Session-Transkripte landen bei Entire.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Entire CLI.
- Rust-Relevanz: Nicht erkennbar.

---

## .entire/settings.json

- Zweck: Entire-Tool-Konfiguration: GitHub-Checkpoint-Repo `nnennandukwe/policyNIM-entire-checkpoints`, push_sessions, Telemetrie.
- Verantwortlichkeit: Externe Session-/Checkpoint-Persistenz.
- Eingaben: Keine zur Laufzeit.
- Ausgaben: Checkpoint-Pushes.
- Datenfluss: Entire → GitHub.
- Persistenz: Remote-Checkpoint-Repo.
- Zustände: Keine.
- APIs: GitHub.
- Ereignisse: Checkpoint-Pushes.
- Nebenwirkungen: Externe Zustandsablage.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Telemetrie aktiv (belegt durch `"telemetry": true`); externe Zustandsablage.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Entire CLI.
- Rust-Relevanz: Nicht erkennbar.

---

## .entire/.gitignore

- Zweck: Ignoriert Entire-lokale Dateien (tmp, settings.local.json, metadata, logs, redactors/local).
- Verantwortlichkeit: Lokale Entire-Zustände aus Versionierung halten.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Redactor-Lokale Daten aus Versionierung.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## .threadloop/config.json

- Zweck: ThreadLoop-Tool-Metadaten (version 1, createdAt 2026-03-23).
- Verantwortlichkeit: Tool-Initialisierung.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/index.md

- Zweck: Dokumentations-Hub nach Zielgruppe und Aufgabe.
- Verantwortlichkeit: Navigation zu README, Contributor Guide, Workflows, Hosted-Beta-Ops, Release, Architektur, Demos, Limits, Tests, Examples.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Verlinkte Docs.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/architecture.md

- Zweck: Ausführliche Architekturbeschreibung: Design-Prinzipien, Ingest-/Retrieval-/Routing-/Compiler-/Preflight-/Regenerations-/Eval-Flows, Package-Boundaries, Import-Regeln, Public Interfaces, Runtime-Boundaries, Failure-States.
- Verantwortlichkeit: Verbindliche Architektur- und Layering-Doku (welches Modul was importieren darf).
- Eingaben: Keine; dokumentiert Quellcode.
- Ausgaben: Keine.
- Datenfluss: Beschreibt alle Hauptdatenflüsse (Korpus→Index→Suche→Route→Compile→Preflight→Regeneration→Eval; Runtime-Decisions).
- Persistenz: Dokumentiert Persistenzpunkte (`data/`, SQLite, Artefakte).
- Zustände: Dokumentiert fail-closed-Zustände und `insufficient_context`.
- APIs: Vollständige CLI-/MCP-/HTTP-Oberflächenliste.
- Ereignisse: Keine direkten; dokumentiert Orchestrierungsflüsse.
- Nebenwirkungen: Keine.
- Fehlerfälle: Liste der explizit zu haltenden Failure-States.
- Sicherheitsrelevanz: Hosted-Auth-Regeln (`/mcp` only), Secrets, Public-Endpoint-Regeln.
- Geschäftslogik: Fail-closed-Grounding-Regeln, Regenerations-Bounds, Eval-Regeln.
- Algorithmen: Task-Profile-Inferenz (deterministisch), deterministische Chunk-IDs, Zitat-Validierung.
- verwendete Datenmodelle: Beschreibt alle Typ-Pakete (SearchResult, PolicySelectionPacket, CompiledPolicyPacket, PreflightResult, PolicyEvidenceTrace, RuntimeActionRequest u. a.).
- Abhängigkeiten: `architecture-diagram.md`.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/architecture-diagram.md

- Zweck: Mermaid-Diagramme für Package-Boundaries und Runtime-Flow.
- Verantwortlichkeit: Visuelle Karte der Komponenten/Abhängigkeiten.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Zwei Flowcharts (Modul-Boundary-Map, Runtime-Flow).
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Mermaid-Rendering.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/contributor-guide.md

- Zweck: Lokales Setup (uv sync, init, Env-Templates), Standalone-Install, Runtime-Settings-Referenz, optionale NVIDIA-Pakete, Default-Modelle, Qualitätsgates.
- Verantwortlichkeit: Entwickler-Onboarding.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine; referenziert CLI.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Installer sammeln keine API-Keys; Secrets-Hinweise.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: uv, pre-commit, ruff, pyright, pytest.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/workflows.md

- Zweck: Kommando-Handbuch: CLI, Quickstart, MCP, Runtime/Evidence, Eval, Troubleshooting, Retrieval-Modell.
- Verantwortlichkeit: Operative Referenz.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Dokumentiert jeden Workflow (init→ingest→dump→search→route→compile→preflight→eval→mcp→runtime→evidence).
- Persistenz: Dokumentiert Datenpfade und SQLite-Tabellen.
- Zustände: Exit-Codes von `runtime execute` (0 allowed/confirmed; 1 blocked/refused/failed).
- APIs: Vollständige CLI-/MCP-/HTTP-Surface.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Troubleshooting-Sektion (Search funktioniert, Preflight fail-closed; negative Scores; fehlender Index; fehlende Credentials).
- Sicherheitsrelevanz: Audit-Log-Redaktion, Bearer-Auth-Nuancen.
- Geschäftslogik: Runtime-Rules-Authoring-Vertrag (action/effect/reason + genau eine Matcher-Familie).
- Algorithmen: Keine.
- verwendete Datenmodelle: `RuntimeActionRequest`, `HealthCheckResult` u. a.
- Abhängigkeiten: Alle CLI-Module.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/hosted-beta-operations.md

- Zweck: Gehosteter Beta-Betrieb: Self-Serve-Flow, 60-/90-Tage-Verifikation, Recovery, Container-Build, Railway-Deploy.
- Verantwortlichkeit: Betriebs- und Recovery-Doku.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Dokumentiert Docker/Railway-Deploy- und Smoke-Flows.
- Persistenz: `/app/state`-Volume (Auth-DB, Evidence-DB).
- Zustände: `/mcp`-Antworten 401/403/429; `/healthz` 200/503.
- APIs: `/beta`, `/auth/github/*`, `/beta/api-key/regenerate`, `/beta/logout`, `/healthz`, `/mcp`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Invalid Token, Upstream-NVIDIA-Failure, Insufficient Context, Service Unavailable.
- Sicherheitsrelevanz: Bearer-Tokens, Break-Glass-Tokens, Session-Secret, GitHub-OAuth, Audit-Redaktion.
- Geschäftslogik: Tagesquota (Standard 500), Rate-Limit (20 Versuche/900s).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `Dockerfile`, `Dockerfile.railway`, `railway.toml`.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/demo-script.md

- Zweck: Schritt-für-Schritt-Demo des Hero-Use-Cases (Refresh-Token-Cleanup-Background-Job).
- Verantwortlichkeit: Präsentations-/Demonstrationsskript.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine; referenziert CLI-/MCP-Kommandos.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Recovery-Notizen.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/limitations.md

- Zweck: Produktlimits und Nicht-Ziele: local-first, Beta-einfache Auth, NVIDIA-Abhängigkeit, offline CI, schmale Retrieval-/Korpus-Breite, fail-closed-Verhalten, Gold-Case-Evals, kein Review-/Approval-Layer.
- Verantwortlichkeit: Ehrliche Grenzdokumentation.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Dokumentiert Auth-Grenzen (nur `/mcp` geschützt).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/public-source-grounding.md

- Zweck: Provenienz-Nachweise des synthetischen Korpus (öffentliche Standards statt interner Dokumente).
- Verantwortlichkeit: Transparenz über die Quelle jeder Policy.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Hält das Repo öffentlich-sicher (kein kopiertes proprietäres Material).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `policies/**`.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/release.md

- Zweck: Release-Checkliste: Gates, GitHub-Release, PyPI (Trusted Publishing), optionaler Hosted-Smoke.
- Verantwortlichkeit: Release-Prozess-Doku.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Tag-Push.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: PyPI-OIDC-Trusted-Publishing.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: `.github/workflows/release.yml`.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/ai-engineer-miami-context-plane.md

- Zweck: Vortrags-/Projekt-Framing: zentralisierte Context-Plane als fehlende Schicht in KI-Coding-Workflows.
- Verantwortlichkeit: Konzeptionelle Einordnung des Produkts.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/extreme-programming-with-agents.md

- Zweck: XP/TDD/BDD-Workflow-Notizen für agentisches Entwickeln (Inner/Outer Loop, Skill-Design-Prompt).
- Verantwortlichkeit: Methodik-Doku.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## docs/assets/readme/policynim-beta-dark-landing-preview.png

- Zweck: Landing-Page-Preview des Beta-Portals (Dark Mode) für README.
- Verantwortlichkeit: Visuelle Vorschau des gehosteten Beta-Portals.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: README-Referenz.
- Persistenz: PNG-Asset (395170 Bytes, 1280x1273, RGB, non-interlaced; belegt durch `file`).
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: README.md.
- Rust-Relevanz: Nicht erkennbar. (Hinweis: visueller Inhalt aus der Datei selbst für dieses Modell nicht prüfbar; Verwendung als Landing-Preview belegt durch `README.md` Zeile 110.)

---

## policies/TEMPLATE.md

- Zweck: Autorenleitfaden für neue Policy-Dokumente (Frontmatter-Form, Sektionen, optionale `runtime_rules`).
- Verantwortlichkeit: Template-Vertrag für das Korpus (kein Laufzeit-Dokument).
- Eingaben: Autoren.
- Ausgaben: Neue Policy-Markdowns.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Matcher-Familien-Regel: genau eine von `path_globs`/`command_regexes`/`url_host_patterns`; `allow` ist kein authored effect.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter-Felder (policy_id, title, doc_type, domain, tags, grounded_in, runtime_rules).
- Abhängigkeiten: Parser-Erwartungen (belegt durch `src/policynim/ingest/parser.py`-Analyse).
- Rust-Relevanz: Nicht erkennbar.

---

## policies/architecture/api-versioning-guidance.md

- Zweck: Policy `ARCH-API-001` — API-Versionierung: additive vs. breaking changes, Deprecation-Pläne, Vertragstests.
- Verantwortlichkeit: Inhaltliche Regelquelle für Retrieval/Preflight.
- Eingaben: Keine (Korpus-Input für Ingest).
- Ausgaben: Keine (Chunks nach Ingest).
- Datenfluss: Korpus → Ingest → Index → Retrieval.
- Persistenz: Chunked im SQLite-Index nach Ingest.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine direkt.
- Geschäftslogik: Versionierungsregeln für API-Änderungen.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `policy_id: ARCH-API-001`, `domain: architecture`.
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## policies/architecture/background-job-design-rules.md

- Zweck: Policy `ARCH-JOB-001` — Background-Job-Regeln: Idempotenz, begrenzte Retries mit Backoff, Operator-Sichtbarkeit, kein Secret in Payloads, begrenzte Concurrency, Heartbeats.
- Verantwortlichkeit: Regelquelle für Job-Design.
- Eingaben: Korpus.
- Ausgaben: Chunks.
- Datenfluss: Korpus → Ingest.
- Persistenz: Index.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: "Job payloads must not contain raw secrets".
- Geschäftslogik: Retry-/Idempotenz-/Observability-Regeln.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `ARCH-JOB-001`.
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## policies/backend/backend-logging-standard.md

- Zweck: Policy `BE-LOG-001` — strukturiertes Logging, stabile IDs, Secret-/PII-Redaktion, Operator-Actionability.
- Verantwortlichkeit: Logging-Regelquelle.
- Eingaben: Korpus.
- Ausgaben: Chunks.
- Datenfluss: Korpus → Ingest.
- Persistenz: Index.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Nie Secrets/Tokens/Session-Cookies loggen; PII-Redaktion.
- Geschäftslogik: Logging-Standards.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `BE-LOG-001`.
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## policies/backend/config-validation-and-fail-closed.md

- Zweck: Policy `BE-CONFIG-001` — zentrale Startup-Validierung, fail-closed, sichere Fehlermeldungen.
- Verantwortlichkeit: Konfigurationsregelquelle.
- Eingaben: Korpus.
- Ausgaben: Chunks.
- Datenfluss: Korpus → Ingest.
- Persistenz: Index.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Kein Secret-Abdruck in Fehlermeldungen.
- Geschäftslogik: Konfigurations-Validierungsregeln.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `BE-CONFIG-001`.
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## policies/backend/request-correlation-and-tracing-standard.md

- Zweck: Policy `BE-TRACE-001` — Korrelations-ID-Propagation, Spans in Logs, keine Secrets in Trace-Metadaten.
- Verantwortlichkeit: Tracing-Regelquelle.
- Eingaben: Korpus.
- Ausgaben: Chunks.
- Datenfluss: Korpus → Ingest.
- Persistenz: Index.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Tracing-Metadaten ohne Secrets/Tokens/Payloads.
- Geschäftslogik: Korrelationsstrategie-Regeln.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `BE-TRACE-001`.
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## policies/security/auth-sensitive-code-review-standard.md

- Zweck: Policy `SEC-AUTH-001` — Threat-Model-Pflicht bei Auth-Änderungen, Token-Lebenszyklus-Doku, fail-closed-Präferenz, unabhängiger Reviewer.
- Verantwortlichkeit: Auth-Code-Review-Regelquelle.
- Eingaben: Korpus.
- Ausgaben: Chunks.
- Datenfluss: Korpus → Ingest.
- Persistenz: Index.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Auth-/Token-/Session-Handling-Regeln.
- Geschäftslogik: Review-Bar-Regeln.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `SEC-AUTH-001`.
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## policies/security/public-endpoint-safety.md

- Zweck: Policy `SEC-PUBLIC-ENDPOINT-002` — öffentliche Endpunkte ohne rohe Exceptions/Pfade/Secrets; allowlist-sanitisierte Fehlermeldungen; zentrale Settings.
- Verantwortlichkeit: Sicherheitsregelquelle für öffentliche HTTP-Surfaces.
- Eingaben: Korpus.
- Ausgaben: Chunks.
- Datenfluss: Korpus → Ingest.
- Persistenz: Index.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Kernregeln für `/healthz`-/`/mcp`-Antworten (stimmt mit `interfaces/mcp.py`-Verhalten überein).
- Geschäftslogik: Sanitisierungs-/Fail-closed-Regeln.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `SEC-PUBLIC-ENDPOINT-002`.
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## policies/security/secrets-redaction-and-handling.md

- Zweck: Policy `SEC-SECRET-001` — Secrets in engstem Boundary, Redaktion vor Logs/Traces, Mock-Werte in Fixtures, Rotation dokumentiert.
- Verantwortlichkeit: Secret-Handling-Regelquelle.
- Eingaben: Korpus.
- Ausgaben: Chunks.
- Datenfluss: Korpus → Ingest.
- Persistenz: Index.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Zentrale Secret-Regeln des Repos.
- Geschäftslogik: Secret-Lebenszyklus-Regeln.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `SEC-SECRET-001`.
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## policies/security/session-lifetime-and-token-boundaries.md

- Zweck: Policy `SEC-SESSION-001` — explizite Session-/Token-Lebenszeiten, Revocation-Pfade, minimale Token-Inhalte, Fehlerunterscheidung invalid/expired/revoked.
- Verantwortlichkeit: Session-/Token-Regelquelle.
- Eingaben: Korpus.
- Ausgaben: Chunks.
- Datenfluss: Korpus → Ingest.
- Persistenz: Index.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Token-Boundary-Regeln; Grounding im Body statt `grounded_in`-Frontmatter (belegt durch public-source-grounding.md).
- Geschäftslogik: Session-Lebenszyklus-Regeln.
- Algorithmen: Keine.
- verwendete Datenmodelle: Frontmatter `SEC-SESSION-001` (ohne grounded_in).
- Abhängigkeiten: Korpus-Loader.
- Rust-Relevanz: Nicht erkennbar.

---

## evals/default_cases.json

- Zweck: Gebündelte Gold-Eval-Suite `day-6-default` mit 6 Fällen (3 search, 3 preflight) inkl. erwarteter Chunk-/Policy-IDs und `expected_insufficient_context`.
- Verantwortlichkeit: Deterministische Regressionsevaluierung von search/preflight.
- Eingaben: Eval-Service lädt die Suite.
- Ausgaben: Eval-Ergebnisse (JSON/HTML-Artefakte).
- Datenfluss: Suite → `EvalService` → `search`/`preflight` → Scoring.
- Persistenz: Ergebnisse unter `data/evals/workspace`.
- Zustände: `expected_insufficient_context` true/false je Fall.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Gold-Case-Erwartungen (Chunk-Recall, Policy-Recall, No-Answer-Fälle).
- Algorithmen: Keine.
- verwendete Datenmodelle: `EvalSuite`, `EvalCase` (belegt durch `types.py`).
- Abhängigkeiten: `services/eval.py`, `runtime_paths.py`.
- Rust-Relevanz: Nicht erkennbar.

---

## examples/codex/README.md

- Zweck: Codex-MCP-Setup-Beispiel (hosted-first, lokaler stdio-Fallback mit `uv run --directory`).
- Verantwortlichkeit: Client-Onboarding für Codex.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `codex mcp add`/`codex mcp get`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Recovery-Notizen (401, Upstream, Insufficient Context, Unavailable).
- Sicherheitsrelevanz: Bearer-Token via Env-Var; `NVIDIA_API_KEY` nicht als `env=$...`-Literal.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## examples/claude-code/README.md

- Zweck: Claude-Code-MCP-Setup-Beispiel (hosted HTTP, lokaler stdio-Fallback via `.mcp.json`).
- Verantwortlichkeit: Client-Onboarding für Claude Code.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: `.mcp.json` (project-scoped).
- Zustände: Keine.
- APIs: `claude mcp add`, `claude mcp add-json`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Recovery-Notizen.
- Sicherheitsrelevanz: Bearer-Header; Env-Passthrough `NVIDIA_API_KEY`.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

## src/policynim/__init__.py

- Zweck: Paket-Marker des `policynim`-Python-Pakets.
- Verantwortlichkeit: Import-Startpunkt; macht `import policynim` ohne Seiteneffekte möglich.
- Eingaben: Keine.
- Ausgaben: Keine.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar.

---

## src/policynim/settings.py

- Zweck: Zentrale Laufzeit-Konfiguration (Pydantic-Basemodel `Settings` mit `get_settings`-Cache) plus `HealthCheckResult`-Modell; regelt Aufbau aller Pfade (Index-DB, Runtime-Rules, Evidence-DB, Eval-Workspace), MCP-Host/Port, API-Key-Quellen, Beta-Auth-Optionen.
- Verantwortlichkeit: Deterministische, validierte Konfiguration für alle Services/CLI/MCP; `get_settings()` (lru_cache) liefert die aktive Instanz; erlaubt `POLICYNIM_CONFIG_FILE`-Umleitung.
- Eingaben: Env-Variablen (`NVIDIA_API_KEY`, `POLICYNIM_*`), `.env`/`config.env`-Dateien (via config_discovery), Plattform-Ports (`PORT`, `RAILWAY_*`).
- Ausgaben: Settings-Objekt mit aufgelösten Pfaden (z. B. `mcp_port`-Default 8123, production `mcp_host` `0.0.0.0`); `default_top_k`; `mcp_bearer_tokens`; `beta_signup_enabled`; `beta_session_secret`.
- Datenfluss: Env/Datei → Settings → Service-Factories (`create_*`).
- Persistenz: Keine direkt; legt Speicherpfade fest (`index_db_path`, `runtime_evidence_db_path`, `eval_workspace_dir`, `beta_auth_db_path`).
- Zustände: `mcp_host`/`mcp_port` (stdio vs. streamable-http), `runtime_mode`-Ableitung (standalone/checkout), `mcp_require_auth` on/off.
- APIs: `Settings`-Modellfelder, `get_settings()`.
- Ereignisse: Keine.
- Nebenwirkungen: `get_settings()` cached; liest Env/Dateien beim ersten Aufruf.
- Fehlerfälle: Fehlender `NVIDIA_API_KEY` führt zu `ConfigurationError` bei Bedarf (pro Service); Validierungsfehler bei Pfadkonflikten (z. B. Verzeichnis als Index-Pfad, deprecated `POLICYNIM_LANCEDB_URI`-Alias).
- Sicherheitsrelevanz: API-Key nur als Env/Datei-Wert, nie geloggt; `beta_session_secret`-Pflicht bei Beta-Signup; Token-Listen für Bearer-Auth.
- Geschäftslogik: Default-`top_k`-Auswahl; production-Port-Host-Wechsel; Konfigurationspräzedenz (plattformspezifisch > explizit > Default).
- Algorithmen: Konfigurationspräzedenz-Resolution.
- verwendete Datenmodelle: `Settings` (Pydantic), `HealthCheckResult`.
- Abhängigkeiten: pydantic, config_discovery, runtime_paths; konsumiert von allen Services/interfaces.
- Rust-Relevanz: Capability "validierte, umgebungsabhängige Konfiguration mit Präzedenz"; Rust: `serde`/`figment`-artige Config-Loader, `OnceCell`-Cache, `tracing`-redact, Typsicherheit via `struct`; Module `config`, `settings`; Error Type `ConfigurationError`.

---

## src/policynim/types.py

- Zweck: Zentrales Datenmodell-Repository (Pydantic-Modelle für Requests/Responses der Services, Ingest, Runtime, Beta) plus Konstanten `MIN_TOP_K`/`MAX_TOP_K`.
- Verantwortlichkeit: Typsichere Verträge zwischen CLI/MCP/Services/Storage; Serialisierung via `model_dump(mode="json")`.
- Eingaben: Konstruktionswerte der jeweiligen Aufrufer (z. B. `PreflightRequest(task, domain, top_k)`, `SearchRequest(query, domain, top_k)`, `PolicyDocument`, `ScoredChunk`, `PolicyMetadata`, `GeneratedPreflightDraft`, `GeneratedPolicyConformanceDraft`, `CompiledPolicyPacket`, `RuntimeRequest`/`RuntimeDecisionResult`, `BetaAccount`, `BetaUsageSnapshot`, `IssuedApiKey`).
- Ausgaben: Validierte Modellinstanzen; JSON-Serialisierung für CLI/MCP-Ausgaben.
- Datenfluss: CLI/MCP → Services → types-Modelle → Storage/JSON.
- Persistenz: Keine direkt; Modelle spiegeln Storage-Records (z. B. `RuntimeEvidenceSessionSummary`).
- Zustände: Enumerierte Felder (z. B. `execution_outcome` allowed/blocked/refused/confirmed/failed, `decision` allow/confirm/block, `confirmation_outcome`).
- APIs: Pydantic-Klassen inkl. Factory-Helper (z. B. `CompiledPolicyPacket`-Identität).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: `ValidationError` bei invaliden Werten (z. B. `top_k` außerhalb `MIN_TOP_K..MAX_TOP_K`, ungültige Kinds in `RuntimeRequest`).
- Sicherheitsrelevanz: Redaction-/Secrets-Felder (z. B. API-Key-Prefix statt Vollwert), `runtime_rules`-Artefakt-Validierung.
- Geschäftslogik: Strukturvorgaben für Grounding (citation_ids, applicable_policies, insufficient_context), Compile-once-Identität.
- Algorithmen: Keine; reine Modelldefinitionen.
- verwendete Datenmodelle: Alle Modelle dieser Datei (siehe Eingaben).
- Abhängigkeiten: pydantic; konsumiert von services/interfaces/storage.
- Rust-Relevanz: Capability "typsichere Domain-Modelle mit Validierung und JSON-Serialisierung"; Rust: `serde` + `serde_json`, `thiserror`-Validierung, `uuid`/`time`-Typen; Module `types`, `contracts`; Error Type `ValidationError`-Äquivalent.

---

## src/policynim/contracts.py

- Zweck: Protocol-Definitionen (Duck-Typing-Verträge) für austauschbare Komponenten (Embedder, Reranker, Generator, PolicyCompiler, IndexStore, EvidenceStore, Preflight/Runtime-Services).
- Verantwortlichkeit: Loose Coupling: Services hängen an Protokollen statt konkreten Klassen; Mocks in Tests implementieren dieselben Protokolle.
- Eingaben: Methodensignaturen (z. B. `embed_query`, `rerank`, `generate`, `compile_policy`, `search`, `store_event`, `close`).
- Ausgaben: Typannotationen für Rückgabewerte (ScoredChunk-Listen, Drafte, Decision-Results).
- Datenfluss: Factory (`service_factories`) injiziert konkrete Implementierungen hinter Protokoll-Referenzen.
- Persistenz: Keine.
- Zustände: Keine; Protokolle enthalten ggf. `closed`-Vertrag.
- APIs: Protocol-Klassen als strukturelle Verträge.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Nicht nachweisbar.
- Sicherheitsrelevanz: `close()`-Lebenszyklus für Ressourcen.
- Geschäftslogik: Keine; Vertragsraster für Fail-closed- und Close-Semantik.
- Algorithmen: Keine.
- verwendete Datenmodelle: Aus types.py referenzierte Modelle.
- Abhängigkeiten: types.py; implementiert in providers/storage/services.
- Rust-Relevanz: Capability "austauschbare Komponentenverträge"; Rust: `trait`-Definitionen (Embedder/Reranker/Generator/Store), `Box<dyn Trait>`-Injektion; Module `contracts`, `services`; Architekturentscheidung: Trait-Objekte statt konkreter Typen.

---

## src/policynim/errors.py

- Zweck: Zentrales Fehlerhierarchie-Modul (`PolicyNIMError`-Basis mit `failure_class`-Attribut und konkrete Ableitungen `ConfigurationError`, `MissingIndexError`, `ProviderError`, `InvalidPolicyDocumentError`, `CompilationError` u. a.).
- Verantwortlichkeit: Einheitliche, maschinenlesbare Fehlerklassifikation (failure_class wird in Hosted-Events, MCP-Responses und CLI-Exit-Pfaden ausgewertet).
- Eingaben: Fehlermeldungen + optionale failure_class.
- Ausgaben: Geworfene Ausnahmen mit `failure_class`-Attribut; CLI: Exit-Code 1 mit klarer stderr-Nachricht; MCP: Fehler-Serialisierung.
- Datenfluss: Services/Storage/Provider → Fehler → CLI/MCP-Handler.
- Persistenz: Keine.
- Zustände: failure_class-Werte (z. B. "non_zero_exit", "confirmation_unavailable", Provider-Fehlerklassen).
- APIs: `PolicyNIMError`-Ableitungen; `failure_class`-Lookup in `_failure_class_from_error`.
- Ereignisse: Hosted `mcp.tool`-Event mit `upstream_failure_class`.
- Nebenwirkungen: Keine.
- Fehlerfälle: Die Fehlerklassen selbst definieren die Fehlerfälle des Systems.
- Sicherheitsrelevanz: Fehlermeldungen dürfen keine Secrets enthalten (Redaction-Vertrag).
- Geschäftslogik: Fehlerklassifikation für Fail-closed-Verhalten.
- Algorithmen: Verkettete failure_class-Suche über `__cause__`/`__context__`.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine externen.
- Rust-Relevanz: Capability "hierarchische, klassifizierbare Fehler"; Rust: `thiserror`/`anyhow`, enum-basierte Error-Types mit `failure_class`-Mapping; Module `errors`; Error Types `ConfigurationError`, `MissingIndexError`, `ProviderError` etc.

---

## src/policynim/config_discovery.py

- Zweck: Auffinden von User-Config (`config.env`), User-Data-Pfad und Projekt-/Checkout-Wurzeln; stellt `user_config_path()`, `user_data_path()`, Checkout-Detection über `__file__`-Lage relativ zu `pyproject.toml` bereit.
- Verantwortlichkeit: Deterministische Ermittlung von Konfigurations-/Datenverzeichnissen für standalone (XDG-artig) und source-checkout (Repo-Wurzel) Modi; `POLICYNIM_CONFIG_FILE`-Umleitung.
- Eingaben: Env (`POLICYNIM_CONFIG_FILE`), Dateisystem-Lage von `config_discovery.py`/`pyproject.toml`, CWD.
- Ausgaben: Pfade (config.env, data/index.sqlite3, data/runtime/..., evals/workspace).
- Datenfluss: Dateisystem → Settings → CLI/MCP.
- Persistenz: Schreibt/Liest config.env (via CLI `init`), legt Datenverzeichnisse an.
- Zustände: runtime_mode (standalone vs. source_checkout).
- APIs: `user_config_path`, `user_data_path`, Checkout-Helfer; `os.replace`-atomares Schreiben.
- Ereignisse: Keine.
- Nebenwirkungen: Erzeugt ggf. Verzeichnisse; `os.replace` beim config-Schreiben (Test: PermissionError → CLI-Fehler).
- Fehlerfälle: Unbeschreibbarer Zielpfad, fehlende Repo-Marker → standalone-Fallback.
- Sicherheitsrelevanz: Verhindert, dass ein beliebiges CWD als "Repo-Wurzel" gilt (Installiert-Modus); `<repo-root>`-Marker.
- Geschäftslogik: Standalone vs. Checkout-Auflösung bestimmt CLI-Kommandos (uv run vs. installed entrypoint).
- Algorithmen: Pfad-Ableitung via `__file__`-Anker + Dateisystem-Marker.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pathlib, os; konsumiert von settings.py und interfaces/cli.py.
- Rust-Relevanz: Capability "plattformbewusste Pfad-Discovery"; Rust: `dirs`-Crate, `std::path`, `env!`-Anker; Module `config_discovery`, `runtime_paths`.

---

## src/policynim/runtime_paths.py

- Zweck: Paket-Resource-Pfadauflösung (`resolve_asset_path`, `resolve_template_root`) für gebündelte Assets (beta), Templates (jinja) und Guardrails-Vorlagen.
- Verantwortlichkeit: Korrekte Pfade sowohl im Entwicklungs- als auch im PyInstaller-Bundle-Modus (Assets als Paketdaten gebündelt).
- Eingaben: Ressourcenname (z. B. `("beta", "beta.css")`, `("nvidia_guardrails/preflight_output", "config.yml")`).
- Ausgaben: Absoluter Pfad zur Ressource oder `InvalidPolicyDocumentError`.
- Datenfluss: MCP-Renderer/Guardrails → resolve_asset_path → Dateisystem.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `resolve_asset_path`, `resolve_template_root`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlende Ressource → `InvalidPolicyDocumentError` (MCP: 404 "Missing beta asset.").
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Pfad-Auflösung relativ zum Paket.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pathlib, `__file__`.
- Rust-Relevanz: Capability "Bundled-Asset-Pfadauflösung"; Rust: `include_str!`/`include_bytes!` oder `rust-embed`-Crate; Module `runtime_paths`; Architekturentscheidung: Assets kompilieren statt zur Laufzeit suchen.

---

## src/policynim/agent_workflows.py

- Zweck: Kuratierte Agent-Workflow-Karten (`agent_workflow_cards()`) mit Titel, Beschreibung, Tool und Prompt für policy_preflight/policy_search; Eingabe in Quickstart-Output und Beta-Dashboard.
- Verantwortlichkeit: Produktseitige Prompt-Bausteine ("Preflight before implementation", "Retrieve policy evidence while debugging", "Verify MCP tool availability").
- Eingaben: Keine (statische Karten).
- Ausgaben: Liste von Dictionaries (title, description, tool, prompt).
- Datenfluss: → CLI quickstart/support-bundle → Beta-Dashboard (`workflow_cards`).
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `agent_workflow_cards()`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Prompts betonen `insufficient_context`-Fail-closed und cited policy lines.
- Geschäftslogik: Workflow-Auswahl spiegelt Tool-Zwecke (preflight vor Implementierung, search beim Debuggen).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine (dict-Listen).
- Abhängigkeiten: Keine.
- Rust-Relevanz: Capability "statische, testbare Content-Karten"; Rust: `const`-Strukturen oder `include_str!`-Prompt-Templates; Module `agent_workflows`.

---

## src/policynim/ingest/__init__.py

- Zweck: Öffentliche API des Ingest-Subpakets (Export von Ladern/Parse/Chunking-Funktionen).
- Verantwortlichkeit: Fassade für Ingest-Pipeline-Bausteine.
- Eingaben: Re-Exportierte Funktionen.
- Ausgaben: Re-Exportierte Funktionen.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Re-Export aus loader/parser/chunking.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: loader, parser, chunking.
- Rust-Relevanz: Nicht erkennbar.

---

## src/policynim/ingest/loader.py

- Zweck: Laden der Policy-Markdown-Dokumente aus dem Korpus (`POLICYNIM_CORPUS_DIR`): rekurseive `.md`-Suche, Validierung, Metadaten-Extraktion.
- Verantwortlichkeit: Wandelt Dateisystem-Korpus in `PolicyDocument`-Objekte um (policy_id, doc_type, domain, title, Quellpfad, Inhalt).
- Eingaben: Korpus-Verzeichnis, `PolicyMetadata`-Defaults, Ausnahme-Pfade/Verzeichnisse.
- Ausgaben: Liste von `PolicyDocument`.
- Datenfluss: Dateisystem → loader → parser → chunking → Embedding/Index.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Ladefunktion(en) für Dokumente + Metadaten.
- Ereignisse: Keine.
- Nebenwirkungen: Liest Dateisystem.
- Fehlerfälle: Fehlender Korpus, ungültige Dokumente → `InvalidPolicyDocumentError`-Pfad.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Korpus-Konventionen (Verzeichnisstruktur → domain, Dateiname → policy_id), TEMPLATE.md-Ausschluss/Handling.
- Algorithmen: Rekursive Dateisystem-Traversierung + Metadaten-Ableitung.
- verwendete Datenmodelle: `PolicyDocument`, `PolicyMetadata`.
- Abhängigkeiten: types.py, errors.py, pathlib.
- Rust-Relevanz: Capability "Korpus-Loading mit Metadaten-Ableitung"; Rust: `walkdir`-Crate, `serde`-Metadaten; Module `ingest`; Error Types `InvalidPolicyDocumentError`.

---

## src/policynim/ingest/parser.py

- Zweck: Parse der Markdown-Policies in Abschnitte (`_SectionExtractor`, `_HeadingEvent`), Ermittlung von Section-Pfaden und Zeilenspannen; Konstante `SECTION_KEY_SEPARATOR = "__"`, `HEADING_PATH_SEPARATOR = " > "`.
- Verantwortlichkeit: Strukturierte Zerlegung einer Policy-Datei in benannte Sektionen mit eindeutigen Schlüsseln für Chunk-IDs.
- Eingaben: Markdown-Text einer Policy.
- Ausgaben: Sektionen (title, key, lines, start/end), Sektionsliste für Chunking.
- Datenfluss: loader → parser → chunking.
- Persistenz: Keine.
- Zustände: `_FenceState` (in/out of code fence) während des Zeilenscans.
- APIs: Parser-Funktionen/`_SectionExtractor`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Unbalancierte Fences/Headings → deterministische Fallbacks.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: ATX-Headings (`_ATX_HEADING_RE`), verschachtelte Heading-Pfade.
- Algorithmen: Zeilenweiser Zustandsautomat (Heading-Events, Fence-Zustand), Identifikator-Normalisierung (`_NORMALIZE_IDENTIFIER_RE`).
- verwendete Datenmodelle: Sektions-Strukturen; Basis für `PolicyChunk`.
- Abhängigkeiten: types.py.
- Rust-Relevanz: Capability "Markdown-Strukturparsing"; Rust: `pulldown-cmark` oder eigene Regex-Zustandsmaschine (`regex`-Crate); Module `ingest::parser`.

---

## src/policynim/ingest/chunking.py

- Zweck: Deterministisches Chunking der geparsten Policies in `PolicyChunk`-Objekte; Konstanten `CHUNK_ID_SEPARATOR = ":"`, `CHUNK_DUPLICATE_SUFFIX_SEPARATOR = "-"`.
- Verantwortlichkeit: Erzeugt stabile Chunk-IDs (policy_id:Sektion:Index), verwaltet Duplikat-Suffixe bei wiederholten Sektionen/Headings; liefert `chunk_policy_documents`/`chunk_policy_document`.
- Eingaben: Geparste Sektionen/Policydokumente.
- Ausgaben: Liste von `PolicyChunk` (chunk_id, section, lines, text, policy-metadata).
- Datenfluss: parser → chunking → Embedding-Index (sqlite_vec).
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `chunk_policy_documents`, `chunk_policy_document`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Leere Sektionen, wiederholte verschachtelte Überschriften (Edge-Case-Tests).
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Chunk-Identitätsregeln (stabil über Re-Ingests).
- Algorithmen: Sektions-zu-Chunk-Zuordnung, Duplikat-Disambiguierung.
- verwendete Datenmodelle: `PolicyChunk`, `PolicyMetadata`.
- Abhängigkeiten: types.py, parser.py.
- Rust-Relevanz: Capability "deterministisches Chunking mit stabilen IDs"; Rust: Hash-/Suffix-Logik, `String`-Building; Module `ingest::chunking`.

## src/policynim/services/__init__.py

- Zweck: Öffentliche Service-Fassade; exportiert alle `create_*`-Factory-Funktionen und Service-Klassen (u. a. `create_preflight_service`, `create_search_service`, `create_runtime_decision_service`, `create_runtime_execution_service`, `create_beta_auth_service`, `create_runtime_health_service`, `ensure_hosted_runtime_ready`, `format_health_failure_reason`).
- Verantwortlichkeit: Einheitlicher Importpunkt für Interfaces (CLI, MCP) und Tests; zentrales Dependency-Wiring.
- Eingaben: Re-Exporte.
- Ausgaben: Re-Exporte.
- Datenfluss: CLI/MCP → services-Fassade → konkrete Service-Implementierungen.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `create_*`-Factories, Service-Klassen, `ensure_hosted_runtime_ready`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Alle Services-Submodule, providers, storage, settings, types, contracts.
- Rust-Relevanz: Capability "zentraler Dependency-Graph"; Rust: Modul-Bündelung, Factory-Funktionen als Modul-API; Module `services`.

---

## src/policynim/services/beta_auth.py

- Zweck: Hosted-Beta-Authentifizierung: GitHub-OAuth (Autorisierungs-URL, Token-Austausch via GitHub-API), Konto-/API-Key-Lebenszyklus (issue/rotate/revoke), Verbrauchszähler (Quota), Suspension, Audit-Log; SQLite-Persistenz via `auth_store`.
- Verantwortlichkeit: Backend des Beta-Portals und der Bearer-Auth des MCP-Endpunkts (`authenticate_api_key`).
- Eingaben: GitHub-OAuth-Code, GitHub-Credentials (Client-ID/Secret aus Settings), Konto-IDs, github_login.
- Ausgaben: `BetaAccount`, `IssuedApiKey` (einmalig voller Secret), `BetaUsageSnapshot`, Auth-Decisions (authorized/suspended/quota_exceeded/unauthorized), Audit-Events.
- Datenfluss: GitHub OAuth → Service → auth_store (SQLite); MCP `_BearerProtectedASGIApp` → `authenticate_api_key`.
- Persistenz: SQLite (`POLICYNIM_BETA_AUTH_DB_PATH`): accounts, api_keys, usage, audit_events.
- Zustände: account.status (active/suspended), api_key aktiv/revoked, quota überschritten/nicht.
- APIs: `build_github_authorize_url`, `complete_github_oauth`, `issue_api_key`, `revoke_api_key`, `get_account`, `get_portal_usage`, `authenticate_api_key`, `list_accounts`, `suspend_account`, `resume_account`, `audit_log`.
- Ereignisse: Audit-Events (u. a. `api_key_rotated` mit key_prefix-Details).
- Nebenwirkungen: Schreibt SQLite; widerruft vorheriges Secret bei Rotation.
- Fehlerfälle: `PolicyNIMError`/`ProviderError` bei OAuth-Fehler (MCP: 502-Seite), fehlende Konten, invalid keys.
- Sicherheitsrelevanz: SHA256-Hash der API-Keys in DB (nur Prefix im Klartext), Secret nur einmal sichtbar, Quota-/Suspension-Durchsetzung, GitHub-Login als Identität.
- Geschäftslogik: Ein aktiver Key pro Konto; Rotation widerruft sofort; Quota-Berechnung (request_count/quota/remaining); Rate-Limit für Auth-Routen.
- Algorithmen: OAuth-State-Validierung, Key-Hashing, Audit-Filterung.
- verwendete Datenmodelle: `BetaAccount`, `BetaUsageSnapshot`, `IssuedApiKey`, `BetaAuthDecision`, Audit-Records.
- Abhängigkeiten: auth_store, providers (GitHub-HTTP), settings, types, errors.
- Rust-Relevanz: Capability "OAuth-Flow + tokenbasierte Kontenverwaltung mit SQLite-Persistenz"; Rust: `oauth2`-Crate, `sha2`-Hashing, `sqlx`/`rusqlite`, `uuid`; Module `services::beta_auth`, `storage::auth_store`; Architekturentscheidung: Secrets gehasht, TOTP-artige Rotation.

---

## src/policynim/services/compiler.py

- Zweck: Policy-Kompilierung: erzeugt `CompiledPolicyPacket` (compiled rules) aus Policy-Chunks; prüft Zitationen gegen vorhandenen Kontext; Fail-closed bei `insufficient_context`; liefert `compile_policy`/`compile_policies`.
- Verantwortlichkeit: Übersetzt Policy-Evidenz in maschinenlesbare Runtime-Regeln; Grundlage für Routing/Decision/Conformance.
- Eingaben: Chunks/Kontext, TaskType, Draft-Guidance.
- Ausgaben: `CompiledPolicyPacket` (mit insufficient_context-Flag, citation-IDs, Identität), Listen davon.
- Datenfluss: preflight → compiler → runtime rules / routing / regeneration.
- Persistenz: Keine direkt; Ergebnisse fließen in `runtime_rules.json`-Artefakt (via Ingest).
- Zustände: sufficient/insufficient context (compile-once-Paketidentität).
- APIs: `compile_policy`, `compile_policies`, Service-Klasse.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Unbekannte/fehlende Citations → `insufficient_context`; Provider-Fehler → fail-closed.
- Sicherheitsrelevanz: Fail-closed-Garantie: kein Compile-Ergebnis ohne belegte Zitationen.
- Geschäftslogik: Zitationsvalidierung gegen Kontext; `_MAX_CHUNKS_PER_POLICY`-Kappung (2 Chunks/Policy).
- Algorithmen: Zitations-Matching, Kontext-Einschränkung.
- verwendete Datenmodelle: `CompiledPolicyPacket`, `CompiledConstraint`, `PolicyChunk`.
- Abhängigkeiten: types, contracts, providers (Generator), errors.
- Rust-Relevanz: Capability "kompilierte, zitationsgebundene Regelpakete"; Rust: `serde`-Paketmodelle, `BTreeMap`-Zitations-Index, `Arc`-Paketidentität; Module `services::compiler`; Architekturentscheidung: Immutable Pakete mit Identitäts-Hash.

---

## src/policynim/services/conformance.py

- Zweck: Policy-Conformance-Scoring: bewertet Antworten gegen Policy-Anforderungen; Backend-Auswahl (`default`, `nemo`, `nemo_evaluator`, `nat`); erzeugt `GeneratedPolicyConformanceDraft` mit Scores; kapselt NVIDIA-Conformance-Response-Validierung.
- Verantwortlichkeit: Qualitäts-/Compliance-Messung von Antworten gegen den Policy-Korpus.
- Eingaben: `PolicyConformanceRequest` (Antwort, Policy-Kontext), Backend-Auswahl, Settings.
- Ausgaben: `GeneratedPolicyConformanceDraft` (Scores, Zitationen, insufficient_context), optional Trace.
- Datenfluss: eval → conformance → provider (NVIDIA/NeMo) → Draft.
- Persistenz: Keine direkt; Eval-Ergebnisse in Workspace.
- Zustände: Backend-Zustand (optionales Paket verfügbar/nicht), closed-Flag.
- APIs: Conformance-Service, `from_settings`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Provider-Aufrufe (kostenpflichtig).
- Fehlerfälle: Fehlendes optionales Paket → `ConfigurationError` mit "uv sync --extra nvidia-eval"-Hinweis; malformed Provider-Responses → Fehlerklassifikation.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Backend-Routing, Bewertungslogik, fail-closed bei fehlender Evidenz.
- Algorithmen: Scoring-Aggregation, Backend-Dispatch.
- verwendete Datenmodelle: `PolicyConformanceRequest`, `GeneratedPolicyConformanceDraft`.
- Abhängigkeiten: providers (nvidia, nvidia_eval), settings, types, contracts.
- Rust-Relevanz: Capability "abstraktes Scoring mit Backend-Plugins"; Rust: Trait `ConformanceEvaluator`, enum `EvalBackend`; Module `services::conformance`, `providers::nvidia_eval`; Architekturentscheidung: Plugin-Backends hinter Trait.

---

## src/policynim/services/dump.py

- Zweck: Index-Dump-Service: liest alle Chunks aus dem SQLite-Index (sortiert) und liefert sie zur Ausgabe; Basis für CLI `dump-index` (inkl. `--count-only`).
- Verantwortlichkeit: Lesbarer Export des Index-Inhalts für Debugging/Audit.
- Eingaben: Settings (Index-Pfad), optional Count-only.
- Ausgaben: Chunk-Liste oder Anzahl ("Indexed chunks: N").
- Datenfluss: Storage (sqlite_vec/index_store) → Service → CLI-Ausgabe.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Dump-Service mit `dump`/`count`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlender Index → `MissingIndexError`/Setup-Hinweis.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Sortierung (policy_id, chunk_id).
- verwendete Datenmodelle: `PolicyChunk`.
- Abhängigkeiten: storage, settings, types.
- Rust-Relevanz: Capability "Index-Export"; Rust: `rusqlite`-Abfrage, `Display`/serde-Ausgabe; Module `services::dump`.

---

## src/policynim/services/eval.py

- Zweck: Eval-Orchestrierung: führt Testfälle (`evals/default_cases.json`) gegen Services/Provider aus, Backend-Auswahl, Rerank on/off-Vergleich, isolierter Live-Eval-Index, Phoenix-Tracing-Reporting (Span-Publishing), Workspace-Output, synchrone Code-Annotationen, `--headless`-UI-Skip.
- Verantwortlichkeit: Messbarer Nachweis der Policy-Conformance-Qualität des Systems.
- Eingaben: Eval-Konfiguration, Testfälle, Settings, Backend, Rerank-Flag, Headless-Flag.
- Ausgaben: Eval-Berichte, Trace-Artefakte (Phoenix), annotierte Ergebnisse, Workspace-Dateien.
- Datenfluss: Testfälle → conformance/preflight-Pipeline → Bewertung → Phoenix-Trace/Report.
- Persistenz: Eval-Workspace (`POLICYNIM_EVAL_WORKSPACE_DIR`), Trace-Daten.
- Zustände: Headless vs. UI, Backend-Auswahl, Rerank an/aus.
- APIs: Eval-Service (`from_settings`), Lauf-Funktionen.
- Ereignisse: Phoenix-Spans, Eval-Ereignisse.
- Nebenwirkungen: Schreibt Workspace, startet ggf. lokales Phoenix (workspace-local), Provider-Aufrufe.
- Fehlerfälle: Fehlendes Optional-Paket → `ConfigurationError`; `--regenerate`-Pfad.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Evaluierungssemantik, Backend-Auswahl, Rerank-Vergleich.
- Algorithmen: Testfall-Schleife, Score-Aggregation, Span-Publishing.
- verwendete Datenmodelle: Eval-Testfälle (aus default_cases.json), Conformance-Drafte.
- Abhängigkeiten: conformance, providers, settings, types, contracts, evals-Korpus.
- Rust-Relevanz: Capability "Evaluierungs-Pipeline mit Tracing"; Rust: `tracing`/`opentelemetry`-Crates, `serde`-Testfälle, Worker-Loop; Module `services::eval`.

---

## src/policynim/services/evidence_trace.py

- Zweck: Materialisierung von Policy-Evidence-Traces: erzeugt `PreflightTraceResult`/Trace-Artefakte, kompakte Eval-Artefakt-Anhänge, Conformance-ID-Erhaltung; Basis für `preflight --trace`.
- Verantwortlichkeit: Nachvollziehbarkeit der Grounding-Kette (welche Chunks/Policies woher).
- Eingaben: Preflight-Ausführung, Trace-Kontext.
- Ausgaben: Trace-Struktur (Schritte, Chunk-Quellen, Conformance-IDs).
- Datenfluss: preflight → evidence_trace → CLI `--trace`/Eval-Artefakt.
- Persistenz: Artefakte in Eval-Workspace.
- Zustände: Keine.
- APIs: Trace-Service-Funktionen.
- Ereignisse: Keine.
- Nebenwirkungen: Schreibt Artefakte.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Trace-Reduktion (kompakt), ID-Erhalt.
- Algorithmen: Trace-Kompression/Zuordnung.
- verwendete Datenmodelle: `PreflightTraceResult`, `PolicyEvidenceTrace`, Conformance-Modelle.
- Abhängigkeiten: types, storage (runtime_evidence?), services (preflight).
- Rust-Relevanz: Capability "Trace-Protokollierung"; Rust: serde-Serialisierung, `Vec`-Schrittketten; Module `services::evidence_trace`.

---

## src/policynim/services/health.py

- Zweck: Runtime-Health-Check: prüft lokalen SQLite-Index (Tabelle `policy_chunks`, Zeilenzahl) und liefert `HealthCheckResult` (ready/status/row_count/mcp_url/table_name/reason).
- Verantwortlichkeit: Readiness-Probe für /healthz (Hosted) und CLI `doctor`.
- Eingaben: Settings.
- Ausgaben: `HealthCheckResult` (JSON).
- Datenfluss: MCP `/healthz`/doctor → health service → storage → Result.
- Persistenz: Keine.
- Zustände: ready/not_ready; status ok/error.
- APIs: `create_runtime_health_service`, `check`, `format_health_failure_reason`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlender/ungültiger Index → reason, 503 (MCP).
- Sicherheitsrelevanz: Keine Secrets im Result.
- Geschäftslogik: Bereitschaft = befüllter Index vorhanden.
- Algorithmen: Row-Count-Prüfung.
- verwendete Datenmodelle: `HealthCheckResult`.
- Abhängigkeiten: storage (create_index_store), settings, types.
- Rust-Relevanz: Capability "Readiness-Probe"; Rust: `rusqlite`-Count-Query, axum-Route; Module `services::health`.

---

## src/policynim/services/ingest.py

- Zweck: Ingest-Orchestrierung: lädt Korpus, parst, chunked, embeddet (Provider), schreibt SQLite-Index (Rebuild bei `rebuild=True`), erzeugt Runtime-Rules-Artefakt; liefert Zusammenfassung (Anzahl Dateien/Chunks, Fehler).
- Verantwortlichkeit: End-to-End-Index-Aufbau für Search/Preflight/Runtime.
- Eingaben: Settings (Korpus, Index-Pfad, Runtime-Rules-Pfad), Settings-Override für Evaluierung (isolierter Index).
- Ausgaben: Ingest-Zusammenfassung; befüllte `index.sqlite3`; `runtime_rules.json`.
- Datenfluss: Korpus → loader → parser → chunking → embedder → sqlite_vec (policy_chunks/policy_vectors) → rules-Artefakt.
- Persistenz: SQLite-Index + Runtime-Rules-Datei (atomar via os.replace).
- Zustände: Index rebuild vs. append; Index vorhanden/fehlt.
- APIs: `IngestService`/`ingest`-Funktion, `from_settings`.
- Ereignisse: Keine.
- Nebenwirkungen: Überschreibt Index bei Rebuild; Provider-Aufrufe (Embeddings).
- Fehlerfälle: Fehlender NVIDIA_API_KEY → `ConfigurationError`; fehlender Korpus; Teilfehler bei Dateien (Count-Reporting).
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Rebuild-Semantik, Fehleraggregation.
- Algorithmen: Pipeline-Orchestrierung, Batch-Embedding.
- verwendete Datenmodelle: `PolicyChunk`, Ingest-Statistik.
- Abhängigkeiten: ingest (loader/parser/chunking), providers (nvidia embedder), storage (sqlite_vec), settings, types.
- Rust-Relevanz: Capability "indexaufbauende Pipeline"; Rust: Pipeline-Traits, `rusqlite`-Transactions, `rayon`-Parallelität; Module `services::ingest`; Architekturentscheidung: Transaktionaler Rebuild.

---

## src/policynim/services/preflight.py

- Zweck: Grounded Preflight: Embedding der Task, Suche im Index (top_k), Reranking (Kandidatenpool), Generator erzeugt Draft-Guidance, Zitationsvalidierung gegen Kontext, Fail-closed bei `insufficient_context`; optional `preflight_with_trace`; `_MAX_CHUNKS_PER_POLICY`-Kappung; `_FINAL_ADHERENCE_THRESHOLD`/`_TRAJECTORY_ADHERENCE_THRESHOLD = 0.75`.
- Verantwortlichkeit: Herzstück: belegte Policy-Guidance für Coding-Tasks vor Implementierung.
- Eingaben: `PreflightRequest` (task, domain, top_k), Settings.
- Ausgaben: `PreflightResult` (plan_steps, applicable_policies, citations, insufficient_context, summary, implementation_guidance) oder `PreflightTraceResult`.
- Datenfluss: CLI/MCP tool `policy_preflight` → Service → embedder → index_store → reranker → generator → compiler → Result.
- Persistenz: Keine (nutzt bestehenden Index).
- Zustände: sufficient/insufficient context; closed.
- APIs: `PreflightService`, `preflight`, `preflight_with_trace`, `from_settings`.
- Ereignisse: Keine.
- Nebenwirkungen: Provider-Aufrufe.
- Fehlerfälle: Fehlender Index → `MissingIndexError`; unbekannte Zitations-IDs → `insufficient_context`; leere Zitationen → fail-closed; dedupliziert Citations, erhält First-seen-Ordnung.
- Sicherheitsrelevanz: Fail-closed-Garantie gegen Halluzination (nur belegte Zitationen).
- Geschäftslogik: Zitations-Deduplizierung, Policy-vs-Draft-Zitationsabgleich, Fallback auf Policy-level-Citations, Kappung pro Policy (2), Rerank-Reihenfolge.
- Algorithmen: Embedding-Retrieval, Rerank (Kandidatenpool `_DEFAULT_RERANK_CANDIDATE_POOL = 15`), Kontext-Beschränkung.
- verwendete Datenmodelle: `PreflightRequest/Result`, `DraftPolicyGuidance`, `ScoredChunk`, `GeneratedPreflightDraft`, `GeneratedCompiledPolicyDraft`.
- Abhängigkeiten: contracts (Embedder/Reranker/Generator/Compiler), storage (index_store), settings, types, errors.
- Rust-Relevanz: Capability "retrieval-gestützte Guidance mit Fail-closed"; Rust: Trait-Pipeline, `async`-Provider, `HashSet`-Dedupe; Module `services::preflight`; Architekturentscheidung: Fail-closed als Typsystem (Result statt Option).

---

## src/policynim/services/regeneration.py

- Zweck: Policy-gebundene Regeneration: Wiederholte Generierung mit Typed-Retry-Triggers, `max_regeneration`-Limit, `insufficient_context`-Stopp, Compile-once-Paketidentität, Zitations-Drift-Ablehnung; Grundlage für `preflight --regenerate` und `eval --regenerate`.
- Verantwortlichkeit: Qualitätsverbesserung durch kontrollierte Retries ohne Identity-Bruch.
- Eingaben: Draft/Ergebnis, Retry-Trigger, Settings.
- Ausgaben: Regeneriertes Ergebnis oder Abbruch mit Begründung.
- Datenfluss: preflight/eval → regeneration → generator → Ergebnis.
- Persistenz: Keine.
- Zustände: max_regeneration erreicht; citation drift; insufficient context.
- APIs: Regenerations-Service.
- Ereignisse: Keine.
- Nebenwirkungen: Zusätzliche Provider-Aufrufe.
- Fehlerfälle: Drift → Ablehnung (fail-closed); Provider-Fehler → fail-closed; Limit → Stop.
- Sicherheitsrelevanz: Drift-Ablehnung verhindert nicht belegte Neuformulierungen.
- Geschäftslogik: Retry-Klassifikation, Limits, Identitätswahrung.
- Algorithmen: Retry-Schleife mit Abbruchbedingungen.
- verwendete Datenmodelle: Drafte (types), CompiledPolicyPacket-Identität.
- Abhängigkeiten: compiler, providers, types, contracts, errors.
- Rust-Relevanz: Capability "kontrollierte Retry-Schleifen"; Rust: `Loop` mit `Result`-Abbruch, `retry`-Muster; Module `services::regeneration`.

---

## src/policynim/services/router.py

- Zweck: Task-aware Routing: `TaskProfile`-Inferenz, Auswahl der Evidenztiefe (`--task-type`-Override, `_DEFAULT_ROUTING_CANDIDATE_POOL = 15`), Grouping der ausgewählten Policies, Weak-Evidence-Fallback; `_LINE_SPAN_RE`-basiert.
- Verantwortlichkeit: Bestimmt, welche Policies/Chunks für eine Task relevant sind (Kandidatenauswahl).
- Eingaben: Task, Domain, TaskType-Override, Kandidaten (Chunks).
- Ausgaben: Routed-Entscheidung (ausgewählte Chunks/Policies, TaskType, Evidenztiefe).
- Datenfluss: CLI `route` → router → storage → Result.
- Persistenz: Keine.
- Zustände: Strong/weak evidence; Fallback.
- APIs: Router-Service, `from_settings`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlender Index → `MissingIndexError`; keine Kandidaten → Weak-Fallback.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: TaskType-Inferenz, Grouping nach Policy, Evidenztiefe.
- Algorithmen: Kandidaten-Pooling, Line-Span-Regex-Auswertung (`_LINE_SPAN_RE`), Fallback-Heuristik.
- verwendete Datenmodelle: Routed-Pakete, `ScoredChunk`, TaskType.
- Abhängigkeiten: storage, settings, types, contracts.
- Rust-Relevanz: Capability "kontextabhängige Auswahl"; Rust: `enum TaskType`, Matching-Heuristiken, `Vec`-Grouping; Module `services::router`.

---

## src/policynim/services/runtime_decision.py

- Zweck: Runtime-Entscheidung: validiert `RuntimeRequest` (shell_command/file_write/http_request), sucht passende kompilierte Runtime-Regeln, wendet Regeln an, liefert `RuntimeDecisionResult` (decision allow/confirm/block, summary, Zitations-IDs).
- Verantwortlichkeit: Policy-konforme Ausführungsentscheidung vor Aktion.
- Eingaben: `RuntimeRequest`, Runtime-Rules-Artefakt, Settings.
- Ausgaben: `RuntimeDecisionResult`.
- Datenfluss: CLI `runtime decide`/MCP → runtime_decision → rules → Result.
- Persistenz: Keine direkt; Evidence wird im Execution-Pfad persistiert.
- Zustände: allow/confirm/block.
- APIs: `RuntimeDecisionService`, `from_settings`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Ungültige Kinds, fehlende Rules → fail-closed (block/error).
- Sicherheitsrelevanz: Entscheidung vor Aktion; block bei fehlenden Rules.
- Geschäftslogik: Regel-Matching auf Request-Felder (task/cwd/command/path), Zitations-Verknüpfung.
- Algorithmen: Regel-Abgleich, Prioritätsauswahl.
- verwendete Datenmodelle: `RuntimeRequest`, `RuntimeDecisionResult`, `CompiledConstraint`-Regeln.
- Abhängigkeiten: storage (runtime_rules), types, errors, contracts.
- Rust-Relevanz: Capability "Regel-Auswertung"; Rust: `serde_json`-Rules, Matching-Muster, `enum Decision`; Module `services::runtime_decision`.

---

## src/policynim/services/runtime_evidence_report.py

- Zweck: Runtime-Evidence-Bericht: aggregiert SQLite-Evidence pro Session zu `RuntimeEvidenceSessionSummary` (counts, executions), Rendering als JSON/Markdown; CLI `evidence report`.
- Verantwortlichkeit: Auditierbarer Nachweis der Ausführungssitzungen.
- Eingaben: session_id, Format (json/markdown), Output-Pfad.
- Ausgaben: Session-Zusammenfassung; Datei-Export (atomar).
- Datenfluss: CLI → Service → runtime_evidence (SQLite) → Report.
- Persistenz: Liest `runtime_evidence.sqlite3`; schreibt Report-Dateien.
- Zustände: Keine.
- APIs: `report_session`, `create_runtime_evidence_report_service`.
- Ereignisse: Keine.
- Nebenwirkungen: Datei-Writes bei `--output`.
- Fehlerfälle: Unbekannte Session → `PolicyNIMError`; Verzeichnis als Output → Fehler; leerer Output-Pfad → Fehler.
- Sicherheitsrelevanz: Redaction von Payloads.
- Geschäftslogik: Zählungen (allowed/confirmed/blocked/refused/failed/incomplete), Execution-Details.
- Algorithmen: SQL-Aggregation, Markdown-Rendering.
- verwendete Datenmodelle: `RuntimeEvidenceSessionSummary`, `RuntimeEvidenceExecutionSummary`.
- Abhängigkeiten: storage (runtime_evidence), types, errors.
- Rust-Relevanz: Capability "Berichtsaggregation"; Rust: `rusqlite`-Aggregate, `tera`/Handlebars-Markdown; Module `services::runtime_evidence_report`.

---

## src/policynim/services/runtime_execution.py

- Zweck: Runtime-Ausführung: führt entschiedene Aktionen aus (shell_command, file_write, http_request) unter Confirmation-Handling (`confirmer`-Callback), Redaction, durable Evidence-Persistenz, Fail-closed; Ergebnis `RuntimeExecutionResult` mit `execution_outcome` und `failure_class`.
- Verantwortlichkeit: Sichere, evidence-pflichtige Ausführung von Entwickleraktionen.
- Eingaben: `RuntimeRequest`, Entscheidung (aus Decision-Service), confirmer, Settings.
- Ausgaben: `RuntimeExecutionResult` (session_id, outcome, output, failure_class).
- Datenfluss: CLI `runtime execute` → Service → (Decision) → Ausführung → Evidence-Store.
- Persistenz: `runtime_evidence.sqlite3` (Events, Executions).
- Zustände: outcomes allowed/confirmed/blocked/refused/failed; confirmation unavailable → failed.
- APIs: `RuntimeExecutionService`, `from_settings`, `execute`.
- Ereignisse: Evidence-Events (Session-Start, Execution-Abschluss).
- Nebenwirkungen: Führt echte Shell-Befehle/Datei-Writes/HTTP-Requests aus; persistiert Evidence.
- Fehlerfälle: Non-zero-Exit (failure_class "non_zero_exit"), Confirmation verweigert (refused), Confirmation nicht verfügbar (failed), Block durch Entscheidung.
- Sicherheitsrelevanz: Redaction von Befehlen/Inhalten, Confirmation für Guarded-Aktionen, Fail-closed.
- Geschäftslogik: Ausführungs-Zulässigkeit, Confirmation-Semantik (stdout vs. stderr Prompt), Session-Resolution.
- Algorithmen: Subprocess-Ausführung (timeout), Atomic-Writes, HTTP-Clients, Evidence-Reihenfolge.
- verwendete Datenmodelle: `RuntimeRequest`, `RuntimeExecutionResult`, `RuntimeEvidenceExecutionSummary`.
- Abhängigkeiten: runtime_decision, storage (runtime_evidence), types, errors.
- Rust-Relevanz: Capability "gesandboxte Ausführung mit Audit"; Rust: `tokio::process`, `tempfile`, `reqwest`, `tracing`; Module `services::runtime_execution`; Architekturentscheidung: Execution als own Trait (testbar, mockbar).

---

## src/policynim/services/search.py

- Zweck: Policy-Suche: Embedding der Query, Suche im Index (top_k, optional Domain-Filter), Reranking, Rückgabe `SearchResult` mit Chunks/Zitationen.
- Verantwortlichkeit: Grounded Retrieval über den Policy-Korpus für `policy_search`/CLI `search`.
- Eingaben: `SearchRequest` (query, domain, top_k), Settings.
- Ausgaben: `SearchResult` (Chunks, Metadaten).
- Datenfluss: CLI/MCP → Service → embedder → index_store → reranker → Result.
- Persistenz: Keine.
- Zustände: Keine; closed.
- APIs: `SearchService`, `search`, `from_settings`.
- Ereignisse: Keine.
- Nebenwirkungen: Provider-Aufrufe.
- Fehlerfälle: Fehlender Index → `MissingIndexError` (CLI: Hinweis `policynim ingest`), fehlender Key → `ConfigurationError`.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Domain-Filterung, top_k-Respektierung, Rerank.
- Algorithmen: Embedding-Retrieval, Rerank-Kandidatenpool.
- verwendete Datenmodelle: `SearchRequest/Result`, `ScoredChunk`.
- Abhängigkeiten: contracts, storage (index_store), settings, types, errors.
- Rust-Relevanz: Capability "Vektor-Suche mit Filtern"; Rust: `sqlite-vec`-FFI oder eigenes HNSW, `async`-Provider; Module `services::search`.

## src/policynim/providers/__init__.py

- Zweck: Export-Fassade für Provider (NVIDIA-Client, Eval-Adapter, Guardrails-Adapter).
- Verantwortlichkeit: Einheitlicher Importpunkt für Provider-Klassen und `installed_version`-Gates.
- Eingaben: Re-Exporte.
- Ausgaben: Re-Exporte.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Re-Exporte aus nvidia/nvidia_eval/nvidia_guardrails.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: nvidia, nvidia_eval, nvidia_guardrails.
- Rust-Relevanz: Nicht erkennbar.

---

## src/policynim/providers/nvidia.py

- Zweck: NVIDIA NIM-Client: `NVIDIAEmbedder` (embed_query), `NVIDIAReranker` (rerank → ScoredChunk-Kandidaten), `NVIDIAGenerator` (Generierung von Drafte/Responses), `NVIDIAPolicyCompiler`-Integration, HTTP-Client mit Timeout `_DEFAULT_HTTP_TIMEOUT_SECONDS = 10.0`, User-Agent `"PolicyNIM Hosted Beta"`, Response-Validierung (malformed Payloads → Fehlerklassifikation).
- Verantwortlichkeit: Gesamter NVIDIA-API-Zugriff (Chat, Embeddings, Reranking, Tool-Calling-Vorbereitung) hinter `Provider`-Verträgen.
- Eingaben: API-Key, Model-Konfiguration (Settings), Texte/Queries/Chunks/Kandidaten.
- Ausgaben: Embedding-Vektoren, gerankte Chunks, generierte Texte/JSON-Drafte.
- Datenfluss: Services → providers.nvidia → NVIDIA NIM (HTTPS) → Responses.
- Persistenz: Keine.
- Zustände: closed; Retry-/Fehlerzustände.
- APIs: `NVIDIAEmbedder.from_settings/embed_query/close`, `NVIDIAReranker.from_settings/rerank/close`, `NVIDIAGenerator` (generate), HTTP-Helfer, `_USER_AGENT`.
- Ereignisse: Hosted `mcp.tool`-Events mit `upstream_failure_class`.
- Nebenwirkungen: Kostenpflichtige externe API-Aufrufe.
- Fehlerfälle: Fehlender Key → `ConfigurationError`; HTTP-/Upstream-Fehler → `ProviderError` mit failure_class; malformed Responses → Validierungsfehler; Timeouts.
- Sicherheitsrelevanz: Key nur via Settings (Env/Datei), nie geloggt; Response-Inhalte ungefiltert an Services.
- Geschäftslogik: Request-Aufbau (Endpoints/Parameter), Fehlerklassifikation, Timeout-Handling.
- Algorithmen: HTTP-Retry, Rerank-Scoring-Nachverarbeitung zu `ScoredChunk`.
- verwendete Datenmodelle: `ScoredChunk`, Drafte, types-Modelle.
- Abhängigkeiten: httpx, pydantic, settings, types, errors, contracts.
- Rust-Relevanz: Capability "externer LLM-HTTP-Client mit Retry/Timeout/Validierung"; Rust: `reqwest`/`reqwest-middleware` mit Retry, `serde`-Strict-Response-Validation, `tracing`; Module `providers::nvidia`; Architekturentscheidung: Response-Schemas streng validieren (serde deny_unknown_fields).

---

## src/policynim/providers/nvidia_eval.py

- Zweck: NeMo-Evaluator-Adapter: `NeMoEvaluatorPolicyConformanceEvaluator` und `NeMoAgentToolkitPolicyConformanceEvaluator`; optionale Package-Gates via `installed_version` (nemo-evaluator/nvidia-simple-evals/nvidia-nat-eval); kapselt Conformance-Evaluation über NeMo-Backends.
- Verantwortlichkeit: Anbindung optionaler NVIDIA-Eval-Backends hinter dem Conformance-Vertrag; sauberes Fehlverhalten ohne installierte Pakete.
- Eingaben: Evaluator-Instanzen, Policy-Conformance-Request, Settings.
- Ausgaben: `GeneratedPolicyConformanceDraft`, `installed_version`-Ergebnis.
- Datenfluss: conformance service → nvidia_eval → NeMo-Paket (falls installiert).
- Persistenz: Keine.
- Zustände: closed; optionales Paket verfügbar/nicht.
- APIs: `installed_version`, `from_settings` (Gate vor Evaluator-Konstruktion), Evaluator-close.
- Ereignisse: Keine.
- Nebenwirkungen: Provider-Aufrufe (nur wenn Paket installiert).
- Fehlerfälle: `PackageNotFoundError` → `ConfigurationError` mit "uv sync --extra nvidia-eval"-Hinweis; schließt übergebene Evaluator-Instanzen (closed=True) im Fehlerfall.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Gate-Reihenfolge: erst Package-Check, dann Evaluator-Konstruktion; Fail-closed.
- Algorithmen: Version-Sniffing.
- verwendete Datenmodelle: `GeneratedPolicyConformanceDraft`, Conformance-Modelle.
- Abhängigkeiten: types, errors, contracts, optional nvidia-Pakete.
- Rust-Relevanz: Capability "optionale Backend-Erkennung"; Rust: `cfg`-Feature-Flags, `Option`-Gate, dynamische Cargo-Features; Module `providers::nvidia_eval`; Architekturentscheidung: Fehlendes Backend als `ConfigurationError` statt Panik.

---

## src/policynim/providers/nvidia_guardrails.py

- Zweck: NeMo-Guardrails-Output-Rail-Wrapper: `_DEFAULT_GUARDRAILS_MODEL = "nvidia/llama-3.3-nemotron-super-49b-v1.5"`, langes Package-Gating (lazy), geladene Assets (config.yml/rails.co), verarbeitet malformed Rail-Output, blocked Output, Zitations-Drift, Regeneration-Kontext-Pass-through; für `nemo_guardrails_preflight_generator`.
- Verantwortlichkeit: Lokale Guardrails-Steuerung der Generator-Ausgaben vor NVIDIA NIM.
- Eingaben: Prompt/Kontext, Regenerations-Einstellungen, Modell-Override.
- Ausgaben: Rail-validierte Generierungsausgabe (blocked oder bereinigt).
- Datenfluss: preflight generator → nvidia_guardrails → rails (colang) → Output.
- Persistenz: Keine (nutzt gepackte Assets).
- Zustände: blocked/nicht; closed.
- APIs: Wrapper-Klasse, `from_settings`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Provider-Aufrufe.
- Fehlerfälle: Malformed Rail-Output → Fehlerklassifikation; blocked Output → Abbruch; Zitations-Drift → fail-closed.
- Sicherheitsrelevanz: Guardrails als Ausgabe-Kontrollschicht.
- Geschäftslogik: Rail-Pflicht, Drift-Erkennung, Regenerations-Pass-through.
- Algorithmen: Rail-Ausführung (NeMo Guardrails), Drift-Vergleich.
- verwendete Datenmodelle: Generierungs-Drafte, Guardrails-Konfiguration (config.yml/rails.co).
- Abhängigkeiten: runtime_paths (Assets), settings, types, errors, optionales neMo-Guardrails-Paket.
- Rust-Relevanz: Capability "Output-Rail-Validierung"; Rust: eigene Rail-Logik (Regex/AST) oder `colang`-Parser-Binding; Module `providers::nvidia_guardrails`; Architekturentscheidung: Guardrails als Pipeline-Stufe.

---

## src/policynim/storage/__init__.py

- Zweck: Export-Fassade für Storage-Schichten (Index, Evidence, Auth) inkl. `create_index_store`.
- Verantwortlichkeit: Einheitlicher Storage-Importpunkt.
- Eingaben: Re-Exporte.
- Ausgaben: Re-Exporte.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `create_index_store`, Store-Klassen.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: sqlite_vec, index_store, runtime_evidence, auth_store.
- Rust-Relevanz: Nicht erkennbar.

---

## src/policynim/storage/sqlite_vec.py

- Zweck: SQLite-Vektor-Index (`SqliteVecIndexStore`): Schema-Version `_SCHEMA_VERSION = "1"`, Tabellen `index_metadata`/`policy_chunks`/`policy_vectors` (vec0 virtual table), SQLite-Vektor-Erweiterung `sqlite-vec` (Version 0.1.9), Domänen-Kandidaten-Multiplikator `_DOMAIN_CANDIDATE_MULTIPLIER = 5`/`_MIN_DOMAIN_CANDIDATES = 20`.
- Verantwortlichkeit: Persistenter Vektor-Retrieval (KNN), Domänen-Filter, Metadaten (version, count), Read/Write-API für Ingest/Search/Health/Dump.
- Eingaben: Chunks + Embeddings (write), Query-Embedding + top_k + domain (search), exists/count.
- Ausgaben: `ScoredChunk`-Kandidaten (sortiert), Metadaten, Row-Count.
- Datenfluss: ingest → write; search/preflight → search; health/dump → count/read.
- Persistenz: SQLite-Datei (`index.sqlite3`) inkl. Vektor-Erweiterung; Schema-Init idempotent.
- Zustände: Schema-Version-Mismatch → Migration/Fehler; Index befüllt/leer.
- APIs: `create_index_store`, `exists`, `count`, `search`, `add_chunks`, `table_name`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Schreibt DB-Dateien; lädt native Erweiterung.
- Fehlerfälle: Fehlende sqlite-vec-Erweiterung, Schema-Versionskonflikt, Verzeichnis als Pfad (doctor-Hinweis).
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Domänen-Kandidaten-Budget (Multiplikator/Mindestanzahl), top_k-Erfüllung.
- Algorithmen: KNN via vec0 (brute-force), Kandidaten-Zusammenführung, Score-Sortierung.
- verwendete Datenmodelle: `PolicyChunk`, `ScoredChunk`, index_metadata-Rows.
- Abhängigkeiten: sqlite-vec (0.1.9, Python-Binding), sqlite3, types, errors.
- Rust-Relevanz: Capability "Vektor-DB-Schicht"; Rust: `sqlite-vec`-Rust-Binding oder `sqlx` + vec0, `Vec<f32>`-Encode; Module `storage::sqlite_vec`; Architekturentscheidung: sqlite-vec statt LanceDB (Migration dokumentiert).

---

## src/policynim/storage/index_store.py

- Zweck: Alternativer/kompatibler Index-Store-Vertrag (`IndexStore`-Interface) für Tests und Duck-Typing (exists/count/search); Mock-freundliche Schnittstelle.
- Verantwortlichkeit: Vertrag + ggf. In-Memory-/Datei-Implementierung für isolierte Nutzung.
- Eingaben: Search-Anfragen, Chunk-Sets.
- Ausgaben: ScoredChunk-Listen, counts.
- Datenfluss: Services → IndexStore.
- Persistenz: Abhängig von Implementierung.
- Zustände: Keine.
- APIs: `exists`, `count`, `search`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: `ScoredChunk`.
- Abhängigkeiten: types.
- Rust-Relevanz: Capability "Storage-Trait"; Rust: `trait IndexStore` mit generischen Implementierungen; Module `storage`.

---

## src/policynim/storage/runtime_evidence.py

- Zweck: SQLite-Evidence-Store (`RuntimeEvidenceStore`): Schema, Event-/Execution-Persistenz, Session-Aggregation, Reopen-/Concurrency-Verhalten; speichert Sitzungen, Aktionen, Outcomes, Redacted-Payloads.
- Verantwortlichkeit: Dauerhafte, auditierbare Aufzeichnung aller Runtime-Entscheidungen/-Ausführungen.
- Eingaben: Session-Events (start/complete), Executions (Request, Decision, Outcome, Fehlerklasse, Zeitstempel).
- Ausgaben: Session-Summaries (counts), Execution-Details.
- Datenfluss: runtime_execution → RuntimeEvidenceStore → `runtime_evidence.sqlite3`; evidence report liest.
- Persistenz: `runtime_evidence.sqlite3` (Schema-Init idempotent).
- Zustände: Session open/complete; Outcome-Klassen.
- APIs: `record_event`, `record_execution`, `summarize_session`, `close`.
- Ereignisse: Session-/Execution-Events.
- Nebenwirkungen: DB-Writes.
- Fehlerfälle: Concurrency-Zugriffe (Tests: Reopen, Reihenfolge, Threadsicherheit).
- Sicherheitsrelevanz: Redaction von Payloads vor Persistenz.
- Geschäftslogik: Zählungs-Aggregation, Reihenfolge-Erhaltung.
- Algorithmen: SQL-Aggregation.
- verwendete Datenmodelle: `RuntimeEvidenceSessionSummary`, `RuntimeEvidenceExecutionSummary`, interne Events.
- Abhängigkeiten: sqlite3, types, errors.
- Rust-Relevanz: Capability "Event-Sourcing-ähnliche Persistenz"; Rust: `rusqlite`-Transaktionen, `serde`-Payloads, `Mutex`-Zugriff; Module `storage::runtime_evidence`.

---

## src/policynim/storage/auth_store.py

- Zweck: SQLite-Auth-Store (`AuthStore`): Konten (GitHub-Login/Email/Status), API-Keys (SHA256-gehasht, nur Prefix im Klartext), Nutzungszähler (quota/request_count), Audit-Events; `_ACCOUNT_SELECT`-SQL.
- Verantwortlichkeit: Persistenz für das Beta-Portal (Konten, Keys, Quota, Audit).
- Eingaben: Konto-Infos, Key-Werte, Usage-Updates, Audit-Einträge.
- Ausgaben: Konto-/Usage-/Audit-Records.
- Datenfluss: beta_auth service → AuthStore → SQLite-DB.
- Persistenz: `POLICYNIM_BETA_AUTH_DB_PATH`; idempotente Schema-Init (zweifache Instanziierung ok).
- Zustände: Konto active/suspended; Key aktiv/revoked.
- APIs: `list_accounts`, `upsert_account_from_github` (account_id stabil), `issue/revoke/rotate` (widerruft vorheriges Secret), `record_usage`, `get_portal_usage`, `audit_log` (Filter github_login/event_type/limit).
- Ereignisse: Audit-Events (z. B. `api_key_rotated` mit `key_prefix`).
- Nebenwirkungen: DB-Writes; Rotation widerruft sofort.
- Fehlerfälle: Fehlende Konten, ungültige Filter.
- Sicherheitsrelevanz: SHA256-Hashing der Keys (`_hash_api_key`), Klartext nur einmal ausgegeben, Auditierbarkeit.
- Geschäftslogik: Ein aktiver Key pro Konto; Upsert ohne Duplikat; Quota-Zähler.
- Algorithmen: SHA256-Hashing, Audit-Filterung.
- verwendete Datenmodelle: `BetaAccount`, `BetaUsageSnapshot`, Audit-Records, Key-Rows.
- Abhängigkeiten: sqlite3, types, errors.
- Rust-Relevanz: Capability "authentifizierte Kontenverwaltung"; Rust: `rusqlite`-Schema, `sha2`-Hashing, `chrono`-UTC; Module `storage::auth_store`; Architekturentscheidung: Secrets niemals im Klartext persistieren.

---

## src/policynim/interfaces/__init__.py

- Zweck: Export-Fassade für CLI und MCP.
- Verantwortlichkeit: Einheitlicher Importpunkt.
- Eingaben: Re-Exporte.
- Ausgaben: Re-Exporte.
- Datenfluss: Keine.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: `cli`, `mcp`-Module.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: cli, mcp.
- Rust-Relevanz: Nicht erkennbar.

---

## src/policynim/interfaces/cli.py

- Zweck: Typer-CLI (3074 Zeilen): Befehle `--version`, `init` (interaktiv), `doctor [--format text|json]`, `quickstart [--target hosted-mcp|local-cli|local-mcp] [--client codex|claude-code] [--hosted-url] [--format json]`, `ingest`, `search`, `route`, `compile`, `preflight [--trace] [--regenerate]`, `dump-index [--count-only]`, `eval [--backend ...] [--headless] [--regenerate]`, `runtime decide/execute --input <path|->`, `evidence report --session-id [--format markdown|json] [--output]`, `mcp [--transport stdio|streamable-http]`, `mcp-config [--target local-stdio|hosted-http] [--client ...] [--repo-root] [--hosted-url] [--uv-command]`, `mcp-smoke [--mcp-config-file] [--format json]`, `support-bundle [--include-mcp-smoke] [--include-local-paths] [--format markdown]`, `beta-admin list-accounts|suspend|resume|revoke-key|audit-log`.
- Verantwortlichkeit: Gesamte Mensch-Maschine-Schnittstelle: Setup, Diagnose, First-run, Ausführung, Berichte, Operator-Admin.
- Eingaben: Kommandozeilen-Argumente, stdin (`--input -`), interaktive Prompts (`init`), Env/Config-Dateien.
- Ausgaben: Text/JSON/Markdown auf stdout; Fehler auf stderr mit Exit-Codes 0/1/2.
- Datenfluss: CLI → Service-Factories → Services → Ergebnisse → Rendering.
- Persistenz: config.env (init, atomar via os.replace), .env (checkout), Report-Dateien, index/rules (via ingest).
- Zustände: runtime_mode standalone/source_checkout; Setup-Zustände (action_required/ok).
- APIs: Typer-App, `app`-Export, `_run_mcp_stdio_smoke`, `_resolve_installed_version`, `_build_cli_confirmer`.
- Ereignisse: Keine.
- Nebenwirkungen: Datei-Writes, Provider-Aufrufe, echte Ausführung (runtime execute), MCP-Launch.
- Fehlerfälle: `ConfigurationError` → Exit 1 mit stderr-Hinweisen (init/ingest-Kommandos); `MissingIndexError` → ingest-Hinweis; Service-close trotz Fehler; Typer-Validierungsfehler (Exit 2).
- Sicherheitsrelevanz: Secrets nie in Output (Redaction-Marker `<config-dir>`, `<local-path>` etc.), Key-Eingabe nie echoed, Validierung von URLs (keine Credentials), frozen-executable-Pfad.
- Geschäftslogik: Setup-Guides (init/quickstart/doctor), Command-Recovery-Hinweise, Hosted-first-Standards, MCP-Config-Kontrakte.
- Algorithmen: Config-Präzedenz, Command-Rendering, Markdown-Wrapping, MCP-Smoke (asyncio stdio_client).
- verwendete Datenmodelle: types-Modelle (PreflightResult, RuntimeExecutionResult, RuntimeEvidenceSessionSummary, BetaAccount u. a.), JSON-Payloads.
- Abhängigkeiten: typer, services, settings, config_discovery, types, errors, mcp (run_server), storage.
- Rust-Relevanz: Capability "Typer-äquivalente CLI"; Rust: `clap`-Derive, `serde_json`-Ausgabe, Exit-Code-Mapping, Subcommands; Module `interfaces::cli`; Architekturentscheidung: Command-Layer delegiert an Services (dünn halten).

---

## src/policynim/interfaces/mcp.py

- Zweck: MCP-Surface (1064 Zeilen): FastMCP-Server "PolicyNIM" (json_response=True), Tools `policy_preflight`/`policy_search` (mit top_k-Validierung, Hosted-Events), Routen `/healthz`, `/beta` (+ Assets, OAuth `start`/`callback`, API-Key-Regenerate, Logout), `_BearerProtectedASGIApp` (exact-match Tokens + Beta-Auth), Streamable-HTTP-Bootstrap mit Port-Check und `ensure_hosted_runtime_ready(rebuild_if_missing=True)`.
- Verantwortlichkeit: KI-Client-Oberfläche (stdio + streamable-http), Hosted-Portal + Bearer-Auth, Readiness.
- Eingaben: MCP-Tool-Argumente (task/query/domain/top_k), HTTP-Requests (Auth-Header, OAuth-Codes, Form-POSTs).
- Ausgaben: JSON-Tool-Responses (model_dump), HTML-Seiten (landing/dashboard), Redirects, Health-JSON, 401/403/429-Antworten.
- Datenfluss: Clients → MCP → Services (create_preflight/search/beta_auth/health) → Storage/Provider.
- Persistenz: Session-Cookies (SessionMiddleware, same_site lax, https_only), beta auth via AuthStore.
- Zustände: Auth-Ergebnis (ContextVar `_HOSTED_AUTH_RESULT`: authorized/suspended/quota_exceeded/unauthorized/not_required); Rate-Limiter-Fenster.
- APIs: `run_server(transport)`, `policy_preflight`, `policy_search`, Modul-level `mcp` (registriert), `_register_tools`.
- Ereignisse: Hosted-JSON-Lines-Events `mcp.tool` (tool_name, latency_ms, upstream_failure_class, auth_result, request_id) und `mcp.auth` (auth_result).
- Nebenwirkungen: Uvicorn-Server; Provider-Aufrufe; Session-Set/Clear; Port-Probe.
- Fehlerfälle: Port belegt → `ConfigurationError` mit Recovery-Text; fehlender Key → `ConfigurationError` (auch aus hosted rebuild); OAuth-State-Mismatch → 400; OAuth-Upstream → 502; fehlender Index → Rebuild-Versuch; fehlendes Asset → 404.
- Sicherheitsrelevanz: Bearer-Auth exact-match + Konten-Auth; Browser-Redirect (303) für menschliche /mcp-Besuche; Secrets nie in HTML/JSON; Rate-Limit auf Auth-Routen; OAuth-State in Session.
- Geschäftslogik: Hosted-Flow (OAuth → Key → Client-Commands), Quota/Suspension an der Auth-Grenze, Placeholder-URL-Handling.
- Algorithmen: Sliding-Window-Rate-Limit (`_InMemoryRateLimiter`), Token-Extraktion (`_extract_bearer_token`), forwarded-IP-Auflösung (`_forwarded_client_address`), top_k-Validierung.
- verwendete Datenmodelle: `PreflightResult`, `SearchResult`, `BetaAccount`, `BetaUsageSnapshot`, `HealthCheckResult`, `IssuedApiKey`.
- Abhängigkeiten: FastMCP (mcp), starlette, jinja2, uvicorn, services, settings, storage, runtime_paths, agent_workflows, types, errors.
- Rust-Relevanz: Capability "MCP-Server (stdio + streamable-http) mit Auth-Middleware"; Rust: `rmcp`-Crate, `axum`-Router, `tower`-Middleware für Bearer-Auth, `jsonwebtoken`/Session; Module `interfaces::mcp`; Architekturentscheidung: Auth als Middleware-Layer, Tools als registrierte Funktionen.

## src/policynim/templates/nvidia_guardrails/preflight_output/config.yml

- Zweck: NeMo-Guardrails-Konfiguration für den Preflight-Generator (Models, Rails, Prompts).
- Verantwortlichkeit: Definiert das Guardrails-Setup (u. a. `_DEFAULT_GUARDRAILS_MODEL` "nvidia/llama-3.3-nemotron-super-49b-v1.5"), Rails-Pfade, Prompts für den Preflight-Output.
- Eingaben: Wird vom Guardrails-Wrapper als gepacktes Asset geladen (lazy Gate).
- Ausgaben: Guardrails-Konfiguration zur Rail-Ausführung.
- Datenfluss: runtime_paths → nvidia_guardrails → NeMo Guardrails → Rails-Konfiguration.
- Persistenz: Als Paket-Asset in PyInstaller-Bundle enthalten ("policynim/policies"-ähnliche Ressourcenliste; Spec enthält `copy_metadata("policynim")`).
- Zustände: Keine.
- APIs: Keine (Datenformat).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Malformed Rail-Output (vom Wrapper behandelt).
- Sicherheitsrelevanz: Rail-Definitionen als Ausgabe-Kontrollschicht.
- Geschäftslogik: Rail-Auswahl für Preflight-Generierung (Adherence-/Trajectory-Schwellen, Regenerations-Kontext).
- Algorithmen: Keine.
- verwendete Datenmodelle: YAML-Konfigurationsschema von NeMo Guardrails.
- Abhängigkeiten: NeMo-Guardrails (optional), runtime_paths.
- Rust-Relevanz: Capability "konfigurierte Output-Rails"; Rust: Config als `include_str!` + eigener Rail-Interpreter; Module `providers::nvidia_guardrails`.

---

## src/policynim/templates/nvidia_guardrails/preflight_output/rails.co

- Zweck: Colang-Rail-Definitionen für den Preflight-Output (Conversation-Flows, Bot-Verhalten, Blocking-Regeln).
- Verantwortlichkeit: Implementiert die Guardrails-Verhaltenslogik für die Preflight-Generierung (Fail-closed, Zitations-Pflicht, Drift-Erkennung).
- Eingaben: Wird als Asset geladen (zusammen mit config.yml).
- Ausgaben: Rail-Verhaltensregeln.
- Datenfluss: config.yml → rails.co → NeMo Guardrails Engine.
- Persistenz: Paket-Asset.
- Zustände: Blocked-Zustände in Rails.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Blocked Output → vom Wrapper als Fehler/Abbruch behandelt.
- Sicherheitsrelevanz: Zentrale Ausgabe-Sicherheitsregeln (Zitationen, Drift).
- Geschäftslogik: Regeln für erlaubte/blockierte Generierungsantworten.
- Algorithmen: Colang-Flow-Ausführung.
- verwendete Datenmodelle: Colang-Syntax.
- Abhängigkeiten: NeMo-Guardrails (optional).
- Rust-Relevanz: Capability "deklarative Output-Regeln"; Rust: eigener Mini-Interpreter oder Regex-basierte Regeln; Module `providers::nvidia_guardrails`.

---

## src/policynim/templates/beta/page.html.j2

- Zweck: Basis-Template aller Beta-Seiten: HTML-Grundgerüst, `<html data-theme>`, `page_class`, Assets-Einbindung (favicon, beta.css, beta-theme-init.js, beta-page.js), Logo (light/dark), Blocks `content`.
- Verantwortlichkeit: Gemeinsames Layout/Dark-Light-Theme-Setup für Landing + Dashboard.
- Eingaben: Kontext aus `_beta_page_context` (document_title, page_class, Asset-URLs, Logo-URLs).
- Ausgaben: Vollständige HTML-Seite.
- Datenfluss: mcp.py `_render_beta_template` → Jinja2 → HTML.
- Persistenz: Keine.
- Zustände: `data-theme` light/dark (via beta-theme-init.js/LocalStorage "policynim-beta-theme").
- APIs: Keine (Server-rendert).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Jinja2-Autoescape aktiv (`select_autoescape` für html/j2).
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Kontext-Dictionaries.
- Abhängigkeiten: Jinja2, partials (logo, theme_toggle), beta.css/JS.
- Rust-Relevanz: Capability "server-rendered Templates"; Rust: `askama`/`tera`/`minijinja`; Module `interfaces::mcp`; Architekturentscheidung: Autoescape als Default.

---

## src/policynim/templates/beta/dashboard.html.j2

- Zweck: Eingeloggte Beta-Seite: Topbar (Status-Pill, Portal-URL), Notices, Account-Fakten (GitHub/Email/Status/Key-Prefix/Key-Erstellungszeit), UTC-Tages-Usage (Bar + Text), API-Key-Generate/Rotate-Form, Sign-Out-Form, Hosted-MCP-Endpunkt, Client-Setup-Command-Cards (Codex/Claude), Agent-Workflow-Cards.
- Verantwortlichkeit: Selbstbedienungs-Portal nach Sign-in.
- Eingaben: Kontext aus `_render_beta_dashboard` (account, usage, new_key, commands, workflow_cards, notices, status_class, mcp_url, portal_url, paths).
- Ausgaben: Dashboard-HTML.
- Datenfluss: mcp.py `beta_dashboard`-Route → Service (get_account/get_portal_usage) → Template.
- Persistenz: Keine (zeigt persistierte Daten).
- Zustände: account.status (active/suspended), new_key sichtbar/nicht.
- APIs: POST `api_key_regenerate_path`, POST `logout_path`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine (Formulare lösen Server-Aktionen aus).
- Fehlerfälle: Suspended-Notiz, abgelaufene Session (Redirect auf Landing).
- Sicherheitsrelevanz: Voller Secret nur direkt nach Erzeugung (`new_key`), sonst nur Prefix; Autoescape.
- Geschäftslogik: Command-Generierung (Codex/Claude mit mcp_url), Usage-Prozent (0-100), Status-Pill-Klassen.
- Algorithmen: Keine.
- verwendete Datenmodelle: `BetaAccount`, `BetaUsageSnapshot`, `IssuedApiKey`.
- Abhängigkeiten: page.html.j2, partials (logo, theme_toggle, notice, command_card, new_key_panel).
- Rust-Relevanz: Capability "dynamisches Portal-UI"; Rust: `askama`-Templates mit Typ-Sicherheit; Module `interfaces::mcp`.

---

## src/policynim/templates/beta/landing.html.j2

- Zweck: Öffentliche Beta-Seite: Hero (Titel, Subtitle, GitHub-Start-Button), Callout (Hosted-Endpunkt, Portal-URL, Flow), 3-Schritte-Quickstart-Karten.
- Verantwortlichkeit: Einstiegsseite für OAuth-Sign-in und Onboarding.
- Eingaben: Kontext aus `_render_beta_landing` (portal_url, mcp_url, github_start_path, notices, steps).
- Ausgaben: Landing-HTML (inkl. 429-Fehleranzeige).
- Datenfluss: mcp.py Beta-Routen → Template.
- Persistenz: Keine.
- Zustände: Notices (Fehler/Sperren).
- APIs: GET `github_start_path` (Link).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Notices für OAuth-Fehler/Rate-Limit (429).
- Sicherheitsrelevanz: Autoescape; keine Secrets.
- Geschäftslogik: Schritt-Flow (Authenticate → Issue key → Paste commands).
- Algorithmen: Keine.
- verwendete Datenmodelle: Schritt-Dictionaries.
- Abhängigkeiten: page.html.j2, partials.
- Rust-Relevanz: Capability "öffentliche Onboarding-Seite"; Rust: Templates via `askama`; Module `interfaces::mcp`.

---

## src/policynim/templates/beta/partials/command_card.html.j2

- Zweck: Wiederverwendbare Command-Karte: eyebrow, title, description, command-Block (`data-copy`), Copy-Button ("Copy command"/"Copy prompt").
- Verantwortlichkeit: Einheitliche Darstellung copy-pastebarer Kommandos/Prompts (Client-Setup, Agent-Workflows).
- Eingaben: `command`-Kontext (title, description, command, button_label, eyebrow).
- Ausgaben: Karten-HTML mit `[data-copy]`.
- Datenfluss: mcp.py → Template → JS (beta-page.js Clipboard).
- Persistenz: Keine.
- Zustände: Copy-State ("copied") via JS.
- APIs: `data-copy`-Attribut.
- Ereignisse: Klick → Clipboard-Write.
- Nebenwirkungen: Keine.
- Fehlerfälle: Clipboard nicht verfügbar → `window.prompt`-Fallback.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Kontext-Dictionary.
- Abhängigkeiten: beta.css (beta-copy-button), beta-page.js.
- Rust-Relevanz: Capability "Client-seitige Interaktion"; Rust: minimal (Templates + JS-Assets).

---

## src/policynim/templates/beta/partials/logo.html.j2

- Zweck: Logo-Baustein mit Light/Dark-Varianten (`compact`-Modus) über `beta-lockup__image--light/dark`-Klassen.
- Verantwortlichkeit: Branding-Konsistenz beider Beta-Seiten.
- Eingaben: `alt`, `compact`, Logo-URLs aus `_beta_page_context`.
- Ausgaben: `<img>`-Markup.
- Datenfluss: page.html.j2/dashboard/landing → Partial.
- Persistenz: Keine.
- Zustände: Theme-Wechsel (CSS blendet Variante aus).
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: assets (policynim_lightmode.png, policynim_darkmode.jpg), beta.css.
- Rust-Relevanz: Nicht erkennbar.

---

## src/policynim/templates/beta/partials/new_key_panel.html.j2

- Zweck: Panel nach Key-Generierung: zeigt den einmaligen Export-Befehl (`export POLICYNIM_TOKEN='<key>'`) mit Copy-Button.
- Verantwortlichkeit: Sichere Einmal-Anzeige des vollen Secrets.
- Eingaben: `new_key`-Kontext (export_command, button_label).
- Ausgaben: Secret-Panel-HTML.
- Datenfluss: mcp.py regenerate-Route → Template.
- Persistenz: Keine (einmalig).
- Zustände: new_key vorhanden/nicht.
- APIs: `data-copy`.
- Ereignisse: Copy.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Voller Secret nur hier; Empfehlung "copy before closing".
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: `IssuedApiKey`.
- Abhängigkeiten: beta.css (beta-card--secret), beta-page.js.
- Rust-Relevanz: Capability "Einmal-Secret-Anzeige"; Rust: gleiches Template-Muster; Sicherheitsentscheidung: Secret nie persistieren.

---

## src/policynim/templates/beta/partials/notice.html.j2

- Zweck: Notiz-Baustein (title, message, tone) für Erfolgs-/Warn-/Fehlermeldungen.
- Verantwortlichkeit: Einheitliche Rückmeldung an Nutzer (OAuth-Fehler, Rate-Limit, Suspension, Key-Erfolg).
- Eingaben: `notice`-Kontext (title, message, tone).
- Ausgaben: Notice-HTML (`beta-notice--<tone>`).
- Datenfluss: Renderer → Template.
- Persistenz: Keine.
- Zustände: tone error/success/warning.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine Secrets in Messages.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: beta.css.
- Rust-Relevanz: Nicht erkennbar.

---

## src/policynim/templates/beta/partials/theme_toggle.html.j2

- Zweck: Theme-Umschalter (aria-pressed, `data-theme-toggle`, `data-theme-label`).
- Verantwortlichkeit: Dark/Light-Umschaltung (Zustand via beta-page.js + LocalStorage).
- Eingaben: Keine.
- Ausgaben: Toggle-HTML.
- Datenfluss: Seiten → JS → `root.dataset.theme`.
- Persistenz: LocalStorage "policynim-beta-theme".
- Zustände: light/dark.
- APIs: `data-theme-toggle`, `data-theme-label`.
- Ereignisse: Klick.
- Nebenwirkungen: Theme-Wechsel.
- Fehlerfälle: Storage-Fehler → In-Memory-Zustand.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: beta-theme-init.js, beta-page.js, beta.css.
- Rust-Relevanz: Nicht erkennbar.

---

## src/policynim/assets/beta/beta-theme-init.js

- Zweck: Früh ladendes Theme-Initialisierungs-Skript (im `<head>`): liest LocalStorage "policynim-beta-theme" bzw. Präferenz und setzt `html[data-theme]` vor dem Rendern (verhindert FOUC).
- Verantwortlichkeit: Korrekte Theme-Bootstrap-Semantik für die Beta-Seiten.
- Eingaben: LocalStorage, System-Präferenz.
- Ausgaben: `data-theme`-Attribut.
- Datenfluss: Browser → DOM.
- Persistenz: LocalStorage-Read.
- Zustände: light/dark.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Setzt DOM-Attribut.
- Fehlerfälle: Storage nicht verfügbar → Fallback.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Keine.
- Rust-Relevanz: Nicht erkennbar (Client-Asset; bei Rust-Neuentwicklung als statisches Asset gebündelt).

---

## src/policynim/assets/beta/beta.css

- Zweck: Vollständige Beta-Portal-Styles (687 Zeilen): CSS-Variablen-Tokens für Light/Dark (Farben, Radii, Shadows), Layout (Shell/Topbar/Hero/Card-Grids), Komponenten (Buttons, Copy-Buttons, Status-Pill, Usage-Bar, Command-Blöcke, Notices, Facts, Step-Index), responsive Breakpoints (900px/640px).
- Verantwortlichkeit: Erscheinungsbild und Responsivität des Hosted-Portals.
- Eingaben: `data-theme`-Attribut, CSS-Klassen der Templates.
- Ausgaben: Gestylte Seite.
- Datenfluss: Templates → Klassen → CSS.
- Persistenz: Keine.
- Zustände: `html[data-theme="light|dark"]`; `body[data-js="ready"]` (Copy-Buttons sichtbar); `[data-copy-state="copied"]`.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Templates, beta-theme-init.js, beta-page.js.
- Rust-Relevanz: Nicht erkennbar (statisches Asset).

---

## src/policynim/assets/beta/beta-page.js

- Zweck: Seitenskript (58 Zeilen): Theme-Toggle-Handler (aria-pressed, Titel, Label, LocalStorage-Write), Copy-Buttons (`[data-copy]` → navigator.clipboard, `window.prompt`-Fallback, Reset nach 1400 ms), setzt `body[data-js="ready"]`.
- Verantwortlichkeit: Client-Interaktivität des Portals (Theme + Copy).
- Eingaben: DOM-Events, `data-copy`-Werte.
- Ausgaben: Clipboard-Writes, Theme-Update, UI-Zustände.
- Datenfluss: Nutzerklick → Handler → DOM/Clipboard.
- Persistenz: LocalStorage-Write (Theme).
- Zustände: theme, copyState (copied), js-ready.
- APIs: `data-theme-toggle`, `data-copy`, `data-theme-label`.
- Ereignisse: click, clipboard.
- Nebenwirkungen: LocalStorage, Clipboard.
- Fehlerfälle: Clipboard nicht verfügbar/nicht secure context → `window.prompt`-Fallback ("Copy again").
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: beta.css.
- Rust-Relevanz: Nicht erkennbar (Client-Asset; statisch gebündelt).

---

## src/policynim/assets/beta/policynim_lightmode.png

- Zweck: Light-Mode-Logo der Beta-Seiten (auch Favicon über `/favicon.ico`-Route, `media_type: image/png`).
- Verantwortlichkeit: Branding-Asset.
- Eingaben: Keine.
- Ausgaben: Bilddaten.
- Datenfluss: Route `_BETA_LIGHT_LOGO_ROUTE` → FileResponse (Cache-Control public, max-age=3600).
- Persistenz: Paket-Asset.
- Zustände: Keine.
- APIs: `/beta/assets/policynim_lightmode.png`, `/favicon.ico`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlende Datei → 404 "Missing beta asset.".
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: runtime_paths, mcp.py.
- Rust-Relevanz: Nicht erkennbar (Binär-Asset; `include_bytes!` bei Neuentwicklung).

---

## src/policynim/assets/beta/policynim_darkmode.jpg

- Zweck: Dark-Mode-Logo der Beta-Seiten (`media_type: image/jpeg`).
- Verantwortlichkeit: Branding-Asset für dunkles Theme.
- Eingaben: Keine.
- Ausgaben: Bilddaten.
- Datenfluss: Route `_BETA_DARK_LOGO_ROUTE` → FileResponse.
- Persistenz: Paket-Asset.
- Zustände: Keine.
- APIs: `/beta/assets/policynim_darkmode.jpg`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlende Datei → 404.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Keine.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: runtime_paths, mcp.py.
- Rust-Relevanz: Nicht erkennbar (Binär-Asset).

## tests/README.md

- Zweck: Testplan-Übersicht: dokumentiert die abgedeckten Bereiche (Parsing/Chunking, Ingest, Search, Runtime, Eval, Preflight, Routing, Compilation, Conformance, Trace, Regeneration, Phoenix-Eval, NeMo-Adapter-Gating, Guardrails, NVIDIA-Response-Validierung, CLI-Outputs, MCP-Parität, Hosted-HTTP/Portal/Auth/Logs, Docker-Contract, Live-Smokes) sowie die Opt-in-Env-Variablen (`POLICYNIM_RUN_DOCKER_TESTS`, `NVIDIA_API_KEY`, `POLICYNIM_BETA_MCP_URL`, `POLICYNIM_BETA_MCP_TOKEN`) und Marker-Semantik (`-m live` vs. `docker_live`).
- Verantwortlichkeit: Orientierung für Entwickler/CI über Teststrategie und Opt-in-Suites.
- Eingaben: Keine.
- Ausgaben: Keine (Doku).
- Datenfluss: Doku → Testausführung.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Dokumentiert pytest-Kommandos (`uv run --group test pytest -q -m docker_live ...`).
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Trennt Beta-Nutzer-Env (`POLICYNIM_TOKEN`) von Operator-Smoke-Env (`POLICYNIM_BETA_MCP_TOKEN`); BuildKit-Secret statt Build-Arg.
- Geschäftslogik: Marker-Zuordnung (live vs. docker_live), Opt-in-Bedingungen.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pytest, alle Testdateien.
- Rust-Relevanz: Nicht erkennbar (Teststrategie-Doku).

---

## tests/test_auth_store.py

- Zweck: Tests für `AuthStore` (SQLite): SHA256-`_hash_api_key`, idempotente Schema-Init (zweite Instanz auf gleichem Pfad ok), Upsert ohne Duplikat (account_id stabil, github_login/email aktualisiert), Key-Rotation widerruft vorheriges Secret, UTC-Zeitstempel.
- Verantwortlichkeit: Verifiziert Persistenz-/Rotations-Verträge des Beta-Auth-Stores.
- Eingaben: `_hash_api_key`, `AuthStore`-Instanzen auf tmp_path-DB, Konto-/Key-Operationen.
- Ausgaben: Assertions auf DB-Zustand (accounts, keys, revoked-Flags).
- Datenfluss: Store-API → SQLite → Assertions.
- Persistenz: tmp-DB (`POLICYNIM_BETA_AUTH_DB_PATH`-Stil).
- Zustände: Konto active; Key aktiv/revoked.
- APIs: `upsert_account_from_github`, `issue_api_key`, `revoke_api_key`, `list_accounts`.
- Ereignisse: Keine.
- Nebenwirkungen: DB-Writes in tmp.
- Fehlerfälle: Duplikat-Upserts, Rotation mit altem Secret.
- Sicherheitsrelevanz: Key-Hashing (SHA256), Prefix statt Klartext.
- Geschäftslogik: Stabilität der account_id, Update-Semantik, Ein-Aktiver-Key-Regel.
- Algorithmen: SHA256.
- verwendete Datenmodelle: `BetaAccount`, Key-Rows.
- Abhängigkeiten: auth_store, types, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Hash-Stabilität, Idempotenz, Rotation (integrationstest gegen rusqlite).

---

## tests/test_beta_auth.py

- Zweck: Tests für den GitHub-OAuth-Flow des Beta-Auth-Service: `_StubResponse`, `_StubGitHubClient` (post/get mit AssertionError bei unerwarteten URLs), `_settings(tmp_path)`-Fixture; prüft Code-Tausch, Konto-Erzeugung, Fehlerpfade.
- Verantwortlichkeit: Verifiziert die OAuth-Kommunikation gegen gestubte GitHub-API.
- Eingaben: Stub-GitHub-Antworten (payload/json_error), OAuth-Codes, Settings mit Test-Credentials.
- Ausgaben: Konten/Fehler-Assertions.
- Datenfluss: Service → Stub-Client → Assertions.
- Persistenz: tmp-DB.
- Zustände: Keine.
- APIs: `complete_github_oauth`, `build_github_authorize_url`, Service-Konstruktion.
- Ereignisse: Keine.
- Nebenwirkungen: Keine (Stubs).
- Fehlerfälle: GitHub-Fehlerantworten → `ProviderError`-Pfade, unerwartete URLs → AssertionError.
- Sicherheitsrelevanz: Client-Secret-Handling (nur in Requests an GitHub).
- Geschäftslogik: OAuth-Code-Austausch, Konto-Upsert.
- Algorithmen: Keine.
- verwendete Datenmodelle: `BetaAccount`.
- Abhängigkeiten: beta_auth service, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: OAuth-Flow mit Mock-HTTP-Server (tower::mock / wiremock).

---

## tests/test_beta_portal.py

- Zweck: Routen-Tests des Beta-Portals über starlette `TestClient` und `StubBetaAuthService` (Konto id=1, github_user_id=123, login "octocat", Prefix "pnm_existing"; UsageSnapshot 2/500/498; oauth_states; `complete_github_oauth` erzwingt code=="oauth-code"); `get_account` für fremde IDs → None.
- Verantwortlichkeit: Verifiziert Portal-Routen (Landing, Dashboard, OAuth start/callback, Regenerate, Logout) inkl. Session-/Redirect-Verhalten.
- Eingaben: TestClient-Requests, Session-Data, Query-Params.
- Ausgaben: Statuscodes, Redirects, HTML-Kontext, Session-Zustand.
- Datenfluss: Route → StubService → Response.
- Persistenz: Test-Sessions (starlette SessionMiddleware).
- Zustände: Session vorhanden/abgelaufen; OAuth-State gesetzt/gepoppt.
- APIs: `_BETA_PATH`, `/auth/github/start`, `/auth/github/callback`, `/beta/api-key/regenerate`, `/beta/logout`.
- Ereignisse: Keine.
- Nebenwirkungen: Session-Mutationen.
- Fehlerfälle: Falscher State → 400, ungültiger Code → Fehler-Notiz, abgelaufene Session → Redirect.
- Sicherheitsrelevanz: OAuth-State-Validierung, Session-Clearing.
- Geschäftslogik: Session-gebundene Dashboard-Anzeige, Key-Ausgabe.
- Algorithmen: Keine.
- verwendete Datenmodelle: `BetaAccount`, `BetaUsageSnapshot`, `IssuedApiKey`.
- Abhängigkeiten: mcp.py (Routen), starlette, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: HTTP-Routentests via axum `tower::ServiceExt::oneshot`.

---

## tests/test_ci_workflows.py

- Zweck: CI-Contract-Tests: CI-Gate exakt `uv run pytest -q -m "not live and not docker_live"` (nicht nur `-m "not live"`); hosted-smoke.yml nur `workflow_dispatch:` (kein pull_request/push), Secrets `POLICYNIM_BETA_MCP_URL`/`POLICYNIM_BETA_MCP_TOKEN`, "Validate hosted smoke secrets"-Schritt, `pytest -q -m live tests/test_hosted_mcp_live.py`, `permissions: contents: read`, Actions auf SHAs gepinnt.
- Verantwortlichkeit: Verhindert CI-Drift (Marker-Eskalation, Workflow-Sicherheit).
- Eingaben: `.github/workflows/*.yml`-Inhalte.
- Ausgaben: Assertions auf YAML-Struktur.
- Datenfluss: YAML → Parse → Assertions.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Weicheres Gate, fehlende Secrets-Validierung, unpinned Actions → Test-Fail.
- Sicherheitsrelevanz: Secrets-Handling, minimale Permissions, SHA-Pinning.
- Geschäftslogik: Marker-Politik (live/docker_live aus Default-Gate ausgeschlossen).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: PyYAML (o. ä.), pytest.
- Rust-Relevanz: Benötigte Rust-Tests: CI-YAML-Schema-Validierung (serde_yaml).

---

## tests/test_cli.py

- Zweck: Umfangreichste CLI-Testdatei (4252 Zeilen): Typer-CLI via `CliRunner` — `--version` (inkl. Fehlerpfad ohne Traceback), `init` (interaktiver Write config.env, Checkout-.env ohne Standalone-Pfade, Blank-Key-Reject, unwritable Ziel), `doctor` (JSON-Status action_required/ok, runtime_mode standalone/source_checkout, checks inkl. `local_index_path`, Legacy-`POLICYNIM_LANCEDB_URI`-Hinweis, Verzeichnis-als-Index-Pfad, MCP-Hints, Quote-Handling für Pfade mit Leerzeichen), `quickstart` (hosted-mcp Default, hosted-url-placeholder, client-commands codex/claude, Agent-Workflow-Karten, local-cli/local-mcp, Prerequisites/Safety, URL-Validierung), `mcp-config` (claude-code stdio server JSON `uv run --directory`, codex CLI+App, hosted-http mit bearer token, Ziel-Optionen-Konflikte, `--uv-command`-Reject, non-checkout `--repo-root`), `mcp-smoke` (Tool-Report, Config-File-Handshake, hosted-config-Reject, missing_tools Exit 1, Recovery-Steps, frozen-executable-Re-Entry), `beta-admin` (list-accounts/suspend/resume/revoke-key/audit-log inkl. Filter und Limit-Validierung), `runtime decide/execute` (Datei/Stdin, malformed JSON, session_id-Resolution, blocked/refused/failed/confirmation_unavailable, stderr-Prompt), `evidence report` (JSON/Markdown, atomarer Datei-Output, Verzeichnis-/Leerpfad-Reject, missing session), `search`/`preflight`-Fehlerpfade (init/ingest-Hinweise, Service-close trotz Fehler), `dump-index --count-only`, Setup-abhängige Kommandos bei umgeleiteter Config (parametrisiert).
- Verantwortlichkeit: Vertragssicherheit der gesamten CLI-Oberfläche (Exit-Codes, stdout/stderr-Trennung, Redaction, Recovery-Hinweise).
- Eingaben: `runner.invoke(app, argv, input=...)`, Env/Config-Fixtures (`configure_standalone_cli_environment`, `configure_checkout_cli_environment`, `write_env_file`, `write_ready_sqlite_index`, `clear_installer_env`, `make_stderr_prompt_confirmer`), Service-Mocks (`MockPreflightService`, `MockBetaAuthService`, `MockRuntimeDecisionService`, `make_runtime_execution_service`, `make_sqlite_runtime_execution_service`, `MockRuntimeEvidenceReportService`), `_run_mcp_stdio_smoke`/`run_server`/Factory-Monkeypatches.
- Ausgaben: Exit-Codes, stdout (Text/JSON/Markdown), stderr.
- Datenfluss: CLI → Factory-Mocks → Rendering → Assertions.
- Persistenz: tmp-Pfade (config.env, .env, index.sqlite3, runtime_evidence.sqlite3, reports).
- Zustände: runtime_mode, Setup-Zustand, decision/outcome-Zustände.
- APIs: Gesamte CLI-Fläche (siehe cli.py-Sektion).
- Ereignisse: Keine.
- Nebenwirkungen: Datei-Writes, Subprocess-Mocks, SQLite-Nutzung.
- Fehlerfälle: Alle Exit-1/2-Pfade mit exakten Meldungen (z. B. "NVIDIA_API_KEY is required for embeddings.", "Configure the path with `POLICYNIM_INDEX_DB_PATH`", "Hosted MCP URL must point to the /mcp endpoint.").
- Sicherheitsrelevanz: Secrets nie in stdout/stderr (Assertions auf `nvapi-test-key`-Abwesenheit), `<repo-root>`-Redaction, URL-Userinfo-Reject.
- Geschäftslogik: Setup-Leitfäden, Command-Erzeugung, MCP-Config-Kontrakte, Confirmation-Semantik.
- Algorithmen: Asyncio-MCP-Smoke, JSON-Rendering.
- verwendete Datenmodelle: types-Modelle, JSON-Payloads (schema_version 1).
- Abhängigkeiten: typer (CliRunner), pytest, services, storage, config_discovery.
- Rust-Relevanz: Benötigte Rust-Tests: Clap-Command-Tests (`assert_cmd`/`clap-verbosity`), stdout/stderr-Assertions, Redaction-Property-Tests.

---

## tests/test_compiler_service.py

- Zweck: Tests des Policy-Compilers: Zitationsvalidierung gegen Kontext, `insufficient_context`-Fail-closed, kompilierte Constraint-Citations, Preflight-Konditionierung, compile-once-Paketidentität.
- Verantwortlichkeit: Sichert den Kompilierungsvertrag (nur belegte Regeln).
- Eingaben: Mock-Generator-Antworten, Chunk-Kontext, TaskTypes.
- Ausgaben: `CompiledPolicyPacket`-Assertions (citations, insufficient_context, Identität).
- Datenfluss: Mock-Generator → Compiler → Paket.
- Persistenz: Keine.
- Zustände: sufficient/insufficient.
- APIs: `compile_policy`/Service.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Unbekannte Citations, Provider-Fehler → fail-closed.
- Sicherheitsrelevanz: Fail-closed.
- Geschäftslogik: Zitations-Pflicht, Identitätswahrung.
- Algorithmen: Zitations-Matching.
- verwendete Datenmodelle: `CompiledPolicyPacket`, `GeneratedCompiledPolicyDraft`.
- Abhängigkeiten: compiler service, types, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Zitations-Matching-Einheitstests, Identitäts-Hash-Stabilität.

---

## tests/test_docker_build_live.py

- Zweck: Opt-in-Docker-Build-Tests: Gate `POLICYNIM_RUN_DOCKER_TESTS=1` + `_docker_ready()` (subprocess `docker info`, Timeout 10); pytestmark `docker_live` + doppelte skipif; Test: Build ohne NVIDIA_API_KEY muss fehlschlagen.
- Verantwortlichkeit: Verhindert Regressionen des Docker-Contracts (BuildKit-Secret statt Build-Arg).
- Eingaben: Docker-Daemon, Build-Argumente.
- Ausgaben: Build-Exit-Codes.
- Datenfluss: Dockerfile → Docker-Build → Assertions.
- Persistenz: Docker-Images/Cache.
- Zustände: docker_live-Marker.
- APIs: `docker build`.
- Ereignisse: Keine.
- Nebenwirkungen: Docker-Systemnutzung.
- Fehlerfälle: Fehlender Key → Build-Failure (erwartet); Docker nicht verfügbar → Skip.
- Sicherheitsrelevanz: Key nur via BuildKit-Secret.
- Geschäftslogik: Opt-in, Marker-Semantik.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: Docker, pytest, subprocess.
- Rust-Relevanz: Benötigte Rust-Tests: Docker-Build-Contract als Integrationstest (skippable).

---

## tests/test_dockerfile_contract.py

- Zweck: Dockerfile-Contract-Tests: Dockerfile ohne `ARG`/`ENV NVIDIA_API_KEY`, `--mount=type=secret,id=nvidia_api_key` + `/run/secrets/nvidia_api_key`; hosted-beta-operations.md dokumentiert `DOCKER_BUILDKIT=1 docker build`, `--secret id=nvidia_api_key,env=NVIDIA_API_KEY`, `-t policynim-hosted .`, und NICHT `--build-arg NVIDIA_API_KEY`; railway.toml `dockerfilePath = "Dockerfile.railway"`; Railway-Dockerfile MIT `ARG NVIDIA_API_KEY` (ohne BuildKit-Secret); beide Dockerfiles bündeln Projekt-Metadaten.
- Verantwortlichkeit: Hält Docker-Contract und Operations-Doku synchron (Secret-Weitergabe je Plattform).
- Eingaben: Dockerfile, Dockerfile.railway, railway.toml, hosted-beta-operations.md.
- Ausgaben: Assertions.
- Datenfluss: Dateien → Parse → Assertions.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Build-Arg-Leak, fehlende Doku → Test-Fail.
- Sicherheitsrelevanz: BuildKit-Secret statt Build-Arg (Key-Leak-Schutz).
- Geschäftslogik: Plattform-Unterschiede (Railway ARG vs. BuildKit-Secret).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Dockerfile-Lint-Assertions (String-/Regex-Checks).

---

## tests/test_docs_hosted_onboarding.py

- Zweck: Hosted-Onboarding-Doku-Tests: `CODEX_HOSTED_COMMAND = "codex mcp add policynim --url https://<railway-domain>/mcp --bearer-token-env-var POLICYNIM_TOKEN"`, `CLAUDE_HOSTED_COMMAND = "claude mcp add --transport http policynim https://<railway-domain>/mcp --header "Authorization: Bearer $POLICYNIM_TOKEN""`; README hosted-first (Hosted-Sektion vor "## Local Contributor Setup"); README verlinkt Split-Doku-Struktur (index/contributor-guide/workflows/hosted-beta-operations); Whitespace-Normalisierung für Containment-Asserts.
- Verantwortlichkeit: Doku-Vertrag für Hosted-First-Onboarding (docs, README, examples).
- Eingaben: README.md, docs/*, examples/*-READMEs.
- Ausgaben: Assertions.
- Datenfluss: Doku → Assertions.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Veraltete Kommandos/Reihenfolge → Test-Fail.
- Sicherheitsrelevanz: Token via Env-Var (kein Literal).
- Geschäftslogik: Hosted-first-Reihenfolge, kanonische Kommandos.
- Algorithmen: Whitespace-Normalisierung.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Doku-Inhalts-Contract (Snapshot/Regex).

## tests/test_docs_runtime_workflows.py

- Zweck: Runtime-Workflows-Doku-Tests: workflows.md dokumentiert `policynim quickstart [--target hosted-mcp|local-cli|local-mcp]`, `doctor [--format text|json]`, `runtime decide|execute --input <path|->`, `evidence report --session-id <id> [--format markdown --output reports/<id>.md]`, `mcp-config [--target local-stdio|hosted-http]`, `mcp-smoke [--mcp-config-file <path>]`, `support-bundle [--include-mcp-smoke]`, `beta-admin list-accounts|suspend|resume|revoke-key|audit-log`; JSON-Kinds "shell_command"/"file_write"/"http_request"; session_id; SQLite-Nutzung; Env-Beispiele (`.env.example` etc.).
- Verantwortlichkeit: Doku-Vertrag für alle Laufzeit-Befehle und Artefakte.
- Eingaben: README, workflows.md, contributor-guide.md, policies/TEMPLATE.md, tests/README.md, docs/release.md, .env-Beispiele.
- Ausgaben: Assertions.
- Datenfluss: Doku → Assertions.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlende/abweichende Befehlsformen → Test-Fail.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Doku-Parität der CLI-Oberfläche.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pytest.
- Rust-Relevanz: Benötigte Rust-Tests: CLI-Usage-Doku-Parität.

---

## tests/test_eval_service.py

- Zweck: Eval-Service-Tests: Orchestrierung, Rerank on/off-Vergleich, isolierter Live-Eval-Index, `--regenerate`-Pfad, Backend-Auswahl (`default|nemo|nemo_evaluator|nat`), Phoenix-Eval-Reporting, Headless-UI-Skip, workspace-lokales Phoenix, deterministisches Span-Publishing, synchrone Code-Annotationen.
- Verantwortlichkeit: Sichert die Eval-Pipeline (Qualitätsnachweis) inkl. optionaler Backends.
- Eingaben: Mock-Services/Provider, Testfälle, Settings-Overrides.
- Ausgaben: Eval-Ergebnisse/Berichte.
- Datenfluss: Eval → Conformance → Provider-Mocks → Bericht/Workspace.
- Persistenz: Eval-Workspace (tmp), Phoenix-Daten.
- Zustände: Headless/UI, Backend, Rerank.
- APIs: Eval-Service, `from_settings`.
- Ereignisse: Spans.
- Nebenwirkungen: Datei-Writes, Provider-Mock-Aufrufe.
- Fehlerfälle: Fehlendes Optional-Paket → `ConfigurationError`; Regenerations-Stops.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Backend-Dispatch, Rerank-Vergleich, Workspace-Isolation.
- Algorithmen: Score-Aggregation, Span-Publishing.
- verwendete Datenmodelle: Eval-Testfälle, Conformance-Drafte.
- Abhängigkeiten: eval service, conformance, providers, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Eval-Pipeline-Integration, Span-Determinismus.

---

## tests/test_health_service.py

- Zweck: Health-Service-Tests: Readiness-Ergebnisse (ready/nicht ready), `HealthCheckResult`-Felder (table_name, row_count, mcp_url, reason), Fehler-Fallbacks bei nicht inspizierbarem Index.
- Verantwortlichkeit: Sichert die /healthz- und doctor-Bereitschaftslogik.
- Eingaben: Settings mit Index-Pfaden, Mock/echte Index-Stores.
- Ausgaben: `HealthCheckResult`-Assertions.
- Datenfluss: Health-Service → Storage → Result.
- Persistenz: tmp-Index.
- Zustände: ready/error.
- APIs: `create_runtime_health_service`, `check`, `format_health_failure_reason`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Ungültiger/fehlender Index → reason + nicht ready.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Bereitschaft = befüllter Index.
- Algorithmen: Row-Count.
- verwendete Datenmodelle: `HealthCheckResult`.
- Abhängigkeiten: health service, storage, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Readiness-Bestimmung mit tmp-DB.

---

## tests/test_hosted_mcp_live.py

- Zweck: Opt-in-Live-Tests gegen die deployte Railway-Beta: Env-Gates `POLICYNIM_BETA_MCP_URL`/`POLICYNIM_BETA_MCP_TOKEN`; `_authenticated_session()` (httpx.AsyncClient + streamable_http_client + `ClientSession.initialize`, Timeout 30/300s), `_structured_payload`, `_hosted_url` (same-origin); prüft Tools/Health gegen echte Deployment.
- Verantwortlichkeit: End-to-End-Nachweis des Hosted-MCP (Live-Smoke).
- Eingaben: Deployte URL, Bearer-Token, Tool-Argumente.
- Ausgaben: MCP-Responses, Health-Checks.
- Datenfluss: Client → Railway → MCP → Services.
- Persistenz: Keine.
- Zustände: live-Marker.
- APIs: `policy_preflight`, `policy_search`, `/healthz`.
- Ereignisse: Keine.
- Nebenwirkungen: Echte Provider-Aufrufe (kostenpflichtig).
- Fehlerfälle: Auth-Fehler, nicht erreichbar → Test-Fail; ohne Env → Skip.
- Sicherheitsrelevanz: Operator-Token (nicht Beta-Nutzer-Token).
- Geschäftslogik: Opt-in, Marker.
- Algorithmen: Keine.
- verwendete Datenmodelle: types-Modelle.
- Abhängigkeiten: httpx, mcp-Client-Bibliothek, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Live-Smoke gegen deployed Endpoint (ignored by default).

---

## tests/test_ingest.py

- Zweck: Ingest-Kerntests: Markdown-Parsing, Metadaten-Normalisierung, deterministisches Chunking, Edge-Cases (leere Sektionen, wiederholte verschachtelte Headings), `chunk_policy_documents`/`chunk_policy_document`-Kontrakte.
- Verantwortlichkeit: Sichert die Korpus-zu-Chunk-Transformation.
- Eingaben: In-Memory-Markdown-Policies, `PolicyMetadata`.
- Ausgaben: Chunk-Listen (IDs, Sektionen, Zeilenspannen, Texte).
- Datenfluss: Markdown → parser → chunking → Assertions.
- Persistenz: Keine.
- Zustände: Fence-Zustand im Parser.
- APIs: Chunking-/Parser-Funktionen.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Edge-Case-Dokumente (leer, Duplikate) → deterministische Ergebnisse.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Chunk-ID-Stabilität, Sektions-Schlüssel.
- Algorithmen: Heading-Erkennung, Duplikat-Suffixe.
- verwendete Datenmodelle: `PolicyChunk`, `PolicyMetadata`.
- Abhängigkeiten: ingest (loader/parser/chunking), pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Parser-/Chunking-Unit-Tests (gleiche Edge-Cases).

---

## tests/test_ingest_service.py

- Zweck: Ingest-Service-Tests: Orchestrierung mit lokalem SQLite-Index-Rebuild, isolierter Index für Evals, Zusammenfassungs-Statistik, Fehleraggregation, `--regenerate`-Interaktion.
- Verantwortlichkeit: Sichert End-to-End-Index-Aufbau.
- Eingaben: Korpus-Fixtures, Settings, Provider-Mocks.
- Ausgaben: Ingest-Zusammenfassung, befüllter Index, Assertions.
- Datenfluss: Korpus → Ingest-Service → Index-DB → Assertions.
- Persistenz: tmp-Index.sqlite3, runtime_rules.json.
- Zustände: Rebuild.
- APIs: `IngestService`/`from_settings`.
- Ereignisse: Keine.
- Nebenwirkungen: DB-Writes.
- Fehlerfälle: Teilfehler, fehlender Key.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Rebuild-Semantik, Isolations-Overrides.
- Algorithmen: Pipeline-Orchestrierung.
- verwendete Datenmodelle: `PolicyChunk`, Ingest-Statistik.
- Abhängigkeiten: ingest service, storage, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Ingest-Pipeline-Integration mit tmp-DB.

---

## tests/test_installer_contract.py

- Zweck: Installer-Contract-Tests: Konstanten `REPO_ROOT`, `INSTALL_SH`/`INSTALL_PS1`, `PYINSTALLER_SPEC`, `PYPROJECT`, `VERSION="0.1.0"`, `LINUX_ASSET="policynim-v0.1.0-linux-amd64.tar.gz"`; PyInstaller nur in `dependency-groups.release` (pinned `pyinstaller==`), nie in project.dependencies/dev/test; pyinstaller.spec enthält `src/policynim/interfaces/cli.py`, `copy_metadata("policynim")`, Ressourcen `"policynim/policies"`, `"policynim/evals"`, `"policynim/assets"`, `"policynim/templates"`.
- Verantwortlichkeit: Hält Release-/Installer-Vertrag (Version, Asset-Name, Bundle-Inhalte, Dependency-Isolation) stabil.
- Eingaben: scripts/install.sh|ps1, packaging/pyinstaller.spec, pyproject.toml.
- Ausgaben: Assertions.
- Datenfluss: Dateien → Parse → Assertions.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Release-Deps in Runtime, fehlende Ressourcen → Test-Fail.
- Sicherheitsrelevanz: Installer-Checksummen-Erwartung (Doku).
- Geschäftslogik: Release-only-Pinning.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pytest, tomllib.
- Rust-Relevanz: Benötigte Rust-Tests: Packaging-Contract (Cargo.toml-Pinning, Asset-Liste).

---

## tests/test_mcp.py

- Zweck: MCP-Oberflächen-Tests (1066 Zeilen): Tool-Parität `policy_preflight`/`policy_search`, top_k-Validierung, `ToolError` aus `mcp.server.fastmcp.exceptions`, Hosted-HTTP-Runtime via starlette `TestClient`, `MockPreflightService` (closed-Flag), PreflightResult "Grounded guidance for refresh-token cleanup.", PolicyGuidance "AUTH-001"/"Auth Reviews", Healthz, Bearer-Auth-Wrapper-Verhalten.
- Verantwortlichkeit: Sichert MCP-Vertrag (Tools, Fehlerserialisierung, Auth).
- Eingaben: TestClient-Requests, Tool-Aufrufe, Mocks.
- Ausgaben: Tool-Responses, Statuscodes, JSON-Fehler.
- Datenfluss: Client → MCP → Service-Mocks.
- Persistenz: Keine.
- Zustände: Auth-Ergebnisse; closed.
- APIs: `policy_preflight`, `policy_search`, `run_server`-Wiring, `/healthz`.
- Ereignisse: Hosted-Events (JSON-Lines).
- Nebenwirkungen: Keine (Mocks).
- Fehlerfälle: Ungültige top_k → `ValueError`/ToolError; Auth-Fehler → 401/403/429.
- Sicherheitsrelevanz: Bearer-Auth, keine Secrets in Responses.
- Geschäftslogik: Tool-Serialisierung (model_dump), Request-Validierung.
- Algorithmen: Keine.
- verwendete Datenmodelle: `PreflightResult`, `SearchResult`, `HealthCheckResult`.
- Abhängigkeiten: mcp.py, starlette, mcp-SDK, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: MCP-Tool-Integration (rmcp-Server gegen Test-Client).

---

## tests/test_nemo_agent_toolkit_policy_conformance.py

- Zweck: Tests für `NeMoAgentToolkitPolicyConformanceEvaluator`: optionales Package-Gate (`installed_version`-Monkeypatch → `PackageNotFoundError`), erwartet `ConfigurationError` mit "uv sync --extra nvidia-eval", `from_settings` prüft optionales Paket VOR Evaluator-Konstruktion, Assertion `evaluator.closed is True`.
- Verantwortlichkeit: Sichert sauberes Gating der optionalen NeMo-Backends (CI ohne Import).
- Eingaben: Monkeypatches (installed_version, `NVIDIAPolicyConformanceEvaluator.from_settings`), Mock-Evaluator.
- Ausgaben: Exception-Assertions, closed-Zustand.
- Datenfluss: `from_settings` → Gate → Fehler.
- Persistenz: Keine.
- Zustände: closed.
- APIs: `from_settings`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: `PackageNotFoundError` → `ConfigurationError` mit Setup-Hinweis.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Gate-Reihenfolge (Paket zuerst), Fail-closed.
- Algorithmen: Keine.
- verwendete Datenmodelle: `GeneratedPolicyConformanceDraft`.
- Abhängigkeiten: nvidia_eval, types, errors, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Feature-Gate-Verhalten (cfg-Features, fehlendes Backend → ConfigurationError).

---

## tests/test_nemo_evaluator_policy_conformance.py

- Zweck: Analoge Tests für `NeMoEvaluatorPolicyConformanceEvaluator` (110 Zeilen): Package-Gate, `ConfigurationError`-Hinweis "uv sync --extra nvidia-eval", Gate vor `from_settings`-Konstruktion, `closed is True` nach Fehler.
- Verantwortlichkeit: Gating-Vertrag des NeMo-Evaluator-Adapters.
- Eingaben: Monkeypatches, Mock-Evaluator.
- Ausgaben: Exception-/Zustands-Assertions.
- Datenfluss: `from_settings` → Gate.
- Persistenz: Keine.
- Zustände: closed.
- APIs: `from_settings`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlendes Paket → `ConfigurationError`.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Fail-closed, Ressourcen-Freigabe.
- Algorithmen: Keine.
- verwendete Datenmodelle: Conformance-Drafte.
- Abhängigkeiten: nvidia_eval, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Feature-Gate + Ressourcen-Close.

---

## tests/test_nemo_guardrails_preflight_generator.py

- Zweck: Tests für den Guardrails-Preflight-Generator: lazy Package-Gating, gepackte Assets (config.yml/rails.co), malformed Rail-Output, blocked Output, Zitations-Drift, Regeneration-Kontext-Pass-through, Default-Factory-Isolation.
- Verantwortlichkeit: Sichert die Guardrails-Ausgabe-Steuerung.
- Eingaben: Mock-Rail-Ergebnisse, Prompt-/Kontext-Fixtures.
- Ausgaben: Generator-Drafte oder Blockierungen.
- Datenfluss: Generator → Guardrails-Wrapper → Mock-Rails.
- Persistenz: Keine.
- Zustände: blocked/nicht.
- APIs: `from_settings`, Generator-Aufruf, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Malformed/blocked Output, Drift → fail-closed.
- Sicherheitsrelevanz: Output-Kontrolle.
- Geschäftslogik: Drift-Erkennung, Regenerations-Pass-through.
- Algorithmen: Rail-Ausführung (gemockt).
- verwendete Datenmodelle: `GeneratedPreflightDraft`.
- Abhängigkeiten: nvidia_guardrails, runtime_paths, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Rail-Logik-Unit-Tests (malformed/blocked/drift).

---

## tests/test_nvidia_embedder.py

- Zweck: NVIDIA-Embedder-Tests: `NVIDIAEmbedder.from_settings`, `embed_query` (Vektor-Form), Response-Validierung, Fehlerklassifikation bei malformed Payloads, closed-Semantik.
- Verantwortlichkeit: Sichert den Embedding-Provider-Vertrag.
- Eingaben: Mock-HTTP-Responses, Texte.
- Ausgaben: Vektor-Assertions.
- Datenfluss: Embedder → Mock-Transport.
- Persistenz: Keine.
- Zustände: closed.
- APIs: `from_settings`, `embed_query`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Malformed Response → Validierungsfehler; fehlender Key → `ConfigurationError`.
- Sicherheitsrelevanz: Key-Handling.
- Geschäftslogik: Request-/Response-Contract.
- Algorithmen: Keine.
- verwendete Datenmodelle: Embedding-Vektoren.
- Abhängigkeiten: nvidia provider, httpx-Mock, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Strict-Response-Deserialisierung, Fehlerpfade.

---

## tests/test_nvidia_generator.py

- Zweck: NVIDIA-Generator-Tests: Generierung von Drafte (Preflight/Conformance), Request-Aufbau, Response-Validierung, malformed Payloads, closed-Semantik.
- Verantwortlichkeit: Sichert den Generierungs-Provider-Vertrag.
- Eingaben: Mock-Responses, Prompts/Kontexte.
- Ausgaben: Generierte Drafte.
- Datenfluss: Generator → Mock-Transport.
- Persistenz: Keine.
- Zustände: closed.
- APIs: `from_settings`, generate, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Malformed/Blocked → Fehlerklassifikation.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Payload-Verträge.
- Algorithmen: Keine.
- verwendete Datenmodelle: Drafte (types).
- Abhängigkeiten: nvidia provider, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Generator-Payload-Validierung.

---

## tests/test_nvidia_live.py

- Zweck: Opt-in-Live-Tests gegen echte NVIDIA-NIM: pytestmark `live` + skipif ohne `NVIDIA_API_KEY`; `test_nvidia_embed_query_live` (`embed_query("PolicyNIM live embedding smoke test")` → nicht-leere Float-Liste); `test_nvidia_rerank_live` mit ScoredChunk-Kandidaten ("Use explicit request ids in logs.", "Rotate session tokens promptly.").
- Verantwortlichkeit: Nachweis echter Provider-Konnektivität (Embedding + Rerank).
- Eingaben: NVIDIA_API_KEY, echte API.
- Ausgaben: Vektoren/ScoredChunks.
- Datenfluss: Provider → NVIDIA NIM → Ergebnisse.
- Persistenz: Keine.
- Zustände: live-Marker.
- APIs: `NVIDIAEmbedder.from_settings`, `NVIDIAReranker.from_settings`.
- Ereignisse: Keine.
- Nebenwirkungen: Kostenpflichtige API-Aufrufe.
- Fehlerfälle: Ohne Key → Skip.
- Sicherheitsrelevanz: Key via Env.
- Geschäftslogik: Smoke-Semantik.
- Algorithmen: Keine.
- verwendete Datenmodelle: `ScoredChunk`.
- Abhängigkeiten: nvidia provider, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Live-Smoke (ignored by default).

---

## tests/test_nvidia_policy_compiler.py

- Zweck: NVIDIA-Policy-Compiler-Tests: Compiler-Payloads (Grounded Generation), Response-Validierung, malformed Payloads, Zitations-/Constraint-Erwartungen, Fail-closed.
- Verantwortlichkeit: Sichert den NVIDIA-gestützten Kompilierungsvertrag.
- Eingaben: Mock-Responses, Policy-Kontext.
- Ausgaben: Kompilierte Pakete.
- Datenfluss: Compiler → Mock-Transport.
- Persistenz: Keine.
- Zustände: closed; insufficient.
- APIs: `from_settings`, compile, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Malformed Responses → Validierungsfehler; unzureichender Kontext → fail-closed.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Payload-Verträge.
- Algorithmen: Keine.
- verwendete Datenmodelle: `CompiledPolicyPacket`.
- Abhängigkeiten: nvidia provider, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Compiler-Payload-Validierung.

---

## tests/test_nvidia_policy_conformance.py

- Zweck: NVIDIA-Conformance-Tests: Conformance-Response-Validierung (malformed grounded-generation Payloads), Backend-Auswahl, Preflight-Trace-Handling, Score-Extraktion, Fehlerklassifikation.
- Verantwortlichkeit: Sichert den Conformance-Provider-Vertrag.
- Eingaben: Mock-Responses, Requests.
- Ausgaben: Conformance-Drafte.
- Datenfluss: Conformance → Mock-Transport.
- Persistenz: Keine.
- Zustände: closed.
- APIs: `from_settings`, score, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Malformed → Validierungsfehler; Provider-Fehler.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Score-Extraktion, Backend-Dispatch.
- Algorithmen: Keine.
- verwendete Datenmodelle: `GeneratedPolicyConformanceDraft`.
- Abhängigkeiten: nvidia provider, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Conformance-Response-Validierung.

## tests/test_nvidia_reranker.py

- Zweck: NVIDIA-Reranker-Tests: `NVIDIAReranker.from_settings`, `rerank` (Kandidaten → ScoredChunk, Score-Übernahme), malformed Payloads, Response-Validierung, closed-Semantik.
- Verantwortlichkeit: Sichert den Rerank-Provider-Vertrag.
- Eingaben: Mock-Responses, Kandidaten.
- Ausgaben: `ScoredChunk`-Listen.
- Datenfluss: Reranker → Mock-Transport.
- Persistenz: Keine.
- Zustände: closed.
- APIs: `from_settings`, `rerank`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Malformed → Validierungsfehler.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Score-Zuordnung zu Chunks.
- Algorithmen: Keine.
- verwendete Datenmodelle: `ScoredChunk`.
- Abhängigkeiten: nvidia provider, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Rerank-Response-Validierung.

---

## tests/test_package_release.py

- Zweck: Package-Release-Contract-Tests: `sqlite-vec==0.1.9` in Runtime-Dependencies; KEIN lancedb/lance-namespace/lance-namespace-urllib3-client in Runtime oder optionalen Dependencies; KEIN Extra `hosted-legacy-index`.
- Verantwortlichkeit: Verhindert Legacy-Dependency-Drift (LanceDB-Rückfall) im Release.
- Eingaben: pyproject.toml.
- Ausgaben: Assertions.
- Datenfluss: TOML → Parse → Assertions.
- Persistenz: Keine.
- Zustände: Keine.
- APIs: Keine.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: LanceDB-Eintrag, fehlendes sqlite-vec → Test-Fail.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Runtime-Dependency-Politik (sqlite-vec-only).
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: pytest, tomllib.
- Rust-Relevanz: Benötigte Rust-Tests: Cargo.toml-Dependency-Contract.

---

## tests/test_policy_conformance_service.py

- Zweck: Conformance-Service-Tests: Orchestrierung, Backend-Auswahl, Scoring, Eval-Artefakt-Trace-Anhänge, Conformance-ID-Erhaltung, Preflight-Trace-Handling, fail-closed.
- Verantwortlichkeit: Sichert den Conformance-Service-Vertrag.
- Eingaben: Mock-Evaluatoren/Provider, Requests.
- Ausgaben: Conformance-Drafte.
- Datenfluss: Service → Mock-Provider.
- Persistenz: Keine.
- Zustände: closed; Backend-Zustand.
- APIs: `from_settings`, evaluate, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlendes Backend, malformed Responses.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Backend-Dispatch, Score-Aggregation.
- Algorithmen: Keine.
- verwendete Datenmodelle: `GeneratedPolicyConformanceDraft`.
- Abhängigkeiten: conformance service, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Conformance-Service-Integration.

---

## tests/test_policy_evidence_trace_service.py

- Zweck: Evidence-Trace-Tests: Materialisierung, `preflight --trace`-CLI-Output, kompakte Eval-Artefakt-Anhänge, Conformance-ID-Erhaltung.
- Verantwortlichkeit: Sichert die Trace-Verträge.
- Eingaben: Preflight-/Eval-Fixtures.
- Ausgaben: Trace-Strukturen.
- Datenfluss: Service → Trace-Artefakt.
- Persistenz: Artefakt-Dateien (tmp).
- Zustände: Keine.
- APIs: Trace-Service.
- Ereignisse: Keine.
- Nebenwirkungen: Datei-Writes.
- Fehlerfälle: Keine.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: ID-Erhalt, Kompaktform.
- Algorithmen: Trace-Reduktion.
- verwendete Datenmodelle: `PreflightTraceResult`.
- Abhängigkeiten: evidence_trace service, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Trace-Serialisierung.

---

## tests/test_policy_regeneration_service.py

- Zweck: Regenerations-Tests: compile-once-Paketidentität, typed Retry-Trigger, max-regeneration-/insufficient-context-Stops, Provider-fail-closed, Zitations-Drift-Ablehnung, `preflight --regenerate`, `eval --regenerate`.
- Verantwortlichkeit: Sichert kontrollierte Retries ohne Identity-Bruch.
- Eingaben: Mock-Generatoren, Drafte, Trigger.
- Ausgaben: Regenerierte Ergebnisse/Abbruch.
- Datenfluss: Service → Mock-Generator.
- Persistenz: Keine.
- Zustände: max/insufficient/drift.
- APIs: Regenerations-Service.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Drift → Ablehnung; Limits → Stop; Provider → fail-closed.
- Sicherheitsrelevanz: Drift-Schutz.
- Geschäftslogik: Retry-Klassifikation.
- Algorithmen: Retry-Schleife.
- verwendete Datenmodelle: Drafte, Paketidentität.
- Abhängigkeiten: regeneration service, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Retry-Schleifen-Semantik.

---

## tests/test_preflight_service.py

- Zweck: Preflight-Service-Tests (756 Zeilen): Mock-Komponenten `MockEmbedder` (Mapping "refresh token cleanup"→[1.0,0.0], "backend guidance"→[0.0,1.0], "missing citations"→[-1.0,-1.0]), `MockReranker(order=[...])`, `MockGenerator(draft)` (last_request/last_context), `MockPolicyCompiler` (calls, insufficient_context-Draft), `MockIndexStore(chunks, exists=True)` (last_query_embedding/last_top_k/last_domain); Szenarien: Happy Path (nicht insufficient), Fail-closed bei Compiler-insufficient, `_MAX_CHUNKS_PER_POLICY`-Kappung (2 Chunks/Policy), unbekannte Zitations-IDs → insufficient, Generator ohne Zitationen → insufficient, Deduplizierung + First-seen-Ordnung (SECURITY-1, BACKEND-1), Policy-Zitations-Disagree → invalid, Fallback auf Policy-level-Citations bei leerer Draft-Liste, fehlender Index → `MissingIndexError`, `close()` schließt owned Komponenten (reranker/generator/compiler).
- Verantwortlichkeit: Umfassender Vertragsnachweis der Grounding-Pipeline (Retrieval → Rerank → Generate → Compile → Validierung).
- Eingaben: PreflightRequests, Mock-Komponenten.
- Ausgaben: `PreflightResult`-Assertions (insufficient_context, citations, applicable_policies, plan_steps).
- Datenfluss: Request → Mocks → Result.
- Persistenz: Keine.
- Zustände: sufficient/insufficient; closed.
- APIs: `PreflightService.preflight`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Unbekannte Citations, leere Zitationen, Drift, fehlender Index.
- Sicherheitsrelevanz: Fail-closed-Garantien (nie ungrounded Ausgabe).
- Geschäftslogik: Zitations-Dedupe/-Validierung, Kappung, Fallbacks.
- Algorithmen: Kontext-Beschränkung, First-seen-Ordnung.
- verwendete Datenmodelle: `PreflightRequest/Result`, `DraftPolicyGuidance`, `ScoredChunk`, `GeneratedPreflightDraft`, `GeneratedCompiledPolicyDraft`.
- Abhängigkeiten: preflight service, types, errors, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Fail-closed-Matrix (alle Zitations-/Kontext-Kombinationen), Kappungs-Logik, Dedupe.

---

## tests/test_router_service.py

- Zweck: Router-Tests: Task-aware Routing, TaskProfile-Inferenz, ausgewählte-Policy-Grouping, Weak-Evidence-Fallback, `--task-type`-Override, `_LINE_SPAN_RE`-Auswertung, Kandidatenpool.
- Verantwortlichkeit: Sichert die Evidenztiefen-Auswahl.
- Eingaben: Mock-Index, Tasks, TaskTypes.
- Ausgaben: Routed-Entscheidungen.
- Datenfluss: Router → Mock-Index → Result.
- Persistenz: Keine.
- Zustände: strong/weak evidence.
- APIs: `from_settings`, route.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlender Index, keine Kandidaten.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Inferenz + Grouping + Fallback.
- Algorithmen: Pooling, Line-Span-Regex.
- verwendete Datenmodelle: `ScoredChunk`, TaskType.
- Abhängigkeiten: router service, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: TaskProfile-Inferenz, Fallback-Matrix.

---

## tests/test_runtime_decision_service.py

- Zweck: Runtime-Decision-Tests: `RuntimeRequest`-Validierung (Kinds), Regel-Matching gegen kompilierte Runtime-Rules, decision allow/confirm/block, Evidence-verknüpfte Zitationen, fail-closed.
- Verantwortlichkeit: Sichert die Entscheidungslogik vor Aktion.
- Eingaben: Requests, Rules-Artefakte (Mock/Datei).
- Ausgaben: `RuntimeDecisionResult`.
- Datenfluss: Service → Rules → Result.
- Persistenz: tmp-rules.
- Zustände: allow/confirm/block.
- APIs: `from_settings`, decide, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Ungültige Kinds, fehlende Rules → block/Fehler.
- Sicherheitsrelevanz: Fail-closed.
- Geschäftslogik: Regel-Abgleich, Zitations-Verknüpfung.
- Algorithmen: Matching.
- verwendete Datenmodelle: `RuntimeRequest`, `RuntimeDecisionResult`.
- Abhängigkeiten: runtime_decision service, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Regel-Matching-Matrix.

---

## tests/test_runtime_evidence_report_service.py

- Zweck: Evidence-Report-Tests: Session-Aggregation (counts, executions), JSON/Markdown-Rendering, Datei-Export, Fehlerpfade (unbekannte Session, Verzeichnis-Output, leerer Pfad).
- Verantwortlichkeit: Sichert den Berichtsvertrag.
- Eingaben: Evidence-Store (SQLite), session_id, Formate.
- Ausgaben: Summaries/Report-Dateien.
- Datenfluss: Store → Service → Report.
- Persistenz: tmp-Evidence-DB, Report-Dateien.
- Zustände: Keine.
- APIs: `report_session`.
- Ereignisse: Keine.
- Nebenwirkungen: Datei-Writes.
- Fehlerfälle: Unbekannte Session → Fehler; ungültige Outputs.
- Sicherheitsrelevanz: Redaction.
- Geschäftslogik: Zählungen, Format-Rendering.
- Algorithmen: SQL-Aggregation, Markdown.
- verwendete Datenmodelle: `RuntimeEvidenceSessionSummary`.
- Abhängigkeiten: runtime_evidence_report service, storage, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Aggregation + Rendering-Snapshots.

---

## tests/test_runtime_evidence_store.py

- Zweck: Evidence-Store-Tests: SQLite-Schema, Event-/Execution-Reihenfolge, Reopen-Verhalten, Concurrency, Session-Zusammenfassungen, Outcome-Klassifikation.
- Verantwortlichkeit: Sichert die persistente Evidence-Schicht.
- Eingaben: Events, Executions.
- Ausgaben: Summaries, Rows.
- Datenfluss: Store-API → SQLite.
- Persistenz: tmp-DB.
- Zustände: Outcomes.
- APIs: `record_event`, `record_execution`, `summarize_session`, `close`.
- Ereignisse: Session-/Execution-Events.
- Nebenwirkungen: DB-Writes.
- Fehlerfälle: Concurrency-Zugriffe, Reopen.
- Sicherheitsrelevanz: Redaction.
- Geschäftslogik: Reihenfolge, Aggregation.
- Algorithmen: SQL.
- verwendete Datenmodelle: `RuntimeEvidenceSessionSummary`.
- Abhängigkeiten: runtime_evidence storage, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Schema/Reopen/Concurrency (rusqlite).

---

## tests/test_runtime_execution_service.py

- Zweck: Runtime-Execution-Tests: Ausführung mit Confirmation-Handling (Callback-Erfolg/Fehler), Redaction, durable Evidence, Fail-closed, failure_class (non_zero_exit, confirmation_unavailable), File-Write/HTTP-Pfade, Session-Resolution.
- Verantwortlichkeit: Sichert die sichere Ausführungsschicht.
- Eingaben: Requests, Decision-Mocks, Confirmer-Mocks.
- Ausgaben: `RuntimeExecutionResult`.
- Datenfluss: Service → Decision → Ausführung → Evidence.
- Persistenz: tmp-Evidence-DB.
- Zustände: allowed/confirmed/blocked/refused/failed.
- APIs: `from_settings`, execute, `close`.
- Ereignisse: Evidence-Events.
- Nebenwirkungen: Gemockte/eingeschränkte Ausführung.
- Fehlerfälle: Non-zero-Exit, Confirmation-Fehler, Block.
- Sicherheitsrelevanz: Redaction, Confirmation, Fail-closed.
- Geschäftslogik: Ausführungs-Zulässigkeit, Outcome-Mapping.
- Algorithmen: Subprocess/File/HTTP-Ausführung (gemockt).
- verwendete Datenmodelle: `RuntimeExecutionResult`, `RuntimeRequest`.
- Abhängigkeiten: runtime_execution service, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Outcome-Matrix, Confirmer-Vertrag, Redaction.

---

## tests/test_runtime_paths.py

- Zweck: Runtime-Pfad-Tests: Auflösung der Daten-/Config-/Artefaktpfade (index, runtime rules, evidence, eval workspace) in standalone- und checkout-Modi, Env-Overrides.
- Verantwortlichkeit: Sichert die Pfad-Verträge (config_discovery/runtime_paths).
- Eingaben: Env, Dateisystem-Lage.
- Ausgaben: Pfad-Assertions.
- Datenfluss: Env → Auflösung → Pfade.
- Persistenz: tmp-Verzeichnisse.
- Zustände: standalone/checkout.
- APIs: Pfad-Funktionen.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Legacy-Pfade, Verzeichnis-Konflikte.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Präzedenz, Modus-Ableitung.
- Algorithmen: Keine.
- verwendete Datenmodelle: Keine.
- Abhängigkeiten: config_discovery, runtime_paths, settings, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Pfad-Auflösung (tempdir).

---

## tests/test_search_service.py

- Zweck: Search-Service-Tests: Orchestrierung (Embed → Retrieval → Rerank), Domain-Filter, top_k, Missing-Index-Handling, closed-Semantik.
- Verantwortlichkeit: Sichert den Suchvertrag.
- Eingaben: Mock-Embedder/Index/Reranker, Queries.
- Ausgaben: `SearchResult`.
- Datenfluss: Service → Mocks → Result.
- Persistenz: Keine.
- Zustände: closed.
- APIs: `from_settings`, search, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Fehlender Index → `MissingIndexError`.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Domain-Filter, Rerank.
- Algorithmen: Retrieval-Pipeline.
- verwendete Datenmodelle: `SearchRequest/Result`, `ScoredChunk`.
- Abhängigkeiten: search service, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Search-Pipeline-Integration.

---

## tests/test_service_factories.py

- Zweck: Factory-Tests: alle `create_*`-Factories (preflight, search, router, compiler, conformance, eval, runtime decision/execution/evidence report, health, beta auth, ingest, dump) — Konstruktion aus Settings, Dependency-Verdrahtung, close-/Fehlerverhalten.
- Verantwortlichkeit: Sichert das zentrale Dependency-Wiring.
- Eingaben: Settings (tmp), Env.
- Ausgaben: Service-Instanzen.
- Datenfluss: Settings → Factory → Service.
- Persistenz: tmp-Pfade.
- Zustände: Keine.
- APIs: Alle `create_*`.
- Ereignisse: Keine.
- Nebenwirkungen: Datei-/DB-Erzeugung (lazy).
- Fehlerfälle: Fehlende Konfiguration → `ConfigurationError`.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Verdrahtungskonsistenz.
- Algorithmen: Keine.
- verwendete Datenmodelle: Settings.
- Abhängigkeiten: services/__init__, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Factory-/DI-Graph-Tests.

---

## tests/test_settings_and_types.py

- Zweck: Settings-/Types-Tests (972 Zeilen): Konfigurationspräzedenz (mcp_port-Default 8123; `POLICYNIM_MCP_PORT` schlägt `PORT`; production + PORT → mcp_host `0.0.0.0`; explizites `POLICYNIM_MCP_HOST` bleibt in production erhalten; außerhalb production bleibt `127.0.0.1` auch mit PORT; User-Config ignoriert bei Plattform-Port), Env/Datei-Loading (`load_settings_without_env_file`-Fixture), Modellvalidierungen.
- Verantwortlichkeit: Sichert Konfigurations-Semantik und Typen-Verträge.
- Eingaben: Env-Sets, tmp-Config-Dateien.
- Ausgaben: Settings-Feld-Assertions.
- Datenfluss: Env/Datei → Settings.
- Persistenz: tmp-Config.
- Zustände: production/nicht; Plattform-Port.
- APIs: `get_settings`, Settings-Modell.
- Ereignisse: Keine.
- Nebenwirkungen: Keine.
- Fehlerfälle: Konflikt-/Präzedenzfälle.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Präzedenzordnung.
- Algorithmen: Präzedenz-Auflösung.
- verwendete Datenmodelle: Settings.
- Abhängigkeiten: settings, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Config-Präzedenz-Matrix (figment-artig).

---

## tests/test_sqlite_vec.py

- Zweck: sqlite-vec-Index-Tests: Schema (Tabellen policy_chunks/policy_vectors/index_metadata), Schema-Version, exists/count, KNN-Suche mit Domänen-Filtern, Kandidaten-Budget (`_DOMAIN_CANDIDATE_MULTIPLIER`/`_MIN_DOMAIN_CANDIDATES`), Add/Read-Zyklen, Fehlerpfade (fehlende Erweiterung, ungültige DB).
- Verantwortlichkeit: Sichert die Vektor-Persistenz- und Retrieval-Schicht.
- Eingaben: Chunks + Embeddings, Query-Vektoren.
- Ausgaben: ScoredChunk-Listen, counts.
- Datenfluss: Store-API → sqlite-vec → Ergebnisse.
- Persistenz: tmp-DB (inkl. Erweiterungs-Loading).
- Zustände: Schema-Version; befüllt/leer.
- APIs: `create_index_store`, `add_chunks`, `search`, `count`, `exists`, `close`.
- Ereignisse: Keine.
- Nebenwirkungen: DB-Writes, native Extension-Load.
- Fehlerfälle: Fehlende Extension, inkorrekte DB → Fehler.
- Sicherheitsrelevanz: Keine.
- Geschäftslogik: Domänen-Kandidaten-Budget, top_k.
- Algorithmen: KNN (vec0), Kandidaten-Zusammenführung.
- verwendete Datenmodelle: `PolicyChunk`, `ScoredChunk`.
- Abhängigkeiten: sqlite_vec storage, sqlite-vec, pytest.
- Rust-Relevanz: Benötigte Rust-Tests: Vektor-Retrieval-Integration (sqlite-vec-Binding).
