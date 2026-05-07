# CRANE
### Cognitive Runtime for Autonomous & Neural Execution

> An enterprise-grade AI agent framework built in Python by [Acren Digital](https://acrendigital.com).  
> Lightweight by design. Powerful by architecture.

---

## What is CRANE?

CRANE is a private AI agent framework that provides the infrastructure to build, deploy, monitor, and evolve two distinct classes of intelligent agents:

- **Conversational Agents** — branded chatbot interfaces embedded on client websites. They respond to users, retrieve information, take actions, and remember who they've spoken to.
- **Autonomous Agents** — background agents that execute tasks on a schedule or on demand, without requiring a human to initiate them. They integrate with external services, process data, and report results.

All agents are managed through two dashboards:

- **CRANE Studio** — the developer dashboard for building, testing, and monitoring agents
- **CRANE Portal** — the production dashboard where business clients interact with their live agents

---

## Core Concepts

CRANE uses a vocabulary grounded in neuroscience and cognitive psychology. These terms are precise — use them exactly as defined throughout the codebase, documentation, and any communication about the framework.

| Term | Definition |
|---|---|
| **Working Memory** | Data held in the agent's active context window. Volatile — exists for the duration of one session only. |
| **Episodic Memory** | The agent's record of its own notable experiences: successes, failures, and novel tasks it has never performed before. |
| **Procedural Memory** | Long-term memory of how the agent performs its functions. Updated only through Neuroplasticity runs. |
| **Neuroplasticity** | The process by which an agent reorganizes its own memory — analyzing episodic patterns and suggesting updates to procedural memory via a GitHub pull request. |
| **Cognitive Dissonance** | The state of an agent operating on conflicting instructions — detected across system prompt, tools, and user input. Surfaced as a warning in CRANE Studio. |
| **Habits** | Scheduled tasks that an autonomous agent performs on a recurring basis. A Habit is an Action with a cron schedule attached. |
| **Actions** | Discrete, executable tasks an agent can perform. Triggered manually via CRANE Portal or automatically via a Habit schedule. |
| **Connectors** | Reusable integration modules that encapsulate communication with an external service (CRM, scheduling software, email, etc.). |

---

## Agent Types

| Type | Role |
|---|---|
| `Agent` | Base type. General-purpose. No orchestration behavior. |
| `OrchestratorAgent` | Receives a high-level task, decomposes it into subtasks, generates instructions for sub-agents, and delegates. |
| `WorkerAgent` | Receives a scoped instruction payload from an Orchestrator and executes it. Returns a structured result. |

---

## Tech Stack

| Layer | Technology |
|---|---|
| API & Backend | FastAPI (Python) |
| Relational Database | PostgreSQL 16 |
| Document Database | MongoDB 7 |
| Vector Database | ChromaDB (local) |
| Embeddings | Voyage AI (`voyage-3`) |
| Task Scheduling | APScheduler |
| Cache & Pub/Sub | Redis 7 |
| Object Storage | MinIO |
| Memory Source of Truth | GitHub (one private repo per client organization) |
| Infrastructure | Docker Compose (local) → Docker on Digital Ocean (production) |
| LLM | Model-agnostic — Claude by default |

---

## Repository Structure

```
crane/                          # Monorepo root
├── pyproject.toml              # Project config and dependencies (uv)
├── .env.example                # All environment variables documented
├── docker-compose.yml          # Full local infrastructure
├── Makefile                    # Developer task runner
│
├── crane/                      # Main Python package
│   ├── core/                   # Base classes, type system, exceptions
│   ├── auth/                   # JWT authentication, RBAC, org isolation
│   ├── memory/                 # Working, episodic, and procedural memory
│   ├── runtime/                # Agent inference pipeline
│   ├── tracers/                # LLM call logging and cost tracking
│   ├── habits/                 # Action registry and Habit scheduler
│   ├── connectors/             # External service connector base and registry
│   ├── neuroplasticity/        # Memory reorganization engine
│   ├── dissonance/             # Cognitive conflict detection
│   ├── tools/                  # Agent tool registry
│   ├── rag/                    # Document ingestion and retrieval pipeline
│   ├── mcp/                    # MCP server base class
│   ├── studio/                 # CRANE Studio API routes
│   └── portal/                 # CRANE Portal API routes
│
├── tests/
│   ├── unit/                   # Unit tests — all external dependencies mocked
│   ├── integration/            # Integration tests — run against live Docker services
│   └── evals/                  # Agent eval test suites
│
├── frontend/
│   ├── studio/                 # CRANE Studio React application
│   ├── portal/                 # CRANE Portal React application
│   └── widget/                 # Embeddable chatbot widget (crane-widget.js)
│
└── docs/
    ├── ARCHITECTURE.md         # Full system architecture specification
    ├── STANDARDS.md            # Coding standards and engineering principles
    └── decisions/              # Architecture Decision Records (ADRs)
```

---

## Getting Started

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- [uv](https://docs.astral.sh/uv/) installed (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- Python 3.12+
- A GitHub personal access token with `repo` scope
- An Anthropic API key
- A Voyage AI API key

### Local Setup

```bash
# 1. Clone the repository
git clone https://github.com/acren-digital/crane.git
cd crane

# 2. Install dependencies
make install

# 3. Configure environment
cp .env.example .env
# Edit .env and fill in all required values

# 4. Start infrastructure
make up

# 5. Run database migrations
make migrate

# 6. Verify everything is working
make test
```

### Environment Variables

Copy `.env.example` to `.env` and fill in the required values. Every variable is documented with a description and example in `.env.example`. Never commit `.env` to version control — it is listed in `.gitignore`.

---

## Development

```bash
make install          # Install all dependencies
make up               # Start all Docker services
make down             # Stop all Docker services
make test             # Run full test suite
make test-unit        # Run unit tests only (no Docker required)
make test-integration # Run integration tests (Docker must be running)
make lint             # Run ruff linter
make format           # Auto-format code
make typecheck        # Run mypy static type checker
make migrate          # Run database migrations
```

### Development Process

CRANE is built with **Test-Driven Development**. The workflow for every feature is:

1. Write a failing test that describes the behavior you want
2. Write the minimum code to make the test pass
3. Refactor without breaking the test
4. Commit with a [Conventional Commit](https://www.conventionalcommits.org/) message

No code is merged to `main` without passing tests. No function is shipped without a docstring and inline comments explaining non-obvious decisions.

---

## Architecture

Full system architecture is documented in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

Key documents:

- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — complete technical specification: memory architecture, agent hierarchy, database design, dashboard specifications, and build phases
- [`docs/STANDARDS.md`](docs/STANDARDS.md) — coding standards and engineering principles that govern all code in this repository
- [`docs/decisions/`](docs/decisions/) — Architecture Decision Records explaining every significant technical choice made during development

---

## Build Phases

CRANE is built in eight sequential phases. Each phase produces a fully tested, standalone module before the next phase begins.

| Phase | Module(s) | Purpose |
|---|---|---|
| 1 | `crane.core`, `crane.auth`, infrastructure | Type system, authentication, Docker environment |
| 2 | `crane.memory` | Working, episodic, and procedural memory + GitHub sync |
| 3 | `crane.runtime`, `crane.tracers` | Agent inference pipeline and LLM call logging |
| 4 | `crane.habits`, `crane.connectors` | Action registry, Habit scheduler, Connector base |
| 5 | `crane.neuroplasticity`, `crane.dissonance` | Memory reorganization engine and conflict detection |
| 6 | `crane.tools`, `crane.rag`, `crane.mcp` | Tool registry, RAG pipeline, MCP server base |
| 7 | `crane.studio` + React frontend | CRANE Studio developer dashboard |
| 8 | `crane.portal` + React frontend + widget | CRANE Portal production dashboard |

---

## Deployment

Local development runs via Docker Compose. Production deployments run in Docker containers on Digital Ocean.

Each client organization's agent knowledge base (episodic memory, procedural memory, instruction library) is stored in a dedicated private GitHub repository owned by Acren Digital. The local database acts as a fast-read cache; GitHub is the source of truth.

---

## License

Private and confidential. All rights reserved by Acren Digital.  
This repository is not open source. Do not distribute.
