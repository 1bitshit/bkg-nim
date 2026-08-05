# ANALYSIS-MANIFEST

| Feld | Wert |
|---|---|
| Analysis ID | ANAL-2026-08-05-001 |
| Source Version | content-anchored via `source-checksum.md` (426 Dateien) |
| Repository List | WebAI2API (foxhui/WebAI2API), mindfoundry (CloseForge-org/mindfoundry), nvidia-nim (miztertea/nim-proxy, Rust), policyNIM (nnennandukwe/policyNIM) |
| Commit Hashes | policyNIM: `0eb2ae72f5560d3ed41aaef2af1c4d3121561e42` (frischer Clone); WebAI2API/mindfoundry/nvidia-nim: keine lokalen `.git`-Metadaten (Inhaltskopien im Eltern-Repo) → Version ausschließlich über `source-checksum.md` belegt |
| Analyzer Version | bkg-nim-studio / opencode big-pickle (2026-08-05) |
| Start Time | 2026-08-05 |
| End Time | pending |
| Total Files | 426 |
| Total Lines | pending |
| Total Capabilities | pending |
| Gate Result | PENDING |

## Nachweis Commit Hashes

```
policyNIM: 0eb2ae72f5560d3ed41aaef2af1c4d3121561e42
```

Nachweis für die Inhaltskopien (kein `.git` vorhanden):

- `ls -d repos/*/.git` → nur `repos/policyNIM/.git` existiert.
- `git -C repos/WebAI2API rev-parse HEAD` etc. löst gegen das Eltern-Repo auf und ist daher KEIN gültiger Quell-Versionsnachweis.
- Gültige Versionsverankerung: `source-checksum.md` (SHA-256 über alle 426 Dateien).
