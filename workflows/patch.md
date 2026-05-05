# Patch Workflow

Bug fixes, hotfixes, typos, config changes, isolated changes (1–5 files). 3 agents: PM, SE, QA.

**Context variables** (from Phase 0):
- `SPEC_FOLDER` · `USER_REQUEST` · `TRIAGE_RESULT` · `TIER` (= `patch`) · `BRANCH_NAME` · `BASE_BRANCH`

**Standing instruction for every agent prompt below**: include the directive `Do not restate the spec or prior docs. Reference section IDs only (e.g., "AC-2", "plan §Changes"). Lead with results.`

---

## Phase 1: Specification

**Inputs Manifest**
- PM (specification): `USER_REQUEST`, `TRIAGE_RESULT`

### Steps

1. Launch `pm` in SPECIFICATION mode (`{MODEL_STANDARD}`):
   - Prompt: "SPECIFICATION mode. Tier: patch — keep scope tight. Request: {USER_REQUEST}. Triage: {TRIAGE_RESULT}. Write to `{SPEC_FOLDER}/01-spec.md`."

2. Read `{SPEC_FOLDER}/01-spec.md`.

3. Present summary to user (title, type, priority, AC count, open questions).

4. **User checkpoint**: approve or revise.

---

## Phase 2: Discovery

**Inputs Manifest**
- SE / QA (discovery): `01-spec.md`
- SE / QA (cross-review): own discovery + counterpart's discovery

### Steps

1. Launch in parallel (`{MODEL_STANDARD}`):

   a. `software-engineer` DISCOVERY: write to `02-discovery-se.md`.
   b. `qa-engineer` DISCOVERY: write to `02-discovery-qa.md`.

2. Cross-review round in parallel (`{MODEL_FAST}`):

   a. SE reads `02-discovery-qa.md`; appends `## Cross-Review Notes` to `02-discovery-se.md`.
   b. QA reads `02-discovery-se.md`; appends `## Cross-Review Notes` to `02-discovery-qa.md`.

3. Read both discovery docs.

4. Present summary to user (≤6 bullets total — root cause/insertion points, test infra, gaps, cross-review highlights).

5. **User checkpoint**.

---

## Phase 3: Planning

**Inputs Manifest**
- SE (planning): `01-spec.md`, `02-discovery-se.md`, `02-discovery-qa.md`
- QA (planning): `01-spec.md`, `02-discovery-qa.md`, `02-discovery-se.md`
- SE/QA (cross-review): own plan + counterpart's plan

### Steps

1. Launch in parallel (`{MODEL_STANDARD}`):

   a. `software-engineer` PLANNING: write to `03-plan-se.md`. Plan must include the **Interface Contract** section (signatures QA tests against).
   b. `qa-engineer` PLANNING: write to `03-plan-qa.md`. Tests must run against scaffolded interfaces (TDD-RED next).

2. Cross-review round in parallel (`{MODEL_FAST}`):

   a. SE reads `03-plan-qa.md`; appends `## Cross-Review Notes` to `03-plan-se.md` (do contracts support QA's tests?).
   b. QA reads `03-plan-se.md`; appends `## Cross-Review Notes` to `03-plan-qa.md` (any missing signatures?).

3. Read both plans.

4. Present summary (SE approach, file count; QA test count, AC coverage; cross-review concerns).

5. **User checkpoint** — **CRITICAL**: do NOT start implementation without explicit approval.

---

## Phase 4: Implementation (TDD)

Red-Green-Refactor: QA writes failing tests first; SE implements to make them pass.

### Phase 4.1 — Scaffold (SE)

**Inputs Manifest**: `01-spec.md`, `03-plan-se.md`

1. Launch `software-engineer` IMPLEMENTATION mode, **SCAFFOLD PHASE** (`{MODEL_STANDARD}`):
   - Prompt: "IMPLEMENTATION, SCAFFOLD PHASE. Create stubs only — no business logic. Project must compile. Write `04-scaffold-summary.md`."

2. Read `04-scaffold-summary.md`.

### Phase 4.2 — TDD Red (QA)

**Inputs Manifest**: `01-spec.md`, `03-plan-qa.md`, `04-scaffold-summary.md`

3. Launch `qa-engineer` IMPLEMENTATION mode, **TDD-RED PHASE** (`{MODEL_STANDARD}`):
   - Prompt: "IMPLEMENTATION, TDD-RED PHASE. Write all planned tests against scaffolds. Run suite. Verify: new tests fail with assertion errors, previously-passing tests still pass. Write `04-tdd-red-report.md`."

4. Read `04-tdd-red-report.md`.
   - If false greens or compile errors: stop, ask user how to proceed.
   - If pre-existing tests now fail: stop, this is a scaffold regression.

### Phase 4.3 — TDD Green (SE)

**Inputs Manifest**: `01-spec.md`, `03-plan-se.md`, `04-scaffold-summary.md`, `04-tdd-red-report.md`

5. Launch `software-engineer` IMPLEMENTATION mode, **TDD-GREEN PHASE** (`{MODEL_STANDARD}`):
   - Prompt: "IMPLEMENTATION, TDD-GREEN PHASE. Implement business logic per plan. Goal: every RED test → GREEN, no regressions. Run full suite. Write `04-implementation-summary.md`."

6. Read `04-implementation-summary.md`.
   - If tests still failing: present to user, do not proceed without explicit approval.

### Phase 4.4 — Cross-Reviews

**Inputs Manifest**: own implementation/test artifact + counterpart's

7. Cross-review round in parallel (`{MODEL_FAST}`):

   a. SE reads `04-tdd-red-report.md`; appends `## Cross-Review Notes` to `04-implementation-summary.md` (all RED → GREEN? interface assumptions?).
   b. QA reads `04-implementation-summary.md`; appends `## Cross-Review Notes` to `04-tdd-red-report.md` AND writes the consolidated `04-test-report.md` (≤40 lines: tests written, pass/fail counts, TDD cycle status).

8. Read all artifacts.

9. Present results (scaffold status, RED tests written, GREEN tests passing, deviations).

10. **User checkpoint**: review and proceed to quality gates.

---

## Phase 5: Quality Gates

**Inputs Manifest**
- QA (quality gate): `01-spec.md`, `04-implementation-summary.md`, `04-tdd-red-report.md`, `04-test-report.md`, modified files
- SE (test review): `04-tdd-red-report.md`, `04-test-report.md`, test files

### Steps

1. Launch in parallel (`{MODEL_STANDARD}`):

   a. `qa-engineer` QUALITY GATE: read code changes, run full suite, validate every AC. Write `05-quality-gate.md`.
   b. `software-engineer` QUALITY GATE (test review): read test code. Write `05-quality-gate-se.md`.

2. After both complete, launch `qa-engineer` final reflection (`{MODEL_FAST}`):
   - Prompt: "Append `## Software Engineer — Test Quality Review` to `05-quality-gate.md` (copy from `05-quality-gate-se.md`), then append `## QA Engineer — Final Reflection` confirming or revising your verdict."

3. Read `05-quality-gate.md`.

4. Present results: QA verdict (APPROVED / WITH CONDITIONS / REJECTED), AC results, SE test assessment, recommendation.

5. Handle verdict:
   - **APPROVED**: → Phase 6.
   - **APPROVED WITH CONDITIONS**: ask user — address now (loop to Phase 4) | accept and merge | stop.
   - **REJECTED**: ask user — fix (loop to Phase 4) | stop | override (warn).

---

## Phase 6: Merge

### Steps

1. **Assemble PR summary** (deterministic — extract, do not paraphrase, no agent invocation):

   Create `{SPEC_FOLDER}/06-pr-summary.md` by extracting (verbatim where possible):

   ```markdown
   # [spec title]

   ## Summary
   [Copy spec §Summary verbatim]

   ## Changes
   [Copy 04-implementation-summary.md §Changes]

   ## Tests
   [Copy 04-test-report.md §Run + §AC Coverage]

   ## Quality Gate
   - **Verdict:** [from 05-quality-gate.md §Verdict]
   - **AC results:** [from 05-quality-gate.md §AC Validation table]
   [Add SE test review verdict line.]

   ## Branch
   `{BRANCH_NAME}` → `{BASE_BRANCH}`

   ## Spec Folder
   `{SPEC_FOLDER}`
   ```

   Target: ≤80 lines. No re-narration; no new commentary.

2. **Verify branch**: confirm on `{BRANCH_NAME}`. Never commit to `main` or to `{BASE_BRANCH}` directly.

3. **Stage and commit**:
   - Stage code changes, test changes, and the spec folder.
   - Commit message: `fix: {short description} (specs/{id}-{slug})`

4. **Push and create PR**:
   - `git push -u origin {BRANCH_NAME}`
   - `gh pr create --base {BASE_BRANCH} --title "{spec title}" --body "$(cat {SPEC_FOLDER}/06-pr-summary.md)"`

5. **GitHub merge checkpoint** — present the PR URL and wait until the user confirms:

   ```
   ⏸ GitHub Merge Required
   PR created: {PR URL}
   Reply: "merged" / "done" → complete · "changes requested" → handle review
   ```

   **If "changes requested"**: loop until merge confirmed.

   a. Launch `software-engineer` in **PR REVIEW RESPONSE mode**:
      - Prompt: "PR REVIEW RESPONSE mode. Fetch comments via `gh pr view {PR_NUMBER} --comments` and `gh pr diff {PR_NUMBER}`. For each: (1) agree → make change, explain; (2) need clarification → formulate question for user; (3) disagree → professional pushback. Commit and push to the same branch. Append to `{SPEC_FOLDER}/06-pr-review-response.md`."

   b. Read `06-pr-review-response.md`. Present each comment + resolution.

   c. If clarification needed: ask user, then re-launch SE.

   d. After all addressed, return to checkpoint.

6. **Mark all todos complete.**

---

### Branching Rules

- **Patch (single cycle)**: `patch/{id}-{slug}` → PR to `main` (or user-specified base).
- **Patch (part of minor multi-cycle)**: → PR to parent feature branch `minor/{parent-id}-{parent-slug}`.
- **Patch (part of major multi-cycle)**: → PR to parent epic branch `epic/{parent-id}-{parent-slug}`.
- **NEVER** push directly to `main`/`master`/shared branches unless explicitly requested.

---

## Error Recovery

- **Agent fails or returns incomplete output**: inform user; offer re-launch or manual.
- **Tests fail in implementation**: document; discuss with user.
- **Quality gate rejects**: expected workflow; loop to Phase 4 with specific fixes.
- **User wants to stop mid-workflow**: spec folder is preserved; resume later.
