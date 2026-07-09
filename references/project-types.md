# Project Types & Recommended Directory Subsets

The canonical directory set is the fullstack superset. Other project types trim it. Pick the row that matches; when between two, pick the larger.

## Canonical directories (the full set)

| Directory | Purpose | Always include? |
|-----------|---------|-----------------|
| `arch/` | System design — how the system runs internally | Yes (any non-trivial project) |
| `adr/` | Architecture Decision Records — why X not Y | Only if real decisions exist |
| `monitor/` | Metrics, logs, cost/usage observability | Only if system emits metrics |
| `eval/` | Evaluation plans, test reports, model Bad Cases | Only if there's something to evaluate |
| `fix/` | Code-level defect修复 records | Only after first real bug fix |
| `runbooks/` | Incident response SOPs | Only if monitor/ exists and is non-trivial |
| `agents/` | Team coordination norms (issue triage, on-call) | Only if team > 1 person |

## By project type

### Fullstack app (frontend + backend)

```
docs/
├── arch/
│   ├── overview/       # 顶层架构、工作流编排、跨模块系统
│   ├── pipeline/       # 后端各 Stage 设计
│   ├── client/         # 前端与 BFF 集成
│   └── api/            # 对外接口契约
├── adr/
├── monitor/
├── eval/
├── fix/
├── runbooks/
└── agents/
```

Use when: project has a UI + API + persistent data. Example: Scira (科研助手 with React frontend + FastAPI backend + LangGraph workflow).

Subdivide `arch/` only when it exceeds ~5 files in one area. A fresh fullstack project can start with flat `arch/` and subdivide later.

### CLI tool / script

```
docs/
├── arch/
│   └── overview/       # 命令结构、配置模型、退出码
├── adr/
├── eval/               # 行为测试 / golden output
└── fix/
```

Skip: `client/` (no frontend), `monitor/` (unless the CLI emits telemetry), `agents/` (usually solo), `runbooks/` (CLI tools rarely have incidents).

Use when: project is a single binary users invoke. Example: a `git-*` subcommand, a build script.

### Library / SDK

```
docs/
├── arch/
│   ├── overview/       # 模块边界、版本策略
│   └── api/            # 公共 API 参考（消费方读这个）
├── adr/
└── eval/               # 兼容性测试、迁移指南
```

`arch/api/` is the centerpiece — consumers live here. Versioning decisions go in `adr/`. Skip `fix/` (library bugs are usually issues/PRs, not docs), `monitor/` (library doesn't run), `runbooks/`.

Use when: project is consumed by other code. Example: an internal SDK, a framework.

### Data pipeline / ETL

```
docs/
├── arch/
│   ├── overview/       # DAG 拓扑、数据流向
│   ├── infrastructure/ # 中间件、部署、网络
│   └── pipeline/       # 各 Stage 设计（提取/清洗/加载）
├── adr/
├── monitor/            # 数据质量指标、SLA
├── runbooks/           # Stage 失败、数据回填
└── eval/               # 数据质量评测
```

`arch/infrastructure/` matters here — Kafka/Spark/Airflow choices belong in `adr/`, deployment topology in `arch/infrastructure/`. `runbooks/` is critical: pipeline failures need SOPs.

Use when: project moves data between systems on a schedule. Example: a nightly ETL, a streaming pipeline.

### ML / LLM service

```
docs/
├── arch/
│   ├── overview/       # 模型拓扑、推理路径
│   ├── pipeline/       # 训练 / 推理 Stage
│   └── api/            # 推理 API 契约
├── adr/                # 模型选型、指标选择
├── monitor/            # 推理延迟、token 成本、漂移
├── eval/
│   └── bad-cases/      # 模型输出质量问题（与代码 fix 隔离）
├── fix/                # 代码级缺陷（推理服务 bug）
└── runbooks/           # 模型降级、限流、回滚
```

The key addition: `eval/bad-cases/` for model quality issues (hallucination, citation errors) — these are **not** code bugs and don't belong in `fix/`. Mixing them scatters修复 records.

Use when: project serves ML/LLM predictions. Example: Scira (LLM workflow — has `eval/bad-cases/` for综述 quality issues), a recommendation service.

### Solo prototype / hackathon

```
docs/
├── README.md           # Everything in one file until it bursts
```

Don't create subdirectories until you have >5 docs. A flat `docs/` with a good README beats a premature taxonomy. Split when README gets >300 lines or when finding a doc takes >30 seconds.

## When between types

- Fullstack + ML service (e.g. LLM app with UI): merge the two — fullstack's `arch/{overview,pipeline,client,api}` + ML's `eval/bad-cases/`.
- CLI that's also a library (e.g. `pytest`): library layout + a `cli/` note in `arch/overview/`.
- Library with a hosted service: library + `monitor/` + `runbooks/` from fullstack.

When genuinely unsure, start with **fullstack minus `client/`** — it covers most cases and trims well.

## Subdivision rules

`arch/` is the directory most likely to bloat. Subdivide when:
- Flat `arch/` exceeds 5 files.
- Files naturally cluster (you find yourself prefixing slugs: `2026-06-01-pipeline-*.md`, `2026-06-05-frontend-*.md`).

Subdivision scheme (pick what fits, don't use all):
- `overview/` — cross-cutting system design (architecture diagrams, workflow orchestration, intent routing).
- `pipeline/` — backend stage-by-stage design (retrieval, parsing, generation).
- `client/` — frontend, BFF, SSE/WebSocket integration.
- `api/` — external interface contracts (REST/OpenAPI, GraphQL schema).
- `infrastructure/` — middleware, deployment topology, networking (data-pipeline projects).

Other directories (`fix/`, `monitor/`, `eval/`, `adr/`, `runbooks/`) rarely need subdivision — if `fix/` has 30 files, that's fine, they sort by date.
