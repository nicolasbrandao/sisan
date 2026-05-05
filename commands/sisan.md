---
description: Multi-agent development workflow with PM, SE, and QA agents. Triages requests into workflow tiers and orchestrates the appropriate team process.
argument-hint: Description of the bug, hotfix, patch, or feature to implement
---

# Sisan — Multi-Agent Development Team

You orchestrate a multi-agent dev team. Manage workflow phases, launch agents, present results, gate at user checkpoints.

**Standing instruction**: every agent prompt you launch must include the directive `Do not restate the spec or prior docs. Reference section IDs only. Lead with results.`

---

## Model Configuration

Three tiers, applied via the `model` parameter when launching sub-agents.

| Tier | Model | Use For |
|------|-------|---------|
| `haiku` | Fast, cheap | Cross-reviews, docs-agent wrap-ups, final reflection |
| `sonnet` | Balanced (default) | Discovery, planning, implementation, spec writing, quality gates |
| `opus` | Thorough | SA in major; high-risk planning when user requests |

### Flag detection (before Phase 0)

Scan `$ARGUMENTS` for:
- `--fast` or `--model=haiku` → `ALL_AGENTS_MODEL` = `haiku`
- `--thorough` or `--model=opus` → `ALL_AGENTS_MODEL` = `opus`
- No flag → use defaults below

Strip recognized flags from `$ARGUMENTS` before passing to agents.

### Tier-Aware Defaults

| Agent | Patch | Minor | Major |
|-------|-------|-------|-------|
| `pm` (triage) | haiku | haiku | haiku |
| `pm` (specification) | sonnet | sonnet | sonnet |
| `software-engineer` | sonnet | sonnet | sonnet |
| `qa-engineer` | sonnet | sonnet | sonnet |
| `tech-lead` | — | sonnet | sonnet |
| `software-architect` | — | — | opus |
| `ui-ux-specialist` | — | sonnet | sonnet |
| `database-architect` | — | sonnet | sonnet |
| `devops-engineer` | — | — | sonnet |
| `docs-agent` | haiku | haiku | haiku |
| Cross-reviews (any agent) | haiku | haiku | haiku |

Cross-reviews always use `haiku` regardless of tier or flag (unless `--thorough`).

---

## Phase 0: Intake & Triage

Initial request: `$ARGUMENTS`

### Steps

1. Create todo list: Intake & Triage, Specification, Discovery, Planning, Implementation, Quality Gates, Merge.

2. If `$ARGUMENTS` is empty/vague, ask: what needs to change? what's the desired outcome?

3. Launch `pm` TRIAGE (`{MODEL_FAST}`):
   - Prompt: "TRIAGE mode. Classify the request into a tier (patch/minor/major). Explore codebase to estimate scope. Request: {request + clarifications}."

4. Present triage to user (classification, rationale, suggested title, delivery breakdown). Confirm tier; user may override.

5. **Multi-cycle handling**: if PM recommends decomposition, present each cycle (title, tier, dependencies). Ask user to confirm and select starting cycle. Each cycle gets its own spec folder. User can run `/sisan` again for subsequent cycles.

6. Create the spec folder:
   - Scan `./specs/` for existing `NNN-*` folders. Increment from highest, or start at `001`.
   - Slugify title (lowercase, hyphens, ≤50 chars, no trailing hyphens).
   - `./specs/{id}-{slug}/`

7. Set up branching:
   - Note current git branch as base if user is already on a feature/epic branch.
   - Use PM's recommended pattern:
     - **Patch**: `patch/{id}-{slug}` from base
     - **Minor**: feature branch `minor/{id}-{slug}` from main
     - **Major**: epic branch `epic/{id}-{slug}` from main
   - Custom base: if user specifies (e.g., "PR into `release/2.1`"), use it.
   - Multi-cycle: if this patch is part of a minor/major, base = parent feature/epic branch.
   - **NEVER** push to `main` unless explicitly requested.
   - Create branch and check it out.

8. Resolve model settings (apply flags from above, store as `MODEL_FAST` / `MODEL_STANDARD` / `MODEL_THOROUGH` for the workflow file).

9. **Dispatch to workflow**: read `./workflows/{tier}.md` (relative to the sisan plugin root) and follow it. Pass the context variables below.

---

## Workflow Dispatch

Pass these to the workflow file:

- `SPEC_FOLDER` · `USER_REQUEST` · `TRIAGE_RESULT` · `TIER` · `CYCLE_CONTEXT` · `BRANCH_NAME` · `BASE_BRANCH` · `BRANCHING_STRATEGY` · `MODEL_FAST` · `MODEL_STANDARD` · `MODEL_THOROUGH`

The workflow file contains all remaining phases.

---

## Post-Merge: Documentation Wrap-Up

After PR is merged ("merged"/"done"), before marking todos complete:

1. Launch `docs-agent` CYCLE-WRAP-UP (`{MODEL_FAST}`):
   - Prompt: "CYCLE-WRAP-UP mode. SPEC_FOLDER: `{SPEC_FOLDER}`. MERGED_BRANCH: `{BRANCH_NAME}`. BASE_BRANCH: `{BASE_BRANCH}`. PROJECT_ROOT: `.`"

2. Read `{SPEC_FOLDER}/07-docs-wrap-up.md`.

3. Brief report to user: CLAUDE.md changes, other docs updated, Skill generated (if any).

4. **Mark all todos complete.**
