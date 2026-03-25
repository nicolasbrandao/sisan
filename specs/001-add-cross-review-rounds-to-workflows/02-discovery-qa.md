# QA Engineer Discovery

## Spec Reference
- **Spec**: `specs/001-add-cross-review-rounds-to-workflows/01-spec.md`
- **Title**: Add Cross-Review Rounds to Workflow Phases
- **Acceptance Criteria Count**: 10

---

## Test Infrastructure

### Framework & Runner
- **Framework**: None — there is no automated test suite. This is a Claude Code plugin consisting entirely of markdown workflow definition files, agent definition files, and a command file. The "tests" are the workflow files themselves; they are read and interpreted by the Claude orchestrator at runtime.
- **Runner**: N/A — no `npm test`, `pytest`, `jest`, or equivalent command exists. There is no `package.json`, `pyproject.toml`, `jest.config.*`, `vitest.config.*`, or any other test runner configuration file in the repository.
- **Config file**: None found.
- **Coverage tool**: None.

### Directory Structure
- **Test location**: N/A — no test directory exists (`__tests__/`, `tests/`, `spec/` are all absent).
- **Naming convention**: N/A
- **Test utilities**: N/A

### CI/CD
- **Pipeline**: No CI configuration files found (no `.github/workflows/`, `Jenkinsfile`, `.gitlab-ci.yml`, etc.).
- **Test commands in CI**: N/A

---

## What "Validation" Means in This Context

Because this project has no automated test suite, validation is entirely manual and structural. Each acceptance criterion is verified by reading the modified workflow files and checking for the presence, content, placement, and formatting of the new cross-review steps. This is a text/structure audit, not a code execution audit.

The validation approach for each acceptance criterion is one of two types:

1. **File-read validation**: The criterion can be fully verified by reading the relevant workflow file and checking for specific textual patterns, structural elements, or the absence of changes. This covers the majority of criteria.

2. **Runtime validation** (not feasible in this cycle): Verifying that an agent actually interprets a prompt correctly when the workflow runs. This would require running a full live workflow with real agents. This is out of scope for this validation cycle — the prompts themselves are the specification, and correct authorship of the prompts is the deliverable.

All 10 acceptance criteria can be validated through file-read validation. None require running the actual workflow.

---

## Existing "Test Coverage"

### Files That Are the Subject of Change
The three files being modified are themselves the objects under validation:

1. `workflows/patch.md` — currently 257 lines. Contains Phases 1–6 for the patch workflow.
2. `workflows/minor.md` — currently 374 lines. Contains Phases 1–6 for the minor workflow.
3. `workflows/major.md` — currently 437 lines. Contains Phases 1–6 for the major workflow.

### What IS Currently Present (Establishes the Pattern to Follow)

**Phase 2 cross-reviews (all three files)**:
- `patch.md` lines 52–58: Parallel cross-review round — SE reads QA's discovery doc and appends `## Cross-Review Notes` to `02-discovery-se.md`; QA reads SE's discovery doc and appends `## Cross-Review Notes` to `02-discovery-qa.md`.
- `minor.md` lines 88–97: Same pattern extended with optional UI/UX cross-review.
- `major.md` lines 127–143: Same pattern extended with SA and optional DevOps cross-reviews.

**Phase 3 cross-reviews (all three files)**:
- `patch.md` lines 88–94: Parallel cross-review round — SE appends `## Cross-Review Notes` to `03-plan-se.md`; QA appends `## Cross-Review Notes` to `03-plan-qa.md`.
- `minor.md` lines 133–143: Same pattern extended with optional UI/UX plan review.
- `major.md` lines 179–194: Same pattern extended with SA and DevOps plan reviews.

**Phase 5 SE→QA review (patch.md only)**:
- `patch.md` lines 150–151: After QA writes the base quality gate report, SE appends `## Software Engineer - Test Quality Review` to `05-quality-gate.md`. This is the one-directional review that the spec identifies as the partial Gap 2.

**Phase 1.9 DevOps Spec Review (major.md)**:
- `major.md` lines 88–98: DevOps reads `01-spec.md` and `01-spec-arch-review.md` and writes `01-spec-infra-review.md`. Phase 1.7 SA review runs before this (lines 58–79). No cross-review between them currently exists.

### What is NOT Present (Coverage Gaps = What This Spec Adds)

1. **Phase 4 cross-review in patch.md**: After step 4 (QA implementation) and before step 5 (read both outputs), there is no cross-review step. Steps 5–7 read and present the outputs without any structured peer reflection.

2. **Phase 4 cross-review in minor.md**: After step 4 (post-all-sub-PRs consolidation) and before step 5 (read both outputs), there is no cross-review step.

3. **Phase 4 cross-review in major.md**: After step 4 (post-all-sub-PRs consolidation) and before step 5 (present results), there is no cross-review step. In major workflows, SA and DevOps also need to cross-review each other's implementation outputs.

4. **Phase 5 QA final reflection in patch.md**: After the SE appends their test quality review to `05-quality-gate.md` (step 2), the workflow proceeds directly to step 3 (read the report) without giving QA a structured opportunity to reflect on the SE's findings.

5. **Phase 1.7/1.9 SA↔DevOps cross-review in major.md**: After Phase 1.7 (SA writes `01-spec-arch-review.md`) and Phase 1.9 (DevOps writes `01-spec-infra-review.md`), there is no cross-review between these two agents. The DevOps reads the SA's review, but the SA never reads the DevOps review.

---

## Test Patterns

### Validation Pattern for Markdown Workflow Files

Since there are no automated tests, the "test pattern" is a structured manual audit. The validation process for each criterion is:

**Pattern: Presence check**
- Verify that a new step appears in the correct location within the correct phase.
- Check: Is the step between the right two existing steps?

**Pattern: Content check**
- Verify that the prompt text uses the correct phrasing.
- Check for key phrases: "You are completing a cross-review", "Append a '## Cross-Review Notes' section", the correct file paths for reading and appending.

**Pattern: Structure check**
- Verify that the step uses the same markdown heading level as adjacent steps in the same section.
- Verify that step numbering is consistent and sequential after any inserted steps.
- Verify that parallel launch steps are expressed as lettered sub-items (a., b.) under a single numbered step — matching the established pattern in Phase 2 and Phase 3.

**Pattern: Sequential vs. parallel check**
- Verify that cross-review agents are launched in parallel (sub-items under one numbered step) versus sequentially (separate numbered steps).
- The Phase 2 and Phase 3 cross-reviews in all three files establish: parallel = one numbered step with lettered sub-items; sequential = separate numbered steps.

**Pattern: Conditionality check**
- Verify that steps that should only run when DevOps is activated are written with appropriate conditional language, mirroring how Phase 1.9 and DevOps participation in other phases is expressed.

**Pattern: Step numbering integrity check**
- Verify that all subsequent step numbers after an insertion point are correctly renumbered.
- Verify that no existing steps were removed, reordered, or altered.

**Pattern: Negative check (no existing step removed)**
- Compare the set of existing step headers and content against the modified file to confirm all pre-existing steps are intact.

### Example: Established Cross-Review Prompt Structure (from patch.md Phase 2)

```
3. **Cross-review round** - Launch two agents in parallel:

   a. **`software-engineer` in DISCOVERY mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's discovery document at `{SPEC_FOLDER}/02-discovery-qa.md`. Then read your own discovery document at `{SPEC_FOLDER}/02-discovery-se.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-se.md` with your observations about the QA findings, any alignment or conflicts, and suggestions for the QA Engineer."

   b. **`qa-engineer` in DISCOVERY mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the Software Engineer's discovery document at `{SPEC_FOLDER}/02-discovery-se.md`. Then read your own discovery document at `{SPEC_FOLDER}/02-discovery-qa.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-qa.md` with your observations about the SE findings, any alignment or conflicts, and suggestions for the Software Engineer."
```

Key structural elements:
- Step label: `**Cross-review round** - Launch [N] agents in parallel:` (or similar)
- Agent label: `**\`agent-name\`` in MODE (cross-review)**:`
- Prompt opener: `"You are completing a cross-review.`
- Reads first: specifies which other agent's document to read first
- Then reads own: specifies own document to read second
- Append instruction: `Append a '## Cross-Review Notes' section to your document at [path]`
- Content description: what observations to make

This is the template the SE must follow when writing the new Phase 4 cross-review prompts.

### Example: Established Parallel vs. Sequential Pattern

**Parallel** (Phase 2, Step 3 in patch.md — both agents launch together):
```
3. **Cross-review round** - Launch two agents in parallel:
   a. **`software-engineer`...
   b. **`qa-engineer`...
```

**Sequential** (Phase 5, Steps 1–2 in patch.md — QA runs first, then SE):
```
1. **Launch `qa-engineer` in QUALITY GATE mode**:
   ...
2. **After QA completes, launch `software-engineer` in QUALITY GATE mode**:
   ...
```

The Phase 5 QA final reflection (Gap 2) must be sequential — QA must run after SE because QA needs to read the SE's appended section. This is a critical distinction that the SE must express correctly using `After SE completes...` language.

---

## Key Files

1. `workflows/patch.md` — Primary file under validation for AC-1, AC-4, AC-5, AC-6, AC-9, AC-10. Changes are additive: one new Phase 4 cross-review step (between current steps 4 and 5), one new Phase 5 QA reflection step (after current step 2, causing renumbering of steps 3–7).

2. `workflows/minor.md` — Primary file for AC-2, AC-4, AC-5, AC-9, AC-10. Changes are additive: one new Phase 4 cross-review step (between current steps 4 and 5, causing renumbering of steps 5–7 to 6–8).

3. `workflows/major.md` — Primary file for AC-3, AC-4, AC-5, AC-7, AC-8, AC-9, AC-10. Most complex file. Changes are additive: one new Phase 4 cross-review step (between current steps 4 and 5, causing renumbering of steps 5–6 to 6–7); one new Phase 1.9 SA↔DevOps cross-review step (at the end of Phase 1.9, after the existing DevOps launch step).

4. `agents/software-engineer.md` — Reference for the SE's `## Cross-Review Notes` output template (lines 102–108 for discovery, lines 184–189 for planning). Confirms the section heading `## Cross-Review Notes` is the established output pattern. No changes needed to this file.

5. `agents/qa-engineer.md` — Reference for the QA's `## Cross-Review Notes` output template (lines 115–121 for discovery, lines 208–214 for planning). Confirms the section heading. No changes needed to this file.

6. `agents/software-architect.md` — Reference for how SA cross-review notes are structured in the discovery output template (lines 276–288). Relevant to AC-7. No changes needed.

7. `agents/devops-engineer.md` — Reference for how DevOps cross-review notes are structured (lines 249–262). Relevant to AC-7. No changes needed.

---

## Validation Approach by Acceptance Criterion

### AC-1: Phase 4 cross-review exists in patch.md
- **Validation method**: File-read
- **What to check**: After the QA IMPLEMENTATION launch step (current step 4) and before the "Read both output documents" step (current step 5), a new parallel cross-review step must exist. Verify SE reads `04-test-report.md` and appends to `04-implementation-summary.md`; QA reads `04-implementation-summary.md` and appends to `04-test-report.md`.

### AC-2: Phase 4 cross-review exists in minor.md
- **Validation method**: File-read
- **What to check**: After the "After all sub-PRs are implemented" consolidation step (current step 4) and before the "Read both output documents" step (current step 5), a new parallel cross-review step must exist. Same SE↔QA pattern as AC-1.

### AC-3: Phase 4 cross-review exists in major.md
- **Validation method**: File-read
- **What to check**: After the "After all sub-PRs are implemented" consolidation step (current step 4) and before the "Present results" step (current step 5), a new parallel cross-review step must exist. Must include SE↔QA cross-review plus conditional SA↔DevOps cross-review (only when DevOps is active).

### AC-4: Phase 4 cross-review prompts match the Discovery/Planning pattern
- **Validation method**: File-read (all three files)
- **What to check**: Each new cross-review prompt for SE must contain: "You are completing a cross-review", reads `04-test-report.md` first, appends to `04-implementation-summary.md`, specifies section heading `## Cross-Review Notes`. Each QA prompt: reads `04-implementation-summary.md` first, appends to `04-test-report.md`.

### AC-5: Phase 4 cross-review prompts follow parallel launch pattern
- **Validation method**: File-read (all three files)
- **What to check**: The cross-review step must be expressed as a single numbered step with lettered sub-items (a., b., and optionally c., d.) — not as separate sequential numbered steps. Compare structural format to Phase 2 Step 3 in each file.

### AC-6: Phase 5 patch QA final reflection step is added
- **Validation method**: File-read (patch.md only)
- **What to check**: After current Step 2 (SE test quality review launch), a new sequential step must exist that launches QA to read the full `05-quality-gate.md` and append a `## QA Engineer - Final Reflection` section. The step must use sequential language (not parallel). Steps after it must be renumbered correctly.

### AC-7: Phase 1.7/1.9 SA↔DevOps cross-review exists in major.md (when DevOps is active)
- **Validation method**: File-read (major.md only)
- **What to check**: At the end of Phase 1.9, after the DevOps SPEC REVIEW launch step, a new parallel cross-review step must exist (conditional on DevOps being active). SA must read `01-spec-infra-review.md` and append `## Cross-Review Notes` to `01-spec-arch-review.md`; DevOps must read the updated `01-spec-arch-review.md` and append `## Cross-Review Notes` to `01-spec-infra-review.md`.

### AC-8: Phase 1.7/1.9 cross-review is skipped when DevOps is not active
- **Validation method**: File-read (major.md only)
- **What to check**: The Phase 1.9 section must retain or include explicit conditional language indicating the cross-review only runs when DevOps is activated. This may be expressed as wrapping the new step in `(only if activated)` language or placing it under the existing `### Steps (only if activated):` section of Phase 1.9.

### AC-9: Workflow document structure remains consistent
- **Validation method**: File-read (all three files)
- **What to check**: New cross-review steps use `### Steps:` format, `**[Number]. **[Step label]**` heading style, and the same prompt wrapper syntax (`- Prompt: "..."`) as adjacent steps. Heading levels (`##` for phases, no sub-heading needed for steps) must match surrounding content.

### AC-10: No existing workflow steps are removed or reordered
- **Validation method**: File-read (all three files, diff-style comparison)
- **What to check**: Every step that existed before the change must still exist after the change, in the same relative order. The only allowed changes are: (a) insertion of new steps, (b) renumbering of step numbers after insertion points, (c) no content changes to existing steps.

---

## Risks and Edge Cases to Watch During Validation

1. **Step numbering after insertion in patch.md Phase 5**: Inserting a QA reflection step after current step 2 means steps 3, 4, and 5 must be renumbered to 4, 5, and 6. Steps 6 and 7 (if present — checking whether patch.md Phase 5 has more steps) or the "Handle the verdict" block must also be renumbered. Risk: SE renumbers only some steps, leaving inconsistencies.

2. **Step numbering after insertion in patch.md Phase 4**: Inserting a cross-review step after current step 4 means current steps 5, 6, and 7 become 6, 7, and 8. Risk: partial renumbering.

3. **Step numbering after insertion in minor.md Phase 4**: Same risk — current steps 5, 6, and 7 become 6, 7, and 8.

4. **Step numbering after insertion in major.md Phase 4**: Current steps 5 and 6 become 6 and 7. Lower risk due to fewer steps.

5. **SA reads updated vs. original SA document in Phase 1.7/1.9 cross-review**: The spec says DevOps reads "the updated `01-spec-arch-review.md`" — meaning after SA has appended their `## Cross-Review Notes`. This implies a sequential sub-step within the cross-review, not a fully parallel launch. The prompt wording must reflect this ordering correctly. Risk: SE writes both as parallel when DevOps needs to read SA's cross-review notes.

6. **Conditionality expression for major.md Phase 4 SA↔DevOps cross-review**: The SA/DevOps sub-items must be marked as conditional on DevOps being active, using language consistent with how other conditional DevOps participation is expressed (e.g., `(only if activated)` or `(if DevOps is active)`). Risk: the conditional language is omitted or inconsistent.

7. **Phase 5 QA reflection must be sequential, not parallel**: The spec is explicit that this step must run after the SE completes. If the SE mistakenly writes it as a parallel launch alongside the SE test quality review step, the step would be functionally broken — QA cannot read SE's section before SE writes it. Risk: SE expresses this as parallel.

8. **Document path consistency**: New prompts must reference the exact same file path patterns as existing prompts (e.g., `{SPEC_FOLDER}/04-test-report.md` not `{SPEC_FOLDER}/test-report.md`). Risk: typos or inconsistent path formatting.

9. **Phase 1.9 placement**: The Phase 1.7/1.9 cross-review belongs at the end of Phase 1.9, inside the `### Steps (only if activated):` section — not after Phase 1.9 as a separate section and not inside Phase 1.7. Risk: incorrect placement.

10. **Scope creep**: The spec explicitly excludes changes to agent files. If any agent file is modified (even trivially), that is a scope violation.

---

## Cross-Review Notes

### Overall Alignment

The SE's discovery and this QA discovery are in strong agreement on all fundamental points: the same 5 gaps are identified, the same 3 workflow files are the only files that need to change, no agent files are modified, and the same "no automated test infrastructure exists" reality is understood. There are no conflicts on scope, affected files, or the nature of the validation work required.

### SE Findings That Strengthen This Document

1. **Exact insertion line numbers**: The SE's discovery provides precise before/after line numbers for every insertion point (patch.md lines 128/129, 151/153; minor.md lines 224/226; major.md lines 96/98 and 278/280). This QA document references insertion points descriptively (e.g., "between step 4 and step 5"). The line numbers from the SE document will be the authoritative reference during implementation validation — particularly for regression Risk 2 (partial renumbering), which is easier to catch when exact line positions are known.

2. **Dash vs. em-dash formatting convention**: The SE correctly identifies that patch.md uses a hyphen-dash (`-`) in step labels while minor.md and major.md use an em-dash (`—`). This QA discovery did not call this out explicitly. It is a concrete formatting check that should be added to the AC-9 validation (workflow document structure consistency). The SE's discovery makes this criterion more specific and checkable.

3. **Agent label mode annotation**: The SE clarifies that the agent label in a parallel cross-review step uses the original working mode (e.g., `IMPLEMENTATION mode (cross-review)`) — not a distinct "CROSS-REVIEW mode". This matches existing Phase 2 and Phase 3 patterns. My QA discovery example shows this correctly but does not call it out as an explicit rule. This is worth verifying explicitly in AC-4 and AC-5.

4. **Phase 5 step count in patch.md**: The SE confirms that Phase 5 of patch.md has exactly 5 numbered steps (steps 3, 4, 5 become 4, 5, 6 after the insertion). The SE's risk analysis in item 2 ("patch.md Phase 5 — no other internal step-number references were found") confirms the loop-back reference in the "Handle the verdict" step uses phase names, not step numbers. This resolves my Risk 1 (step numbering after insertion in Phase 5) — renumbering steps 3–5 to 4–6 is sufficient and no cross-references will break.

5. **Track activation status paragraph is not a step**: The SE explicitly identifies that the `**Track activation status**` paragraph in major.md Phase 1.9 is a standalone prose block, not a numbered step, and must not be renumbered. This is not called out in my QA discovery. This is a concrete negative check to add during AC-7/AC-8 validation.

### One Substantive Divergence: Phase 1.9 SA↔DevOps Parallel vs. Sequential

My Risk 5 flags a potential ordering issue: the spec says DevOps reads "the updated `01-spec-arch-review.md`" — implying DevOps should read the SA's cross-review notes, which means the SA must write first. This would make the Phase 1.9 cross-review sequential (SA first, then DevOps), not parallel.

The SE's Gap 4 description plans both SA and DevOps as a parallel launch ("cross-review round, parallel SA+DevOps"). The SE does not address this ordering ambiguity.

This divergence needs to be resolved before the SE writes the implementation. The recommended approach is to re-read the spec's exact wording for AC-7 to determine whether "the updated `01-spec-arch-review.md`" is prescriptive (implying sequential) or whether a simpler interpretation is intended (SA reads DevOps's infra review, DevOps reads SA's arch review, both launched in parallel with each reading the version produced during Phase 1.7/1.9, not each other's cross-review notes). The Phase 2/3 cross-reviews are always parallel — each agent reads the other's original document, not the other's cross-review notes — which suggests parallel is the correct pattern for Phase 1.9 as well. If the spec indeed uses "updated" loosely, the SE's parallel interpretation is correct and my Risk 5 is a false concern. The SE should confirm this with the spec text before writing the Phase 1.9 insertion.

### Suggestions for the Software Engineer

1. **Resolve the Phase 1.9 ordering question** before writing that insertion. Re-read the spec's AC-7 wording. If "updated" means "the document as it exists after Phase 1.9 completes" (i.e., after DevOps has written it — not after SA appends cross-review notes), then parallel is correct and both agents are reading the other's primary deliverable. This is the most likely intent given every other cross-review in the codebase is parallel.

2. **Add an explicit wait step for major.md Phase 4** if the SA↔DevOps cross-review is included in the same numbered step as SE↔QA. The SE's existing step format for parallel + conditional agents (e.g., Phase 2 in major.md) uses a wait step after the parallel launch. Verify whether a wait step is present or needs to be added at the new insertion point.

3. **Verify Phase 5 step count carefully**: The SE's discovery states existing steps 3, 4, 5 in Phase 5 become 4, 5, 6. However, the QA discovery notes patch.md is 257 lines. Confirm whether Phase 5 currently has 5 steps (1–5) or 7 steps (1–7, matching Phase 4's structure). The step count after renumbering depends on whether the current maximum step number is 5 or 7. Both QA and SE documents agree the final "Handle the verdict" step uses phase-name references only, so renumbering is safe regardless — but the final step number must be correct.

4. **Confirm major.md Phase 4 conditional agent labeling pattern**: When writing the SA and DevOps sub-items in the major.md Phase 4 cross-review step, use the exact conditional language pattern already established in that file (e.g., `(only if DevOps is active)` or `(only if activated)`) — not ad-hoc language. The SE's discovery correctly identifies this risk (item 7) and identifies the correct pattern. Cross-checking against an existing conditional agent sub-item in major.md Phase 2 or Phase 3 will confirm the exact phrasing to replicate.
