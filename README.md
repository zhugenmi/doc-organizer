# doc-organizer

An Agent skill that organizes a project's documentation into a navigable, boundary-clear structure under `docs/` — and guides the content of each doc so the result is professionally usable.

Organize docs by **intent**, not filename: "I'm debugging a bug" → `fix/`, "why did we pick Kafka?" → `adr/`.

## What & Why

Two ideas drive this skill:

1. **Boundary declarations** — each directory has a sharp, mutually-exclusive rule for what belongs there. The declaration is the contract; the directory is just where contracts live. A folder called `fix/` means nothing until the README says "code-level defects only, not model quality issues".
2. **Date-based naming** — `YYYY-MM-DD-事宜.md` so `ls` gives you a timeline. Phase numbers (`phase14_*.md`) sort by writing order, which is meaningless to a future reader.

A tidy folder structure alone is **not** the goal. The goal is **captured knowledge**: an architecture doc that lets a new engineer understand the system, a fix doc that prevents the same bug from being rediscovered, an ADR that explains why a choice was made. This skill therefore guides both *where* a doc goes and *what* it should contain.

## When to use

- New project, want to set up `docs/` properly before docs accumulate.
- Existing flat `docs/` with 10+ files, hard to find anything.
- After a burst of phase-based development (e.g. `phase1_*.md` … `phase27_*.md`) that needs reorganizing into lasting categories.
- User asks to "tidy up" / "整理" / "规范" the docs folder.

Do **not** use for: a project with <5 docs (flat is fine), or single-purpose docs like a README that lives at repo root.

## Installation

This is an Agent skill (a `SKILL.md`-based skill) compatible with any agent that supports the skill standard: Claude Code, Codex, OpenCode, OpenClaw, and others.

Install by placing the `doc-organizer/` directory into your agent's skills folder, or simply ask your agent: "install the skill from https://github.com/zhugenmi/doc-organizer" (if your agent supports remote skill installation).

The skill is auto-discovered on startup. Invoke it by asking your agent to "organize docs", "整理项目文档", "规范文档目录", or whenever you have a messy `docs/` folder you want to structure.

## How it works

The skill runs an 8-step workflow:

1. **Inventory** — build a table of every doc: date, topic, type. Extract dates from the doc body first (`日期：`, `> Date:`, frontmatter); fall back to mtime only when the body has no date.
2. **Classify project type** — fullstack / CLI / library / pipeline / ML. Determines the directory subset. See [`references/project-types.md`](references/project-types.md).
3. **Assign boundary declarations** — one-line, mutually-exclusive rule per directory. Write them as a table in `docs/README.md`. See [`references/boundary-declarations.md`](references/boundary-declarations.md).
4. **Move + rename** — `YYYY-MM-DD-事宜.md`. Date first → timeline. Slug rewritten to describe the topic, not the phase number.
5. **Merge duplicates** — same topic across two docs, or sequential fixes to the same root cause. Concatenate with `---` separator + provenance line (`> 合并自 X + Y`). Never delete content during a merge.
6. **Seed foundational docs** — only create a directory when you have real content for it. Empty directories rot. Distinguish "will fill this sprint" (create + `.gitkeep`) from "might never fill" (declare intent in README, create on first use).
7. **Write `docs/README.md` as a router** — not an intro. Engineers come with a task; the README routes them to the right doc in one hop. Boundary table near the top, then directory tree, then "by scenario" sections.
8. **Fix cross-references** — grep for stale paths, fix them, re-run grep to verify zero residuals. A reorg with broken links is worse than the original mess.

## Directory layout

Default (fullstack) layout — trim to your project type:

```
docs/
├── README.md              # router: boundary table + scenario routing
├── arch/                  # system design: how the system runs
│   ├── overview/
│   ├── pipeline/
│   ├── client/
│   └── api/
├── adr/                   # architecture decision records: why X over Y
├── fix/                   # problems & solutions: code/perf/integration/deploy/data
├── eval/
│   └── bad-cases/         # model output quality issues (not engineering bugs)
├── monitor/               # metrics definitions
└── runbooks/              # incident SOPs (pairs with monitor/)
```

The boundary-declaration table in `docs/README.md` is the contract that keeps these mutually exclusive. Example disambiguations:

| Ambiguity | Resolution |
|---|---|
| `arch/` vs `agents/` | arch = system internals; agents = people process |
| `fix/` vs `eval/bad-cases/` | fix = fix is in code/config/process; bad-cases = fix is in prompt/data/model |
| `arch/` vs `adr/` | arch = living summary of what's in use now; adr = the decision record that argues the choice |
| `monitor/` vs `runbooks/` | monitor = metrics definition; runbooks = incident SOP |

## Doc types & templates

Each template in `assets/` defines the content baseline for its doc type.

| Type | Template | What makes it valuable |
|---|---|---|
| Architecture | [`assets/arch.template.md`](assets/arch.template.md) | Module breakdown + **operating mechanism** (lifecycle, data flow, state machine, concurrency, failure handling) + tech-stack table linking to ADRs + known limits |
| Fix | [`assets/fix.template.md`](assets/fix.template.md) | **Root-cause analysis** (排查路径, direct vs root cause) + executable verification + retrospective with reusable lessons |
| ADR | [`assets/adr.template.md`](assets/adr.template.md) | **Alternatives considered** and why each was rejected — the whole point of an ADR |
| Runbook | [`assets/runbook.template.md`](assets/runbook.template.md) | **Symptom lookup table** — on-call finds the right step in 30 seconds, not by reading prose |
| docs/README | [`assets/README.template.md`](assets/README.template.md) | Boundary-declaration table + directory tree + "by scenario" routing sections |

A doc in the right directory with shallow content is still a bad doc. The test: *six months from now, will this doc save someone an hour?* If no, it's clutter, not knowledge.

## Project type adaptations

The canonical directory set is large; most projects use a subset. See [`references/project-types.md`](references/project-types.md) for the full matrix.

- **Fullstack app** — `arch/{overview,pipeline,client,api}`. The superset; trim from here.
- **CLI / script** — no `client/`, no `monitor/` unless it emits metrics.
- **Library / SDK** — `arch/api/` is the centerpiece.
- **Data pipeline / ETL** — `arch/infrastructure/` matters; add `schemas/` or `contracts/`.
- **ML / LLM service** — needs `eval/bad-cases/` for model quality issues; add `prompts/` and `data-models/`.

When unsure, default to the fullstack layout — trimming later is cheap.

## License

MIT-0. Published by [zhugenmi](https://github.com/zhugenmi).
