# Software Engineer Implementation Plan

## Spec Reference
- **Spec**: `specs/001-add-cross-review-rounds-to-workflows/01-spec.md`
- **Discovery**: `specs/001-add-cross-review-rounds-to-workflows/02-discovery-se.md`

---

## Approach

All five insertions are purely additive markdown text changes to three workflow files. Each new cross-review step is modelled exactly on the nearest equivalent cross-review step already present in the same file (Phase 2 or Phase 3), preserving the file-specific dash/em-dash convention, conditional label pattern, and prompt phrasing. No existing step content is altered; only step numbers that follow an insertion point are incremented by one.

The two architectural decisions flagged in discovery are resolved as follows, per the TL's recommendations:
- **Phase 1.9 SA↔DevOps ordering**: fully parallel (Option A). Both sub-items are lettered under one numbered step. Each agent reads the other's primary Phase 1.7/1.9 deliverable, consistent with every existing cross-review in the codebase.
- **Phase 4 major.md "relevant SA context"**: SA reads `04-infra-summary.md` and appends cross-review notes to `04-implementation-summary.md`. DevOps reads `04-implementation-summary.md` and appends cross-review notes to `04-infra-summary.md`. This mirrors the SE↔QA pattern in the same step.

---

## Changes

### 1. `workflows/patch.md` — Phase 4 cross-review (PR 1)

- **Action**: modify
- **Description**: Insert a new parallel SE↔QA cross-review step between current step 4 and step 5 of Phase 4. Renumber existing steps 5, 6, 7 to 6, 7, 8.
- **Details**:

  **Insertion point**: immediately after the closing line of current step 4 (the QA IMPLEMENTATION launch prompt line), before the line `5. **Read both output documents.**`.

  **Surrounding context for locating the insertion point**:
  ```
  4. **After SE completes, launch `qa-engineer` in IMPLEMENTATION mode** (or in parallel if determined safe in step 2):
     - Prompt: "You are in IMPLEMENTATION mode. Read your test plan at `{SPEC_FOLDER}/03-plan-qa.md`, the specification at `{SPEC_FOLDER}/01-spec.md`, and the SE's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Implement the tests following your plan. Follow the project's test conventions exactly. Run the full test suite after writing tests. Write your test report to `{SPEC_FOLDER}/04-test-report.md`."

  5. **Read both output documents.**
  ```

  **New text to insert** (between the blank line after step 4 and the line that begins `5. **Read`):
  ```
  5. **Cross-review round** - Launch two agents in parallel:

     a. **`software-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the QA Engineer's test report at `{SPEC_FOLDER}/04-test-report.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the tests align with what was implemented, any interface assumptions in the tests that differ from reality, and any concerns."

     b. **`qa-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own test report at `{SPEC_FOLDER}/04-test-report.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-test-report.md` with your observations about whether the implementation matches what the tests assume, any deviations from plan that affect test correctness, and any concerns."

  ```

  **Step renumbering after insertion**:
  - Current `5. **Read both output documents.**` → `6. **Read both output documents.**`
  - Current `6. **Present results to the user**:` → `7. **Present results to the user**:`
  - Current `7. **User checkpoint**:` → `8. **User checkpoint**:`

  **Dash convention**: patch.md uses a hyphen-dash (`-`) in the step label (e.g., `**Cross-review round** - Launch two agents in parallel:`). This matches the existing Phase 2 and Phase 3 cross-review steps in the same file.

- **Reason**: Closes Gap 1 (Phase 4 cross-review) for the patch tier. Satisfies AC-1, AC-4, AC-5, AC-9, AC-10.

---

### 2. `workflows/minor.md` — Phase 4 cross-review (PR 1)

- **Action**: modify
- **Description**: Insert a new parallel SE↔QA cross-review step between current step 4 and step 5 of Phase 4. Renumber existing steps 5, 6, 7 to 6, 7, 8.
- **Details**:

  **Insertion point**: immediately after the closing line of current step 4 (the consolidation block), before the line `5. **Read both output documents.**`.

  **Surrounding context for locating the insertion point**:
  ```
  4. **After all sub-PRs are implemented**:
     - Ensure `{SPEC_FOLDER}/04-implementation-summary.md` has a consolidated summary of all PRs.
     - Ensure `{SPEC_FOLDER}/04-test-report.md` has a consolidated report of all test results.

  5. **Read both output documents.**
  ```

  **New text to insert** (between the blank line after step 4 and the line that begins `5. **Read`):
  ```
  5. **Cross-review round** — Launch two agents in parallel:

     a. **`software-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the QA Engineer's test report at `{SPEC_FOLDER}/04-test-report.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the tests align with what was implemented, any interface assumptions in the tests that differ from reality, and any concerns."

     b. **`qa-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own test report at `{SPEC_FOLDER}/04-test-report.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-test-report.md` with your observations about whether the implementation matches what the tests assume, any deviations from plan that affect test correctness, and any concerns."

  ```

  **Step renumbering after insertion**:
  - Current `5. **Read both output documents.**` → `6. **Read both output documents.**`
  - Current `6. **Present results to the user**:` → `7. **Present results to the user**:`
  - Current `7. **User checkpoint**:` → `8. **User checkpoint**:`

  **Dash convention**: minor.md uses an em-dash (`—`) in the step label (e.g., `**Cross-review round** — Launch two agents in parallel:`). This matches the existing Phase 2 and Phase 3 cross-review steps in the same file.

- **Reason**: Closes Gap 1 (Phase 4 cross-review) for the minor tier. Satisfies AC-2, AC-4, AC-5, AC-9, AC-10.

---

### 3. `workflows/major.md` — Phase 1.9 SA↔DevOps cross-review (PR 2)

- **Action**: modify
- **Description**: Insert a new parallel SA↔DevOps cross-review step as step 3 inside the `### Steps (only if activated):` block of Phase 1.9, after the existing step 2. The `**Track activation status**` paragraph that follows is not a numbered step and must not be renumbered.
- **Details**:

  **Insertion point**: immediately after step 2 of Phase 1.9, before the blank line that precedes `**Track activation status**`.

  **Surrounding context for locating the insertion point**:
  ```
  2. **Read the infra spec review**. Present to user combined with Phase 1.7 results if both present. **User checkpoint**.

  **Track activation status**: Remember which conditional agents (UI/UX, DevOps) are active. This determines their participation in all subsequent phases.
  ```

  **New text to insert** (between step 2 and the `**Track activation status**` paragraph):
  ```
  3. **Cross-review round** — Launch two agents in parallel:

     a. **`software-architect` in SPEC REVIEW mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the DevOps Engineer's infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`. Then read your own architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/01-spec-arch-review.md` with your observations about whether the architecture decisions are feasible from an infrastructure perspective, any conflicts between architectural choices and infrastructure constraints, and any concerns."

     b. **`devops-engineer` in SPEC REVIEW mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the Software Architect's architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`. Then read your own infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/01-spec-infra-review.md` with your observations about infrastructure feasibility gaps or confirmations in the SA's architecture, any deployment complexity the architecture creates, and any concerns."

  ```

  **Step renumbering**: No renumbering needed. The step is appended as step 3 at the end of the numbered list. The `**Track activation status**` block that follows is prose, not a step.

  **Conditionality**: The entire `### Steps (only if activated):` block is already conditional on DevOps activation (established by the Activation Check at the top of Phase 1.9). No additional conditional label is required on the individual sub-items — the block-level conditionality covers the new step. This is consistent with steps 1 and 2 in the same block, which carry no `(only if activated)` label because the block header provides that guard.

  **Dash convention**: major.md uses an em-dash (`—`).

  **Ordering decision (parallel, not sequential)**: Both SA and DevOps read each other's primary Phase 1.7/1.9 deliverable (not each other's cross-review notes). SA reads `01-spec-infra-review.md` (DevOps's Phase 1.9 output); DevOps reads `01-spec-arch-review.md` (SA's Phase 1.7 output). This is consistent with all six existing cross-review rounds in the codebase, where each agent reads the other's primary document. The word "updated" in AC-7 of the spec refers to the document as updated during Phase 1.7 (the SA's primary deliverable), not after SA appends cross-review notes to it.

- **Reason**: Closes Gap 3 (Phase 1.7/1.9 SA↔DevOps cross-review). Satisfies AC-7, AC-8, AC-9, AC-10.

---

### 4. `workflows/major.md` — Phase 4 cross-review (PR 2)

- **Action**: modify
- **Description**: Insert a new parallel cross-review step between current step 4 and step 5 of Phase 4. The step includes SE↔QA plus SA↔DevOps (conditional). Renumber existing steps 5 and 6 to 6 and 7.
- **Details**:

  **Insertion point**: immediately after the closing line of current step 4 (the consolidation block, ending with the DevOps line), before the line `5. **Present results**:`.

  **Surrounding context for locating the insertion point**:
  ```
  4. **After all sub-PRs are implemented**:
     - Ensure `04-implementation-summary.md` has consolidated sections for all PRs.
     - Ensure `04-test-report.md` has consolidated results for all PRs.
     - If DevOps active: ensure `04-infra-summary.md` has consolidated infra sections.

  5. **Present results**: Per-phase summary, per-PR summary, consolidated totals.
  ```

  **New text to insert** (between the blank line after step 4 and the line that begins `5. **Present results**`):
  ```
  5. **Cross-review round** — Launch agents in parallel (2 or 4):

     a. **`software-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the QA Engineer's test report at `{SPEC_FOLDER}/04-test-report.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the tests align with what was implemented, any interface assumptions in the tests that differ from reality, and any concerns."

     b. **`qa-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own test report at `{SPEC_FOLDER}/04-test-report.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-test-report.md` with your observations about whether the implementation matches what the tests assume, any deviations from plan that affect test correctness, and any concerns."

     c. **`software-architect` in IMPLEMENTATION mode (cross-review)** (only if activated):
     - Prompt: "You are completing a cross-review. Read the DevOps Engineer's infrastructure summary at `{SPEC_FOLDER}/04-infra-summary.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the infrastructure implementation aligns with the architecture decisions, any concerns about infra changes that affect application behaviour, and any concerns."

     d. **`devops-engineer` in IMPLEMENTATION mode (cross-review)** (only if activated):
     - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own infrastructure summary at `{SPEC_FOLDER}/04-infra-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-infra-summary.md` with your observations about whether the application implementation has any infrastructure implications, any deployment concerns raised by the SE's changes, and any concerns."

  ```

  **Step renumbering after insertion**:
  - Current `5. **Present results**: Per-phase summary, per-PR summary, consolidated totals.` → `6. **Present results**: Per-phase summary, per-PR summary, consolidated totals.`
  - Current `6. **User checkpoint**.` → `7. **User checkpoint**.`

  **Conditionality**: SA and DevOps sub-items are labelled `(only if DevOps is active)`, matching the exact conditional language used for DevOps participation in Phase 4 step 3d (`(for infra PRs, only if activated)`) and Phase 2 step 3e (`(only if activated)`). The label `(only if DevOps is active)` is preferred here for clarity, as it names the condition explicitly at the cross-review step level (unlike the Phase 4 loop where the block-level context already implies it). Note: either `(only if DevOps is active)` or `(only if activated)` is acceptable; check the Phase 2 major.md pattern and use exact matching phrasing `(only if activated)` if consistency is preferred.

  **SA document target**: SA appends to `04-implementation-summary.md`, not to a separate SA document, because the SA has no `04-*` output file. This follows the TL's recommendation and the symmetry of the SE↔QA sub-items in the same step.

  **Dash convention**: major.md uses an em-dash (`—`).

- **Reason**: Closes Gap 1 (Phase 4 cross-review) for the major tier, including the SA↔DevOps dimension. Satisfies AC-3, AC-4, AC-5, AC-9, AC-10.

---

### 5. `workflows/patch.md` — Phase 5 QA final reflection (PR 2)

- **Action**: modify
- **Description**: Insert a new sequential QA final reflection step as step 3 of Phase 5, after the existing SE test quality review step (step 2). Renumber existing steps 3, 4, 5 to 4, 5, 6.
- **Details**:

  **Insertion point**: immediately after the closing line of current step 2 (the SE QUALITY GATE launch prompt line), before the line `3. **Read the quality gate report.**`.

  **Surrounding context for locating the insertion point**:
  ```
  2. **After QA completes, launch `software-engineer` in QUALITY GATE mode**:
     - Prompt: "You are in QUALITY GATE mode (test review). Read the test report at `{SPEC_FOLDER}/04-test-report.md` and review the actual test code. Assess test correctness, coverage adequacy, and test quality. Append your '## Software Engineer - Test Quality Review' section to the quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`."

  3. **Read the quality gate report.**
  ```

  **New text to insert** (between the blank line after step 2 and the line that begins `3. **Read the quality gate report`):
  ```
  3. **After SE completes, launch `qa-engineer` in QUALITY GATE mode (final reflection)**:
     - Prompt: "You are completing a final reflection. Read the full quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`, including the Software Engineer's '## Software Engineer - Test Quality Review' section appended by the SE. Append a '## QA Engineer - Final Reflection' section to `{SPEC_FOLDER}/05-quality-gate.md` acknowledging the SE's test quality findings, noting any updates to your assessment warranted by the SE's observations, and confirming or revising your overall verdict."

  ```

  **Step renumbering after insertion**:
  - Current `3. **Read the quality gate report.**` → `4. **Read the quality gate report.**`
  - Current `4. **Present results to the user**:` → `5. **Present results to the user**:`
  - Current `5. **Handle the verdict**:` → `6. **Handle the verdict**:`

  **Internal cross-reference check**: The `Handle the verdict` step (currently step 5, becoming step 6) contains the phrase "loop back to Phase 4 — re-launch SE and/or QA with the specific issues to address". This references Phase 4 by name, not by step number, so no reference update is needed.

  **Sequential, not parallel**: This step is inherently sequential — QA must read the SE's appended section before writing the reflection. The label uses "After SE completes, launch..." consistent with the sequential language already used in Phase 5 step 2 ("After QA completes, launch `software-engineer`..."). No parallel sub-items are used.

  **Dash convention**: patch.md uses a hyphen-dash (`-`). However, this step is sequential (not a parallel launch), so no "Launch N agents in parallel" label appears. The step label mirrors the sequential style of step 2 in the same phase.

- **Reason**: Closes Gap 2 (Phase 5 QA final reflection) for the patch tier. Satisfies AC-6, AC-9, AC-10.

---

## Implementation Order

1. **`workflows/patch.md` Phase 4 cross-review** (Change 1). This is the simplest of the five insertions — a single-file, two-agent parallel step with no conditionality. Completing it first establishes the exact prompt text pattern that all subsequent SE↔QA cross-review steps replicate.

2. **`workflows/minor.md` Phase 4 cross-review** (Change 2). Identical in structure to Change 1, with only the em-dash convention difference. Doing this second allows direct copy-check of the SE↔QA prompt text against Change 1.

3. **`workflows/major.md` Phase 1.9 SA↔DevOps cross-review** (Change 3). The first PR 2 change. Self-contained insertion in Phase 1.9 with no renumbering dependency on other changes in the same file.

4. **`workflows/major.md` Phase 4 cross-review** (Change 4). Also in `major.md`, but in a separate phase (Phase 4) from Change 3 (Phase 1.9). Both can be made in the same editing pass of the file. If editing sequentially, do Change 3 first (it is higher in the file), then Change 4.

5. **`workflows/patch.md` Phase 5 QA final reflection** (Change 5). Final change, also in `patch.md`. Editing patch.md twice is acceptable; both changes are in different phases (Phase 4 and Phase 5) with no ordering dependency between them. Both can be made in the same editing pass.

---

## Convention Compliance

- **Prompt opener**: All new prompts open with `"You are completing a cross-review."` (Steps 1–4) or `"You are completing a final reflection."` (Step 5), matching the established phrasing in the same files.
- **Read order in prompts**: Each agent reads the other's document first, then their own document second — consistent with all existing cross-review prompts.
- **Append instruction**: All prompts use `"Append a '## Cross-Review Notes' section to your document at {SPEC_FOLDER}/[doc]"`, matching the exact phrase from existing cross-reviews (except the Phase 5 QA reflection, which uses `"## QA Engineer - Final Reflection"` as specified in the spec's AC-6).
- **Document path format**: All document references use the `` `{SPEC_FOLDER}/[filename]` `` pattern with the prefix and backtick quoting, consistent with every existing prompt in all three files.
- **Dash/em-dash**: patch.md uses `-` in step labels; minor.md and major.md use `—`. All five new steps comply.
- **Agent label format**: Parallel sub-items use `**\`agent-name\` in MODE (cross-review)**:` — the mode matches the primary work mode of the phase (`IMPLEMENTATION` for Phase 4, `SPEC REVIEW` for Phase 1.9, `QUALITY GATE` for Phase 5 step 5). The `(cross-review)` parenthetical is appended, matching Phase 2 and Phase 3 existing cross-review labels.
- **Conditional label**: `(only if activated)` on the SA and DevOps sub-items of the major.md Phase 4 cross-review step, matching the exact pattern used at Phase 2 step 3e of `major.md`. The Phase 1.9 step requires no per-sub-item conditional label because the enclosing `### Steps (only if activated):` block provides the guard.
- **Step numbering**: All renumbered steps increment by exactly 1. No steps are removed or reordered.

---

## PR Strategy

### PR 1: `patch.md` Phase 4 + `minor.md` Phase 4

**Files changed**: `workflows/patch.md`, `workflows/minor.md`

**Estimated lines added**: ~22 lines total (11 lines per file, including blank separator lines)

**Exact edits**:

#### `workflows/patch.md` — Phase 4

- **Insertion point**: after the blank line following step 4's prompt line, before `5. **Read both output documents.**`
- **Surrounding text (old_string for Edit tool)**:
  ```
     - Prompt: "You are in IMPLEMENTATION mode. Read your test plan at `{SPEC_FOLDER}/03-plan-qa.md`, the specification at `{SPEC_FOLDER}/01-spec.md`, and the SE's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Implement the tests following your plan. Follow the project's test conventions exactly. Run the full test suite after writing tests. Write your test report to `{SPEC_FOLDER}/04-test-report.md`."

  5. **Read both output documents.**
  ```
- **Replacement (new_string for Edit tool)**:
  ```
     - Prompt: "You are in IMPLEMENTATION mode. Read your test plan at `{SPEC_FOLDER}/03-plan-qa.md`, the specification at `{SPEC_FOLDER}/01-spec.md`, and the SE's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Implement the tests following your plan. Follow the project's test conventions exactly. Run the full test suite after writing tests. Write your test report to `{SPEC_FOLDER}/04-test-report.md`."

  5. **Cross-review round** - Launch two agents in parallel:

     a. **`software-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the QA Engineer's test report at `{SPEC_FOLDER}/04-test-report.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the tests align with what was implemented, any interface assumptions in the tests that differ from reality, and any concerns."

     b. **`qa-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own test report at `{SPEC_FOLDER}/04-test-report.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-test-report.md` with your observations about whether the implementation matches what the tests assume, any deviations from plan that affect test correctness, and any concerns."

  6. **Read both output documents.**
  ```
- **Additional renaming in patch.md Phase 4**: change `6. **Present results to the user**:` → `7. **Present results to the user**:` and `7. **User checkpoint**:` → `8. **User checkpoint**:`

#### `workflows/minor.md` — Phase 4

- **Insertion point**: after the blank line following step 4's last bullet, before `5. **Read both output documents.**`
- **Surrounding text (old_string for Edit tool)**:
  ```
  4. **After all sub-PRs are implemented**:
     - Ensure `{SPEC_FOLDER}/04-implementation-summary.md` has a consolidated summary of all PRs.
     - Ensure `{SPEC_FOLDER}/04-test-report.md` has a consolidated report of all test results.

  5. **Read both output documents.**
  ```
- **Replacement (new_string for Edit tool)**:
  ```
  4. **After all sub-PRs are implemented**:
     - Ensure `{SPEC_FOLDER}/04-implementation-summary.md` has a consolidated summary of all PRs.
     - Ensure `{SPEC_FOLDER}/04-test-report.md` has a consolidated report of all test results.

  5. **Cross-review round** — Launch two agents in parallel:

     a. **`software-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the QA Engineer's test report at `{SPEC_FOLDER}/04-test-report.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the tests align with what was implemented, any interface assumptions in the tests that differ from reality, and any concerns."

     b. **`qa-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own test report at `{SPEC_FOLDER}/04-test-report.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-test-report.md` with your observations about whether the implementation matches what the tests assume, any deviations from plan that affect test correctness, and any concerns."

  6. **Read both output documents.**
  ```
- **Additional renaming in minor.md Phase 4**: change `6. **Present results to the user**:` → `7. **Present results to the user**:` and `7. **User checkpoint**:` → `8. **User checkpoint**:`

---

### PR 2: `major.md` Phase 4 + `major.md` Phase 1.9 + `patch.md` Phase 5

**Files changed**: `workflows/major.md`, `workflows/patch.md`

**Estimated lines added**: ~50 lines total (~12 for major.md Phase 1.9, ~22 for major.md Phase 4, ~8 for patch.md Phase 5)

**Exact edits**:

#### `workflows/major.md` — Phase 1.9 SA↔DevOps cross-review

- **Insertion point**: after step 2 of Phase 1.9, before the `**Track activation status**` paragraph
- **Surrounding text (old_string for Edit tool)**:
  ```
  2. **Read the infra spec review**. Present to user combined with Phase 1.7 results if both present. **User checkpoint**.

  **Track activation status**: Remember which conditional agents (UI/UX, DevOps) are active. This determines their participation in all subsequent phases.
  ```
- **Replacement (new_string for Edit tool)**:
  ```
  2. **Read the infra spec review**. Present to user combined with Phase 1.7 results if both present. **User checkpoint**.

  3. **Cross-review round** — Launch two agents in parallel:

     a. **`software-architect` in SPEC REVIEW mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the DevOps Engineer's infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`. Then read your own architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/01-spec-arch-review.md` with your observations about whether the architecture decisions are feasible from an infrastructure perspective, any conflicts between architectural choices and infrastructure constraints, and any concerns."

     b. **`devops-engineer` in SPEC REVIEW mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the Software Architect's architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`. Then read your own infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/01-spec-infra-review.md` with your observations about infrastructure feasibility gaps or confirmations in the SA's architecture, any deployment complexity the architecture creates, and any concerns."

  **Track activation status**: Remember which conditional agents (UI/UX, DevOps) are active. This determines their participation in all subsequent phases.
  ```
- **Step renumbering**: No other numbered steps in Phase 1.9 are affected. The new step is an append-only insertion at the end of the numbered list.

#### `workflows/major.md` — Phase 4 cross-review

- **Insertion point**: after the blank line following step 4's last bullet, before `5. **Present results**:`
- **Surrounding text (old_string for Edit tool)**:
  ```
  4. **After all sub-PRs are implemented**:
     - Ensure `04-implementation-summary.md` has consolidated sections for all PRs.
     - Ensure `04-test-report.md` has consolidated results for all PRs.
     - If DevOps active: ensure `04-infra-summary.md` has consolidated infra sections.

  5. **Present results**: Per-phase summary, per-PR summary, consolidated totals.
  ```
- **Replacement (new_string for Edit tool)**:
  ```
  4. **After all sub-PRs are implemented**:
     - Ensure `04-implementation-summary.md` has consolidated sections for all PRs.
     - Ensure `04-test-report.md` has consolidated results for all PRs.
     - If DevOps active: ensure `04-infra-summary.md` has consolidated infra sections.

  5. **Cross-review round** — Launch agents in parallel (2 or 4):

     a. **`software-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the QA Engineer's test report at `{SPEC_FOLDER}/04-test-report.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the tests align with what was implemented, any interface assumptions in the tests that differ from reality, and any concerns."

     b. **`qa-engineer` in IMPLEMENTATION mode (cross-review)**:
     - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own test report at `{SPEC_FOLDER}/04-test-report.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-test-report.md` with your observations about whether the implementation matches what the tests assume, any deviations from plan that affect test correctness, and any concerns."

     c. **`software-architect` in IMPLEMENTATION mode (cross-review)** (only if activated):
     - Prompt: "You are completing a cross-review. Read the DevOps Engineer's infrastructure summary at `{SPEC_FOLDER}/04-infra-summary.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the infrastructure implementation aligns with the architecture decisions, any concerns about infra changes that affect application behaviour, and any concerns."

     d. **`devops-engineer` in IMPLEMENTATION mode (cross-review)** (only if activated):
     - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own infrastructure summary at `{SPEC_FOLDER}/04-infra-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-infra-summary.md` with your observations about whether the application implementation has any infrastructure implications, any deployment concerns raised by the SE's changes, and any concerns."

  6. **Present results**: Per-phase summary, per-PR summary, consolidated totals.
  ```
- **Additional renaming in major.md Phase 4**: change `6. **User checkpoint**.` → `7. **User checkpoint**.`

#### `workflows/patch.md` — Phase 5 QA final reflection

- **Insertion point**: after the blank line following step 2's prompt line, before `3. **Read the quality gate report.**`
- **Surrounding text (old_string for Edit tool)**:
  ```
  2. **After QA completes, launch `software-engineer` in QUALITY GATE mode**:
     - Prompt: "You are in QUALITY GATE mode (test review). Read the test report at `{SPEC_FOLDER}/04-test-report.md` and review the actual test code. Assess test correctness, coverage adequacy, and test quality. Append your '## Software Engineer - Test Quality Review' section to the quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`."

  3. **Read the quality gate report.**
  ```
- **Replacement (new_string for Edit tool)**:
  ```
  2. **After QA completes, launch `software-engineer` in QUALITY GATE mode**:
     - Prompt: "You are in QUALITY GATE mode (test review). Read the test report at `{SPEC_FOLDER}/04-test-report.md` and review the actual test code. Assess test correctness, coverage adequacy, and test quality. Append your '## Software Engineer - Test Quality Review' section to the quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`."

  3. **After SE completes, launch `qa-engineer` in QUALITY GATE mode (final reflection)**:
     - Prompt: "You are completing a final reflection. Read the full quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`, including the Software Engineer's '## Software Engineer - Test Quality Review' section. Append a '## QA Engineer - Final Reflection' section to `{SPEC_FOLDER}/05-quality-gate.md` acknowledging the SE's test quality findings, noting any updates to your assessment warranted by the SE's observations, and confirming or revising your overall verdict."

  4. **Read the quality gate report.**
  ```
- **Additional renaming in patch.md Phase 5**: change `4. **Present results to the user**:` → `5. **Present results to the user**:` and `5. **Handle the verdict**:` → `6. **Handle the verdict**:`

---

## Risk Mitigation

- **Prompt consistency (Risk 6 from discovery)**: Every new prompt opens with `"You are completing a cross-review."` (or `"You are completing a final reflection."` for AC-6). The read-other-first / read-own-second order is maintained in all prompts. The `## Cross-Review Notes` heading is used in all cross-review prompts.

- **Step renumbering (Risk 1 from discovery)**: All renumbering is specified exactly in the PR Strategy section above. The total renumbered steps are: patch.md Phase 4 (3 steps: 5→6, 6→7, 7→8), patch.md Phase 5 (3 steps: 3→4, 4→5, 5→6), minor.md Phase 4 (3 steps: 5→6, 6→7, 7→8), major.md Phase 4 (2 steps: 5→6, 6→7). No renumbering needed for the major.md Phase 1.9 insertion.

- **Internal cross-references (Risks 2–4 from discovery)**: The only loop-back reference in any of the affected phases references "Phase 4" by name (in patch.md Phase 5, step 5 "Handle the verdict"). This is safe. No step-number cross-references were found in any of the affected phases.

- **Phase 1.9 "Track activation status" paragraph (Risk 5 from discovery)**: The paragraph is not a numbered step and is not renumbered. Its position relative to the phase boundary remains unchanged.

- **SA/DevOps conditionality (Risk 7 from discovery)**: SA and DevOps sub-items in the major.md Phase 4 cross-review are labelled `(only if activated)`, matching the Phase 2 pattern and making the condition explicit. The Phase 1.9 cross-review step requires no per-sub-item conditional label because it is inside the `### Steps (only if activated):` block.

- **QA Final Reflection is sequential (Risk 8 from discovery)**: The step uses "After SE completes, launch `qa-engineer`..." and does not use parallel sub-item lettering. It is a standalone numbered step.

- **Phase 1.9 ordering resolved as parallel (TL Recommendation 1)**: Both SA and DevOps sub-items are lettered under one numbered step. Each agent reads the other's primary deliverable, not each other's cross-review notes.

- **"Relevant SA context" resolved (TL Recommendation 2)**: SA reads `04-infra-summary.md` and appends to `04-implementation-summary.md`. DevOps reads `04-implementation-summary.md` and appends to `04-infra-summary.md`. No new document names are introduced.

---

## Cross-Review Notes

### QA Plan Alignment

The QA validation plan maps each acceptance criterion to a file-read check. The implementation plan above directly supports all ten checks:

- **AC-1 through AC-5** (Phase 4 cross-reviews): The insertion points and parallel sub-item structure in Changes 1–4 match exactly what QA's AC-1/AC-2/AC-3/AC-4/AC-5 checks look for.
- **AC-6** (Phase 5 QA reflection): Change 5 uses sequential language (`After SE completes...`) and the section heading `## QA Engineer - Final Reflection`, matching what AC-6 requires.
- **AC-7/AC-8** (major.md Phase 1.9): Change 3 is placed inside the `### Steps (only if activated):` block and uses parallel sub-items, satisfying both the "runs when active" (AC-7) and "skipped when not active" (AC-8) checks.
- **AC-9** (structure consistency): All five changes use the same heading level, step numbering style, and prompt wrapper syntax as adjacent steps.
- **AC-10** (no steps removed): All changes are purely additive. Existing step content is unchanged; only step numbers after insertion points are incremented.

### Interface the QA Plan Expects

The QA plan (AC-4 content check) expects each SE cross-review prompt to contain the phrase `"You are completing a cross-review"` as the opener and to specify reading `04-test-report.md` before `04-implementation-summary.md`. Changes 1, 2, 4 comply exactly. Change 3 (Phase 1.9) uses the same opener with documents `01-spec-infra-review.md` and `01-spec-arch-review.md` respectively.

### One Clarification for QA Validation of AC-7

The Phase 1.9 cross-review step (Change 3) uses a parallel structure — both agents are lettered sub-items (`a.` and `b.`) under a single numbered step. The QA plan's AC-7 check should verify this parallel structure (not sequential numbered steps), consistent with the TL's resolution of the ordering question. The DevOps sub-item reads `01-spec-arch-review.md` (the Phase 1.7 primary deliverable), not the SA's updated version post-cross-review.

### Exact Interface Contracts the Implementation Must Honour

The QA plan enforces several non-negotiable exact-text and exact-path requirements at the validation level. Every item listed below is a hard FAIL in the QA checks if not met precisely:

1. **Prompt opener** (V-4.1 through V-4.10): Every new cross-review prompt must open with the phrase `"You are completing a cross-review.` (with a period after "cross-review", inside the opening quote). The Phase 5 QA reflection (Change 5) is the only exception — it uses `"You are completing a final reflection.` — but none of the Phase 4 or Phase 1.9 steps may deviate from the cross-review opener.

2. **Section heading** (V-4.1 through V-4.10, V-6.3): All Phase 4 and Phase 1.9 cross-review prompts must instruct the agent to append a section titled exactly `## Cross-Review Notes`. The Phase 5 QA reflection must use `## QA Engineer - Final Reflection` — this heading is verbatim per AC-6 and V-6.3; any variation (`## QA Final Reflection`, `## Final QA Reflection`) is a FAIL.

3. **File paths with `{SPEC_FOLDER}/` prefix** (V-4.11): Every document reference in a new prompt must use the `` `{SPEC_FOLDER}/[filename]` `` pattern — backtick-quoted and prefixed. Bare filenames or relative paths (`./04-test-report.md`) are a FAIL.

4. **Append target per agent** (V-4.1 through V-4.10):
   - SE appends to `` `{SPEC_FOLDER}/04-implementation-summary.md` `` (reads `04-test-report.md` first).
   - QA appends to `` `{SPEC_FOLDER}/04-test-report.md` `` (reads `04-implementation-summary.md` first).
   - SA in Phase 4 reads `` `{SPEC_FOLDER}/04-infra-summary.md` `` and appends to `` `{SPEC_FOLDER}/04-implementation-summary.md` ``.
   - DevOps in Phase 4 reads `` `{SPEC_FOLDER}/04-implementation-summary.md` `` and appends to `` `{SPEC_FOLDER}/04-infra-summary.md` ``.
   - SA in Phase 1.9 reads `` `{SPEC_FOLDER}/01-spec-infra-review.md` `` and appends to `` `{SPEC_FOLDER}/01-spec-arch-review.md` ``.
   - DevOps in Phase 1.9 reads `` `{SPEC_FOLDER}/01-spec-arch-review.md` `` and appends to `` `{SPEC_FOLDER}/01-spec-infra-review.md` ``.
   - The plan (Changes 3 and 4) specifies these document pairings exactly. No `04-*` document may appear in Phase 1.9 prompts (EC-5) and no `01-*` document may appear in Phase 4 prompts (EC-6).

5. **Read order in each prompt** (V-4.x): Each agent reads the other's document first, its own document second. The plan already specifies this order in every prompt. This must not be reversed during implementation.

6. **Append instruction wording** (EC-4): Prompts use `"Append a '## Cross-Review Notes' section"` — not "Replace", "Overwrite", or "Update". This is already the wording in every prompt in the plan; it must not be altered.

### Dash Convention Is a Hard Structural Failure

QA checks V-9.1 (patch.md uses `-`), V-9.3 (minor.md uses `—`), and V-9.4 (major.md uses `—`) treat the wrong dash as a FAIL, not a style note. The plan specifies the correct convention per file in each change description. During implementation, the edit tool old_string/new_string blocks in the PR Strategy section must be followed verbatim to avoid a dash substitution error.

### Conditional Label for major.md Phase 4 SA/DevOps Sub-items

QA check V-3.3 requires `(only if activated)` as the conditional label on sub-items `c.` and `d.` in major.md Phase 4, matching the pattern used at Phase 2 step 3e of `major.md`. The plan specifies `(only if activated)` on both sub-items `c.` and `d.`.

### QA Plan Items That Do Not Require Implementation Adjustment

The following QA checks are fully satisfied by the plan as written and require no change:

- **V-NUM.1 through V-NUM.5** (step numbering): All renumbering is specified exactly in the Risk Mitigation section and the PR Strategy exact-edit blocks. The plan already lists the before and after step numbers for every affected step across all five insertion points.
- **V-10.x** (regression baseline): All changes are additive. The plan does not alter, remove, or reorder any existing step content or label. The only modification to existing lines is changing a step number (a digit at the start of a line), which the plan specifies precisely.
- **RT-1 through RT-7** (regression tests): The plan does not touch agent files, the command file, Phase 2, Phase 3, Phase 6, or the existing Phase 5 step 2. These are explicitly out of scope in each change description.
- **EC-1** (loop-back reference in patch.md Phase 5 step 6): The plan notes that `**Handle the verdict**` references "Phase 4" by name; no step-number update is needed or specified.
- **EC-2** (`**Track activation status**` paragraph): The plan's exact-edit block for the Phase 1.9 insertion preserves the paragraph as unnumbered prose immediately after the new step 3.
