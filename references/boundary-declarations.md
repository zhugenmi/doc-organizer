# Boundary Declarations

Each directory needs a one-line rule that's **mutually exclusive** with every other directory. If a doc could plausibly go in two dirs, the rule is too fuzzy - tighten it. This file gives the canonical rules and disambiguates the common overlaps.

## Canonical rules

| Directory | Rule (one line) |
|-----------|----------------|
| `arch/` | System internals - how the system runs: architecture design, module breakdown, operating mechanism (lifecycle/flow/state/concurrency/failure), interface contracts, current tech stack. Technical view, as-built. |
| `adr/` | Decisions - why we chose X over Y. Timeless; not "what we built" but "why we picked it". Individual selection decisions with alternatives considered. |
| `monitor/` | Observability definitions - what metrics exist, log formats, cost formulas. |
| `eval/` | Evaluation - test plans, benchmark reports, model quality analysis. |
| `eval/bad-cases/` | Model output quality issues (hallucination, wrong citation). Fix lives in prompt/data/model, NOT code. |
| `fix/` | Problems we solved - code bugs, perf regressions, integration failures, deployment breakage, data issues. Fix lives in code/config/process. NOT model behavior. |
| `runbooks/` | Incident SOPs - what to do when a metric breaks. Pairs with `monitor/`. |
| `agents/` | Team coordination - issue triage, on-call, domain ownership. People view. |

## Common ambiguities and how to disambiguate

### `arch/` vs `agents/` - "workflow docs"

Both could be called "workflow". The split:

- `arch/` = **technical** workflow (data ETL flow, retry/compensation mechanism, LangGraph DAG). The system follows these.
- `agents/` = **human** workflow (how issues get triaged, who is on-call, how to label PRs). The team follows these.

Test: "Does a new engineer need this, or a new system component?" - engineer -> `agents/`, component -> `arch/`.

### `fix/` vs `eval/bad-cases/` - "things that went wrong"

Both are post-mortems. The split is **where the fix lives**, not how severe the symptom was:

- `fix/` = fix is in **code / config / process**. Includes: null deref, transaction not committing, slow query, memory leak, API contract mismatch, deployment config error, failed data migration. The common thread: an engineer changed code or config to resolve it.
- `eval/bad-cases/` = fix is in **prompt / training data / model**. Includes: hallucinated citation, refused valid request, biased classification, wrong tone. The surrounding code is fine; the model's behavior is the problem.

Test: "To fix this, do I edit code/config, or do I edit the prompt/data/model?" - code/config -> `fix/`, prompt/data/model -> `eval/bad-cases/`.

### `arch/` vs `adr/` - "design docs" and "tech selection"

Both are architectural, and both can touch technology selection. The split is **descriptive vs justificational**:

- `arch/` = **what we built and how it runs** - the current design, module breakdown, operating mechanism, API contract, and the *current tech stack as a table*. Descriptive. The tech-stack table lists what's in use; it links to ADRs for the "why", it doesn't argue the choice.
- `adr/` = **why we picked it** - the alternatives considered, the decision, the consequences. Justificational. Each "why X not Y" gets its own ADR.

Test: "Does this doc describe the system as it is, or argue for a choice made?" - describe -> `arch/`, argue -> `adr/`.

An `arch/` doc references ADRs ("see ADR-0003 for why we use LangGraph" in the tech-stack table), but the ADR stands alone as a decision record. Don't duplicate the reasoning in both - the arch doc links, the ADR argues.

### `monitor/` vs `runbooks/` - "ops docs"

Both are operational. The split:

- `monitor/` = **what to watch** - metric definitions, log field schemas, cost calculation formulas.
- `runbooks/` = **what to do** - step-by-step incident response when a metric goes wrong.

Test: "Does this doc answer 'what is this metric?' or 'this metric is broken, what now?'" - definition -> `monitor/`, response -> `runbooks/`.

They pair: every `runbooks/` doc should reference a `monitor/` doc (which metric triggered it), and `monitor/` docs about incident-prone metrics should reference their `runbooks/`.

### `arch/` vs `fix/` - "design problem or bug?"

Sometimes a bug reveals a design flaw, and the fix involves a design change. Both dirs could claim it. The split:

- `fix/` = **reactive** - something broke, we investigated, we solved it. The doc centers on the problem and the investigation. Even if the fix is a design change, the doc's purpose is "here's what went wrong and how we solved it".
- `arch/` = **descriptive** - the current design as it stands. If a fix changes the architecture, update the `arch/` doc to reflect the new state, and let `fix/` record what happened.

Test: "Is this doc about an incident and its resolution, or about the system's current design?" - incident -> `fix/`, current design -> `arch/`. Both may exist for the same change: `fix/` records the journey, `arch/` records the destination.

### `eval/` vs `fix/` - "test docs"

Both can involve failures. The split:

- `eval/` = **planned evaluation** - test plans, benchmark methodology, periodic quality reports. Proactive.
- `fix/` = **reactive修复** - a specific problem was found and solved. Reactive.

Test: "Was this doc written before (plan) or after (post-mortem) the work?" - before -> `eval/`, after -> `fix/`.

## When a doc spans two categories

Sometimes a doc genuinely spans two. Two strategies:

1. **Split the doc.** If half is "why we chose X" and half is "how X is built", split into `adr/0007-chose-x.md` and `arch/.../x-implementation.md`, cross-link them.
2. **Primary home + cross-link.** If one aspect dominates, file under the primary and add a "See also" pointer in the other directory's README section.

Don't duplicate the content into both dirs - that breaks the "single source of truth" and the copies will drift.

## Declaring boundaries in README

The boundary table belongs near the top of `docs/README.md`, bolded. Format:

```markdown
## 📌 各目录边界声明

| 内容 | 归属目录 |
|------|----------|
| 代码 Bug / 性能 / 集成 / 部署 / 数据问题修复 | `fix/` |
| 模型 Bad Case 分析（修复在 prompt/数据/模型） | `eval/bad-cases/` |
| 技术设计 / 模块划分 / 运行机制 / 技术栈总览 | `arch/` |
| 架构决策记录（为什么选 X 不选 Y） | `adr/` |
| 人机协作流程 | `agents/` |
| 监控指标定义 | `monitor/` |
| 故障应急操作 | `runbooks/` |
```

The table is the contract. Directories enforce it; the README explains it. When a new doc arrives and doesn't fit a row, that's a signal to either tighten an existing rule or (rarely) add a new directory - but only after writing the new rule.
