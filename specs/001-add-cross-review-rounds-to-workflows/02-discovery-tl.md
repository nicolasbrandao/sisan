# Tech Lead Discovery Review

## Spec Reference
- **Spec**: `specs/001-add-cross-review-rounds-to-workflows/01-spec.md`
- **SE Discovery**: `specs/001-add-cross-review-rounds-to-workflows/02-discovery-se.md`
- **QA Discovery**: `specs/001-add-cross-review-rounds-to-workflows/02-discovery-qa.md`
- **UI/UX Discovery**: N/A

---

## Architectural Context

### Module Boundaries

The repository has a flat, two-directory structure with a clean ownership model:

- `workflows/` — Orchestration instructions. Each file (`patch.md`, `minor.md`, `major.md`) is a standalone, self-contained workflow definition consumed by the Claude orchestrator at runtime. There are no cross-file dependencies within this directory: `patch.md` does not import or reference `minor.md` or `major.md`, and vice versa. Each file owns its own phase sequence, step numbering, and prompt text.
- `agents/` — Agent persona definitions. Each file defines an agent's modes, output templates, and behavioral constraints. These files are referenced by name from workflow files (e.g., "Launch `software-engineer`"), but there is no runtime linkage — the orchestrator resolves agent names at runtime.
- `specs/` — Audit trail documents produced during workflow execution. Not part of the plugin source; produced as output.
- `.claude/commands/` — The `/sisan` command entry point. Phase 0 (intake and triage) lives here. No changes needed.

**Boundary crossed by this change**: NO. All five insertions stay within `workflows/`. No agent files are touched. The change is entirely within the orchestration layer. There is no shared abstraction being modified — each workflow file is modified independently.

### Dependency Graph

The workflow files are terminal consumers: nothing in the repository depends on them being a particular length or having a particular step count. The only "consumer" is the live Claude orchestrator instance that reads and executes them.

- **Upstream** (depends on this code): The Claude orchestrator at runtime; the human operator reading the workflow.
- **Downstream** (this code depends on): Agent files by name (soft reference only — no import mechanism), and the `{SPEC_FOLDER}` document paths referenced in prompts.
- **Shared abstractions**: The `{SPEC_FOLDER}` path variable and document naming conventions (`04-implementation-summary.md`, `04-test-report.md`, etc.) are shared conventions across all three workflow files. None of these are changed by this spec.

### Abstraction Layers

The relevant layers within the orchestration system:

- **Workflow layer** (`workflows/*.md`): Defines phase structure, step sequencing, parallelism, and prompt text. This is where all changes land.
- **Agent layer** (`agents/*.md`): Defines agent output templates and behavioral rules. Not modified.
- **Command layer** (`.claude/commands/sisan.md`): Entry point and Phase 0 triage. Not modified.

**Layer violations**: None. The spec is correctly scoped to the workflow layer only. The SE and QA discovery documents both confirm that agent files already contain `## Cross-Review Notes` template sections and require no modification — the cross-review behavior is driven by the workflow's prompt instructions, not by agent persona definitions.

### Design Patterns in Use

- **Parallel launch pattern**: A single numbered step introduces multiple lettered sub-items (`a.`, `b.`, `c.`) that the orchestrator launches simultaneously. Used in Phase 2 and Phase 3 across all three files. This is the established pattern for cross-review rounds.
- **Sequential step pattern**: A numbered step with "After [agent] completes, launch..." language, or a standalone numbered step after an explicit "Wait" step. Used for the TL review (Phase 2 step 5), the SA and DevOps plan reviews (Phase 3 cross-review round), and the Phase 5 SE test review in `patch.md`. The QA final reflection (Gap 2) must follow this pattern.
- **Conditional activation pattern**: `(only if activated)` appended to an agent sub-item label, with a preceding activation check block (`### Activation Check: ... ### Steps (only if activated):`). Used for UI/UX and DevOps throughout all workflow files.
- **Additive append pattern**: Cross-review agents are never given their own output file; they append a `## Cross-Review Notes` section to their own primary document. This keeps document count stable and preserves atomic authorship (one agent per document).

---

## Architectural Concerns

### 1. The Phase 1.9 SA↔DevOps ordering question is architecturally significant and must be resolved before implementation

Both the SE and QA discovery documents flag this question, and it is the most important architectural decision in this spec.

The spec's AC-7 text states: "DevOps reads the updated `01-spec-arch-review.md`". The word "updated" is ambiguous. It could mean (a) the document as it exists at the point the cross-review step runs — i.e., after SA has appended their cross-review notes — which implies sequential ordering, or (b) the document as updated during Phase 1.7 (the SA's primary deliverable), which implies parallel ordering where both agents read each other's original documents.

I have examined every existing cross-review round across all three workflow files:

- `patch.md` Phase 2 step 3: parallel — SE reads QA's doc (no cross-review notes on it yet), QA reads SE's doc (no cross-review notes on it yet).
- `patch.md` Phase 3 step 3: parallel — same pattern.
- `minor.md` Phase 2 step 3: parallel — SE reads QA's doc, QA reads SE's doc, UI/UX reads SE's doc. None read the other's cross-review notes.
- `minor.md` Phase 3 step 3: parallel — same pattern.
- `major.md` Phase 2 step 3: parallel — SA reads SE's doc and DevOps's doc simultaneously with DevOps reading SE's doc and SA's doc. Critically: SA and DevOps are launched at the same time, each reading the other's primary discovery document. Neither reads the other's cross-review notes (which do not exist yet at that point).
- `major.md` Phase 3 step 3: parallel — same structure.

The universal pattern across all six existing cross-review rounds is: each agent reads the other's primary deliverable, not the other's cross-review notes. The "updated" word in AC-7 is best read as "the document updated during Phase 1.7" (SA's primary deliverable), not "after SA appends cross-review notes to it".

**Recommendation: Use fully parallel ordering (Option A).** The reasoning:

1. **Consistency with all existing cross-review patterns**: Every prior cross-review in all three files is parallel. Each agent reads the other's primary output document. Introducing a sequential exception here creates a two-tier cross-review model that adds cognitive overhead for future maintainers without proportional benefit.

2. **What the agents actually need**: The SA's most valuable architecture decisions are already captured in `01-spec-arch-review.md` (Phase 1.7 output). The DevOps's most valuable infrastructure feasibility findings are in `01-spec-infra-review.md` (Phase 1.9 output). Each agent reading the other's primary document is sufficient to surface architectural-versus-infra conflicts. The SA's cross-review notes on the infra document would primarily just restate the same architectural position — the marginal value of DevOps reading that over reading `01-spec-arch-review.md` directly is low.

3. **Symmetry of the cross-review**: In a parallel cross-review, both agents start from the same informational baseline. SA reads the DevOps's fresh perspective on infrastructure; DevOps reads the SA's fresh architecture decisions. In a sequential model, the second agent (DevOps) would read the first agent's (SA's) preliminary reaction, which may be lower quality than the primary document itself.

4. **Latency**: Parallel is faster. This matters in a workflow already structured around keeping phases crisp.

5. **The spec's AC-7 wording does not actually require sequential**: Re-reading the full expected behavior section (Gap 3), it says "SA reads the DevOps infrastructure spec review (`01-spec-infra-review.md`) and appends... DevOps reads the SA's cross-review notes". Only that second sentence, read in isolation, implies sequential ordering. But this is in the "Expected Behavior" narrative, not a normative constraint. The actual AC-7 acceptance criterion says only: "SA reads `01-spec-infra-review.md` and appends `## Cross-Review Notes` to `01-spec-arch-review.md`; DevOps reads the updated `01-spec-arch-review.md` and appends `## Cross-Review Notes` to `01-spec-infra-review.md`." Given the universal parallel pattern in the codebase, "the updated `01-spec-arch-review.md`" means the version produced in Phase 1.7 — the same document DevOps already reads during Phase 1.9 step 1 (it is explicitly in the DevOps SPEC REVIEW prompt: "Read the specification... and the architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`"). DevOps is already reading `01-spec-arch-review.md` as its primary input. The cross-review simply asks it to read it again with a cross-review lens and append notes to `01-spec-infra-review.md`. Parallel is coherent.

**If sequential is chosen instead**: The SE must use "After SA completes their cross-review notes, launch `devops-engineer`..." language (not parallel sub-items), and the QA validation criterion for AC-7 must check for sequential structure rather than parallel sub-items. This is a valid but non-preferred approach.

### 2. The Phase 4 major.md SA↔DevOps cross-review scope is slightly under-specified in the spec

The spec's AC-3 and Gap 1 describe the Phase 4 major.md cross-review as including "SA reads `04-infra-summary.md` and DevOps reads relevant SA context, each appending cross-review notes to their respective documents". The phrase "relevant SA context" is vague — the SA does not have a `04-*` document at Phase 4. The SA's Phase 4 role in major.md is to have written API contracts in `03-plan-sa.md` and to produce a system integration review in Phase 5. There is no `04-implementation-summary-sa.md`.

This is not a blocking concern, but the SE will need to decide what "relevant SA context" means when writing the DevOps cross-review prompt for Phase 4. The most defensible interpretation: DevOps reads `04-implementation-summary.md` (SE's implementation output) and appends cross-review notes to `04-infra-summary.md`, commenting on whether the application implementation has any infra implications. The SA, reading `04-infra-summary.md`, appends cross-review notes commenting on whether the infra implementation aligns with the architecture decisions. This is consistent with what the Phase 5 quality gate already does for both agents.

**Impact**: The SE must make an explicit choice about what document the DevOps cross-review agent reads in Phase 4. This should be `04-implementation-summary.md` (not some SA-specific doc that doesn't exist), following the same structure as SE↔QA in the same step.

**Recommendation for planning**: The SE should model the SA and DevOps sub-items in the Phase 4 major.md cross-review step on the Phase 2 major.md cross-review (lines 135–142), where SA reads SE's discovery and DevOps reads SA's discovery. The Phase 4 analog: SA reads `04-infra-summary.md`, DevOps reads `04-implementation-summary.md`. Both append to their own respective documents.

### 3. Patch.md Phase 5 step count — a renumbering verification is warranted

The SE's discovery states Phase 5 of `patch.md` has steps 1–5, with steps 3–5 shifting to 4–6 after the QA reflection insertion. I can confirm from direct file inspection: `patch.md` Phase 5 has exactly 5 numbered steps (1: QA quality gate, 2: SE test review, 3: read report, 4: present results, 5: handle verdict). After inserting the QA reflection as new step 3, the final step ("Handle the verdict") becomes step 6. The loop-back inside that step references "Phase 4" by name ("loop back to Phase 4"), not a step number — safe. This is consistent with what both the SE and QA discovery documents report.

**Impact**: None — the analysis is correct. This is a verification point, not a concern.

---

## Scale & Performance Considerations

Not applicable to this change. The workflow files are markdown documents consumed once per workflow run by a single orchestrator instance. There are no performance-critical paths, no data volume concerns, and no concurrency implications. Adding ~60–90 lines of instruction text has no measurable effect on runtime or agent behavior beyond the intended functionality.

---

## Scope Assessment

- **Tier confirmation**: CONFIRMED as minor. The change touches 3 files, adds 5 insertion points, and involves approximately 60–90 lines of new text. All changes are purely additive. No existing behavior is modified, removed, or reordered. This is well within the minor workflow scope.
- **Decomposition**: The 2-PR decomposition in the spec is appropriate and well-motivated. PR 1 (patch.md Phase 4 + minor.md Phase 4) establishes the simple SE↔QA pattern without any conditional complexity. PR 2 (major.md Phase 4, patch.md Phase 5, major.md Phase 1.9) handles the more nuanced cases. A reviewer seeing PR 2 will already understand the base pattern from PR 1. This is a sound sequencing strategy.
- **Rationale**: The work is structurally uniform (markdown insertions following an established pattern), the scope is bounded to three files with zero inter-file dependencies, and the 2-PR split is meaningful rather than arbitrary.

---

## Recommendations for Planning Phase

1. **Resolve Phase 1.9 SA↔DevOps ordering as parallel.** The SE should write both agents as lettered sub-items under a single numbered step (e.g., `3. **Cross-review round** — Launch two agents in parallel:`), with sub-items `a.` for SA and `b.` for DevOps (both marked `(only if activated)` since the entire step is inside the `### Steps (only if activated):` block). The SA sub-item should read `01-spec-infra-review.md` and append to `01-spec-arch-review.md`; the DevOps sub-item should read `01-spec-arch-review.md` (the Phase 1.7 primary deliverable) and append to `01-spec-infra-review.md`. This is the interpretation most consistent with every other cross-review in the codebase.

2. **Explicitly define what document the DevOps agent reads in the Phase 4 major.md cross-review.** The SE should read `04-implementation-summary.md` (the SE's consolidated implementation output) and append to `04-implementation-summary.md`. The SA should read `04-infra-summary.md` and append to `01-spec-arch-review.md` — or, if the SA's cross-review appends to a new `## Cross-Review Notes` section within `04-implementation-summary.md` as the spec's AC-3 loosely implies, that must be made explicit. The cleanest interpretation: SA reads `04-infra-summary.md` and appends to `04-implementation-summary.md`; DevOps reads `04-implementation-summary.md` and appends to `04-infra-summary.md`. This mirrors the SE↔QA pattern exactly and does not require any new document names.

3. **Use the em-dash (not hyphen-dash) in `minor.md` and `major.md` step labels.** The SE's discovery documents this correctly (patch.md uses `-`, minor and major use `—`). This is a formatting convention that affects the QA AC-9 check and should be treated as a hard constraint, not a stylistic preference.

4. **Confirm the `(only if activated)` label placement for major.md Phase 4.** When the Phase 4 cross-review step includes SA and DevOps sub-items, those sub-items must be marked `(only if DevOps is active)` or `(only if activated)` — the exact phrasing used in the rest of `major.md` for conditional DevOps participation. Check `major.md` Phase 2 step 3e (line 141) for the exact conditional label pattern: `**`devops-engineer` cross-review** (only if activated):`. Replicate that label format.

5. **Do not add a wait step before the Phase 4 cross-review step in patch.md.** Unlike `minor.md` and `major.md` where an explicit "Wait for all agents to complete" step precedes the cross-review round, `patch.md` Phase 4 uses "After SE completes, launch qa-engineer" sequential phrasing (no explicit wait step). The new cross-review step in patch.md should follow the existing flow: it runs after QA completes step 4, so the natural language "After both SE and QA complete, launch cross-review" or just a numbered parallel step following step 4 is appropriate. Do not add a standalone "Wait for both agents to complete" step if no such step exists in the current Phase 4 of patch.md — that would change the formatting style of the section.
