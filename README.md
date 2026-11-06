# UnvibeCode Engineering Challenge 2026 — Cohort 3 Submission

**Candidate:** Ram Chandra Gupta  
**Evaluation Track:** Track 1 (Engineering Issue Investigation) & Track 2 (Developer Adoption Challenge)  
**Host Organization:** [Alphashots.AI](https://github.com/FinanceFlash/unvibecode)  
**Target Repository Analyzed:** [FastAPI (`tiangolo/fastapi`)](https://github.com/fastapi/fastapi) (1,231 source files, ~1.5M tokens)

---

## 📌 Overview

This repository documents the end-to-end analysis, technical defect investigation, and product adoption proposal developed for the **UnvibeCode Engineering Challenge (Cohort 3)**.

UnvibeCode was evaluated against the production codebase of **FastAPI** (`v0.162.0`). The review surfaced both the power of UnvibeCode's 27 reconstructed business workflows and an **original, high-impact pipeline defect** in how unresolved relationships and static assets are extracted into LLM context packages.

---

## 🔧 Track 1: Engineering Issue Investigation

### Title: `[Bug] context-pack & static-linker — Multi-schema inconsistency and static web-asset pollution in unresolved_relationships.jsonl`

* **Issue Status:** Submitted to [FinanceFlash/unvibecode/issues](https://github.com/FinanceFlash/unvibecode/issues)
* **Impact Level:** High (Data contract violation / Pipeline failure for LLM consumers)

### 1. Problem Summary
When UnvibeCode compiles the repository context bundle (`<repo>-repository-context.zip`), the exported `unresolved_relationships.jsonl` file suffers from two critical, interconnected pipeline defects:
1. **Schema Non-Uniformity / Data Corruption:** The file bundles three conflicting JSON schemas into a single `.jsonl` document without normalization. 3,756 records use the key `source_file`, while 2,215 records use the key `source`, with completely different property signatures. Any typed parser (e.g., Pydantic or TypeScript interfaces) crashes with runtime `KeyError` or schema validation exceptions.
2. **False-Positive Dependency Pollution:** The linker treats documentation HTML anchors (`<a>`), stylesheet `@import` URLs, and inline SVG data URIs (`data:image/svg+xml...`) as external module dependencies. In FastAPI alone, this inflates the unresolved dependency count by **2,188 false positives**, corrupting the dependency graph presented to developers and AI coding assistants.

### 2. Test Environment
- **UnvibeCode Version:** `shipreadyv2.customer_mvp.v10` (`schema_version: shipreadyv2.customer_analysis_status.v10`)
- **OS / Python Version:** Windows 11 / Python 3.11+
- **Public Repository URL:** `https://github.com/fastapi/fastapi` at commit `69c67a167`
- **Command:** `python -m unvibecode review --repository "<path>/fastapi"`

### 3. Empirical Verification & Schema Breakdown
Out of 5,971 lines generated in `unresolved_relationships.jsonl`:
* **Schema A (3,756 lines - Python AST parser):**
  `detail`, `line`, `module`, `raw`, `reason`, `resolution_status`, `source_file`, `symbol_resolutions`, `symbols`
* **Schema B (2,188 lines - Web/static link scraper):**
  `import_style`, `language`, `line`, `module`, `raw`, `reason`, `source` *(Missing `source_file`, `detail`, `resolution_status`)*
* **Schema C (27 lines - Uncertain symbol linker):**
  `confidence`, `is_uncertain`, `line`, `raw`, `resolution_strategy`, `source`, `symbol`, `target`, `type`, `uncertainty_reasons`

### 4. Root Cause & Recommended Fix
* **Cause:** Multiple extractor stages append raw intermediate dictionaries directly to the export stream without passing through a unifying Data Transfer Object (DTO).
* **Fix:** Introduce a single Pydantic schema for `UnresolvedRelationship` mapping `source` -> `source_file`, and route web URIs (`https://`, `data:`) to a separate `external_links.jsonl` file.

---

## 💡 Track 2: Developer Adoption & Repeat-Usage Proposal

### Approach: "UnvibeCode Blast-Radius Lens" (GitHub PR Review Bot & Pre-Push Hook)

Rather than positioning UnvibeCode as a slow, one-off full-repo auditor, deliver it as an incremental, daily code-review gate that comments directly on pull requests with an interactive "Blast-Radius Diff": revealing which business workflows, downstream endpoints, and risk surfaces are altered by the proposed changes before merging.

### Key Rationale:
1. **Fits Daily Habits:** Developers rarely re-audit an entire codebase, but they review PRs and check git diffs daily. Delta reviews answer: *"Which business workflows does this change touch?"*
2. **Prominent Brand Recall:** Avoids the invisibility of background MCP tools. High-visibility badges and visual SVG blast-radius maps on every PR ensure team-wide brand exposure.
3. **Frictionless LLM Export:** Provides a 1-click *"Copy LLM Patch Prompt"* containing only the affected workflow chunks, eliminating context bloat in tools like Cursor and Claude.

### Implementation Plan:
* **MVP:** A lightweight GitHub Action (`unvibecode/blast-radius-action`) computing graph resolution on `git diff origin/main...HEAD`.
* **Local Hook:** An `unvibecode diff` pre-commit CLI command.
* **Metric:** 14-day repeat PR interaction rate across 50 beta test repositories.

---

## 📊 Summary of FastAPI Codebase Analysis

The repository analysis executed on FastAPI (`0.162.0`) mapped:
- **1,231 Source Files** (~4.8 MB code, ~1.5M tokens)
- **27 Reconstructed Business Workflows** across 13 functional domains
- **1 Application-Level Flaw in FastAPI:** Identified a bug in [`fastapi/scripts/translate.py`](file:///fastapi/scripts/translate.py#L182-L186) where translation retry exhaustion still writes corrupted Markdown translations to disk.
- **3 Architectural Coupling Hotspots:** Identified cyclic dependency loops centered around `fastapi/__init__.py` (647 incoming edges) and `fastapi/_compat/`.

---

## 📜 Author
* **Ram Chandra Gupta**
* Lovely Professional University
* GitHub: [github.com](https://github.com)
