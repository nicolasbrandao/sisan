# Software Engineer Discovery

## Spec Reference
- **Spec**: `specs/001-add-cross-review-rounds-to-workflows/01-spec.md`
- **Title**: Add Cross-Review Rounds to Workflow Phases
- **Type**: small-feature

---

## Affected Code Paths

### Primary Path

- `workflows/patch.md:109–138` — Phase 4 (Implementation): SE launches at step 3, QA at step 4, then steps 5–7 read docs, present results, user checkpoint. No cross-review between steps 4 and 5.
- `workflows/patch.md:141–176` — Phase 5 (Quality Gates): QA at step 1, SE test review at step 2, then steps 3–5 read/present/handle verdict. No QA final reflection after step 2.
- `workflows/minor.md:182–234` — Phase 4 (Implementation): per-sub-PR loop at step 3 (a–f), then step 4 consolidates, step 5 reads docs, step 6 presents, step 7 user checkpoint. No cross-review between steps 4 and 5.
- `workflows/major.md:238–283` — Phase 4 (Implementation): per-phase/per-sub-PR loop at step 3 (a–g), then step 4 consolidates, step 5 presents, step 6 user checkpoint. No cross-review between steps 4 and 5.
- `workflows/major.md:82–99` — Phase 1.9 (DevOps Spec Review): step 1 launches DevOps, step 2 reads and presents. No cross-review with SA after both Phase 1.7 and 1.9 complete.

### Secondary Paths (side effects, related flows)

- `agents/software-engineer.md:102–108` — The SE's discovery output template already includes a `## Cross-Review Notes` section as the last block. This section header is the exact heading the new prompts must instruct agents to append.
- `agents/qa-engineer.md:115–121` — The QA's discovery output template similarly includes `## Cross-Review Notes` as the last section.
- `agents/software-architect.md:276–289` — The SA's discovery output template includes a `## Cross-Review Notes` section with `### SE Discovery Review` and `### DevOps Discovery Review (if present)` subsections.
- `agents/devops-engineer.md:249–262` — The DevOps discovery output template includes a `## Cross-Review Notes` section.

---

## Root Cause / Insertion Points

New steps should be inserted at the following locations:

### Gap 1 — patch.md Phase 4 cross-review

**Insertion point**: Between step 4 and step 5 of Phase 4 (`workflows/patch.md`, after line 128, before "5. **Read both output documents.**").

A new numbered step 5 (cross-review round) should be inserted. Existing steps 5, 6, 7 become steps 6, 7, 8.

### Gap 2 — patch.md Phase 5 QA final reflection

**Insertion point**: Between step 2 and step 3 of Phase 5 (`workflows/patch.md`, after line 151, before "3. **Read the quality gate report.**").

A new numbered step 3 (QA final reflection) should be inserted. Existing steps 3, 4, 5 become steps 4, 5, 6.

### Gap 3 — minor.md Phase 4 cross-review

**Insertion point**: Between step 4 and step 5 of Phase 4 (`workflows/minor.md`, after line 224, before "5. **Read both output documents.**").

A new numbered step 5 (cross-review round) should be inserted. Existing steps 5, 6, 7 become steps 6, 7, 8.

### Gap 4 — major.md Phase 1.9 SA↔DevOps cross-review

**Insertion point**: After step 2 of Phase 1.9 (`workflows/major.md`, after line 96, before "**Track activation status**...").

A new step 3 is inserted inside the "Steps (only if activated):" block. The `**Track activation status**` note is not a numbered step — it is a standalone paragraph — so it is not renumbered.

### Gap 5 — major.md Phase 4 cross-review

**Insertion point**: Between step 4 and step 5 of Phase 4 (`workflows/major.md`, after line 278, before "5. **Present results**:").

A new numbered step 5 (cross-review round) should be inserted. Existing steps 5 and 6 become steps 6 and 7.

---

## Dependencies

- **Internal**: The workflow files are standalone markdown consumed by the Claude orchestrator. No code file imports or calls them. Changes are purely additive text.
- **External**: None. No APIs, services, or databases are involved.
- **Shared state**: The new cross-review steps reference the same document paths already used in adjacent steps (`{SPEC_FOLDER}/04-implementation-summary.md`, `{SPEC_FOLDER}/04-test-report.md`, `{SPEC_FOLDER}/04-infra-summary.md`, `{SPEC_FOLDER}/01-spec-arch-review.md`, `{SPEC_FOLDER}/01-spec-infra-review.md`). No new files are introduced.

---

## Conventions Observed

### Heading levels
- Phase headers use `##` (e.g., `## Phase 4: Implementation`)
- "Goal" and "Steps" use `###`
- Numbered steps use `1.`, `2.`, etc. at the top level under `### Steps:`
- Sub-items within a step use `a.`, `b.`, `c.` lettering and indentation
- Sub-sub-items use `- **Label**: content` bullet lists

### Step numbering style
- Steps are plain Arabic numerals followed by a period and bold label: `3. **Cross-review round** - Launch two agents in parallel:`
- Steps containing parallel agent launches use the label pattern `**Cross-review round** - Launch [N] agents in parallel:` or `**Cross-review round** — Launch agents in parallel (N or M):`
- patch.md uses a hyphen-dash (`-`) after "round"; minor.md and major.md use an em-dash (`—`). The new steps must match the file they are inserted into.

### Parallel-launch step format
Existing cross-review steps in Phase 2 and Phase 3 follow this pattern:

**patch.md (Phase 2, step 3):**
```
3. **Cross-review round** - Launch two agents in parallel:

   a. **`software-engineer` in DISCOVERY mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's discovery document at `{SPEC_FOLDER}/02-discovery-qa.md`. Then read your own discovery document at `{SPEC_FOLDER}/02-discovery-se.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-se.md` with your observations about the QA findings, any alignment or conflicts, and suggestions for the QA Engineer."

   b. **`qa-engineer` in DISCOVERY mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the Software Engineer's discovery document at `{SPEC_FOLDER}/02-discovery-se.md`. Then read your own discovery document at `{SPEC_FOLDER}/02-discovery-qa.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/02-discovery-qa.md` with your observations about the SE findings, any alignment or conflicts, and suggestions for the Software Engineer."
```

**minor.md (Phase 2, step 3):**
```
3. **Cross-review round** — Launch agents in parallel (2 or 3):
```
(Uses em-dash `—`, not hyphen-dash `-`.)

**major.md (Phase 2, step 3):**
```
3. **Cross-review round** — Launch agents in parallel (3-5):
```
(Uses em-dash `—`, not hyphen-dash `-`.)

### Agent label format in parallel steps
- `a. **\`software-engineer\` in IMPLEMENTATION mode**:`
- `b. **\`qa-engineer\` in IMPLEMENTATION mode**:`
- The mode in the label matches the mode the agent was launched in for the original work (not "cross-review mode" — cross-review is indicated only in the prompt text)
- Exception: in Phase 2 and 3, the label does append `(cross-review)` parenthetically, e.g., `**\`software-engineer\` in DISCOVERY mode (cross-review)**:`

### Prompt string conventions
All prompts use double-quoted strings. Key phrases used in existing cross-review prompts (must be replicated exactly):

| Phrase | Source |
|--------|--------|
| `"You are completing a cross-review."` | patch.md Phase 2 step 3a, 3b; Phase 3 step 3a, 3b; minor.md Phase 2 step 3a–c; Phase 3 step 3a–b; major.md Phase 2 step 3a–e; Phase 3 step 3a–b |
| `"Read [agent]'s [phase] at \`{SPEC_FOLDER}/[doc]\`."` | all cross-review prompts |
| `"Then read your own [phase] at \`{SPEC_FOLDER}/[doc]\`."` | all cross-review prompts |
| `"Append a '## Cross-Review Notes' section to your [doc] at \`{SPEC_FOLDER}/[doc]\`"` | all cross-review prompts |

### Document path variable format
All document references inside prompts use backtick-quoted paths with the `{SPEC_FOLDER}/` prefix: `` `{SPEC_FOLDER}/04-implementation-summary.md` ``

### Conditional phrasing pattern (major.md)
Conditional agent participation is written as: `(only if activated)` appended to the agent sub-item label, e.g.:
```
   d. **`devops-engineer` cross-review** (only if activated):
```
The condition for Phase 1.9 activation is stated as a block at the top of the phase:
```
### Activation Check:
1. **Read the spec** `{SPEC_FOLDER}/01-spec.md`.
2. **Check `Affected Areas > Infrastructure`**: If empty/"None"/"N/A" → SKIP. Otherwise → ACTIVATE.
```
The Phase 1.7/1.9 cross-review is conditional on DevOps being activated, and must follow the same `(only if activated)` labeling or equivalent conditional language.

### Sequential (non-parallel) step format
Sequential steps after parallel rounds use:
```
5. **Launch `tech-lead` in DISCOVERY REVIEW mode** (sequential — runs AFTER cross-reviews):
```
The QA final reflection in patch.md Phase 5 is sequential (QA reads SE's section first) and should use language like "After SE completes their test quality review, launch `qa-engineer`..." rather than parallel launch language.

### Wait step format
After parallel launches: `2. **Wait for both agents to complete.**` or `4. **Wait for cross-reviews to complete.**`

In patch.md Phase 4, there is no explicit "wait" step between steps 4 and 5; the instructions use "After SE completes, launch qa-engineer" phrasing. This means the new cross-review step (to be inserted as new step 5) does not need a preceding wait step, but should itself be explicit about whether it is parallel or sequential. Given that the spec requires parallel launch for SE/QA cross-review in Phase 4, it should follow the same pattern as Phase 2/3 cross-reviews.

---

## Risks Identified

1. **Step renumbering**: In patch.md Phase 4, inserting a new step 5 shifts existing steps 5, 6, 7 to 6, 7, 8. In patch.md Phase 5, inserting a new step 3 shifts existing steps 3, 4, 5 to 4, 5, 6. In minor.md Phase 4, inserting a new step 5 shifts existing steps 5, 6, 7 to 6, 7, 8. In major.md Phase 4, inserting a new step 5 shifts existing steps 5 and 6 to 6 and 7. All internal references to these step numbers within the same phase must be checked.

2. **Internal step number references — patch.md Phase 5**: Step 5 ("Handle the verdict") contains a loop-back reference: "loop back to Phase 4 — re-launch SE and/or QA with the specific issues to address". This refers to Phase 4 by name, not by step number, so it is safe. No other internal step-number references were found in Phase 4 or Phase 5 of patch.md.

3. **Internal step number references — minor.md Phase 4**: Step 3c reads "same logic as patch", and step 3f references "if this is NOT the last sub-PR". Step 7 references "discuss with user before proceeding" — no step number cross-references found that would break.

4. **Internal step number references — major.md Phase 4**: Step 3b references "determine implementer", step 3g references "if not last sub-PR". The "loop back to Phase 4" phrasing in Phase 5 references Phase by name only. No step-number cross-references found that would break.

5. **Phase 1.9 insertion location**: Phase 1.9 has an activation check block and then a "Steps (only if activated):" section with two numbered steps. The new step 3 (cross-review) must be placed within the "only if activated" block. The `**Track activation status**` paragraph follows the entire phase and is not a step — it must not be renumbered.

6. **Prompt consistency**: New prompts must use the exact phrasing `"You are completing a cross-review."` as the opening sentence. Any deviation risks the orchestrator interpreting them differently.

7. **Phase 4 major.md — SA/DevOps cross-review conditionality**: The SA↔DevOps cross-review in Phase 4 only applies when DevOps is active. The insertion at Phase 4 step 5 must label SA and DevOps as `(only if DevOps is active)` or equivalent, consistent with how other conditional agents are marked in the same file.

8. **QA Final Reflection is sequential, not parallel**: The patch.md Phase 5 QA final reflection must run after the SE completes step 2. This is a one-agent sequential step, not a parallel launch. The formatting must not use the parallel sub-item lettering style.

---

## Scope Fit Assessment

- **Estimated files to change**: 3 (`workflows/patch.md`, `workflows/minor.md`, `workflows/major.md`)
- **Estimated lines added**: ~60–90 lines total across all three files (each new cross-review step is approximately 10–18 lines; 5 new steps total)
- **Fits within tier**: YES — well within minor workflow scope. PR 1 (patch.md Phase 4 + minor.md Phase 4) is approximately 25–35 lines. PR 2 (major.md Phase 4, patch.md Phase 5, major.md Phase 1.9) is approximately 35–55 lines.

---

## Key Files

1. `workflows/patch.md` — Primary file for Gap 1 (Phase 4 cross-review) and Gap 2 (Phase 5 QA reflection). Insertion points at lines 128 and 151.
2. `workflows/minor.md` — Primary file for Gap 1 extension to minor tier. Insertion point at line 224.
3. `workflows/major.md` — Primary file for Gap 1 extension to major tier (Phase 4) and Gap 3 (Phase 1.9 SA↔DevOps). Insertion points at lines 96 and 278.
4. `agents/software-engineer.md` — Confirms the `## Cross-Review Notes` section heading exists in the SE output template, validating that the section heading used in new prompts is consistent with agent output expectations.
5. `agents/qa-engineer.md` — Confirms the `## Cross-Review Notes` section heading exists in the QA output template.
6. `agents/software-architect.md` — Confirms SA discovery and planning templates include `## Cross-Review Notes` sections; informs the SA-side prompt for Phase 1.9 cross-review.
7. `agents/devops-engineer.md` — Confirms DevOps discovery template includes `## Cross-Review Notes` section; informs the DevOps-side prompt for Phase 1.9 cross-review.

---

## Exact Insertion Points with Line Context

### patch.md — Phase 4, after step 4 (line 128), before step 5 (line 129)

Current line 128:
```
4. **After SE completes, launch `qa-engineer` in IMPLEMENTATION mode** (or in parallel if determined safe in step 2):
   - Prompt: "You are in IMPLEMENTATION mode. Read your test plan at `{SPEC_FOLDER}/03-plan-qa.md`, the specification at `{SPEC_FOLDER}/01-spec.md`, and the SE's implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Implement the tests following your plan. Follow the project's test conventions exactly. Run the full test suite after writing tests. Write your test report to `{SPEC_FOLDER}/04-test-report.md`."
```

Current line 129 (becomes step 6 after insertion):
```
5. **Read both output documents.**
```

New step 5 to insert between them (cross-review round, parallel SE+QA):
- SE reads `04-test-report.md`, appends cross-review notes to `04-implementation-summary.md`
- QA reads `04-implementation-summary.md`, appends cross-review notes to `04-test-report.md`

### patch.md — Phase 5, after step 2 (line 151), before step 3 (line 153)

Current line 151:
```
2. **After QA completes, launch `software-engineer` in QUALITY GATE mode**:
   - Prompt: "You are in QUALITY GATE mode (test review). Read the test report at `{SPEC_FOLDER}/04-test-report.md` and review the actual test code. Assess test correctness, coverage adequacy, and test quality. Append your '## Software Engineer - Test Quality Review' section to the quality gate report at `{SPEC_FOLDER}/05-quality-gate.md`."
```

Current line 153 (becomes step 4 after insertion):
```
3. **Read the quality gate report.**
```

New step 3 to insert between them (sequential QA final reflection):
- QA reads full `05-quality-gate.md` (including SE's test quality review), appends `## QA Engineer - Final Reflection`

### minor.md — Phase 4, after step 4 (line 224), before step 5 (line 226)

Current lines 222–224:
```
4. **After all sub-PRs are implemented**:
   - Ensure `{SPEC_FOLDER}/04-implementation-summary.md` has a consolidated summary of all PRs.
   - Ensure `{SPEC_FOLDER}/04-test-report.md` has a consolidated report of all test results.
```

Current line 226 (becomes step 6 after insertion):
```
5. **Read both output documents.**
```

New step 5 to insert between them (cross-review round, parallel SE+QA):
- Same pattern as patch.md Phase 4 cross-review

### major.md — Phase 1.9, after step 2 (line 96), before "Track activation status" paragraph (line 98)

Current lines 93–96:
```
1. **Launch `devops-engineer` in SPEC REVIEW mode**:
   - Prompt: "..."

2. **Read the infra spec review**. Present to user combined with Phase 1.7 results if both present. **User checkpoint**.
```

Current line 98 (not a step — standalone paragraph, stays as-is):
```
**Track activation status**: Remember which conditional agents (UI/UX, DevOps) are active. This determines their participation in all subsequent phases.
```

New step 3 to insert between step 2 and the "Track activation status" note (cross-review round, parallel SA+DevOps):
- SA reads `01-spec-infra-review.md`, appends cross-review notes to `01-spec-arch-review.md`
- DevOps reads updated `01-spec-arch-review.md`, appends cross-review notes to `01-spec-infra-review.md`
- Only runs when DevOps is activated (step is inside the "only if activated" block)

### major.md — Phase 4, after step 4 (line 278), before step 5 (line 280)

Current lines 275–278:
```
4. **After all sub-PRs are implemented**:
   - Ensure `04-implementation-summary.md` has consolidated sections for all PRs.
   - Ensure `04-test-report.md` has consolidated results for all PRs.
   - If DevOps active: ensure `04-infra-summary.md` has consolidated infra sections.
```

Current line 280 (becomes step 6 after insertion):
```
5. **Present results**: Per-phase summary, per-PR summary, consolidated totals.
```

New step 5 to insert between them (cross-review round):
- SE reads `04-test-report.md`, appends cross-review notes to `04-implementation-summary.md`
- QA reads `04-implementation-summary.md`, appends cross-review notes to `04-test-report.md`
- SA reads `04-infra-summary.md`, appends cross-review notes (only if DevOps active)
- DevOps reads relevant SA context, appends cross-review notes to `04-infra-summary.md` (only if DevOps active)

---

## Cross-Review Notes

### Alignment with QA Discovery

The QA Engineer's discovery is highly aligned with this document. Both identify the same five gaps (named as AC-1 through AC-8 on the QA side and Gaps 1–5 here), the same three files to modify, and the same insertion points. The QA document cross-validates the exact line references cited in the "Exact Insertion Points" section above: QA's listing of `patch.md` currently at 257 lines, `minor.md` at 374 lines, and `major.md` at 437 lines is consistent with the line numbers identified in this discovery.

Key areas of confirmation:

- **Gap 1 / AC-1** (patch.md Phase 4): Both agree the new step goes between current step 4 and step 5, and that it is a parallel SE↔QA cross-review.
- **Gap 2 / AC-6** (patch.md Phase 5 QA reflection): Both agree this is a sequential step (QA runs after SE), not a parallel launch. The QA document explicitly flags the risk of writing it as parallel — which matches Risk 8 in this document.
- **Gap 3 / AC-2** (minor.md Phase 4): Full agreement on insertion point and pattern.
- **Gap 4 / AC-7** (major.md Phase 1.9 SA↔DevOps): Both agree the new step belongs inside the "Steps (only if activated):" block and must be conditional.
- **Gap 5 / AC-3** (major.md Phase 4): Both agree on the SE↔QA plus conditional SA↔DevOps structure.

### One Conflict to Resolve: Phase 1.9 SA↔DevOps Parallelism

The QA document raises a concern in its "Risks and Edge Cases" section (risk 5) that the SE did not explicitly address: whether the SA↔DevOps cross-review in Phase 1.9 is truly parallel, or whether DevOps must read SA's updated document after SA has appended their cross-review notes. The spec says DevOps reads "the updated `01-spec-arch-review.md`" — implying SA goes first, then DevOps.

This discovery document (Gap 4 insertion note) states both agents run in parallel but does not resolve the ordering concern raised by QA. This is a genuine conflict that requires a decision during planning:

- **Option A**: Fully parallel — both agents read the other's document at the time the step launches. SA would read the original `01-spec-infra-review.md` (no cross-review notes on it yet); DevOps would read the original `01-spec-arch-review.md` (no cross-review notes on it yet). This is consistent with how Phase 2 and Phase 3 cross-reviews work across all three workflow files.
- **Option B**: Sequential — SA goes first, appends cross-review notes to `01-spec-arch-review.md`, then DevOps reads the updated document before appending to `01-spec-infra-review.md`. This is what the spec text implies with the phrase "updated `01-spec-arch-review.md`".

**Recommendation**: The planning phase should explicitly resolve this. The Phase 2 and Phase 3 patterns (fully parallel) are simpler and consistent. If the spec explicitly requires sequential ordering here, Option B should be used and the step should use sequential language ("After SA completes...") rather than parallel sub-items. The QA Engineer should update AC-7 validation criteria once this is resolved, since the correct structural pattern (parallel vs. sequential) affects the format check.

### QA Validation Pattern Observations

The QA document identifies six structural validation patterns (presence, content, structure, sequential vs. parallel, conditionality, step numbering integrity, and negative check). These patterns are well-suited to validating this type of markdown-only change. One observation worth noting: the "content check" pattern (AC-4) verifies that each new prompt contains `"You are completing a cross-review"` as the opening phrase, reads the other agent's document first, then reads the agent's own document. The conventions section in this discovery document confirms that all existing cross-review prompts follow this exact order — this is a valid constraint and the implementation must match it.

### Suggestions for the QA Engineer

1. **AC-7 validation**: Clarify whether the check for SA's step uses parallel sub-items (`a.` and `b.` under one numbered step) or sequential numbered steps, pending resolution of the ordering decision above.

2. **AC-6 step renumbering in patch.md Phase 5**: The QA document (risk 1) notes that steps 3, 4, and 5 shift to 4, 5, and 6, and asks whether there are steps 6 and 7 or a "Handle the verdict" block. This discovery confirms (Risk 2 above) that the loop-back reference in Phase 5's final step uses "Phase 4" by name, not a step number, so no internal cross-references break. QA can confirm this is safe during file-read validation.

3. **Document path format**: The QA document's content-check pattern (AC-4) should also verify that the `{SPEC_FOLDER}/` prefix is used consistently (not a bare filename), as identified in the conventions section of this document. This is a low-risk but easily missed typo vector.

4. **No agent file modifications**: Both discovery documents agree that `agents/software-engineer.md`, `agents/qa-engineer.md`, `agents/software-architect.md`, and `agents/devops-engineer.md` require no changes. The QA "scope creep" risk (risk 10) is a valid negative check — if any agent file is modified, it is a violation.

5. **minor.md Phase 4 step count**: The QA document references "current steps 5, 6, 7 become 6, 7, 8" for minor.md (matching Gap 3 here). The QA validation for AC-10 should explicitly confirm that the renumbered steps 6, 7, and 8 in minor.md match the original steps 5, 6, and 7 content-for-content, not just that numbering is correct.
