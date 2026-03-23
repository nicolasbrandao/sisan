# Sisan

A Claude Code plugin that orchestrates a multi-agent development team with structured workflows, document-based collaboration, and quality gates.

## Overview

Sisan provides a team of up to 7 specialized AI agents that work together through structured workflows to handle development tasks. The team scales per workflow tier: patch uses 3 agents, minor uses up to 5, major uses up to 7. Each workflow produces a complete audit trail of markdown documents.

**Current workflow tiers:**

| Tier | PRs | Files | Services | Branching | Agents | Status |
|------|-----|-------|----------|-----------|--------|--------|
| **Patch** | 1 | 1-5 | 1 | Work branch → PR to main | PM, SE, QA | Available |
| **Minor** | 1-3 | 5-15 | 1 | Feature branch + sub-PRs | PM, SE, QA, TL, UI/UX* | Available |
| **Major** | 4+ | 15+ | 1+ | Epic branch + sub-PRs | PM, SE, QA, TL, SA, UI/UX*, DevOps* | Available |

*UI/UX Specialist and DevOps Engineer are conditionally activated based on spec content.

The PM agent triages requests and can recommend **decomposing** larger requests into multiple smaller cycles (e.g., a "minor" request delivered as 3 patch cycles), keeping PRs small and easy to review.

## Installation

Add the sisan plugin to your Claude Code configuration. The plugin directory should contain:

```
sisan/
├── .claude-plugin/plugin.json
├── agents/
│   ├── pm.md
│   ├── software-engineer.md
│   ├── qa-engineer.md
│   ├── tech-lead.md
│   ├── ui-ux-specialist.md
│   ├── software-architect.md
│   └── devops-engineer.md
├── commands/
│   └── sisan.md
└── workflows/
    ├── patch.md
    ├── minor.md
    └── major.md
```

## Usage

```
/sisan Fix the login timeout bug that occurs when session expires
```

Or simply:

```
/sisan
```

The command will guide you through the entire process interactively.

## How It Works

### Agents

| Agent | Role | Modes | Tiers |
|-------|------|-------|-------|
| **PM** | Triages requests, writes specifications | Triage, Specification | All |
| **Software Engineer** | Explores code, plans changes, implements | Discovery, Planning, Implementation, Quality Gate | All |
| **QA Engineer** | Explores tests, plans tests, implements tests, validates | Discovery, Planning, Implementation, Quality Gate | All |
| **Tech Lead** | Reviews code-level architecture, validates approaches, gates implementation | Discovery Review, Planning Review, Quality Gate | Minor, Major |
| **UI/UX Specialist** | Validates design consistency, accessibility, responsive behavior | Spec Review, Discovery, Quality Gate | Minor*, Major* |
| **Software Architect** | Reviews system-level architecture, defines API contracts, produces ADRs | Spec Review, Discovery, Planning + Verdict, Quality Gate | Major |
| **DevOps Engineer** | Plans and implements infrastructure, CI/CD, deployment, monitoring | Spec Review, Discovery, Planning, Implementation, Quality Gate | Major* |

*UI/UX Specialist activated when spec has UI changes. DevOps Engineer activated when spec has infrastructure changes.

### Patch Workflow Phases

```
Phase 0: Intake & Triage     PM classifies the request
Phase 1: Specification        PM writes detailed spec with acceptance criteria
Phase 2: Discovery            SE + QA explore codebase in parallel, then cross-review
Phase 3: Planning             SE + QA create plans in parallel, then cross-review
Phase 4: Implementation       SE writes code, QA writes tests
Phase 5: Quality Gates        QA validates against spec, SE reviews tests
Phase 6: Merge                Create PR with full audit trail
```

User approval is required between every phase.

### Minor Workflow Phases

```
Phase 0:   Intake & Triage     PM classifies the request
Phase 1:   Specification        PM writes detailed spec with acceptance criteria
Phase 1.5: UI/UX Spec Review   UI/UX augments spec with UI criteria (conditional)
Phase 2:   Discovery            SE + QA + UI/UX (conditional) explore in parallel,
                                  then cross-review, then TL reviews everything
Phase 3:   Planning             SE + QA plan in parallel, cross-review, UI/UX reviews
                                  SE plan (conditional), then TL renders verdict
Phase 4:   Implementation       SE writes code + QA writes tests, per sub-PR
Phase 5:   Quality Gates        QA validates, SE reviews tests, TL reviews architecture,
                                  UI/UX reviews design (conditional)
Phase 6:   Merge                Create sub-PRs → feature branch, final PR → main
```

The Tech Lead's **planning verdict can block implementation** (APPROVED / APPROVED WITH CHANGES / REJECTED). User approval is required between every phase.

### Major Workflow Phases

```
Phase 0:   Intake & Triage       PM classifies the request
Phase 1:   Specification          PM writes spec (with UI, Infra, Cross-Service sections)
Phase 1.5: UI/UX Spec Review     UI/UX augments spec with UI criteria (conditional)
Phase 1.7: Architecture Review   SA maps system topology, defines API contracts (always)
Phase 1.9: Infra Spec Review     DevOps identifies infra requirements (conditional)
Phase 2:   Discovery              SE + QA + SA + UI/UX + DevOps explore in parallel,
                                    cross-review, then TL reviews everything
Phase 3:   Planning               SE + QA + SA + DevOps plan in parallel, cross-review,
                                    then SA renders system verdict + TL renders code verdict
Phase 4:   Implementation         SE writes app code, DevOps writes infra code,
                                    QA writes tests — per sub-PR, phased
Phase 5:   Quality Gates          QA validates, SE reviews tests, TL reviews code arch,
                                    SA reviews system integration, UI/UX reviews design,
                                    DevOps reviews deployment readiness
Phase 6:   Merge                  Sub-PRs → epic branch, final PR → main
```

**Dual blocking verdicts**: Both the Software Architect (system-level) and Tech Lead (code-level) can independently block implementation. The most restrictive verdict wins. User approval is required between every phase.

### Spec Folder Structure

Each workflow run creates a folder in `./specs/` with auto-incrementing IDs:

**Patch workflow** (9 documents):
```
specs/001-fix-login-timeout/
├── 01-spec.md                      # PM specification
├── 02-discovery-se.md              # SE codebase discovery + cross-review notes
├── 02-discovery-qa.md              # QA test infrastructure discovery + cross-review notes
├── 03-plan-se.md                   # SE implementation plan + cross-review notes
├── 03-plan-qa.md                   # QA test plan + cross-review notes
├── 04-implementation-summary.md    # SE implementation summary
├── 04-test-report.md               # QA test report
├── 05-quality-gate.md              # Quality gate (QA + SE sections)
└── 06-pr-summary.md                # PR description
```

**Minor workflow** (up to 14 documents):
```
specs/002-add-user-profile/
├── 01-spec.md                      # PM specification
├── 01-spec-ui-review.md            # UI/UX spec review (conditional)
├── 02-discovery-se.md              # SE discovery + cross-review notes
├── 02-discovery-qa.md              # QA discovery + cross-review notes
├── 02-discovery-ui.md              # UI/UX discovery (conditional)
├── 02-discovery-tl.md              # Tech Lead discovery review
├── 03-plan-se.md                   # SE plan + cross-review notes
├── 03-plan-qa.md                   # QA plan + cross-review notes
├── 03-plan-ui-review.md            # UI/UX plan review (conditional)
├── 03-plan-tl.md                   # Tech Lead plan review (with verdict)
├── 04-implementation-summary.md    # SE implementation summary (consolidated)
├── 04-test-report.md               # QA test report (consolidated)
├── 05-quality-gate.md              # Quality gate (QA + SE + TL + UI/UX sections)
└── 06-pr-summary.md                # PR description
```

**Major workflow** (up to 21 documents):
```
specs/003-new-auth-system/
├── 01-spec.md                      # PM specification
├── 01-spec-ui-review.md            # UI/UX spec review (conditional)
├── 01-spec-arch-review.md          # SA architecture spec review (always)
├── 01-spec-infra-review.md         # DevOps infra spec review (conditional)
├── 02-discovery-se.md              # SE discovery + cross-review notes
├── 02-discovery-qa.md              # QA discovery + cross-review notes
├── 02-discovery-ui.md              # UI/UX discovery (conditional)
├── 02-discovery-sa.md              # SA discovery + cross-review notes
├── 02-discovery-devops.md          # DevOps discovery (conditional)
├── 02-discovery-tl.md              # TL discovery review
├── 03-plan-se.md                   # SE plan + cross-review notes
├── 03-plan-qa.md                   # QA plan + cross-review notes
├── 03-plan-sa.md                   # SA plan + API contracts + ADRs + verdict
├── 03-plan-devops.md               # DevOps infra plan (conditional)
├── 03-plan-ui-review.md            # UI/UX plan review (conditional)
├── 03-plan-tl.md                   # TL plan review + verdict
├── 04-implementation-summary.md    # SE implementation summary (consolidated)
├── 04-test-report.md               # QA test report (consolidated)
├── 04-infra-summary.md             # DevOps infra summary (conditional)
├── 05-quality-gate.md              # QG (QA + SE + TL + SA + UI/UX + DevOps sections)
└── 06-pr-summary.md                # PR description + ADRs
```

### Agent Communication

Agents communicate through documents, not directly. The orchestrator controls the flow:

1. Each agent writes its output to a known file in the spec folder
2. The orchestrator tells each agent which documents to read
3. Cross-review rounds let agents review each other's work and append notes
4. All decisions and trade-offs are documented in the audit trail

## Branching Strategy

Sisan never pushes directly to `main`. Every change goes through a PR.

**Patch** (single PR):
```
main
 └── patch/001-fix-login-timeout   ← work branch, PR targets main
```

**Minor** (feature branch with sub-PRs):
```
main
 └── minor/002-add-retry-logic           ← feature branch
      ├── minor/002-add-retry-logic/pr-1-retry-util    ← PR targets feature branch
      └── minor/002-add-retry-logic/pr-2-apply-to-api  ← PR targets feature branch
                                          └── final PR: feature branch → main
```

**Major** (epic branch with sub-PRs):
```
main
 └── epic/003-new-auth-system              ← epic branch
      ├── epic/003-new-auth-system/pr-1-db-schema     ← PR targets epic branch
      ├── epic/003-new-auth-system/pr-2-auth-service   ← PR targets epic branch
      └── epic/003-new-auth-system/pr-3-api-endpoints  ← PR targets epic branch
                                            └── final PR: epic branch → main
```

**Multi-cycle with decomposition**: When a minor/major request is decomposed into patch cycles, each patch cycle can target the parent feature/epic branch instead of main:
```
main
 └── minor/002-add-retry-logic           ← feature branch (created once)
      ├── patch/004-retry-util            ← cycle 1, PR targets feature branch
      └── patch/005-apply-retry-to-api    ← cycle 2, PR targets feature branch
                                          └── final PR: feature branch → main
```

## Multi-Cycle Delivery

Large requests don't have to run through a single large workflow. The PM and SE analyze the request and can recommend breaking it into multiple smaller cycles:

```
User request: "Add retry logic with exponential backoff to all API calls"

PM Triage → Recommends 3 patch cycles:
  Cycle 1: Add retry utility function (patch)
  Cycle 2: Apply retry to auth API calls (patch)
  Cycle 3: Apply retry to data API calls (patch)

Each cycle runs independently through the full workflow,
producing its own spec folder, PR, and audit trail.
```

This keeps PRs small (< ~300 lines), making them easier for humans to review.

## Extensibility

The architecture is designed for adding new workflow tiers and agents:

**To add a new agent:**
1. Create `agents/{agent-name}.md` with frontmatter and mode definitions
2. Reference the agent in the appropriate `workflows/{tier}.md` file
3. Existing agents don't need modification — the orchestrator prompts control what documents each agent reads

**To customize an existing workflow:**
Each workflow file (`workflows/{tier}.md`) is self-contained. Modify the phase structure, cross-review rounds, or agent participation without affecting other tiers.

## License

MIT
