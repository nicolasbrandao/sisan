# Patch Workflow

This workflow handles bug fixes, hotfixes, typos, config changes, and small isolated changes (1-3 files). It uses 3 agents: PM, Software Engineer, and QA Engineer.

**Context variables** (provided by the orchestrator from Phase 0):
- `SPEC_FOLDER`: Path to the spec folder (e.g., `./specs/001-fix-login-timeout/`)
- `USER_REQUEST`: The user's request with clarifications
- `TRIAGE_RESULT`: PM's triage classification and rationale
- `TIER`: `patch`
- `BRANCH_NAME`: The work branch for this cycle (e.g., `patch/001-fix-login-timeout`)
- `BASE_BRANCH`: The branch the PR targets (e.g., `main`, or a feature/epic branch if this cycle is part of a larger delivery)

---

## Phase 1: Specification

**Goal**: Create a precise spec document that both SE and QA can work from independently.

### Steps:

1. **Launch the `pm` agent in SPECIFICATION mode**:
   - Prompt: "You are in SPECIFICATION mode. Create a detailed specification for the following request. Explore the codebase to understand the affected area, identify risks, and define precise acceptance criteria. Write the spec to `{SPEC_FOLDER}/01-spec.md`. The workflow tier is `patch` -- keep the scope tight. Request: {USER_REQUEST}. Triage context: {TRIAGE_RESULT}"

2. **Read the spec**: After the PM agent completes, read `{SPEC_FOLDER}/01-spec.md`.

3. **Present the spec to the user**:
   - Show a summary: title, type, priority, summary, acceptance criteria count, and any open questions.
   - If there are open questions, ask the user to answer them.

4. **User checkpoint**: Ask the user to approve the spec or request changes.
   - If changes requested: either re-launch the PM agent with feedback, or edit the spec directly based on user input.
   - Do not proceed until the user approves the spec.

---

## Phase 2: Discovery

**Goal**: SE and QA independently explore the codebase to understand the work ahead.

### Steps:

1. **Launch two agents in parallel**:

   a. **`software-engineer` in DISCOVERY mode**:
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`. Explore the codebase to understand the affected code paths, identify the root cause or insertion points, map dependencies, and document conventions. Write your findings to `{SPEC_FOLDER}/02-discovery-se.md`."

   b. **`qa-engineer` in DISCOVERY mode**:
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`. Explore the test infrastructure, identify existing test coverage for the affected area, document test patterns and conventions, and identify coverage gaps. Write your findings to `{SPEC_FOLDER}/02-discovery-qa.md`."

2. **Wait for both agents to complete.**

3. **Cross-review round** - Launch two agents in parallel:

   a. **`software-engineer` in DISCOVERY mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's discovery document at `{SPEC_FOLDER}/02-discovery-qa.md`. Then read your own discovery document at `{SPEC_FOLDER}/02-discovery-se.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-se.md` with your observations about the QA findings, any alignment or conflicts, and suggestions for the QA Engineer."

   b. **`qa-engineer` in DISCOVERY mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the Software Engineer's discovery document at `{SPEC_FOLDER}/02-discovery-se.md`. Then read your own discovery document at `{SPEC_FOLDER}/02-discovery-qa.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-qa.md` with your observations about the SE findings, any alignment or conflicts, and suggestions for the Software Engineer."

4. **Read both discovery documents** (including cross-review notes).

5. **Present a summary to the user**:
   - Key findings from SE: root cause, affected files, risks
   - Key findings from QA: test infrastructure, existing coverage, gaps
   - Any cross-review observations worth highlighting

6. **User checkpoint**: Ask the user if the findings look correct and if they want to proceed to planning.
   - If the user has additional context, note it for the planning phase.

---

## Phase 3: Planning

**Goal**: SE and QA independently create detailed plans for their respective work.

### Steps:

1. **Launch two agents in parallel**:

   a. **`software-engineer` in PLANNING mode**:
   - Prompt: "You are in PLANNING mode. Read the following documents: specification at `{SPEC_FOLDER}/01-spec.md`, your discovery at `{SPEC_FOLDER}/02-discovery-se.md`, and the QA discovery at `{SPEC_FOLDER}/02-discovery-qa.md`. Create a detailed implementation plan with the minimal change set needed to satisfy the spec. Write your plan to `{SPEC_FOLDER}/03-plan-se.md`."

   b. **`qa-engineer` in PLANNING mode**:
   - Prompt: "You are in PLANNING mode. Read the following documents: specification at `{SPEC_FOLDER}/01-spec.md`, your discovery at `{SPEC_FOLDER}/02-discovery-qa.md`, and the SE discovery at `{SPEC_FOLDER}/02-discovery-se.md`. Create a detailed test plan mapping every acceptance criterion to specific test cases. Include regression and edge case tests. Write your plan to `{SPEC_FOLDER}/03-plan-qa.md`."

2. **Wait for both agents to complete.**

3. **Cross-review round** - Launch two agents in parallel:

   a. **`software-engineer` in PLANNING mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's test plan at `{SPEC_FOLDER}/03-plan-qa.md`. Then read your own implementation plan at `{SPEC_FOLDER}/03-plan-se.md`. Append a '## Cross-Review Notes' section to your plan at `{SPEC_FOLDER}/03-plan-se.md`. Comment on whether your implementation supports the QA test cases, any interface contracts QA expects, and any suggested adjustments."

   b. **`qa-engineer` in PLANNING mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation plan at `{SPEC_FOLDER}/03-plan-se.md`. Then read your own test plan at `{SPEC_FOLDER}/03-plan-qa.md`. Append a '## Cross-Review Notes' section to your plan at `{SPEC_FOLDER}/03-plan-qa.md`. Comment on whether the SE's approach is testable, any dependencies on specific implementation choices, and any suggested adjustments."

4. **Read both plans** (including cross-review notes).

5. **Present the plans to the user**:
   - SE plan: approach, number of files to change, implementation order
   - QA plan: test strategy, number of test cases, coverage of acceptance criteria
   - Any cross-review concerns

6. **User checkpoint**: Ask the user to approve both plans before implementation.
   - **CRITICAL**: Do NOT start implementation without explicit user approval.
   - If the user wants changes, re-launch the relevant agent(s) with feedback.

---

## Phase 4: Implementation

**Goal**: SE implements the code changes, QA implements the tests.

### Steps:

1. **Confirm user approval**: If the user has not explicitly approved, ask now.

2. **Determine execution order**:
   - Read both plans (`03-plan-se.md` and `03-plan-qa.md`).
   - If the QA test plan depends on new interfaces, functions, or code being created by the SE (i.e., tests import or reference code that doesn't exist yet), run SE first, then QA.
   - If the QA tests can be written independently (testing existing interfaces, behavior changes in existing functions), run both in parallel.
   - When in doubt, run SE first, then QA -- it's safer.

3. **Launch `software-engineer` in IMPLEMENTATION mode**:
   - Prompt: "You are in IMPLEMENTATION mode. Read your implementation plan at `{SPEC_FOLDER}/03-plan-se.md` and the specification at `{SPEC_FOLDER}/01-spec.md`. Implement the changes following your plan exactly. If you must deviate, document why. Keep changes minimal -- only what the plan calls for. Write your implementation summary to `{SPEC_FOLDER}/04-implementation-summary.md`."

4. **After SE completes, launch `qa-engineer` in IMPLEMENTATION mode** (or in parallel if determined safe in step 2):
   - Prompt: "You are in IMPLEMENTATION mode. Read your test plan at `{SPEC_FOLDER}/03-plan-qa.md`, the specification at `{SPEC_FOLDER}/01-spec.md`, and the SE's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Implement the tests following your plan. Follow the project's test conventions exactly. Run the full test suite after writing tests. Write your test report to `{SPEC_FOLDER}/04-test-report.md`."

5. **Cross-review round** - Launch two agents in parallel:

   a. **`software-engineer` in IMPLEMENTATION mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's test report at `{SPEC_FOLDER}/04-test-report.md`. Then read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-implementation-summary.md` with your observations about whether the tests align with what was implemented, any interface assumptions in the tests that differ from reality, and any concerns."

   b. **`qa-engineer` in IMPLEMENTATION mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the Software Engineer's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Then read your own test report at `{SPEC_FOLDER}/04-test-report.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/04-test-report.md` with your observations about whether the implementation matches what the tests assume, any deviations from plan that affect test correctness, and any concerns."

6. **Read both output documents.**

7. **Present results to the user**:
   - SE: files changed, any deviations from plan
   - QA: tests written, test results (pass/fail counts)
   - Highlight any failures or concerns

8. **User checkpoint**: Ask the user to review and proceed to quality gates.
   - If there are test failures, discuss with the user before proceeding.

---

## Phase 5: Quality Gates

**Goal**: Final validation that the implementation meets the spec.

### Steps:

1. **Launch `qa-engineer` in QUALITY GATE mode**:
   - Prompt: "You are in QUALITY GATE mode. This is the final validation. Read the specification at `{SPEC_FOLDER}/01-spec.md`, the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`, and the test report at `{SPEC_FOLDER}/04-test-report.md`. Review the actual code changes by reading the modified files. Run the full test suite. Validate every acceptance criterion. Write the quality gate report to `{SPEC_FOLDER}/05-quality-gate.md`."

2. **After QA completes, launch `software-engineer` in QUALITY GATE mode**:
   - Prompt: "You are in QUALITY GATE mode (test review). Read the test report at `{SPEC_FOLDER}/04-test-report.md` and review the actual test code. Assess test correctness, coverage adequacy, and test quality. Append your '## Software Engineer - Test Quality Review' section to the quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`."

3. **After SE completes, launch `qa-engineer` in QUALITY GATE mode (final reflection)**:
   - Prompt: "You are completing a final reflection. Read the full quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`, including the Software Engineer's '## Software Engineer - Test Quality Review' section. Append a '## QA Engineer - Final Reflection' section to `{SPEC_FOLDER}/05-quality-gate.md` acknowledging the SE's test quality findings, noting any updates to your assessment warranted by the SE's observations, and confirming or revising your overall verdict."

4. **Read the quality gate report.**

5. **Present results to the user**:
   - QA verdict: APPROVED / APPROVED WITH CONDITIONS / REJECTED
   - Acceptance criteria results (PASS/FAIL per criterion)
   - Any blockers or conditions
   - SE's test quality assessment
   - Overall recommendation

6. **Handle the verdict**:

   a. **APPROVED**: Proceed to Phase 6 (Merge).

   b. **APPROVED WITH CONDITIONS**: Present the conditions to the user. Ask if they want to:
      - Address the conditions now (loop back to Phase 4 with specific fixes)
      - Accept the conditions and proceed to merge (create follow-up tasks)
      - Stop and address later

   c. **REJECTED**: Present the required fixes to the user. Ask if they want to:
      - Fix the issues (loop back to Phase 4 -- re-launch SE and/or QA with the specific issues to address)
      - Stop and address later
      - Override the rejection and merge anyway (warn that this bypasses quality gates)

---

## Phase 6: Merge

**Goal**: Create a PR with a comprehensive summary targeting the correct base branch.

### Steps:

1. **Generate the PR summary**: Create `{SPEC_FOLDER}/06-pr-summary.md` with this structure:

   ```markdown
   # PR Summary

   ## Title
   [Short descriptive title from the spec]

   ## Summary
   [2-3 sentences from the spec summary]

   ## Changes Made
   [From 04-implementation-summary.md]

   ## Tests
   [From 04-test-report.md - summary of tests written and results]

   ## Quality Gate
   [From 05-quality-gate.md - verdict and key findings]

   ## Acceptance Criteria Status
   [From 05-quality-gate.md - PASS/FAIL per criterion]

   ## Branch Info
   - **Work branch**: `{BRANCH_NAME}`
   - **Target branch**: `{BASE_BRANCH}`

   ## Spec Folder
   `{SPEC_FOLDER}`
   ```

2. **Verify branch state**:
   - Confirm you are on the correct work branch (`{BRANCH_NAME}`). If not, check it out.
   - The work branch should have been created during Phase 0 (Intake & Triage).
   - If the branch does not exist for some reason, create it now from `{BASE_BRANCH}`.
   - **NEVER commit or push directly to `main` or to the base branch.** All changes go through a PR.

3. **Stage and commit changes**:
   - Stage the code changes and test changes.
   - Stage the spec folder so the audit trail is preserved in the repo.
   - Commit with a descriptive message referencing the spec ID and title.
   - Example: `fix: resolve login timeout on session expiry (specs/001-fix-login-timeout)`

4. **Push and create PR**:
   - Push the work branch to the remote with `-u` flag.
   - Create a PR using `gh pr create`:
     - `--base {BASE_BRANCH}` to target the correct branch (this is critical -- do NOT default to main if a different base was specified)
     - `--title` from the spec title
     - `--body` from `06-pr-summary.md`
   - Example: `gh pr create --base main --title "Fix login timeout on session expiry" --body "$(cat specs/001-fix-login-timeout/06-pr-summary.md)"`

5. **GitHub merge checkpoint** — present the PR and wait in a loop until it is merged:

   ```
   ⏸ GitHub Merge Required
   PR has been created: {PR URL}
   Please go to GitHub, review the PR, and merge it into `{BASE_BRANCH}`.

   Once done, reply with one of:
   - "merged" / "done" → to complete the workflow
   - "changes requested" → I'll read the review comments and address them
   ```

   **If the user replies "changes requested"** — repeat the following until the user confirms the merge:

   a. Launch `software-engineer` in **PR REVIEW RESPONSE mode**:
      - Prompt: "You are in PR REVIEW RESPONSE mode. Fetch all review comments using `gh pr view {PR_NUMBER} --comments` and `gh pr diff {PR_NUMBER}`. For each comment: (1) If you agree, make the code change and explain what you did. (2) If you need clarification, formulate a specific question for the user. (3) If you disagree, prepare a concise, professional pushback explaining your reasoning. Commit and push any code changes to the same branch. Write a response summary to `{SPEC_FOLDER}/06-pr-review-response.md` (append if it already exists) listing each comment and how it was handled."

   b. Read `{SPEC_FOLDER}/06-pr-review-response.md`. Present each comment and its resolution.

   c. If any comments required user clarification: ask the user, then re-launch SE with the answers and repeat from (a).

   d. Once all comments are addressed, loop back to the checkpoint:
      ```
      ⏸ GitHub Merge Required (updated)
      Changes have been pushed. Please re-review the PR on GitHub.

      Reply with:
      - "merged" / "done" → to complete the workflow
      - "changes requested" → I'll address the new round of comments
      ```

   **This loop repeats until the user replies "merged" / "done".** Do not proceed past this point until that confirmation is received.

6. **Mark all todos complete.**


### Branching Rules

- **Patch (single cycle)**: `patch/{id}-{slug}` → PR to `main` (or user-specified base)
- **Patch (part of minor multi-cycle)**: `patch/{id}-{slug}` → PR to the parent feature branch `minor/{parent-id}-{parent-slug}`
- **Patch (part of major multi-cycle)**: `patch/{id}-{slug}` → PR to the parent epic branch `epic/{parent-id}-{parent-slug}`
- **NEVER** push directly to `main`, `master`, or any shared branch unless the user explicitly requests it
- If the user overrides and asks to push directly, warn them once, then comply if they confirm

---

## Error Recovery

- **Agent fails or returns incomplete output**: Inform the user. Offer to re-launch the agent or proceed manually.
- **Tests fail during implementation**: Document in the test report. Discuss with user during the checkpoint -- it may be a test issue or an implementation issue.
- **Quality gate rejects**: This is expected workflow. Loop back to implementation with specific fixes.
- **User wants to stop mid-workflow**: That's fine. All documents written so far are preserved in the spec folder. The user can resume later by re-running `/sisan` and pointing to the existing spec folder (future enhancement).
