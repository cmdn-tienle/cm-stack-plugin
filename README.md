# CM Stack Plugin Marketplace

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin marketplace hosting **CM Stack Workflows** — a structured multi-agent development team that orchestrates specialized AI agents through a complete SDLC workflow.

> **One plugin, four agents, eight skills — a full development team in your terminal.**

---

## Table of Contents

- [Installation](#installation)
- [The Workflow Pipeline](#the-workflow-pipeline)
- [Agents](#agents)
- [Skills](#skills)
- [Usage](#usage)
- [Plugin Structure](#plugin-structure)
- [How It Works](#how-it-works)
- [Configuration](#configuration)
- [Contributing](#contributing)
- [License](#license)

---

## Installation

### Option 1: Install from Marketplace

Add the marketplace and install the plugin:

```bash
/plugin marketplace add cmdn-labs/cm-stack-plugin
/plugin install cm-stack-workflows@cm-stack-plugin
```

### Option 2: Per-Session (CLI Flag)

Load the plugin for a single Claude Code session:

```bash
claude --plugin-dir /path/to/cm-stack-plugin/plugins/cm-stack-workflows
```

### Option 3: Project-Level (Persistent)

Add to your project's `.claude/plugins.json` to auto-load the plugin for all sessions in that project:

```json
{
  "plugins": [
    {
      "path": "/path/to/cm-stack-plugin/plugins/cm-stack-workflows"
    }
  ]
}
```

> **Tip:** Use an absolute path, or a path relative to your project root.

---

## The Workflow Pipeline

CM Stack provides multiple entry points depending on how well-defined your requirements are:

```
                              Vague idea? ──▶ BRAINSTORM ──▶ lead_XXX.md ──┐
                                                                           │
 ┌──────────┐     ┌───────────┐     ┌──────────┐     ┌──────────┐     ┌────▼────┐
 │ ANALYZE  │────▶│  DESIGN   │────▶│   PLAN   │────▶│   TASK   │────▶│  COMMIT │
 │          │     │           │     │          │     │          │     │         │
 │ PRD      │     │ System    │     │ Impl.    │     │ Code +   │     │ Clean   │
 │ Document │     │ Design    │     │ Plan     │     │ Review   │     │ Commits │
 └──────────┘     └───────────┘     └──────────┘     └──────────┘     └─────────┘
       ▲
       │       ┌─────────────────────────────────────────────────────────────┐
       └───────│ Well-defined, small scope? ──▶ FAST-TRACK ──▶ Code (skip)   │
               └─────────────────────────────────────────────────────────────┘
```

### Full Workflow (Documentation-Heavy)

| Phase | Skill | Input | Output | Agents Involved |
|-------|-------|-------|--------|-----------------|
| **0. Brainstorm** | `brainstorm` | Vague idea / rough concept | `docs/leads/lead_XXX.md` | Business Analyst + Tech Lead |
| **1. Analyze** | `analyze` | Feature request / lead document | `docs/requirements/prd_XXX.md` | Business Analyst + Tech Lead |
| **2. Design** | `design` | PRD document | `docs/designs/design_XXX.md` | 4× Tech Lead (parallel) |
| **3. Plan** | `plan` | Design document | `docs/plans/plan_XXX.md` | Tech Lead + Business Analyst |
| **4. Task** | `task` | Plan document | Implemented code (reviewed) | Senior Engineer + 3 reviewers |
| **5. Commit** | `git-commit` | Staged changes | Clean git commits | — (utility skill) |
| **Review** | `review-branch` | Branch diff | Structured review report | — (utility skill) |

### Fast-Track (Skip Documentation)

| Phase | Skill | Input | Output | Agents Involved |
|-------|-------|-------|--------|-----------------|
| **Fast-Track** | `fast-track` | Lead doc or well-defined prompt | Implemented code (light review) | Senior Engineer + QA |

---

## Agents

### Business Analyst

| | |
|---|---|
| **File** | `agents/business-analyst.md` |
| **Model** | Opus |
| **Specialty** | Requirements engineering, PRDs, stakeholder analysis |

A Senior Business Analyst with 15+ years of experience. Transforms vague user needs into structured, actionable Product Requirement Documents.

**Capabilities:**
- Requirements elicitation using 5 Whys, MoSCoW prioritization, user story mapping
- Gap identification and clarifying question generation
- Acceptance criteria definition with testable conditions
- Stakeholder mapping and constraint discovery
- PRD authoring with full document structure

---

### Technical Lead / Architect

| | |
|---|---|
| **File** | `agents/technical-lead-architect.md` |
| **Model** | Opus |
| **Specialty** | System design, architecture decisions, design documentation |

An elite Technical Leader with 15+ years designing enterprise-grade systems. Provides high-level guidance on architecture, component design, and technical trade-offs.

**Capabilities:**
- System architecture analysis and design
- Component boundary definition and communication patterns
- API design with versioning, authentication, and error handling
- Resilience patterns (circuit breakers, retries, fallbacks)
- Design documentation with ADR (Architecture Decision Record) format
- Security, scalability, and observability assessment

---

### Senior Engineer

| | |
|---|---|
| **File** | `agents/senior-engineer.md` |
| **Model** | Opus |
| **Specialty** | Implementation, database design, PRD-to-code translation |

A Senior Software Engineer who translates complex requirements into production-ready code. Adapts to the project's language, framework, and conventions.

**Capabilities:**
- Clean, idiomatic code following project conventions
- Database schema design with proper indexing and constraints
- Query optimization and N+1 prevention
- PRD analysis and implementation step breakdown
- Framework-native patterns (ORM, DI, jobs, events, validation)
- Comprehensive test writing (happy paths, failures, edge cases)

---

### QA Engineer

| | |
|---|---|
| **File** | `agents/qa-engineer.md` |
| **Model** | Opus |
| **Specialty** | Code review, testing strategy, defect detection, requirement verification |

A Senior QA Engineer and the last line of defense before code reaches production. Performs thorough code reviews, designs test strategies, and validates implementations against requirements.

**Capabilities:**
- Line-by-line code review (correctness, security, maintainability, performance)
- Testing strategy design using the test pyramid (unit → integration → E2E)
- Security vulnerability detection (SQL injection, XSS, CSRF, mass assignment)
- Requirements traceability — mapping tests to acceptance criteria
- Structured QA reports with severity-ranked findings (Critical / Major / Minor)
- Bug investigation and reproduction

---

## Skills

### `brainstorm` — Discovery & Clarification

> **Command:** Triggered when user has a vague idea, rough concept, or unstructured lead

Transforms vague ideas into clear, actionable requirements through collaborative analysis and targeted questioning. Spawns `business-analyst` and `technical-lead-architect` to examine the lead from multiple perspectives.

```
               ┌── business-analyst ──────────┐
Vague idea ───▶│   (business questions)       │──▶ Q&A ──▶ lead_XXX.md
               └───technical-lead-architect───┘
                         (technical questions)
```

**When to use:**
- User says "I'm thinking about..." or "We might need..."
- Requirements are incomplete or ambiguous
- Starting discovery for a new initiative

**Output:** `docs/leads/lead_XXX.md`

---

### `analyze` — Requirements Analysis

> **Command:** Triggered when user provides feature requirements or asks to create a PRD

Orchestrates parallel analysis by spawning `business-analyst` and `technical-lead-architect` simultaneously. Both agents analyze the requirements from their respective perspectives, and their outputs are synthesized into a unified PRD.

```
                     ┌── business-analyst ─────────┐
User requirement ──▶ │   (business perspective)    │──▶ Synthesize ──▶ prd_XXX.md
                     └── technical-lead-architect ─┘
                         (technical perspective)
```

**Output:** `docs/requirements/prd_XXX.md`

---

### `design` — System Design

> **Command:** Triggered when user provides a PRD and asks to create a system design

Spawns **four** `technical-lead-architect` agents in parallel, each researching a specific architectural concern:

| Agent | Focus Area |
|-------|-----------|
| Architect 1 | Architecture & Components — system structure, tech stack, design patterns |
| Architect 2 | External Services & APIs — integrations, endpoints, rate limiting |
| Architect 3 | Data Model & Migration — schema changes, relationships, rollback plans |
| Architect 4 | Security & Performance — auth, encryption, caching, scalability |

All findings are synthesized into a single comprehensive design document.

**Output:** `docs/designs/design_XXX.md`

---

### `plan` — Implementation Planning

> **Command:** Triggered when user provides a design document and asks to plan implementation

Analyzes the design to produce a detailed implementation plan with:

- **Task breakdown** — Granular, 2–4 hour tasks with acceptance criteria
- **Dependency graph** — Visual representation of task relationships
- **Conflict analysis** — File conflicts, database conflicts, interface conflicts
- **Parallelization opportunities** — Which tasks can run simultaneously
- **Risk register** — Identified risks with likelihood, impact, and mitigation
- **Rollback strategy** — Per-phase rollback procedures

Spawns `technical-lead-architect` (task breakdown & deps) and `business-analyst` (acceptance criteria mapping) in parallel.

**Output:** `docs/plans/plan_XXX.md`

---

### `task` — Task Execution

> **Command:** Triggered when user provides a plan file and asks to execute tasks

The core execution engine. Reads the plan, identifies available tasks, and spawns `senior-engineer` agents to implement them — parallelizing independent tasks.

After implementation, a **triple review gate** validates the work:

```
                                          ┌─ business-analyst ──────────┐
                                          │  (business logic check)     │
senior-engineer(s) ──▶ Implementation ──▶ ├─ technical-lead-architect ──┤──▶ Pass/Fail
                                          │  (architecture review)      │
                                          └─ qa-engineer ───────────────┘
                                             (code review & testing)
```

Tasks are only marked `✅ DONE` when all three reviewers pass. Failed reviews trigger fixes and re-review.

**Task Status Indicators:**

| Indicator | Meaning |
|-----------|---------|
| *(none)* | Pending — not started |
| ⏳ | In progress |
| ✅ DONE | Completed and reviewed |
| 🚫 | Blocked by dependency |

---

### `fast-track` — Skip Documentation, Ship Fast

> **Command:** Triggered when user has well-defined requirements and wants to skip documentation

Bypasses the full workflow for small, clear-scope work. Spawns `senior-engineer` to implement directly, followed by lightweight `qa-engineer` review. No PRD, design, or plan documents created.

```
Well-defined ──▶ senior-engineer ──▶ Implementation ──▶ qa-engineer ──▶ Done
    prompt           (implement)                        (light review)
```

**When to use:**
- User provides a lead document and says "just implement this"
- User says "skip the paperwork" or "fast-track this"
- Scope is small (1-3 files, clear boundaries)

**When NOT to use:**
- Requirements are vague (use `brainstorm` first)
- Scope is large or complex (use full workflow)
- Multiple components/systems involved

**Output:** Implemented code + optional summary at `docs/implementation/impl_XXX.md`

---

### `review-branch` — Branch Code Review

> **Command:** Triggered when user asks to review a branch, code changes, or PR

Performs a comprehensive code review by comparing the current branch against a target branch. Covers security, correctness, performance, error handling, maintainability, testing, and cross-file impact.

```
Target branch ──▶ git diff ──▶ Systematic review ──▶ Structured report
(main/develop)     (gather)     (8 categories)       (docs/code-review/)
```

**Review categories:**
- Correctness & Logic — bugs, edge cases, race conditions
- Security — injection, auth, data exposure
- Error Handling & Reliability — cleanup, retry, swallowed exceptions
- Performance — N+1 queries, memory, unnecessary sync ops
- API Design & Contracts — conventions, breaking changes, backward-compat
- Data Integrity — transactions, migrations, rollback safety
- Maintainability & Readability — abstraction level, naming, DRY
- Testing — coverage, edge cases, deterministic tests

**Severity levels:** CRITICAL / HIGH / MEDIUM / LOW / INFO

**Output:** `docs/code-review/<timestamp>_<source>_<target>.md`

---

### `git-commit` — Clean Git Commits

> **Command:** Triggered when user asks to commit changes

A utility skill for creating clean, professional git commits:

- **No AI signatures** — Never adds "Co-Authored-By" lines
- **Specific file staging** — Uses `git add <file>`, never `git add .`
- **Concise messages** — Focuses on _what_ changed and _why_
- **Safety checks** — Reviews for secrets or unexpected changes before committing

---

## Usage

### Full Workflow Example (Complex Feature)

```
You:    "I need a subscription billing system for my SaaS app"
        ↓ (analyze runs → spawns business-analyst + tech lead)

Output: docs/requirements/prd_001.md

You:    "Create a system design from docs/requirements/prd_001.md"
        ↓ (design runs → spawns 4 tech lead agents in parallel)

Output: docs/designs/design_001.md

You:    "Create an implementation plan from docs/designs/design_001.md"
        ↓ (plan runs → spawns tech lead + business analyst)

Output: docs/plans/plan_001.md

You:    "Execute tasks from docs/plans/plan_001.md"
        ↓ (task runs → spawns senior engineers → triple review)

Output: Implementation + updated plan with ✅ DONE tasks

You:    "Commit the changes"
        ↓ (git-commit runs)

Output: Clean git commit(s)
```

### Discovery Workflow Example (Vague Idea)

```
You:    "I'm thinking about adding SSO to our app"
        ↓ (brainstorm runs → spawns business-analyst + tech lead)

Output: Clarifying questions about providers, users, flows...

You:    [Answers questions]
        ↓ (synthesizes responses)

Output: docs/leads/lead_001.md with clear requirements

You:    "Now analyze and create a PRD"
        ↓ (continues to full workflow)
```

### Fast-Track Example (Skip Documentation)

```
You:    "Add a rate limiter to the API, max 100 req/min per IP"
        ↓ (fast-track runs → spawns senior-engineer)

Output: Rate limiting middleware implemented

        ↓ (qa-engineer reviews)

Output: QA review passed, done!

Total: 2 agents, no documentation, fast delivery
```

### Individual Skill Usage

You can use any skill independently:

```
# Brainstorm a vague idea
"I'm considering adding real-time notifications"

# Just analyze requirements
"Analyze the requirements for a user authentication system"

# Just design from an existing PRD
"Design the system from docs/requirements/prd_003.md"

# Just execute tasks from a plan
"Work on the tasks in docs/plans/plan_002.md"

# Fast-track a small, clear task
"Add input validation to the contact form"

# Just commit
"Commit my changes"

# Review a branch
"Review my branch against main"
```

### Choosing the Right Workflow

| Your Situation | Use | Why |
|----------------|-----|-----|
| "I have a vague idea" | `brainstorm` | Clarify before committing |
| "I know what I want, but it's complex" | `analyze` → full workflow | Complex needs thorough planning |
| "Just implement this small thing" | `fast-track` | Skip docs, ship fast |
| "I have a PRD, now what?" | `design` | Continue the pipeline |
| "I have a plan, execute it" | `task` | Start implementation |
| "Review my branch/changes" | `review-branch` | Catch issues before merge |

### Checking Progress

```
"Check progress on docs/plans/plan_001.md"
```

Returns a summary of completed, in-progress, pending, and blocked tasks with percentage completion.

---

## Plugin Structure

This repository is a **plugin marketplace** that hosts the `cm-stack-workflows` plugin in a monorepo layout:

```
cm-stack-plugin/                            # Marketplace repository
├── .claude-plugin/
│   └── marketplace.json                    # Marketplace catalog
├── plugins/
│   └── cm-stack-workflows/                 # The plugin
│       ├── .claude-plugin/
│       │   └── plugin.json                 # Plugin manifest
│       ├── agents/
│       │   ├── business-analyst.md         # Requirements & PRD specialist
│       │   ├── qa-engineer.md              # Code review & testing specialist
│       │   ├── senior-engineer.md          # Implementation specialist
│       │   └── technical-lead-architect.md # Architecture & design specialist
│       └── skills/
│           ├── brainstorm/
│           │   └── SKILL.md                # brainstorm: Vague idea → Lead document
│           ├── analyze/
│           │   └── SKILL.md                # analyze: Requirements → PRD
│           ├── design/
│           │   └── SKILL.md                # design: PRD → System Design
│           ├── plan/
│           │   └── SKILL.md                # plan: Design → Implementation Plan
│           ├── task/
│           │   └── SKILL.md                # task: Plan → Code (with review gate)
│           ├── fast-track/
│           │   └── SKILL.md                # fast-track: Lead/prompt → Code (skip docs)
│           ├── review-branch/
│           │   ├── SKILL.md                # review-branch: Branch diff → Structured review
│           │   ├── evals/
│           │   │   └── evals.json          # Evaluation configurations
│           │   └── references/
│           │       ├── performance-checklist.md  # Performance review criteria
│           │       ├── review-template.md        # Report template
│           │       └── security-checklist.md     # Security review criteria
│           └── git-commit/
│               └── SKILL.md                # git-commit: Clean commit workflow
├── LICENSE                                 # MIT License
└── README.md
```

### Generated Artifacts (in your project)

When you use the workflow skills, they produce documents in your project:

```
your-project/
└── docs/
    ├── leads/
    │   ├── lead_260401_sso.md         # Clarified requirements from brainstorming
    │   └── lead_260415_reporting.md
    ├── requirements/
    │   ├── prd_001.md                 # Product Requirement Document
    │   └── prd_002.md
    ├── designs/
    │   ├── design_001.md              # System Design Document
    │   └── design_002.md
    ├── plans/
    │   ├── plan_001.md                # Implementation Plan
    │   └── plan_002.md
    └── implementation/
        └── impl_001.md                # Optional fast-track summary
    └── code-review/
        └── 20260407-143025_feature-login_main.md  # Branch review report
```

---

## How It Works

### Agent Orchestration

The plugin uses Claude Code's **Agent tool** to spawn specialized subagents. Each agent definition includes:

- **Frontmatter** — Name, description with usage examples, model selection, and color coding
- **System prompt** — Identity, core competencies, behavioral guidelines, and output formats

Agents are spawned on-demand by skills and can run in parallel using `run_in_background: true`.

### Multi-Perspective Review

The `task` skill implements a **review gate** that requires three independent reviews before work is marked complete:

1. **Business Analyst** — Validates acceptance criteria and business logic correctness
2. **Technical Lead** — Reviews architectural alignment, code quality, and security
3. **QA Engineer** — Performs code review, identifies defects, verifies test coverage

If any reviewer flags issues, the implementation is revised and re-reviewed.

### Agent Memory

All agents are designed to **build institutional knowledge** across conversations. They record:

- Architectural patterns and conventions discovered in the codebase
- Database schemas and relationships
- Integration points and API contracts
- Testing patterns and factory configurations
- Common bug patterns and performance benchmarks

---

## Configuration

### Plugin Manifest

The plugin is defined in `plugins/cm-stack-workflows/.claude-plugin/plugin.json`:

```json
{
  "name": "cm-stack-workflows",
  "version": "1.0.0",
  "description": "A structured development team plugin for Claude Code - 4 specialized agents and 8 workflow skills for the full SDLC.",
  "author": {
    "name": "Tien Le H."
  },
  "license": "MIT"
}
```

### Agent Model Selection

All agents currently use the `opus` model for highest quality reasoning. You can modify the `model` field in each agent's frontmatter to change this:

```yaml
---
name: senior-engineer
model: opus       # Change to sonnet for faster/cheaper execution
color: yellow
---
```

---

## Contributing

1. **Agents** are defined in `plugins/cm-stack-workflows/agents/<name>.md` with YAML frontmatter + markdown system prompts
2. **Skills** are defined in `plugins/cm-stack-workflows/skills/<name>/SKILL.md` with workflow instructions
3. Follow the existing patterns for consistency — each skill should document:
   - When to use / when not to use
   - Workflow diagram (Mermaid)
   - Step-by-step implementation
   - Common mistakes
   - File output conventions

---

## License

[MIT](LICENSE)
