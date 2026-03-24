# Add Cross-Review Rounds to Workflow Phases

## Metadata
- **Type**: small-feature
- **Workflow Tier**: minor
- **Priority**: medium
- **Date**: 2026-03-23

## Summary
Three workflow files (`patch.md`, `minor.md`, `major.md`) are missing cross-review rounds at key phases, leaving gaps where agents complete their work in isolation without peer validation. This spec defines exactly where those gaps are and what cross-review content should be added to close them.

## Background / Context
The sisan multi-agent workflow system is built around the principle that agents improve their outputs when they read each other's work. Cross-review rounds already exist in Phase 2 (Discovery) and Phase 3 (Planning) across all tiers, demonstrating the pattern. However, three gaps were identified through triage analysis:

1. **Phase 4 (Implementation) has no cross-review in any tier.** After SE implements code and QA implements tests, neither agent reads the other's output before the Quality Gate phase. This means implementation misalignments (e.g., SE deviates from plan in a way that breaks QA's test setup) are not caught until Phase 5.

2. **Phase 5 patch workflow — one-directional SE→QA review only.** The patch Quality Gate has SE reviewing QA's tests, but QA does not similarly reflect on the SE's implementation quality in a structured cross-review pass. The SE review is already present but is a separate agent launch rather than a true peer cross-review.

3. **Phase 1.7 major workflow — SA reviews alone.** The Software Architect writes their architecture spec review with no cross-review from the DevOps Engineer (who runs in Phase 1.9). When there are infrastructure-affecting changes, the SA's architecture decisions and the DevOps feasibility assessment are developed in isolation. The SA may specify an integration pattern that creates deployment complexity the DevOps Engineer would flag if consulted earlier.

Note: The Phase 5 patch workflow gap (#2 above) is already partially present as a sequential SE launch after QA. The true gap is the absence of a QA→SE reflection at that stage. For consistency with Discovery and Planning, both agents should have a structured cross-review section. However, since Phase 5 patch already has the SE review (it is just one-directional), the primary implementation focus should be the Phase 4 gap and the Phase 1.7 SA/DevOps gap.

## Current Behavior

### patch.md

**Phase 4 (Implementation)** (`workflows/patch.md`, lines 109–138):
- Step 3 launches SE in IMPLEMENTATION mode.
- Step 4 launches QA in IMPLEMENTATION mode, referencing the SE's implementation summary.
- No cross-review step exists after both agents complete. Neither agent reads the other's output for a structured reflection.

**Phase 5 (Quality Gates)** (`workflows/patch.md`, lines 141–176):
- Step 1 launches QA in QUALITY GATE mode.
- Step 2 launches SE in QUALITY GATE mode (test review).
- The SE reviews QA's tests and appends to the quality gate report. QA does not have a structured opportunity to review the SE's final implementation before completing the quality gate report (QA reads `04-implementation-summary.md` but does not append cross-review notes to the SE's document).

### minor.md

**Phase 4 (Implementation)** (`workflows/minor.md`, lines 183–234):
- For each sub-PR: SE launches, then QA launches (or parallel if safe).
- No cross-review step exists after all sub-PRs are implemented. The consolidated output documents are read but no structured peer review occurs.

### major.md

**Phase 1.7 (Architecture Spec Review)** (`workflows/major.md`, lines 58–79):
- SA launches, writes `01-spec-arch-review.md`.
- Phase 1.9 (DevOps Spec Review, conditional) launches after Phase 1.7.
- The DevOps Spec Review reads the SA's review (`01-spec-arch-review.md`) but the SA does not read the DevOps review. The SA's architecture decisions are finalized before the DevOps Engineer provides infrastructure feasibility input, and there is no loop back.

**Phase 4 (Implementation)** (`workflows/major.md`, lines 238–283):
- Per-phase, per-sub-PR: SE launches, then DevOps launches (if active), then QA launches.
- No cross-review step exists after all phases are implemented.

## Expected Behavior

After this change is implemented:

### Gap 1 — Phase 4 cross-review (all tiers)

After SE and QA both complete their implementation work (all sub-PRs if multi-PR), a cross-review round is inserted before the user checkpoint. Each agent reads the other's output document and appends a `## Cross-Review Notes` section, mirroring the Discovery and Planning cross-review patterns:

- SE reads `04-test-report.md` and appends cross-review notes to `04-implementation-summary.md`, commenting on whether the tests align with what was implemented, any interface assumptions in the tests that differ from reality, and any concerns.
- QA reads `04-implementation-summary.md` and appends cross-review notes to `04-test-report.md`, commenting on whether the implementation matches what the tests assume, any deviations from plan that affect test correctness, and any concerns.

In major workflows, the SA and DevOps Engineer (if active) also participate: SA reads DevOps's infra summary and appends cross-review notes to `04-implementation-summary.md` (or a separate SA cross-review section); DevOps reads the SA's implementation context and appends to `04-infra-summary.md`.

### Gap 2 — Phase 5 patch QA reflection (patch.md only)

The existing SE→QA review in patch Phase 5 is already present. The gap is that QA writes the base quality gate report but does not have a dedicated cross-review reflection step on the SE's actual test quality assessment. Since the SE appends to the same `05-quality-gate.md` document (sequential), QA could in principle read it — but the workflow does not instruct QA to do so. A final QA reflection step should be added after the SE completes their test quality review, so QA can append a `## QA Engineer - Final Reflection` section to the quality gate document, acknowledging the SE's test quality findings and updating their verdict if warranted.

### Gap 3 — Phase 1.7/1.9 SA↔DevOps cross-review (major.md only)

After both Phase 1.7 (SA) and Phase 1.9 (DevOps) complete, a cross-review step is added:
- SA reads the DevOps infrastructure spec review (`01-spec-infra-review.md`) and appends a `## Cross-Review Notes` section to their architecture spec review (`01-spec-arch-review.md`), commenting on whether the SA's architecture decisions are feasible from an infrastructure perspective.
- DevOps reads the SA's cross-review notes (already has access to `01-spec-arch-review.md`) and appends a `## Cross-Review Notes` section to their infra spec review (`01-spec-infra-review.md`), commenting on infrastructure feasibility gaps or confirmations in the SA's architecture.

This cross-review only runs when DevOps is activated (i.e., the `Affected Areas > Infrastructure` section in the spec is non-empty).

## Acceptance Criteria

1. **Phase 4 cross-review exists in patch.md**
   - Given: SE and QA have both completed their implementation steps in Phase 4
   - When: the orchestrator reaches the post-implementation checkpoint
   - Then: the workflow instructs a parallel cross-review round where SE reads `04-test-report.md` and appends cross-review notes to `04-implementation-summary.md`, and QA reads `04-implementation-summary.md` and appends cross-review notes to `04-test-report.md`, before presenting results to the user

2. **Phase 4 cross-review exists in minor.md**
   - Given: all sub-PRs have been implemented (SE and QA completed for each sub-PR)
   - When: the orchestrator reaches the consolidated post-implementation step (step 4 of Phase 4)
   - Then: the workflow instructs a parallel cross-review round where SE reads `04-test-report.md` and appends cross-review notes to `04-implementation-summary.md`, and QA reads `04-implementation-summary.md` and appends cross-review notes to `04-test-report.md`, before presenting results to the user

3. **Phase 4 cross-review exists in major.md**
   - Given: all sub-PRs across all phases have been implemented (SE, DevOps if active, and QA completed)
   - When: the orchestrator reaches the consolidated post-implementation step (step 4 of Phase 4)
   - Then: the workflow instructs a parallel cross-review round where SE reads `04-test-report.md` and appends cross-review notes to `04-implementation-summary.md`, QA reads `04-implementation-summary.md` and appends cross-review notes to `04-test-report.md`, and (if DevOps is active) SA reads `04-infra-summary.md` and DevOps reads relevant SA context, each appending cross-review notes to their respective documents

4. **Phase 4 cross-review prompts match the Discovery/Planning pattern**
   - Given: any workflow file (patch, minor, or major)
   - When: the Phase 4 cross-review prompt for SE is read
   - Then: the prompt follows the established pattern — instructs the agent to identify themselves as "completing a cross-review", specifies which document to read first, specifies which document to append to, and specifies the section heading `## Cross-Review Notes`

5. **Phase 4 cross-review prompts follow parallel launch pattern**
   - Given: any workflow file (patch, minor, or major)
   - When: the Phase 4 cross-review step is executed
   - Then: SE and QA cross-review agents are launched in parallel (not sequentially), consistent with how cross-reviews are structured in Phase 2 and Phase 3

6. **Phase 5 patch QA final reflection step is added**
   - Given: QA has written the base quality gate report and SE has appended their test quality review to `05-quality-gate.md`
   - When: the SE's test quality review step completes
   - Then: the workflow instructs a final QA reflection step where QA reads the full `05-quality-gate.md` (including the SE's test quality review section) and appends a `## QA Engineer - Final Reflection` section, allowing QA to acknowledge SE's findings and update their overall verdict if warranted

7. **Phase 1.7/1.9 SA↔DevOps cross-review exists in major.md (when DevOps is active)**
   - Given: both Phase 1.7 (SA architecture spec review) and Phase 1.9 (DevOps infra spec review) have completed, and DevOps is active
   - When: the orchestrator reaches the end of Phase 1.9
   - Then: the workflow instructs a parallel cross-review round where SA reads `01-spec-infra-review.md` and appends `## Cross-Review Notes` to `01-spec-arch-review.md`, and DevOps reads the updated `01-spec-arch-review.md` and appends `## Cross-Review Notes` to `01-spec-infra-review.md`

8. **Phase 1.7/1.9 cross-review is skipped when DevOps is not active**
   - Given: Phase 1.7 (SA) has completed but Phase 1.9 (DevOps) was skipped because `Affected Areas > Infrastructure` was "None"
   - When: the orchestrator reaches the end of Phase 1.9
   - Then: the cross-review step does not run (there is no DevOps review document to cross-review against)

9. **Workflow document structure remains consistent**
   - Given: any of the three modified workflow files
   - When: the file is read
   - Then: the new cross-review steps follow the same formatting conventions as existing cross-review steps in the same file — same markdown heading level, same step numbering style, same agent prompt structure

10. **No existing workflow steps are removed or reordered**
    - Given: any of the three modified workflow files
    - When: the changes are applied
    - Then: all existing phase steps remain present and in their original order; new steps are additions only, not replacements

## Delivery Strategy
- **Total cycles**: 1
- **This spec covers**: Full delivery
- **PRs expected from this cycle**: 2
  - PR 1: Add Phase 4 cross-review to `patch.md` and `minor.md`
  - PR 2: Add Phase 4 cross-review to `major.md`, Phase 5 QA reflection to `patch.md`, and Phase 1.7/1.9 SA↔DevOps cross-review to `major.md`
- **Related specs**: N/A

PR 1 is self-contained and targets the two simplest, most structurally similar workflows. PR 2 extends the same Phase 4 pattern to the more complex major workflow and adds the two more nuanced improvements. PR 1 is a prerequisite for PR 2 only in the sense that reviewing PR 1 first gives a reviewer the established pattern to compare against when reviewing the major workflow changes in PR 2.

## Scope

### In Scope
- Adding a Phase 4 cross-review step to `workflows/patch.md` (SE↔QA)
- Adding a Phase 4 cross-review step to `workflows/minor.md` (SE↔QA)
- Adding a Phase 4 cross-review step to `workflows/major.md` (SE↔QA, plus SA↔DevOps when DevOps is active)
- Adding a Phase 5 QA final reflection step to `workflows/patch.md`
- Adding a Phase 1.7/1.9 SA↔DevOps cross-review step to `workflows/major.md` (conditional on DevOps activation)
- Agent prompt text for all new cross-review steps, following existing prompt patterns in the same files

### Out of Scope
- Changes to agent definition files (`agents/*.md`) — the cross-review behavior is orchestrated by the workflow, not defined in the agent files. Agent files already describe cross-review in their output templates (e.g., the SE's `## Cross-Review Notes` template section in `agents/software-engineer.md` lines 102–107). No agent file changes are needed.
- Adding cross-review to Phase 5 of the minor or major workflows — those phases already have multiple sequential reviewers (SE, TL, UI/UX, SA, DevOps) that read prior reports. Adding further cross-review there is a separate concern and would require careful sequencing analysis.
- Adding cross-review to Phase 6 (Merge) in any tier — that phase is administrative and does not benefit from agent peer review.
- Phase 3 cross-review in the major workflow between SA and DevOps — the SA already reviews the SE plan at Phase 3 and the DevOps already reviews the SE plan for deployment feasibility. The SA↔DevOps gap is best addressed at Phase 1 (spec review) and Phase 4 (post-implementation). Adding it to Phase 3 is out of scope for this cycle.
- Refactoring existing cross-review prompts or changing existing workflow behavior — this is additive only.

## Affected Areas
- **Files**:
  - `workflows/patch.md` — add Phase 4 cross-review step (between steps 4 and 5), add Phase 5 QA final reflection step (after step 2)
  - `workflows/minor.md` — add Phase 4 cross-review step (between steps 4 and 5)
  - `workflows/major.md` — add Phase 1.7/1.9 cross-review step (at end of Phase 1.9), add Phase 4 cross-review step (between steps 4 and 5)
- **APIs**: None
- **Database**: None
- **UI**: None — these are workflow instruction files consumed by the Claude orchestrator, not user-facing interfaces.
- **Infrastructure**: None
- **Cross-Service**: None — all changes are within a single repository and affect only markdown workflow definition files.

## Risks & Dependencies

- **Prompt consistency risk**: New cross-review prompts must closely match the style and structure of existing prompts in each file, or the orchestrator may interpret them differently from established cross-reviews. Mitigation: the SE should directly model new prompts on the nearest existing cross-review prompt in the same file, preserving exact phrasing patterns like "You are completing a cross-review", "Append a '## Cross-Review Notes' section", and the document path reference format.

- **Step numbering disruption**: Inserting new steps between existing numbered steps requires renumbering subsequent steps. If step numbers are referenced elsewhere in the same file (e.g., "See step 2" or "loop back to step N"), those references must be updated. Mitigation: review each affected section for internal step number references before inserting.

- **Phase 4 cross-review ordering in major.md**: The major Phase 4 is structured around sub-PR phases (Phase A → B → C → D), not a single post-implementation moment. The cross-review should only run once, after ALL sub-PRs across ALL phases are complete — not after each sub-PR phase. The new step belongs in the "After all sub-PRs are implemented" section (step 4 of Phase 4 in major.md), not inside the per-phase loop. Mitigation: placement must be after step 4 of Phase 4 in major.md, before step 5 (present results).

- **SA/DevOps cross-review conditionality**: The Phase 1.7/1.9 cross-review in major.md only runs when DevOps is activated. The conditional logic must be explicit in the workflow instructions. Mitigation: follow the same conditional activation pattern already used in Phase 1.9 ("only if activated").

- **QA Final Reflection ordering in patch.md Phase 5**: The QA final reflection must run after the SE appends their section to `05-quality-gate.md`. This is an inherently sequential step (QA must read the SE's section before reflecting). The workflow must not launch this step in parallel with the SE review. Mitigation: use explicit sequential language ("After SE completes their test quality review...").

## Open Questions
- None. The gaps are well-defined from triage analysis and the existing cross-review pattern in the codebase provides a clear template to follow.
