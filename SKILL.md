---
name: doc-organizer
description: Organize a project's documentation into a navigable, boundary-clear structure under docs/, AND guide the content of each doc type so the result is professionally usable. Use whenever the user asks to "organize docs", "set up documentation structure", "整理项目文档", "规范文档目录", "reorganize docs/", or has a messy flat docs/ folder they want to structure. Also use proactively when starting a new project that will accumulate technical docs, or when an existing docs/ folder has grown past ~10 flat files. Covers directory layout by project type, date-based naming, duplicate merging, README navigation hub, content templates for architecture/fix/ADR/runbook docs (covering architecture design, module design, operating mechanism, tech selection, bug fixes, problem-solving), and cross-reference fixing. Use this whenever the user wants docs that capture project knowledge - not just a tidy folder structure.
---

# doc-organizer

Organize project documentation into a structure where any engineer can find a doc by intent ("I'm debugging a bug" -> `fix/`, "why did we pick Kafka" -> `adr/`) rather than by guessing filenames. Built around two ideas: **boundary declarations** (each directory has a sharp rule for what belongs there) and **naming by date** (`YYYY-MM-DD-事宜.md` so timeline + topic are both sortable).

A tidy folder structure alone is not the goal. The goal is **captured knowledge**: an architecture doc that lets a new engineer understand the system, a fix doc that prevents the same bug from being rediscovered, an ADR that explains why a choice was made. This skill therefore guides both *where* a doc goes and *what* it should contain. Use the templates in `assets/` as the content baseline for each doc type.

## When to use

- New project, want to set up `docs/` properly before docs accumulate.
- Existing flat `docs/` with 10+ files, hard to find anything.
- After a burst of phase-based development (e.g. `phase1_*.md` … `phase27_*.md`) that needs reorganizing into lasting categories.
- User asks to "tidy up" / "整理" / "规范" the docs folder.

Do **not** use for: a project with <5 docs (flat is fine), or single-purpose docs like a README that lives at repo root.

## Workflow

Run these in order. Each step feeds the next.

### 1. Inventory what exists

Before moving anything, build a mental (or scratch) table of every doc: **date, topic, type** (architecture / fix / eval / decision / ops). Extract dates from the doc body first (look for `日期：`, `> Date:`, frontmatter); fall back to file mtime only when the body has no date. This matters because dates drive the final filename and getting them wrong makes the timeline lie.

```bash
# Quick inventory: dates + first heading
for f in docs/**/*.md; do echo "$(date -r "$f" +%Y-%m-%d) $f"; done | sort
```

If there are no existing docs, skip to step 3.

### 2. Classify the project type

The directory structure is a **subset chosen by project type**, not a fixed list. Read `references/project-types.md` for the full matrix, but the quick decision tree:

- Has a frontend + backend? -> fullstack layout (`arch/{overview,pipeline,client,api}`)
- Pure CLI / script? -> CLI layout (no `client/`, no `monitor/` unless it emits metrics)
- Library / SDK consumed by others? -> library layout (`arch/api/` is the centerpiece)
- Data pipeline / ETL? -> pipeline layout (`arch/infrastructure/` matters)
- ML / LLM service? -> ML layout (needs `eval/bad-cases/` for model quality issues)

When unsure, default to the **fullstack layout** - it's the superset and trimming later is cheap.

Beyond the canonical directories, add project-specific dirs when the project's core value lives in a specific artifact type. Examples: an LLM-extraction project should have `prompts/` (prompts are product logic, not config) and `data-models/` (extraction schemas); a data pipeline should have `schemas/` or `contracts/`. If a category of doc would accumulate >5 files and doesn't fit `arch/` or `fix/`, it deserves its own top-level dir. Read `references/project-types.md` for the full per-type recommendations.

### 3. Assign boundary declarations

This is the most important step. For each directory you'll create, write a one-line rule for what belongs there. The rules must be **mutually exclusive** - if a doc could go in two dirs, the rule is too fuzzy. Common ambiguity traps and how to disambiguate:

- `arch/` (technical "how the system runs") vs `agents/` (human "how the team coordinates") - both could be called "workflow docs". Declare: arch = system internals, agents = people process.
- `fix/` (problems we solved: code bugs, perf issues, integration snags, deployment failures) vs `eval/bad-cases/` (model output quality) - both are "things that went wrong". Declare: fix = engineering-side problems where the fix is in code/config/process, bad-cases = model behavior issues where the fix is in prompt/data/model. The split is *where the fix lives*, not how severe the symptom was.
- `arch/` (current tech stack as-built) vs `adr/` (why we chose X over Y) - both touch "technology selection". Declare: arch = living summary of what's in use now (a table with links to ADRs), adr = the decision record that argues the choice. A tech-stack table in `arch/overview/` points to ADRs; it doesn't duplicate their reasoning.
- `monitor/` (what to watch) vs `runbooks/` (what to do when it breaks) - declare: monitor = metrics definition, runbooks = incident SOP.

Write the declarations as a table in `docs/README.md` (see `assets/README.template.md`). The table is the contract; directories are the enforcement. See `references/boundary-declarations.md` for the full canonical rules and disambiguation cases.

### 4. Move + rename

Naming convention: `YYYY-MM-DD-事宜.md`. The date goes first so `ls` gives you a timeline. `事宜` is a short kebab-case slug of the topic, not the original filename - `phase14_paper_reading.md` becomes `2026-06-15-paper-reading-module.md` (date extracted from the doc, slug rewritten to describe the topic, not the phase number).

Why not keep phase numbers? Phase numbers are about the *writing process*, not the *content*. Six months from now nobody remembers what "phase 14" was; everyone can remember "paper-reading-module". The date preserves the timeline; the slug preserves the topic.

Use `mv` (or `git mv` if docs are tracked - but most projects gitignore `docs/`, so check first). Don't rename inside doc bodies yet - that's step 7.

### 5. Merge duplicates

After moving, look for docs that overlap. Signals to merge:
- Same topic across two docs (e.g. "SSE backend" + "frontend SSE integration" - same feature, two halves).
- A doc that's really just a subset of another.
- Sequential fixes to the same root cause (e.g. "retrieval approval" + "reference newline fix" done in the same session).

To merge: concatenate with a `---` separator between original bodies, write a new combined title, and add a `> 合并自 X + Y` line under the title so provenance isn't lost. Delete the originals.

```markdown
# Combined Title

> 合并自 Phase X 与 Phase Y。日期：2026-06-18。

---

<original body of doc 1>

---

<original body of doc 2>
```

Do **not** merge docs that merely share a theme but cover different time periods or different subsystems - a timeline of distinct work stays distinct. Over-merging destroys history; under-merging scatters related info. When in doubt, keep separate.

### 6. Seed foundational docs (only if you have real content)

Four doc types earn their existence through seed content with real substance - don't pre-create them empty. Each has a template in `assets/`; the template defines the content baseline.

- `arch/` (architecture design): write one when the system has enough shape to describe. Use `assets/arch.template.md` as the baseline - it covers background & goals, **module breakdown**, **operating mechanism** (lifecycle, data flow, state machine, concurrency, failure handling), interface contracts, and a **tech stack table** that links to ADRs. Subdivide into `overview/`, `pipeline/`, `client/`, `api/` only when flat `arch/` exceeds ~5 files. A new engineer should be able to read the arch doc and understand how the system runs.
- `adr/` (Architecture Decision Records): write one when there was a real "why X not Y" choice. Format: see `assets/adr.template.md`. Number them `0001-`, `0002-`, never reuse numbers. If no decisions worth recording yet, skip the dir. The tech-stack table in `arch/` links here; don't duplicate the reasoning in both places.
- `fix/` (problems & solutions): write one whenever the team solved a non-trivial problem - code bug, performance regression, integration failure, deployment breakage, data issue. Use `assets/fix.template.md` as the baseline: it forces a root-cause section (not just "fixed it"), a verification step, and a retrospective that extracts reusable lessons. The doc's value is the retrospective - without it, you've written a changelog entry, not a fix record. Model output quality issues go in `eval/bad-cases/`, not here.
- `runbooks/`: write one when a monitor metric has a non-obvious incident response. Format: see `assets/runbook.template.md`. Pairs with `monitor/` - every runbook should reference a monitor doc and vice versa.

Empty directories rot - but the rule has nuance. Distinguish two cases:
- **Will fill this sprint** (e.g. `arch/overview/` for a greenfield project where the first architecture doc is days away): create the dir now, with `.gitkeep` if you need git to track it. Confidence: high.
- **Might never fill** (e.g. `adr/` when no real trade-off decisions exist yet, `runbooks/` before any metrics are defined): don't create the dir. Declare the intent in README ("ADRs -> `adr/`, to be created on first real decision") and create it on first use. Confidence: low.

The test: "Am I certain a doc lands here within weeks?" Yes -> create. No -> declare intent.

### 7. Write `docs/README.md` as navigation hub

The README is not an introduction - it's a **router**. Someone lands on it with an intent ("I'm debugging X", "I want to understand Y"); the README routes them to the right directory. Structure:

1. One-line intro.
2. **Boundary declaration table** (the contract from step 3) - put this near the top, bolded.
3. Directory tree.
4. "By scenario" sections: for each common intent, list the relevant docs with one-line descriptions.

Use `assets/README.template.md` as the starting point. Every doc in the tree should appear in at least one scenario section; if a doc isn't reachable from the README, it's orphaned and will be lost.

### 8. Fix cross-references

After all moves/renames, grep for stale paths and fix them. The common stale patterns:

```bash
# Find references to old paths
grep -rn "docs/" docs/ README.md CLAUDE.md AGENTS.md 2>/dev/null
# Look specifically for old filenames or old dir structures
grep -rn "phase[0-9]\|api\.md\|<old-dir>/" docs/ 2>/dev/null
```

Fix each to the new path. After fixing, re-run the grep with the old patterns filtered out to verify zero stale refs. This is tedious but critical - a docs reorg that leaves broken links is worse than the original mess.

## Content guidance by doc type

Spatial organization gets engineers to the right directory; **content guidance** gets them a doc worth reading when they arrive. Each doc type has a template in `assets/` - use it as the baseline, not a rigid form. The sections that matter most:

**Architecture doc (`arch/`, use `assets/arch.template.md`)**. The four sections that make or break it:
- **Module breakdown** - a table of modules with one-line responsibilities and dependencies. If two modules' responsibilities overlap, the breakdown is wrong; fix it before writing more.
- **Operating mechanism** - lifecycle (startup/shutdown), request/data flow, state machines, concurrency model, failure & retry behavior. This is what a new engineer needs to debug the system. Skip it and the doc is just a pretty diagram.
- **Tech stack table** - what's in use, at what version, with links to ADRs for the "why". Living doc, updated when the stack changes.
- **Known limits & risks** - honest record of current shortcomings, so future optimization has a starting point instead of rediscovering them.

**Fix doc (`fix/`, use `assets/fix.template.md`)**. The sections that distinguish a fix record from a changelog entry:
- **Root cause analysis** with a *排查路径* (investigation path), not just the answer. Future readers need to learn how to debug, not just what this bug was. Distinguish 直接原因 (direct cause, where code broke) from 根本原因 (root cause, why the bug was possible at all).
- **Verification** - executable steps that confirm the fix works and didn't break anything else. "已验证" is not verification.
- **Retrospective** - trigger conditions, prevention measures, reusable lessons. This is the section that pays off over time. Without it, the doc is a changelog entry.

**ADR (`adr/`, use `assets/adr.template.md`)**. The section that matters:
- **理由 (rationale)** - alternatives considered and why each was rejected. This is the whole point of an ADR. Without alternatives, it's just a decision log entry that tells future-you nothing about whether the decision still holds.

**Runbook (`runbooks/`, use `assets/runbook.template.md`)**. The section that matters:
- **症状速查 (symptom lookup table)** - a table mapping observable symptoms to investigation steps. On-call at 3am needs to find the right step in 30 seconds, not read prose. Every runbook pairs with a monitor doc.

## Principles (the why behind the rules)

**Boundary declarations beat directory names.** A directory called `fix/` means nothing until README says "code-level defects only, not model quality issues". The declaration is the contract; the directory is just where contracts live. Always write the declaration before creating the directory.

**Don't pre-create directories you might never fill.** `eval/bad-cases/`, `adr/`, `runbooks/` often never get content - declare intent in README, create on first use. But directories you'll fill within weeks (e.g. `arch/overview/` on a greenfield project) can be pre-created with `.gitkeep`. The distinction is confidence: "certain within weeks" -> create; "might never" -> declare intent. See step 6 for the full rule.

**Dates first, slugs second.** `2026-06-15-paper-reading.md` sorts chronologically and reads topically. Phase numbers (`phase14_*.md`) sort by writing order, which is meaningless to a future reader. Always extract the real date from the doc body when available; mtime is a fallback, not the source of truth.

**Merge to preserve, not to compress.** The goal of merging isn't fewer files - it's keeping related info together so it's found together. A merged doc should retain both original bodies separated by `---` plus a provenance line. Never delete content during a merge; only restructure the container.

**The README is a router, not an intro.** Engineers come to docs with a task ("debug this", "understand that"). The README's job is to route them to the right doc in one hop. If they have to read three paragraphs of project intro before finding the doc list, the README failed.

**Content depth is the point.** A doc in the right directory with shallow content is still a bad doc. The templates exist to enforce a content baseline - root cause for fixes, alternatives for ADRs, operating mechanism for architecture. If a doc doesn't meet the baseline, it's not done. The test: "Six months from now, will this doc save someone an hour?" If the answer is no, the doc is clutter, not knowledge.

## Adapting to project type

The canonical directory set is large; most projects use a subset. See `references/project-types.md` for the full matrix and recommended subsets for: fullstack app, CLI tool, library/SDK, data pipeline, ML/LLM service. When in doubt, start small - you can always add a directory later, but removing an established one (and migrating its docs) is painful.

## Common pitfalls

- **Putting everything in `arch/`.** `arch/` is for system design, not "all technical docs". Fixes go in `fix/`, decisions in `adr/`, metrics in `monitor/`. If `arch/` has >15 files, subdivide (overview / pipeline / client / api) rather than letting it grow flat.
- **Keeping phase numbers in filenames.** `phase14_paper_reading.md` tells future-you nothing. Rewrite to `2026-06-15-paper-reading-module.md`.
- **Forgetting to fix refs after moving.** Always end with a grep for stale paths. A reorg with broken links is worse than the original mess.
- **Pre-creating `adr/` and `runbooks/` empty.** Only create when you have a real decision / procedure to write. Declare intent in README instead.
- **Merging unrelated docs to "reduce clutter".** Merge for relatedness, not for count. Two docs on different subsystems stay separate even if they share a theme.
- **Writing fix docs without a root-cause section.** "Changed X, now it works" is a changelog entry, not a fix record. The root cause and retrospective are what make the doc worth keeping. If you can't write a root cause, you don't understand the bug yet - keep investigating.
- **Writing arch docs as diagrams only.** A diagram shows structure; an arch doc must also explain the *operating mechanism* - how the system actually runs, where it fails, how it recovers. Without that, the doc doesn't help anyone debug.
- **Duplicating tech-stack reasoning in both `arch/` and `adr/`.** The arch doc has the stack table; the ADR has the reasoning. Link, don't copy - copies drift.

## References

- `references/project-types.md` - full directory matrix + recommended subsets by project type.
- `references/boundary-declarations.md` - canonical boundary rules for each directory, including disambiguation for common overlaps.
- `assets/README.template.md` - `docs/README.md` template with boundary table + scenario sections.
- `assets/arch.template.md` - architecture doc template (背景 / 模块划分 / 运行机制 / 技术栈 / 接口 / 已知限制).
- `assets/fix.template.md` - fix doc template (问题背景 / 症状 / 根因分析 / 解决方案 / 验证 / 复盘).
- `assets/adr.template.md` - ADR template (背景 / 决策 / 理由 / 后果).
- `assets/runbook.template.md` - runbook template (症状速查 / 排查步骤 / 验证).
