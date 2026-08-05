# BKG NIM Studio — Evidence-Based Analysis & Rust-2024 Rewrite Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Execute `/workspaces/bkg-nim/AGENTS.md` to completion (validate the existing inventory, finish all repo analyses, pass the GOAL-CHECK), then — and only then — implement a brand-new Rust-2024 app in `/workspaces/bkg-nim/code/bkg-nim/v0.1` that re-implements the combined capabilities of all analyzed repos with fresh architecture, naming, and design.

**Architecture:** Two strictly gated phases. Phase A (Analysis) consolidates the existing evidence-based inventory at `/workspaces/bkg-nim/inventory`, validates it against the actual repos in `/workspaces/bkg-nim/repos`, completes the missing WebAI2API analysis, and produces the canonical AGENTS.md document set in `/workspaces/bkg-nim/code/bkg-nim/docs/inventory` (01–11 + manifest + checksum). Phase B (Implementation) creates a Cargo project in `code/bkg-nim/v0.1` and implements each capability module-by-module with TDD. Phase B **must not start** until the GOAL-CHECK gate (Task 17) passes.

**Tech Stack:** Analysis = shell (`sha256sum`, `wc`, `git log`) + Markdown. Implementation = Rust 2024 (`rustup`, `cargo`), tokio async runtime, HTTP via axum, `reqwest` (rustls), serde/serde_json; verification via `cargo test` + `cargo clippy`.

**Decision (user-confirmed):** Target app = Rust-2024 combined re-implementation of all analyzed repos' capabilities. Missing repo policyNIM must be cloned into `repos/` and analyzed. The existing `/workspaces/bkg-nim/inventory` is the primary prior-analysis source and must be inspected, validated, and consolidated — not redone from scratch.

## Global Constraints

Copied verbatim from `/workspaces/bkg-nim/AGENTS.md`. Every task implicitly includes this section.

- Analysis docs go in `/workspaces/bkg-nim/code/bkg-nim/docs/inventory`; new code goes in `/workspaces/bkg-nim/code/bkg-nim/v0.1`.
- Checksum freeze first: `sha256sum` of all repo sources → `docs/inventory/source-checksum.md` before any analysis.
- **Never** translate code, copy function/variable/class names, copy files, or "port" anything. The Rust implementation must have its own modules, structs, traits, enums, error handling, and architecture.
- A file is NOT `FERTIG_ANALYSIERT` while any of these fields is `Nicht ermittelt`, `Unbekannt`, `Nicht analysiert`, `Nicht überprüft`, `Vermutlich`, or `Wahrscheinlich`: Verantwortlichkeit, Eingaben, Ausgaben, Datenfluss, Geschäftslogik, Abhängigkeiten, APIs, Persistenz, Algorithmen.
- Every claim needs evidence (`EV-[REPO]-000001` format: Repository, Commit, Datei, Zeilenbereich, Beziehung, Typ, Aussage). Negative findings require documented negative proof ("Keine Persistenz festgestellt." + Nachweis). "Unbekannt" is forbidden after evidence validation.
- Every file read needs a Read Evidence block: File Hash, Byte Size, Line Count, Encoding, Read Timestamp, Reader Result.
- File statuses (tracked per file in `01-source-file-index.md`): `DISCOVERED → READING → CONTENT_CAPTURED → REFERENCE_ANALYSIS → DEPENDENCY_CLOSURE → CAPABILITY_ANALYSIS → EVIDENCE_VALIDATION → FERTIG_ANALYSIERT` (or `ANALYSIS_BLOCKED`).
- Save progress after every step; resume at the first file with status `DISCOVERED`.
- GOAL-CHECK before completion: no Markdown file may contain `Nicht ermittelt`, `Unbekannt`, `Nicht analysiert`, `TODO`, or `TBD`. If any hit exists, re-analyze; only a clean sweep allows goal completion.
- Binary files that cannot be analyzed need a documented non-analyzable block (Hash, Typ, Metadaten, Verwendung) — not a per-byte dissection.

---

## Phase A — Analysis (per AGENTS.md)

### Task 1: Clone policyNIM, scaffold output dirs, checksum freeze, analysis manifest

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/source-checksum.md`
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/analysis-manifest.md`
- Directory: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/`

**Interfaces:**
- Produces: `source-checksum.md` (line format `sha256sum  <repo>/<relpath>`) and `analysis-manifest.md` (Analysis ID, Source Version, Repository List, Commit Hashes, Analyzer Version, Start Time, End Time, Total Files, Total Lines, Total Capabilities, Gate Result) that later tasks append to.

- [ ] **Step 1: Clone the missing repo**

```bash
cd /workspaces/bkg-nim/repos
git clone https://github.com/nnennandukwe/policyNIM
```

Verify: `ls /workspaces/bkg-nim/repos/policyNIM/README.md` exists and `git -C /workspaces/bkg-nim/repos/policyNIM log --oneline -1` prints a commit.

- [ ] **Step 2: Verify all four repos are present**

Run: `ls /workspaces/bkg-nim/repos/`
Expected: `WebAI2API/ mindfoundry/ nvidia-nim/ policyNIM/`

- [ ] **Step 3: Create the docs directory**

```bash
mkdir -p /workspaces/bkg-nim/code/bkg-nim/docs/inventory
```

- [ ] **Step 4: Create the checksum freeze**

```bash
cd /workspaces/bkg-nim/repos
sha256sum $(find . -type f -not -path './*/\.git/*') > /workspaces/bkg-nim/code/bkg-nim/docs/inventory/source-checksum.md
```

Verify: `wc -l /workspaces/bkg-nim/code/bkg-nim/docs/inventory/source-checksum.md` equals the count from `find . -type f -not -path './*/\.git/*' | wc -l`.

- [ ] **Step 5: Record commit hashes for the manifest**

```bash
cd /workspaces/bkg-nim/repos
for d in WebAI2API mindfoundry nvidia-nim policyNIM; do echo "$d: $(git -C $d rev-parse HEAD)"; done
```

- [ ] **Step 6: Write the manifest skeleton**

Write `analysis-manifest.md` with: `Analysis ID: ANAL-2026-08-05-001`, Source Version, the four repo URLs + commit hashes from Step 5, Analyzer Version, Start Time (now), Total Files/Total Lines/Total Capabilities = `pending`, Gate Result = `PENDING`. Use the exact `sha256sum` output as the freeze evidence block.

- [ ] **Step 7: Commit**

```bash
git add docs/superpowers/plans/2026-08-05-bkg-nim-analysis-and-rust-rewrite.md code/bkg-nim/docs/inventory/source-checksum.md code/bkg-nim/docs/inventory/analysis-manifest.md
git commit -m "chore(analysis): checksum freeze and analysis manifest"
```

---

### Task 2: Build the canonical per-file index `01-source-file-index.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/01-source-file-index.md`

**Interfaces:**
- Consumes: file list from all four repos (WebAI2API 114, mindfoundry 48, nvidia-nim 113, policyNIM from clone).
- Produces: one record per file with `File-ID: FILE-000001` (sequential across all repos), Repository, Pfad, Dateityp, Sprache, Zeilen, File Hash, Read Timestamp, Reader Status, Evidence Count, Cross Reference Count, and AGENTS.md status (initially `DISCOVERED`).

- [ ] **Step 1: Enumerate every file**

```bash
cd /workspaces/bkg-nim/repos
find . -type f -not -path './*/\.git/*' -not -path '*/node_modules/*' | sort
```

Expected: every file in all four repos, no `.git` internals, no `node_modules`.

- [ ] **Step 2: Compute line counts and hashes per file**

Use `wc -l` and `sha256sum` per file. For binary assets (PNG/SVG/etc.), record `binary` for Zeilen and mark for a non-analyzable evidence block.

- [ ] **Step 3: Write `01-source-file-index.md`**

For every file, emit the table row: `FILE-000001 | repo | path | type | language | lines | hash | read-timestamp | DISCOVERED | 0 | 0`. Assign `File-ID` sequentially. One table row per file — no exceptions, no omissions.

- [ ] **Step 4: Verify index completeness**

Run: `grep -c '^FILE-' docs/inventory/01-source-file-index.md` and compare with the `find` count from Step 1. They must match exactly.

- [ ] **Step 5: Commit**

```bash
git add code/bkg-nim/docs/inventory/01-source-file-index.md
git commit -m "docs(analysis): canonical per-file index 01-source-file-index"
```

---

### Task 3: Validate the existing mindfoundry inventory against `repos/mindfoundry`

**Files:**
- Read: `/workspaces/bkg-nim/inventory/01-mindfoundry/*.md` (8 files)
- Read: all files in `/workspaces/bkg-nim/repos/mindfoundry/` (48 files)
- Modify: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/01-source-file-index.md`

**Interfaces:**
- Consumes: per-file records from Task 2 for `mindfoundry/`.
- Produces: for each mindfoundry file — `FERTIG_ANALYSIERT` status in the index, a Read Evidence block, and an evidence count; any inventory claims that fail validation get corrected with the actual file content as source.

- [ ] **Step 1: Read every mindfoundry file**

Read all 48 files under `/workspaces/bkg-nim/repos/mindfoundry/` (source, tests, configs, JSONL, CSV, Markdown, and binary assets). Record Read Evidence (File Hash, Byte Size, Line Count, Encoding, Read Timestamp, Reader Result) per file.

- [ ] **Step 2: Cross-check every claim in `01-mindfoundry/index.md` and `files.md`**

For each claim, locate the cited file + line range in the actual repo. Mark `CONFIRMED_PRESENT` with `EV-MF-…` evidence or `CONFIRMED_ABSENT` with documented negative evidence. Fix any discrepancy in a new consolidated note (never silently drop a claim).

- [ ] **Step 3: Mark files `FERTIG_ANALYSIERT`**

For each mindfoundry file whose every required field (Verantwortlichkeit, Eingaben, Ausgaben, Datenfluss, Geschäftslogik, Abhängigkeiten, APIs, Persistenz, Algorithmen) is evidence-backed and free of forbidden terms, set the index status to `FERTIG_ANALYSIERT` and record the Read Timestamp.

- [ ] **Step 4: Verify the GOAL-CHECK terms are absent for these files**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/01-source-file-index.md`
Expected: no output for the mindfoundry rows.

- [ ] **Step 5: Commit**

```bash
git add code/bkg-nim/docs/inventory/01-source-file-index.md
git commit -m "docs(analysis): validate mindfoundry inventory (FERTIG_ANALYSIERT)"
```

---

### Task 4: Validate the existing nim-proxy inventory against `repos/nvidia-nim`

**Files:**
- Read: `/workspaces/bkg-nim/inventory/02-nim-proxy/*.md` (8 files)
- Read: all files in `/workspaces/bkg-nim/repos/nvidia-nim/` (113 files, incl. `src/*.rs`, `knowledge/`, `fuzz/`, tests, workflows)
- Modify: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/01-source-file-index.md`

**Interfaces:**
- Consumes: per-file records from Task 2 for `nvidia-nim/`.
- Produces: `FERTIG_ANALYSIERT` status + Read Evidence + `EV-NIMPROXY-…` evidence IDs for every nvidia-nim file.

- [ ] **Step 1: Read every nvidia-nim file**

Read all 113 files (`Cargo.toml`, `src/*.rs` incl. `lib.rs main.rs proxy.rs pool.rs dispatch.rs governor.rs history.rs auth.rs config.rs settings.rs`, `tests/e2e.rs`, `tests/support/mod.rs`, `fuzz/`, `knowledge/`, `.github/workflows/`, Dockerfile, compose files, `docs/`, binary assets). Record Read Evidence per file.

- [ ] **Step 2: Cross-check every claim in `02-nim-proxy/index.md` and `files.md`**

Each claim must map to a real file + line range in the repo. Emit `EV-NIMPROXY-…` evidence for `CONFIRMED_PRESENT`; document negative evidence for anything absent.

- [ ] **Step 3: Mark files `FERTIG_ANALYSIERT`**

Same rule as Task 3 Step 3: every required field evidence-backed and free of forbidden terms.

- [ ] **Step 4: Verify GOAL-CHECK terms absent for these files**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/01-source-file-index.md`
Expected: no output for the nvidia-nim rows.

- [ ] **Step 5: Commit**

```bash
git add code/bkg-nim/docs/inventory/01-source-file-index.md
git commit -m "docs(analysis): validate nim-proxy inventory (FERTIG_ANALYSIERT)"
```

---

### Task 5: Validate the existing policyNIM inventory against the cloned repo

**Files:**
- Read: `/workspaces/bkg-nim/inventory/03-policyNIM/*.md` (4 files)
- Read: all files in `/workspaces/bkg-nim/repos/policyNIM/`
- Modify: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/01-source-file-index.md`

**Interfaces:**
- Consumes: per-file records from Task 2 for `policyNIM/`.
- Produces: `FERTIG_ANALYSIERT` status + Read Evidence + `EV-POLICY-…` evidence IDs for every policyNIM file.

- [ ] **Step 1: Read every policyNIM file**

Read every file in the cloned repo (Python package `src/policynim/`, `interfaces/`, `policies/`, `evals/`, `tests/`, `docs/`, configs). Record Read Evidence per file.

- [ ] **Step 2: Cross-check every claim in `03-policyNIM/index.md` and `files.md`**

Map each claim to file + line range; emit `EV-POLICY-…` evidence for confirmed claims, documented negative evidence for absent ones.

- [ ] **Step 3: Mark files `FERTIG_ANALYSIERT`**

Same rule as Task 3 Step 3.

- [ ] **Step 4: Verify GOAL-CHECK terms absent for these files**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/01-source-file-index.md`
Expected: no output for the policyNIM rows.

- [ ] **Step 5: Commit**

```bash
git add code/bkg-nim/docs/inventory/01-source-file-index.md
git commit -m "docs(analysis): validate policyNIM inventory (FERTIG_ANALYSIERT)"
```

---

### Task 6: Complete the missing WebAI2API analysis

**Files:**
- Read: all 114 files in `/workspaces/bkg-nim/repos/WebAI2API/` (79 `.js`, 11 `.vue`, configs, `src/backend|config|server|utils`, `supervisor.js`, scripts, Dockerfile, webui)
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/04-webai2api/` with `index.md`, `files.md`, `functions.md`, `data-models.md`, `architecture.md`, `tests.md`, `rust-foundation.md`
- Modify: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/01-source-file-index.md`

**Interfaces:**
- Consumes: per-file records from Task 2 for `WebAI2API/`; the documentation format of `files.md` used by the three validated inventories (per-file blocks with Zweck/Verantwortlichkeit/Eingaben/Ausgaben/Datenfluss/Persistenz/Zustände/APIs/Ereignisse/Nebenwirkungen/Fehlerfälle/Sicherheitsrelevanz/Geschäftslogik/Algorithmen/verwendete Datenmodelle/Abhängigkeiten/Rust-Relevanz).
- Produces: complete per-file documentation, function analysis, data models, architecture, tests, and a rust-foundation section — all with `EV-WEB2API-…` evidence IDs — plus `FERTIG_ANALYSIERT` status for all 114 files.

- [ ] **Step 1: Read every WebAI2API file**

Read all 114 files. For binary assets (`.png`) and the huge `pnpm-lock.yaml`, record Read Evidence with byte size/line count and a non-analyzable note where applicable.

- [ ] **Step 2: Write `04-webai2api/index.md`**

Repo overview in the same format as the three existing index files: Zweck, Verantwortlichkeit, Komponenten, Datenfluss, Persistenz, Abhängigkeiten, Eingaben/Ausgaben, Zustände — every claim backed by a file path.

- [ ] **Step 3: Write `04-webai2api/files.md`**

One section per file using the exact per-file field blocks from the interface section. No file may be left with `Nicht ermittelt` in any field.

- [ ] **Step 4: Write `functions.md`, `data-models.md`, `architecture.md` (incl. Mermaid), `tests.md`, `rust-foundation.md`**

Follow the content contracts of the corresponding 01/02 inventory files. Cross-reference every entry to `EV-WEB2API-…` evidence.

- [ ] **Step 5: Mark all 114 files `FERTIG_ANALYSIERT`**

Update the index after every file's documentation is complete (save progress per step; resume at the first `DISCOVERED`/`READING` file on interruption).

- [ ] **Step 6: Verify GOAL-CHECK terms absent for WebAI2API**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" code/bkg-nim/docs/inventory/04-webai2api/`
Expected: no output.

- [ ] **Step 7: Commit**

```bash
git add code/bkg-nim/docs/inventory/04-webai2api/ code/bkg-nim/docs/inventory/01-source-file-index.md
git commit -m "docs(analysis): complete WebAI2API analysis (FERTIG_ANALYSIERT)"
```

---

### Task 7: `02-repository-inventory.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/02-repository-inventory.md`

**Interfaces:**
- Consumes: validated per-repo data from Tasks 3–6.
- Produces: for each of the four repositories — Verantwortlichkeit, Hauptmodule, Datenfluss, APIs, Persistenz, Konfiguration, Threads, Async, State, Risiken.

- [ ] **Step 1: Write the four repository sections**

One section per repo (mindfoundry, nim-proxy, policyNIM, WebAI2API). Every statement carries its `EV-…` evidence ID; negative facts carry documented negative evidence.

- [ ] **Step 2: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/02-repository-inventory.md`
Expected: no output.

- [ ] **Step 3: Commit**

```bash
git add code/bkg-nim/docs/inventory/02-repository-inventory.md
git commit -m "docs(analysis): 02-repository-inventory"
```

---

### Task 8: `03-function-analysis.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/03-function-analysis.md`

**Interfaces:**
- Consumes: functions.md content from all four inventories.
- Produces: per important capability — Datei, Capability, Warum existiert sie, Input, Output, State, Seiteneffekte, Algorithmus, Abhängigkeiten, Welche Rust-Konzepte werden benötigt, Was DARF NICHT übernommen werden. No code translation, no variable names.

- [ ] **Step 1: Aggregate every documented function/capability**

Walk the four inventories' function analyses; deduplicate across repos (e.g. rate limiting appears in nim-proxy and policyNIM).

- [ ] **Step 2: Write the capability entries**

Use the exact field list from the interface section, each backed by evidence IDs.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/03-function-analysis.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/03-function-analysis.md
git commit -m "docs(analysis): 03-function-analysis"
```

---

### Task 9: `04-data-model-analysis.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/04-data-model-analysis.md`

**Interfaces:**
- Consumes: data-models.md from all four inventories.
- Produces: for every class/struct/dataclass/Pydantic/TypedDict/enum/JSON structure — Zweck, Felder, Lebensdauer, Beziehungen, Ownership, Persistenz, Validierung. Concepts only, no translation.

- [ ] **Step 1: Aggregate every data model**

Cover: Rust structs in nim-proxy (Config, GovernorSettings, AppState, NimKey, ClientKey, User, StoredConfig), Python models in mindfoundry and policyNIM, JS/Vue structures in WebAI2API.

- [ ] **Step 2: Write the model entries**

One section per model with the field list from the interface section, each backed by evidence IDs.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/04-data-model-analysis.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/04-data-model-analysis.md
git commit -m "docs(analysis): 04-data-model-analysis"
```

---

### Task 10: `05-architecture-analysis.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/05-architecture-analysis.md`

**Interfaces:**
- Consumes: architecture.md from all four inventories + integration observations.
- Produces: components/services/layers/modules/pipelines/queues/events/schedulers/workers/agent loops/memory/RAG/policies/security/APIs/data flow with Mermaid diagrams. Every claim evidence-backed.

- [ ] **Step 1: Compose the cross-repo architecture**

Integrate the four repos' architectures, including where the systems overlap (NVIDIA NIM as shared upstream, policy gating in policyNIM/mindfoundry, rate limiting in nim-proxy).

- [ ] **Step 2: Write the document with Mermaid diagrams**

One Mermaid diagram per major flow; each diagram box maps to a capability with its evidence ID.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/05-architecture-analysis.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/05-architecture-analysis.md
git commit -m "docs(analysis): 05-architecture-analysis"
```

---

### Task 11: `06-integration-analysis.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/06-integration-analysis.md`

**Interfaces:**
- Consumes: data-flow and API observations from Tasks 3–6.
- Produces: which components talk to which, how, with which data/protocols/events/APIs/formats.

- [ ] **Step 1: Map every integration point**

Include: proxy → NVIDIA NIM (`/v1/chat/completions`), harness → proxy (OpenAI-compatible), mindfoundry retrieval API → Nemotron, policyNIM CLI/MCP interfaces, WebAI2API upstream adapters, dashboard/SSE polling, Discord API, Google Workspace APIs.

- [ ] **Step 2: Write the document**

Per integration: components, protocol, data format, event flow, error/retry behavior — with evidence IDs.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/06-integration-analysis.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/06-integration-analysis.md
git commit -m "docs(analysis): 06-integration-analysis"
```

---

### Task 12: `07-rust-rewrite-analysis.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/07-rust-rewrite-analysis.md`

**Interfaces:**
- Consumes: rust-foundation.md from all four inventories + function analysis.
- Produces: per capability — Beobachtete Fähigkeit, Problem, Verhalten, Randbedingungen, Rust Design Anforderungen, Neue Architekturentscheidung. Explicitly NOT a translation.

- [ ] **Step 1: Aggregate the per-repo rust-foundation content**

Walk the four `rust-foundation.md` files and the `03-function-analysis.md` capability list.

- [ ] **Step 2: Write the rewrite analysis**

For every capability, complete all six fields. Every "Neue Architekturentscheidung" must be a fresh design, not a port.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/07-rust-rewrite-analysis.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/07-rust-rewrite-analysis.md
git commit -m "docs(analysis): 07-rust-rewrite-analysis"
```

---

### Task 13: `08-memory-analysis.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/08-memory-analysis.md`

**Interfaces:**
- Consumes: mindfoundry memory/replicant analysis + policyNIM vector index analysis + nim-proxy metrics history.
- Produces: concepts-only description of Window Memory, Summary Memory, Facts, Episodic, Semantic, Graph, Consolidation, Retrieval, Embeddings.

- [ ] **Step 1: Extract memory-related capabilities**

From mindfoundry (replicants, discord memories, RAG retrieval, freshness scoring), policyNIM (SQLite vector index, chunking, embeddings, rerank), nim-proxy (metrics history snapshots).

- [ ] **Step 2: Write the document**

Describe each memory concept, what each repo actually implements, and what the combined design needs — concepts only, no code.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/08-memory-analysis.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/08-memory-analysis.md
git commit -m "docs(analysis): 08-memory-analysis"
```

---

### Task 14: `09-policy-security-analysis.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/09-policy-security-analysis.md`

**Interfaces:**
- Consumes: policyNIM policy engine, mindfoundry policy gate + PII redaction, nim-proxy auth (PBKDF2, roles, client keys, fail-closed).
- Produces: analysis of Validation, Permissions, Policy Engine, Audit, Security, Secrets, Auth, PII, Rate Limits.

- [ ] **Step 1: Extract all policy/security capabilities**

Walk the four inventories for auth, secrets, PII, rate limits, validation, audit.

- [ ] **Step 2: Write the document**

One section per listed topic; every claim evidence-backed; negative findings documented.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/09-policy-security-analysis.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/09-policy-security-analysis.md
git commit -m "docs(analysis): 09-policy-security-analysis"
```

---

### Task 15: `10-test-analysis.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/10-test-analysis.md`

**Interfaces:**
- Consumes: tests.md from all four inventories + the actual test files.
- Produces: per test file — getestete Fähigkeit, Randfälle, Annahmen, was später in Rust neu getestet werden muss.

- [ ] **Step 1: Inventory every test file**

Cover: nim-proxy `tests/e2e.rs` + `tests/support/mod.rs` + `mock_nim.py` + `loadtest.py` + `test_release_contract.py` + fuzz targets; mindfoundry `tests/`; policyNIM `tests/`; WebAI2API tests.

- [ ] **Step 2: Write the analysis**

For each test file, complete the four required fields. Note which tests will map to Rust-2024 test requirements.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/10-test-analysis.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/10-test-analysis.md
git commit -m "docs(analysis): 10-test-analysis"
```

---

### Task 16: `11-final-rewrite-foundation.md`

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/11-final-rewrite-foundation.md`

**Interfaces:**
- Consumes: all of Tasks 7–15.
- Produces: vollständige Capability-Liste, benötigte Rust-Module, benötigte Crates, Risiken, unbekannte Bereiche, offene Fragen, Reihenfolge der Rust-Neuentwicklung. This document is the contract that gates Phase B.

- [ ] **Step 1: Aggregate the capability list**

Enumerate every capability from `03-function-analysis.md` and `07-rust-rewrite-analysis.md` with its evidence IDs.

- [ ] **Step 2: Define modules, crates, risks, open questions, build order**

Propose the fresh Rust-2024 module graph and the ordered development sequence. Any genuinely open area must be listed as an explicit offene Frage — never as `Unbekannt`.

- [ ] **Step 3: Verify no forbidden terms**

Run: `grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" docs/inventory/11-final-rewrite-foundation.md`
Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add code/bkg-nim/docs/inventory/11-final-rewrite-foundation.md
git commit -m "docs(analysis): 11-final-rewrite-foundation"
```

---

### Task 17: GOAL-CHECK gate

**Files:**
- Modify: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/analysis-manifest.md`

**Interfaces:**
- Consumes: all of Tasks 1–16.
- Produces: a passing gate — manifest updated with End Time, Total Files, Total Lines, Total Capabilities, Gate Result. **Phase B may only start after this task passes.**

- [ ] **Step 1: Sweep for forbidden terms in every Markdown file**

```bash
cd /workspaces/bkg-nim/code/bkg-nim/docs/inventory
grep -rn "Nicht ermittelt\|Unbekannt\|Nicht analysiert\|TODO\|TBD" .
```

Expected: **no output.** For every hit found: re-analyze the affected file and fix it; re-run the sweep until empty. Do not proceed on any hit.

- [ ] **Step 2: Verify all files are `FERTIG_ANALYSIERT`**

Run: `grep -c "FERTIG_ANALYSIERT" 01-source-file-index.md` and compare against the total file count from Task 1. Also run: `grep -E "DISCOVERED|READING" 01-source-file-index.md` — expected: no output.

- [ ] **Step 3: Verify the checksum freeze is still current**

```bash
cd /workspaces/bkg-nim/repos
sha256sum $(find . -type f -not -path './*/\.git/*') > /tmp/recheck.md
diff /tmp/recheck.md /workspaces/bkg-nim/code/bkg-nim/docs/inventory/source-checksum.md
```

Expected: no diff. If the repos changed, re-freeze and re-validate affected files.

- [ ] **Step 4: Update the manifest**

Set Total Files, Total Lines, Total Capabilities from the index and capability list; set Gate Result = `PASS`; record End Time.

- [ ] **Step 5: Commit**

```bash
git add code/bkg-nim/docs/inventory/analysis-manifest.md
git commit -m "docs(analysis): GOAL-CHECK gate PASS"
```

---

## Phase B — Rust-2024 Implementation (gated by Task 17)

### Task 18: Install Rust toolchain and scaffold the project

**Files:**
- Create: `/workspaces/bkg-nim/code/bkg-nim/v0.1/Cargo.toml`
- Create: `/workspaces/bkg-nim/code/bkg-nim/v0.1/src/lib.rs`
- Create: `/workspaces/bkg-nim/code/bkg-nim/v0.1/src/main.rs`

**Interfaces:**
- Consumes: `11-final-rewrite-foundation.md` (must exist and be GOAL-CHECK-clean).
- Produces: a compiling Cargo project with `edition = "2024"` and a placeholder smoke test.

- [ ] **Step 1: Verify the gate**

Run: `grep -n "Gate Result" code/bkg-nim/docs/inventory/analysis-manifest.md`
Expected: `Gate Result = PASS`. If not `PASS`, stop and return to Task 17.

- [ ] **Step 2: Install the Rust toolchain**

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain stable
```

Verify: `rustc --version` and `cargo --version` both succeed and `rustc` reports edition-2024-capable stable.

- [ ] **Step 3: Scaffold the project**

```bash
mkdir -p /workspaces/bkg-nim/code/bkg-nim/v0.1/src
cd /workspaces/bkg-nim/code/bkg-nim/v0.1
cargo init --name bkg-nim-v01 --vcs none
```

Edit `Cargo.toml`: set `edition = "2024"`; add `[lib]` and `[[bin]]` sections pointing at `src/lib.rs` and `src/main.rs`; add `[profile.release]` with `lto = true`, `strip = true`, `opt-level = 3`, `codegen-units = 1` (reference: nim-proxy `Cargo.toml`). `--vcs none` keeps the crate inside the parent repo's git history (no nested `.git`).

- [ ] **Step 4: Write the lib module and smoke test**

`src/lib.rs`:
```rust
pub fn version() -> &'static str {
    env!("CARGO_PKG_VERSION")
}
```

`src/main.rs` (binary entry calling `bkg_nim_v01::version()`).

- [ ] **Step 5: Run the smoke test**

Add to `src/lib.rs`:
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn scaffold_smoke() {
        assert_eq!(super::version(), env!("CARGO_PKG_VERSION"));
    }
}
```

Run: `cargo test`
Expected: 1 passed.

- [ ] **Step 5: Run the test**

Run: `cargo test`
Expected: 1 passed.

- [ ] **Step 6: Commit**

```bash
git add code/bkg-nim/v0.1/
git commit -m "feat(v0.1): scaffold rust-2024 cargo project"
```

### Task 19: Decompose the foundation doc into module tasks and extend this plan

**Files:**
- Read: `/workspaces/bkg-nim/code/bkg-nim/docs/inventory/11-final-rewrite-foundation.md`
- Modify: this plan document

**Interfaces:**
- Consumes: the capability list, module graph, crate list, and build order from `11-final-rewrite-foundation.md`.
- Produces: Tasks 20…N appended to this plan, one per module in the foundation's build order, each with Files / Interfaces / TDD steps (write failing test → verify fail → implement → verify pass → commit) in the exact format of the tasks above.

- [ ] **Step 1: Read the foundation doc**

- [ ] **Step 2: Append one TDD task per foundation module**

Each appended task must name the concrete module file, its interfaces (exact function names + signatures produced/consumed), and a real test-first cycle. No placeholder steps.

- [ ] **Step 3: Run the self-review checklist on the extended plan**

Check: (a) every capability in the foundation has a task; (b) no `TBD`/`TODO`/`Nicht ermittelt`/`Unbekannt` anywhere; (c) signatures are consistent across tasks; (d) each task is independently testable.

- [ ] **Step 4: Commit**

```bash
git add docs/superpowers/plans/2026-08-05-bkg-nim-analysis-and-rust-rewrite.md
git commit -m "docs(plan): decompose rust-2024 implementation into TDD tasks"
```

### Task 20+: Module implementation (TDD)

Appended by Task 19. Minimum expected module sequence derived from the confirmed target (combined re-implementation) — final interfaces come from the foundation doc:

- Proxy/rate-limiting core (sliding-window limiter, key pool, affinity)
- Request dispatcher / concurrency governor
- Policy engine (validation, permissions, PII redaction, audit)
- Memory/RAG layer (chunking, embeddings store, retrieval, consolidation)
- Auth & config store (hashing, sessions, roles, fail-closed)
- HTTP API + SSE relay + metrics/history + dashboard
- End-to-end tests against a scriptable mock upstream

Each module task: `cargo test` red → green → commit. See Task 19 Step 2 for the concrete appended tasks.

---

## Execution Handoff

After the plan is approved, offer the standard choice: subagent-driven development (fresh subagent per task, two-stage review — recommended for this large multi-phase plan) or inline execution (executing-plans with batch checkpoints). Phase A tasks 3–5 can run in parallel once Task 2 completes (they touch disjoint repos).
