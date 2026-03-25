# Sisan

> **A Claude Code plugin that orchestrates a multi-agent AI development team through structured, document-driven workflows.**

Sisan gives you a full engineering org inside your IDE — a Product Manager, Software Engineer, QA Engineer, Tech Lead, Software Architect, UI/UX Specialist, DevOps Engineer, and Database Architect — all working together through proven, phase-gated workflows. Every decision is documented. Every change goes through a PR. Nothing gets skipped.

---

## Table of Contents

- [Installation & Usage](#installation--usage)
- [Why Sisan?](#why-sisan)
- [How It Works](#how-it-works)
- [Agent Team](#agent-team)
- [Workflow Tiers](#workflow-tiers)
  - [Patch Workflow](#patch-workflow)
  - [Minor Workflow](#minor-workflow)
  - [Major Workflow](#major-workflow)
- [Branching Strategy](#branching-strategy)
- [Spec Folder & Audit Trail](#spec-folder--audit-trail)
- [Multi-Cycle Delivery](#multi-cycle-delivery)
- [Extensibility](#extensibility)
- [License](#license)

---

## Installation & Usage

### Option 1 — Marketplace (recommended)

```bash
# 1. Add the Sisan marketplace
/plugin marketplace add nicolasbrandao/sisan

# 2. Install the plugin
/plugin install sisan@sisan-plugins
```

### Option 2 — Manual Clone

```bash
git clone https://github.com/nicolasbrandao/sisan ~/.claude/plugins/sisan
```

Then in Claude Code:

```
/reload-plugins
```

> Use `/reload-plugins` anytime you update the plugin to apply changes without restarting your session.

### Plugin Structure

```
sisan/
├── .claude-plugin/
│   └── plugin.json           # Plugin manifest
├── agents/
│   ├── pm.md                 # Product Manager
│   ├── software-engineer.md  # Software Engineer
│   ├── qa-engineer.md        # QA Engineer
│   ├── tech-lead.md          # Tech Lead
│   ├── ui-ux-specialist.md   # UI/UX Specialist
│   ├── software-architect.md # Software Architect
│   ├── devops-engineer.md    # DevOps Engineer
│   └── database-architect.md # Database Architect
├── commands/
│   └── sisan.md              # Orchestrator command
└── workflows/
    ├── patch.md              # Patch tier workflow
    ├── minor.md              # Minor tier workflow
    └── major.md              # Major tier workflow
```

### Running Sisan

```
/sisan Fix the login timeout bug that occurs when session expires
```

Or without arguments to start interactively:

```
/sisan
```

### Example Session

```
User:  /sisan Add password reset via email

PM:    Classifying request...
       → Tier: PATCH (single service, 3–4 files, no infra changes)
       → Suggested title: "Add password reset flow"
       → Delivery: single cycle

       Shall I proceed? (y/n)

User:  y

PM:    Writing specification... [creates specs/001-add-password-reset/01-spec.md]
       ✅ Spec complete. Review at specs/001-add-password-reset/01-spec.md

       Shall I proceed to Discovery? (y/n)

User:  y

SE+QA: Running discovery in parallel...
       ✅ Discovery complete. Review findings before planning.

       Shall I proceed to Planning? (y/n)

...and so on through every phase.
```

### Overriding the Tier

If the PM recommends `patch` but you know it's more complex, just say so at the triage step:

```
PM suggested: patch
User: No, treat this as minor — it touches the auth module and needs a TL review.
```

---

## Why Sisan?

Most AI coding tools give you a single "agent" that writes code and hopes for the best. Sisan is different:

| Problem with single-agent tools | How Sisan solves it |
|----------------------------------|---------------------|
| No upfront spec or acceptance criteria | PM writes a formal spec before any code is touched |
| Code written without understanding the codebase | SE + QA do structured **discovery** first |
| No test strategy — tests come as an afterthought | QA plans + writes tests in parallel with implementation |
| No architectural review | Tech Lead and Software Architect gate planning with formal verdicts |
| Giant, unreviable PRs | Tiered workflows keep PRs small; decomposition splits large requests into multiple cycles |
| No audit trail | Every phase produces markdown documents that form a complete audit trail |
| Pushing directly to main | Enforced branch-per-change with mandatory PRs |

---

## How It Works

```
User Request → PM Triage → Spec → Discovery → Planning → Implementation → Quality Gates → PR
                                               ↑                                          ↑
                                     Tech Lead / SA gate                         QA validates
```

The `/sisan` command acts as the **orchestrator**: it runs agents one at a time (or in parallel), passes outputs between phases, and requires **user approval at every phase boundary**. You stay in control; the agents do the heavy lifting.

### Core Principles

- **No phase skipping** — every phase exists for a reason.
- **User checkpoints** — you approve or reject before every transition.
- **Document-based communication** — agents never talk to each other directly; they communicate through files.
- **Verdicts can block progress** — Tech Lead and Software Architect can reject plans before a single line of production code is written.
- **Full audit trail** — a `specs/` folder is created for every run with all decisions documented.

---

## Agent Team

| Agent | Abbrev. | Core Responsibility | Active Tiers |
|-------|---------|---------------------|--------------|
| **Product Manager** | PM | Triage, specification, acceptance criteria, decomposition | All |
| **Software Engineer** | SE | Codebase discovery, implementation planning, code implementation | All |
| **QA Engineer** | QA | Test infrastructure discovery, test planning, test implementation, quality gating | All |
| **Tech Lead** | TL | Code-level architecture review, planning verdict (can block), quality gate | Minor, Major |
| **UI/UX Specialist** | UI | Design consistency, accessibility, responsive review | Minor*, Major* |
| **Software Architect** | SA | System topology, API contracts, ADRs, system-level planning verdict (can block) | Major |
| **DevOps Engineer** | DevOps | Infra planning, CI/CD, deployment, monitoring implementation | Major* |
| **Database Architect** | DBA | Schema design, migration safety, query performance, ORM conventions | Minor*, Major* |

*Conditionally activated based on spec content (UI changes, infrastructure changes, or DB schema changes).

### Agent Modes

Each agent operates across multiple **modes** depending on the phase:

| Agent | Modes |
|-------|-------|
| PM | Triage · Specification |
| SE | Discovery · Planning · Implementation · Quality Gate |
| QA | Discovery · Planning · Implementation · Quality Gate |
| TL | Discovery Review · Planning Review (w/ Verdict) · Quality Gate |
| UI/UX | Spec Review · Discovery · Plan Review · Quality Gate |
| SA | Spec Review · Discovery · Planning + Verdict · Quality Gate |
| DevOps | Spec Review · Discovery · Planning · Implementation · Quality Gate |
| DBA | Spec Review · Discovery · Planning · Implementation · Quality Gate |

---

## Workflow Tiers

Sisan routes every request to one of three tiers. The PM recommends the tier; you confirm it.

| Tier | Scope | PRs | Files Changed | Services | Agents | Branching |
|------|-------|-----|---------------|----------|--------|-----------|
| **Patch** | Bug fixes, hotfixes, targeted changes | 1 | 1–5 | 1 | PM, SE, QA | Work branch → PR to main |
| **Minor** | Features, refactors, moderate changes | 1–3 | 5–15 | 1 | PM, SE, QA, TL, UI/UX*, DBA* | Feature branch + sub-PRs |
| **Major** | Cross-service features, architecture, migrations | 4+ | 15+ | 1+ | PM, SE, QA, TL, SA, UI/UX*, DevOps*, DBA* | Epic branch + sub-PRs |

---

### Patch Workflow

The simplest tier. Three agents, one PR, full documentation.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Phase 0 │ Intake & Triage      PM classifies the request               │
│  Phase 1 │ Specification        PM writes spec with acceptance criteria  │
│  Phase 2 │ Discovery            SE + QA explore codebase (parallel)     │
│           │                       → cross-review each other's findings  │
│  Phase 3 │ Planning             SE + QA create plans (parallel)         │
│           │                       → cross-review each other's plans     │
│  Phase 4 │ Implementation       SE writes code · QA writes tests        │
│  Phase 5 │ Quality Gates        QA validates against spec               │
│           │                       → SE reviews test coverage            │
│  Phase 6 │ Merge                PR created with full audit trail        │
└─────────────────────────────────────────────────────────────────────────┘
                        ↕ User approval required between every phase
```

**Produces:** 9 documents in `specs/{id}-{slug}/`

---

### Minor Workflow

Adds a Tech Lead with a **blocking planning verdict** and optional UI/UX + DBA participation.

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Phase 0   │ Intake & Triage    PM classifies                           │
│  Phase 1   │ Specification      PM writes spec                          │
│  Phase 1.5 │ UI/UX Spec Review  UI/UX augments spec [conditional]       │
│  Phase 2   │ Discovery          SE + QA + UI/UX explore (parallel)      │
│             │                     → cross-review → TL reviews all       │
│  Phase 3   │ Planning           SE + QA plan (parallel)                 │
│             │                     → cross-review → UI/UX reviews SE     │
│             │                     → TL renders VERDICT ◀ can block      │
│  Phase 4   │ Implementation     SE code + QA tests, per sub-PR          │
│  Phase 5   │ Quality Gates      QA + SE + TL + UI/UX [conditional]      │
│  Phase 6   │ Merge              Sub-PRs → feature branch → main         │
└─────────────────────────────────────────────────────────────────────────┘
                        ↕ User approval required between every phase
```

> **Tech Lead Verdict options:** `APPROVED` / `APPROVED WITH CHANGES` / `REJECTED`
> A `REJECTED` verdict sends the plan back to the SE for revision before any code is written.

**Produces:** Up to 14 documents in `specs/{id}-{slug}/`

---

### Major Workflow

The full team. **Dual blocking verdicts** from both the Software Architect (system level) and Tech Lead (code level).

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Phase 0   │ Intake & Triage      PM classifies                         │
│  Phase 1   │ Specification        PM writes spec (UI + Infra sections)  │
│  Phase 1.5 │ UI/UX Spec Review    UI/UX augments spec [conditional]     │
│  Phase 1.7 │ Architecture Review  SA maps topology + API contracts       │
│  Phase 1.9 │ Infra Spec Review    DevOps identifies requirements [cond.] │
│  Phase 2   │ Discovery            SE + QA + SA + UI/UX + DevOps (par.)  │
│             │                       → cross-review → TL reviews all     │
│  Phase 3   │ Planning             SE + QA + SA + DevOps plan (parallel) │
│             │                       → cross-review                      │
│             │                       → SA renders SYSTEM VERDICT ◀ block │
│             │                       → TL renders CODE VERDICT   ◀ block │
│  Phase 4   │ Implementation       SE + DevOps code + QA tests, phased  │
│  Phase 5   │ Quality Gates        QA + SE + TL + SA + UI/UX + DevOps   │
│  Phase 6   │ Merge                Sub-PRs → epic branch → main          │
└─────────────────────────────────────────────────────────────────────────┘
                        ↕ User approval required between every phase
```

> **Dual blocking verdicts:** The most restrictive verdict wins. If SA says `REJECTED` and TL says `APPROVED`, implementation is blocked.

**Produces:** Up to 21 documents in `specs/{id}-{slug}/`

---

## Branching Strategy

Sisan **never pushes directly to `main`**. Every change lives on its own branch and goes through a PR.

### Patch — Single Branch
```
main
 └── patch/001-fix-login-timeout   ── PR ──▶ main
```

### Minor — Feature Branch with Sub-PRs
```
main
 └── minor/002-add-retry-logic          (feature branch)
      ├── minor/002-add-retry-logic/pr-1-retry-util   ── PR ──▶ feature branch
      └── minor/002-add-retry-logic/pr-2-apply-to-api ── PR ──▶ feature branch
                                         └── final PR: feature branch ──▶ main
```

### Major — Epic Branch with Sub-PRs
```
main
 └── epic/003-new-auth-system            (epic branch)
      ├── epic/003-new-auth-system/pr-1-db-schema     ── PR ──▶ epic branch
      ├── epic/003-new-auth-system/pr-2-auth-service  ── PR ──▶ epic branch
      └── epic/003-new-auth-system/pr-3-api-endpoints ── PR ──▶ epic branch
                                         └── final PR: epic branch ──▶ main
```

### Multi-Cycle Decomposition
When a request is decomposed into multiple patch cycles targeting a parent feature branch:
```
main
 └── minor/002-add-retry-logic           (feature branch, created once)
      ├── patch/004-retry-util           ── PR ──▶ feature branch
      └── patch/005-apply-retry-to-api  ── PR ──▶ feature branch
                                         └── final PR: feature branch ──▶ main
```

---

## Spec Folder & Audit Trail

Every run creates a numbered folder under `./specs/` with a complete audit trail:

### Patch (9 documents)

```
specs/001-fix-login-timeout/
├── 01-spec.md                   # PM specification + acceptance criteria
├── 02-discovery-se.md           # SE codebase discovery + cross-review notes
├── 02-discovery-qa.md           # QA test infrastructure discovery + cross-review
├── 03-plan-se.md                # SE implementation plan + cross-review notes
├── 03-plan-qa.md                # QA test plan + cross-review notes
├── 04-implementation-summary.md # SE implementation summary
├── 04-test-report.md            # QA test report
├── 05-quality-gate.md           # Quality gate (QA + SE sections)
└── 06-pr-summary.md             # PR description
```

### Minor (up to 14 documents)

```
specs/002-add-user-profile/
├── 01-spec.md                   # PM specification
├── 01-spec-ui-review.md         # UI/UX spec review [conditional]
├── 02-discovery-se.md           # SE discovery + cross-review
├── 02-discovery-qa.md           # QA discovery + cross-review
├── 02-discovery-ui.md           # UI/UX discovery [conditional]
├── 02-discovery-tl.md           # Tech Lead discovery review
├── 03-plan-se.md                # SE plan + cross-review
├── 03-plan-qa.md                # QA plan + cross-review
├── 03-plan-ui-review.md         # UI/UX plan review [conditional]
├── 03-plan-tl.md                # Tech Lead plan review + VERDICT
├── 04-implementation-summary.md # SE implementation summary
├── 04-test-report.md            # QA test report
├── 05-quality-gate.md           # Quality gate (QA + SE + TL + UI/UX)
└── 06-pr-summary.md             # PR description
```

### Major (up to 21 documents)

```
specs/003-new-auth-system/
├── 01-spec.md                   # PM specification (UI + Infra + Cross-Service sections)
├── 01-spec-ui-review.md         # UI/UX spec review [conditional]
├── 01-spec-arch-review.md       # SA architecture spec review (always)
├── 01-spec-infra-review.md      # DevOps infra spec review [conditional]
├── 02-discovery-se.md           # SE discovery + cross-review
├── 02-discovery-qa.md           # QA discovery + cross-review
├── 02-discovery-ui.md           # UI/UX discovery [conditional]
├── 02-discovery-sa.md           # SA discovery + cross-review
├── 02-discovery-devops.md       # DevOps discovery [conditional]
├── 02-discovery-tl.md           # TL discovery review
├── 03-plan-se.md                # SE plan + cross-review
├── 03-plan-qa.md                # QA plan + cross-review
├── 03-plan-sa.md                # SA plan + API contracts + ADRs + SYSTEM VERDICT
├── 03-plan-devops.md            # DevOps infra plan [conditional]
├── 03-plan-ui-review.md         # UI/UX plan review [conditional]
├── 03-plan-tl.md                # TL plan review + CODE VERDICT
├── 04-implementation-summary.md # SE implementation summary
├── 04-test-report.md            # QA test report
├── 04-infra-summary.md          # DevOps infra summary [conditional]
├── 05-quality-gate.md           # Quality gate (QA + SE + TL + SA + UI/UX + DevOps)
└── 06-pr-summary.md             # PR description + ADRs
```

---

## Multi-Cycle Delivery

Large requests don't have to be one giant PR. The PM can recommend breaking the work into multiple smaller cycles, each producing its own branch, spec folder, and PR.

**Example:**

```
User request: "Add retry logic with exponential backoff to all API calls"

PM Triage → Recommends 3 patch cycles:
  ├── Cycle 1: Add retry utility function      (patch/004-retry-util)
  ├── Cycle 2: Apply retry to auth API calls   (patch/005-retry-auth)
  └── Cycle 3: Apply retry to data API calls   (patch/006-retry-data)

Each cycle → full workflow → own spec folder → own PR → reviewed independently
```

**Why this matters:**

| Approach | PR size | Reviewability | Risk per merge |
|----------|---------|---------------|----------------|
| Single large PR | 500–2000 lines | Very hard | High |
| Multi-cycle decomposition | < 300 lines | Easy | Low |

Run `/sisan` once per cycle. Each cycle picks up where the branching strategy left off.

---

---

## Extensibility

Sisan is designed to be modular. Each component is self-contained.

### Adding a New Agent

1. Create `agents/{agent-name}.md` with a YAML frontmatter header and mode definitions.
2. Reference the agent in the appropriate `workflows/{tier}.md` file.
3. Existing agents require no modification — the orchestrator prompt controls what documents each agent reads.

### Adding a New Workflow Tier

1. Create `workflows/{tier}.md` with the phase definitions.
2. Register the tier in `commands/sisan.md`.
3. The PM triage and orchestrator dispatch will automatically route to it.

### Customizing an Existing Workflow

Each workflow file (`workflows/{tier}.md`) is self-contained. You can:

- Add or remove phases
- Change which agents participate
- Adjust cross-review rounds
- Modify the verdict mechanism

Changes to one tier do not affect others.

---

## License

MIT

---

<div align="center">

Built for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) · by [Nicolas Brandao](https://github.com/nicolasbrandao)

</div>
