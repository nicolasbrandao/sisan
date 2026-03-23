# Minor Workflow

This workflow handles small features, enhancements, refactors, new API endpoints, and new UI components (1-3 PRs, 5-15 files, single service). It uses 5 agents: PM, Tech Lead, Software Engineer, QA Engineer, and optionally UI/UX Specialist.

**Context variables** (provided by the orchestrator from Phase 0):
- `SPEC_FOLDER`: Path to the spec folder (e.g., `./specs/002-add-retry-logic/`)
- `USER_REQUEST`: The user's request with clarifications
- `TRIAGE_RESULT`: PM's triage classification and rationale
- `TIER`: `minor`
- `BRANCH_NAME`: The feature branch for this workflow (e.g., `minor/002-add-retry-logic`)
- `BASE_BRANCH`: The branch the final PR targets (e.g., `main`)
- `CYCLE_CONTEXT`: Multi-cycle context if applicable
- `BRANCHING_STRATEGY`: The full branching plan from triage

---

## Phase 1: Specification

**Goal**: Create a precise spec document that all agents can work from independently.

### Steps:

1. **Launch the `pm` agent in SPECIFICATION mode**:
   - Prompt: "You are in SPECIFICATION mode. Create a detailed specification for the following request. Explore the codebase to understand the affected area, identify risks, and define precise acceptance criteria. Write the spec to `{SPEC_FOLDER}/01-spec.md`. The workflow tier is `minor` — this is a small feature or enhancement. Be thorough with the 'Affected Areas > UI' section: if any user interface changes are involved, describe them clearly. If there are no UI changes, mark that section as 'None'. The UI section determines whether a UI/UX Specialist will be activated. Also detail the delivery strategy — how many PRs are recommended and what each covers. Request: {USER_REQUEST}. Triage context: {TRIAGE_RESULT}"

2. **Read the spec**: After the PM agent completes, read `{SPEC_FOLDER}/01-spec.md`.

3. **Present the spec to the user**:
   - Show a summary: title, type, priority, summary, acceptance criteria count, PR decomposition strategy, and any open questions.
   - Highlight: **UI changes detected** (or "No UI changes — UI/UX Specialist will be skipped").
   - If there are open questions, ask the user to answer them.

4. **User checkpoint**: Ask the user to approve the spec or request changes.
   - If changes requested: either re-launch the PM agent with feedback, or edit the spec directly based on user input.
   - Do not proceed until the user approves the spec.

---

## Phase 1.5: UI/UX Spec Review (CONDITIONAL)

**Goal**: Augment the spec with UI-specific acceptance criteria, accessibility requirements, and design system guidance.

### Activation Check:

1. **Read the spec** `{SPEC_FOLDER}/01-spec.md`.
2. **Check the `Affected Areas > UI` section**:
   - If the section contains "None", "N/A", or is empty: **SKIP this phase entirely**. Print: "No UI changes detected — skipping UI/UX Spec Review." Proceed to Phase 2.
   - If the section has any content describing UI changes: **ACTIVATE the UI/UX Specialist** and continue with the steps below.

### Steps (only if UI/UX is activated):

1. **Launch the `ui-ux-specialist` agent in SPEC REVIEW mode**:
   - Prompt: "You are in SPEC REVIEW mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`. Explore the codebase to understand the existing design system, component library, CSS methodology, accessibility patterns, and responsive breakpoints. Augment the spec with UI-specific acceptance criteria, accessibility requirements, responsive behavior expectations, and design system components to reuse. Write your review to `{SPEC_FOLDER}/01-spec-ui-review.md`."

2. **Read the UI spec review**: After the UI/UX agent completes, read `{SPEC_FOLDER}/01-spec-ui-review.md`.

3. **Present the UI-specific criteria to the user**:
   - Show: additional UI acceptance criteria, accessibility requirements, design system components to reuse, responsive behavior expectations.
   - Note: "These UI criteria will be treated as requirements for the SE's implementation and QA's validation."

4. **User checkpoint**: Ask the user to approve the augmented requirements.
   - If changes requested: re-launch the UI/UX agent with feedback, or edit the review directly.
   - Do not proceed until the user approves.

**Track UI/UX activation status**: Remember whether the UI/UX Specialist was activated. This determines their participation in Phase 2 and Phase 5.

---

## Phase 2: Discovery

**Goal**: SE, QA, and optionally UI/UX explore the codebase independently. Then the Tech Lead reviews everything from an architectural perspective.

### Steps:

1. **Launch agents in parallel** (2 or 3 agents depending on UI/UX activation):

   a. **`software-engineer` in DISCOVERY mode**:
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ' and the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}. Explore the codebase to understand the affected code paths, identify the root cause or insertion points, map dependencies, and document conventions. Write your findings to `{SPEC_FOLDER}/02-discovery-se.md`."

   b. **`qa-engineer` in DISCOVERY mode**:
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ' and the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}. Explore the test infrastructure, identify existing test coverage for the affected area, document test patterns and conventions, and identify coverage gaps. Write your findings to `{SPEC_FOLDER}/02-discovery-qa.md`."

   c. **`ui-ux-specialist` in DISCOVERY mode** (only if activated in Phase 1.5):
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md` and your spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`. Map the existing UI architecture of the affected area — component tree, styling patterns, accessibility state, and reusable elements. Write your findings to `{SPEC_FOLDER}/02-discovery-ui.md`."

2. **Wait for all agents to complete.**

3. **Cross-review round** — Launch agents in parallel (2 or 3):

   a. **`software-engineer` in DISCOVERY mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's discovery at `{SPEC_FOLDER}/02-discovery-qa.md`. Then read your own discovery at `{SPEC_FOLDER}/02-discovery-se.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-se.md` with observations about QA findings, alignment or conflicts, and suggestions."

   b. **`qa-engineer` in DISCOVERY mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the Software Engineer's discovery at `{SPEC_FOLDER}/02-discovery-se.md`. Then read your own discovery at `{SPEC_FOLDER}/02-discovery-qa.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-qa.md` with observations about SE findings, alignment or conflicts, and suggestions."

   c. **`ui-ux-specialist` in DISCOVERY mode (cross-review)** (only if activated):
   - Prompt: "You are completing a cross-review. Read the Software Engineer's discovery at `{SPEC_FOLDER}/02-discovery-se.md`. Then read your own discovery at `{SPEC_FOLDER}/02-discovery-ui.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-ui.md` with observations about the SE's findings and their UI implications."

4. **Wait for cross-reviews to complete.**

5. **Launch `tech-lead` in DISCOVERY REVIEW mode** (sequential — runs AFTER cross-reviews):
   - Prompt: "You are in DISCOVERY REVIEW mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ', the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}, the SE discovery at `{SPEC_FOLDER}/02-discovery-se.md`, the QA discovery at `{SPEC_FOLDER}/02-discovery-qa.md`{if UI/UX active: ', and the UI/UX discovery at `{SPEC_FOLDER}/02-discovery-ui.md`'}. Assess the architectural implications of this change. Write your findings to `{SPEC_FOLDER}/02-discovery-tl.md`."

6. **Read all discovery documents** (SE, QA, UI/UX if active, TL).

7. **Present a summary to the user**:
   - Key findings from SE: affected files, root cause/insertion points, risks
   - Key findings from QA: test infrastructure, existing coverage, gaps
   - Key findings from UI/UX (if active): component architecture, design system usage, a11y gaps
   - Key findings from TL: architectural concerns, scope assessment, recommendations
   - Highlight: cross-review observations worth noting
   - **If TL recommends re-triage or further decomposition**: present this prominently and ask the user how to proceed.

8. **User checkpoint**: Ask the user if the findings look correct and if they want to proceed to planning.

---

## Phase 3: Planning

**Goal**: SE and QA create detailed plans. Then the Tech Lead validates the approach and renders a verdict that can block implementation.

### Steps:

1. **Launch two agents in parallel**:

   a. **`software-engineer` in PLANNING mode**:
   - Prompt: "You are in PLANNING mode. Read all documents: specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ', UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}, your discovery at `{SPEC_FOLDER}/02-discovery-se.md`, QA discovery at `{SPEC_FOLDER}/02-discovery-qa.md`{if UI/UX active: ', UI/UX discovery at `{SPEC_FOLDER}/02-discovery-ui.md`'}, and Tech Lead discovery review at `{SPEC_FOLDER}/02-discovery-tl.md`. Create a detailed implementation plan. Pay special attention to the TL's recommendations and address them in your plan. Include a PR Strategy section specifying how many sub-PRs this should be decomposed into and what each covers. Write your plan to `{SPEC_FOLDER}/03-plan-se.md`."

   b. **`qa-engineer` in PLANNING mode**:
   - Prompt: "You are in PLANNING mode. Read all documents: specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ', UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}, your discovery at `{SPEC_FOLDER}/02-discovery-qa.md`, SE discovery at `{SPEC_FOLDER}/02-discovery-se.md`{if UI/UX active: ', UI/UX discovery at `{SPEC_FOLDER}/02-discovery-ui.md`'}, and Tech Lead discovery review at `{SPEC_FOLDER}/02-discovery-tl.md`. Create a detailed test plan mapping every acceptance criterion{if UI/UX active: ' (including UI acceptance criteria from the spec review)'} to specific test cases. Include regression and edge case tests. Write your plan to `{SPEC_FOLDER}/03-plan-qa.md`."

2. **Wait for both agents to complete.**

3. **Cross-review round** — Launch agents in parallel (2 or 3):

   a. **`software-engineer` in PLANNING mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's test plan at `{SPEC_FOLDER}/03-plan-qa.md`. Then read your own implementation plan at `{SPEC_FOLDER}/03-plan-se.md`. Append a '## Cross-Review Notes' section to your plan with comments on whether your implementation supports QA's test cases, interface contracts, and any adjustments."

   b. **`qa-engineer` in PLANNING mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation plan at `{SPEC_FOLDER}/03-plan-se.md`. Then read your own test plan at `{SPEC_FOLDER}/03-plan-qa.md`. Append a '## Cross-Review Notes' section to your plan with comments on testability, implementation dependencies, and any adjustments."

   c. **`ui-ux-specialist` in PLANNING REVIEW** (only if activated):
   - Prompt: "You are reviewing the SE's plan for UI compliance. Read the SE's implementation plan at `{SPEC_FOLDER}/03-plan-se.md` and your spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`. Assess whether the plan adequately addresses your UI acceptance criteria, accessibility requirements, and design system recommendations. Write your review to `{SPEC_FOLDER}/03-plan-ui-review.md`."

4. **Wait for cross-reviews to complete.**

5. **Launch `tech-lead` in PLANNING REVIEW mode** (sequential — runs AFTER cross-reviews):
   - Prompt: "You are in PLANNING REVIEW mode. Read all documents: specification at `{SPEC_FOLDER}/01-spec.md`, all discovery documents (`02-discovery-se.md`, `02-discovery-qa.md`, `02-discovery-tl.md`{if UI/UX active: ', `02-discovery-ui.md`'}), the SE plan at `{SPEC_FOLDER}/03-plan-se.md` (with cross-review notes), the QA plan at `{SPEC_FOLDER}/03-plan-qa.md` (with cross-review notes){if UI/UX active: ', and the UI/UX plan review at `{SPEC_FOLDER}/03-plan-ui-review.md`'}. Evaluate the SE's approach, PR decomposition, and render your verdict. Write your review to `{SPEC_FOLDER}/03-plan-tl.md`."

6. **Read the TL's planning review** at `{SPEC_FOLDER}/03-plan-tl.md`. Check the verdict.

7. **Handle the TL verdict**:

   a. **APPROVED**: Proceed to step 8.

   b. **APPROVED WITH CHANGES**:
      - Present the required changes to the user.
      - Ask the user: "The Tech Lead has approved the plan with required changes. Should the SE revise the plan?"
      - If yes: re-launch `software-engineer` in PLANNING mode with the TL's required changes as additional input. Then re-run the TL PLANNING REVIEW to confirm.
      - If the user wants to override: note it and proceed.

   c. **REJECTED**:
      - Present the rejection reasons and recommended path forward to the user.
      - Ask the user: "The Tech Lead has rejected the plan. Options: (1) SE revises the plan, (2) Override and proceed anyway, (3) Stop here."
      - If revise: re-launch `software-engineer` in PLANNING mode with the TL's feedback. Then re-run the TL PLANNING REVIEW.
      - If override: warn that this bypasses architectural review, note it, and proceed.
      - If stop: halt the workflow.

8. **Present all plans to the user**:
   - SE plan: approach, number of PRs, files to change per PR, implementation order
   - QA plan: test strategy, test cases per acceptance criterion, coverage
   - TL verdict: APPROVED/WITH CHANGES/REJECTED and rationale
   - UI/UX plan review (if active): coverage of UI criteria
   - Any cross-review concerns

9. **User checkpoint**: Ask the user to approve both plans before implementation.
   - **CRITICAL**: Do NOT start implementation without explicit user approval.
   - If the user wants changes, re-launch the relevant agent(s) with feedback.

---

## Phase 4: Implementation

**Goal**: SE implements the code changes, QA implements the tests. For multi-PR workflows, iterate sequentially per sub-PR.

### Steps:

1. **Confirm user approval**: If the user has not explicitly approved, ask now.

2. **Determine sub-PR structure**:
   - Read the SE's plan (`03-plan-se.md`), specifically the PR Strategy section.
   - Determine the number of sub-PRs and what each covers.
   - If 1 PR: the implementation phase behaves identically to the patch workflow (single branch, no sub-branches).
   - If 2-3 PRs: iterate sequentially per sub-PR (steps 3-6).

3. **For each sub-PR** (iterate sequentially — PR 1 first, then PR 2, etc.):

   a. **Create the sub-branch**:
      - If multiple PRs: create `{BRANCH_NAME}/pr-{N}-{description}` from the feature branch `{BRANCH_NAME}`.
      - If single PR: work directly on `{BRANCH_NAME}` (no sub-branch needed).

   b. **Launch `software-engineer` in IMPLEMENTATION mode**:
      - Prompt: "You are in IMPLEMENTATION mode. Read your implementation plan at `{SPEC_FOLDER}/03-plan-se.md` and the specification at `{SPEC_FOLDER}/01-spec.md`. {If multi-PR: 'This is PR {N} of {total}. Implement ONLY the scope for this PR: {PR scope description from the plan}.'} Implement the changes following your plan exactly. If you must deviate, document why. Keep changes minimal. Write your implementation summary to `{SPEC_FOLDER}/04-implementation-summary.md`{if multi-PR: ' (append a section for PR {N})'}."

   c. **Determine execution order for QA** (same logic as patch):
      - If QA tests depend on new code the SE just wrote: run SE first, then QA.
      - If QA tests can be written independently: run in parallel.
      - When in doubt: SE first, then QA.

   d. **Launch `qa-engineer` in IMPLEMENTATION mode**:
      - Prompt: "You are in IMPLEMENTATION mode. Read your test plan at `{SPEC_FOLDER}/03-plan-qa.md`, the specification at `{SPEC_FOLDER}/01-spec.md`, and the SE's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. {If multi-PR: 'This is PR {N} of {total}. Implement ONLY the tests for this PR scope: {PR scope description}.'} Implement the tests following your plan. Follow the project's test conventions exactly. Run the test suite after writing tests. Write your test report to `{SPEC_FOLDER}/04-test-report.md`{if multi-PR: ' (append a section for PR {N})'}."

   e. **Commit changes to the branch**:
      - Stage code changes and test changes.
      - Commit with a descriptive message: `feat: {PR description} (specs/{id}-{slug}, PR {N}/{total})`

   f. **Intermediate checkpoint** (if this is NOT the last sub-PR):
      - Present results for this PR: files changed, tests written, test results.
      - Ask the user if they want to proceed to the next PR.
      - If the user wants changes: address them before proceeding.

4. **After all sub-PRs are implemented**:
   - Ensure `{SPEC_FOLDER}/04-implementation-summary.md` has a consolidated summary of all PRs.
   - Ensure `{SPEC_FOLDER}/04-test-report.md` has a consolidated report of all test results.

5. **Read both output documents.**

6. **Present results to the user**:
   - Per-PR summary: files changed, tests written, deviations from plan
   - Consolidated: total files changed, total tests, overall test results
   - Highlight any failures or concerns

7. **User checkpoint**: Ask the user to review and proceed to quality gates.
   - If there are test failures, discuss with the user before proceeding.

---

## Phase 5: Quality Gates

**Goal**: Final validation by all agents that the implementation meets the spec and quality standards.

### Steps:

1. **Launch `qa-engineer` in QUALITY GATE mode** (runs first — produces the base report):
   - Prompt: "You are in QUALITY GATE mode. This is the final validation. Read the specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ', the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}, the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`, and the test report at `{SPEC_FOLDER}/04-test-report.md`. Review the actual code changes by reading the modified files. Run the full test suite. Validate every acceptance criterion{if UI/UX active: ' (including UI criteria)'}. Write the quality gate report to `{SPEC_FOLDER}/05-quality-gate.md`."

2. **After QA completes, launch the remaining reviewers in parallel** (2 or 3 agents):

   a. **`software-engineer` in QUALITY GATE mode** (test review):
   - Prompt: "You are in QUALITY GATE mode (test review). Read the test report at `{SPEC_FOLDER}/04-test-report.md` and review the actual test code. Assess test correctness, coverage adequacy, and test quality. Append your '## Software Engineer - Test Quality Review' section to the quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`."

   b. **`tech-lead` in QUALITY GATE mode** (architecture review):
   - Prompt: "You are in QUALITY GATE mode (architecture review). Read the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`, the SE's plan at `{SPEC_FOLDER}/03-plan-se.md`, and your own plan review at `{SPEC_FOLDER}/03-plan-tl.md`. Review the actual code changes by reading the modified files. Assess plan adherence, architectural quality, and convention compliance. Append your '## Tech Lead - Architecture Review' section to the quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`."

   c. **`ui-ux-specialist` in QUALITY GATE mode** (design review, only if activated):
   - Prompt: "You are in QUALITY GATE mode (design review). Read your spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`, the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`, and the test report at `{SPEC_FOLDER}/04-test-report.md`. Review the actual UI code changes. Validate design system compliance, accessibility, and responsive behavior. Append your '## UI/UX Specialist - Design Review' section to the quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`."

3. **Read the full quality gate report** at `{SPEC_FOLDER}/05-quality-gate.md`.

4. **Present results to the user**:
   - QA verdict: APPROVED / APPROVED WITH CONDITIONS / REJECTED
   - Acceptance criteria results (PASS/FAIL per criterion)
   - SE's test quality assessment
   - TL's architecture review: verdict and any concerns
   - UI/UX design review (if active): compliance, accessibility, responsive behavior
   - Overall recommendation: consolidate all verdicts into a single recommendation.

5. **Handle the verdict** (use the most restrictive verdict across all reviewers):

   a. **All APPROVED/ADEQUATE**: Proceed to Phase 6 (Merge).

   b. **Any APPROVED WITH CONDITIONS or NEEDS IMPROVEMENT (non-blocking)**:
      - Present all conditions to the user. Ask if they want to:
        - Address the conditions now (loop back to Phase 4 with specific fixes)
        - Accept the conditions and proceed to merge (create follow-up tasks)
        - Stop and address later

   c. **QA REJECTED or any NEEDS IMPROVEMENT (blocking)**:
      - Present the required fixes. Ask if they want to:
        - Fix the issues (loop back to Phase 4 with specific issues to address)
        - Stop and address later
        - Override and merge anyway (warn that this bypasses quality gates)

---

## Phase 6: Merge

**Goal**: Create PRs with comprehensive summaries, targeting the correct branches.

### Steps:

1. **Generate the PR summary**: Create `{SPEC_FOLDER}/06-pr-summary.md` with this structure:

   ```markdown
   # PR Summary

   ## Title
   [Short descriptive title from the spec]

   ## Summary
   [2-3 sentences from the spec summary]

   ## Changes Made
   [From 04-implementation-summary.md — consolidated across all sub-PRs]

   ## Tests
   [From 04-test-report.md — summary of tests written and results]

   ## Quality Gate
   [From 05-quality-gate.md — all verdicts and key findings]
   - QA Verdict: [verdict]
   - SE Test Review: [verdict]
   - TL Architecture Review: [verdict]
   - UI/UX Design Review: [verdict or "N/A"]

   ## Acceptance Criteria Status
   [From 05-quality-gate.md — PASS/FAIL per criterion]

   ## Branch Info
   - **Feature branch**: `{BRANCH_NAME}`
   - **Target branch**: `{BASE_BRANCH}`
   - **Sub-PRs**: [list if applicable]

   ## Spec Folder
   `{SPEC_FOLDER}`
   ```

2. **Handle sub-PR branches** (if multi-PR):
   - For each sub-branch `{BRANCH_NAME}/pr-N-{description}`:
     - Verify the branch has the correct commits.
     - Push the sub-branch to the remote with `-u` flag.
     - Create a PR targeting the feature branch `{BRANCH_NAME}`:
       - `gh pr create --base {BRANCH_NAME} --title "{PR title}" --body "{PR description for this sub-PR}"`
     - Present the PR link to the user.

3. **Handle the feature branch** (single PR or final PR):
   - If single PR (no sub-branches): push `{BRANCH_NAME}` to remote, create PR targeting `{BASE_BRANCH}`.
   - If multi-PR: ask the user how to proceed with the feature-branch-to-main PR:
     - **Option A**: Create the final PR now (feature branch → BASE_BRANCH) with the full summary. Sub-PRs should be merged into the feature branch first.
     - **Option B**: Wait for sub-PRs to be reviewed/merged by humans first, then create the final PR later.
   - Use `gh pr create --base {BASE_BRANCH} --title "{spec title}" --body "$(cat {SPEC_FOLDER}/06-pr-summary.md)"`

4. **Stage and commit the spec folder**:
   - Stage the spec folder so the audit trail is preserved.
   - Commit: `docs: add spec audit trail for {id}-{slug}`
   - Push the updated branch.

5. **Present the results to the user**:
   - All PR links (sub-PRs + final PR if created)
   - Feature branch → target branch mapping
   - Total files changed, tests written
   - If multi-cycle: remind the user which cycles remain and what to do next.

6. **Mark all todos complete.**

### Branching Rules

- **Minor (single PR)**: `minor/{id}-{slug}` → PR to `{BASE_BRANCH}` (default: `main`)
- **Minor (multi-PR)**: Feature branch `minor/{id}-{slug}` stays. Sub-branches `minor/{id}-{slug}/pr-N-{desc}` → PRs to `minor/{id}-{slug}`. Final PR: `minor/{id}-{slug}` → `{BASE_BRANCH}`.
- **NEVER** push directly to `main`, `master`, or any shared branch unless the user explicitly requests it.
- If the user overrides and asks to push directly, warn them once, then comply if they confirm.

---

## Error Recovery

- **Agent fails or returns incomplete output**: Inform the user. Offer to re-launch the agent or proceed manually.
- **TL rejects the plan**: This is expected workflow. The SE revises the plan and the TL re-reviews. If the user disagrees with the TL, they can override.
- **Tests fail during implementation**: Document in the test report. Discuss with user during the checkpoint.
- **Quality gate rejects**: This is expected workflow. Loop back to implementation with specific fixes.
- **Sub-PR has issues while later PRs are fine**: Address the specific sub-PR. The sequential nature means later PRs can be re-implemented if earlier ones change.
- **User wants to stop mid-workflow**: All documents written so far are preserved in the spec folder. The user can resume later.
- **UI/UX Specialist finds issues late in quality gate**: Present to the user. If blocking, loop back. If non-blocking, note for follow-up.
