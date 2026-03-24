# QA Engineer Test Plan

## Spec Reference
- **Spec**: `specs/001-add-cross-review-rounds-to-workflows/01-spec.md`
- **Discovery (QA)**: `specs/001-add-cross-review-rounds-to-workflows/02-discovery-qa.md`
- **Discovery (SE)**: `specs/001-add-cross-review-rounds-to-workflows/02-discovery-se.md`
- **Discovery (TL)**: `specs/001-add-cross-review-rounds-to-workflows/02-discovery-tl.md`

---

## Test Strategy

All validation is structural (file-read). There is no automated test suite. Each check reads a modified workflow file and verifies the presence, content, placement, formatting, and step numbering of the new or renumbered steps. Validation is organized into five insertion points across three files, verified in order from simplest to most complex. The first checks (regression baseline) confirm that pre-existing content is intact; the subsequent checks verify new content is correct. This ordering means a renumbering failure (which corrupts step references) will be caught before prompt-content errors, giving a clear failure triage path.

---

## Validation Ordering Strategy

### Why this order

1. **Regression checks first (AC-10)**: Confirm that no existing steps were removed, altered, or reordered. If existing content is corrupted, all subsequent content checks may produce false positives (the new content might be in the right place relative to each other but displace original content). Verifying the original steps are intact is the foundation for every other check.

2. **Structure and numbering checks second (AC-9, step numbering)**: After confirming original content is present, verify that renumbering after each insertion point is correct and consistent. A numbering error is a hard defect even if the new step content is perfect.

3. **Existence checks third (AC-1, AC-2, AC-3, AC-6, AC-7)**: Confirm each new cross-review step exists at the correct location (between the right two existing steps).

4. **Content checks fourth (AC-4)**: Verify that the prompt text inside each new step uses the required phrases, correct file paths, and correct section heading.

5. **Structural pattern checks fifth (AC-5, parallel vs. sequential)**: Verify the launch format matches the established parallel or sequential pattern.

6. **Conditionality check (AC-8)**: Verify that the Phase 1.9 cross-review and the SA/DevOps sub-items in Phase 4 major.md are gated by appropriate conditional language.

### Run order summary

1. AC-10 (no existing steps removed or reordered) — all three files
2. AC-9 (document structure consistency) — all three files
3. Step numbering integrity — per insertion point
4. AC-1, AC-2, AC-3, AC-6, AC-7 (existence + placement) — per file
5. AC-4 (prompt content matches pattern) — per file
6. AC-5 (parallel launch format) — per file
7. AC-8 (conditionality language) — major.md only

---

## Validation Checks by Acceptance Criterion

---

### AC-10: No existing workflow steps are removed or reordered

This is the regression baseline. Run this first for every file before verifying any new content.

#### V-10.1: patch.md — Phase 4 existing steps intact

- **File**: `workflows/patch.md`
- **What to check**: Phase 4 `### Steps:` section must contain all six original step labels in their original relative order. The labels are:
  1. `**Confirm user approval**`
  2. `**Determine execution order**`
  3. `**Launch \`software-engineer\` in IMPLEMENTATION mode**`
  4. `**After SE completes, launch \`qa-engineer\` in IMPLEMENTATION mode**`
  5. `**Read both output documents.**`
  6. `**Present results to the user**`
  7. `**User checkpoint**`
- **PASS**: All seven labels are present in the listed order, with a new numbered step inserted between the original step 4 and step 5. The original step 5 now carries the number 6, step 6 carries 7, step 7 carries 8.
- **FAIL**: Any label is missing, any label content is altered, or any label appears out of order relative to the others.

#### V-10.2: patch.md — Phase 5 existing steps intact

- **File**: `workflows/patch.md`
- **What to check**: Phase 5 `### Steps:` section must contain all five original step labels in their original relative order. The original labels were:
  1. `**Launch \`qa-engineer\` in QUALITY GATE mode**`
  2. `**After QA completes, launch \`software-engineer\` in QUALITY GATE mode**`
  3. `**Read the quality gate report.**`
  4. `**Present results to the user**`
  5. `**Handle the verdict**`
- **PASS**: All five labels are present and a new step is inserted after the original step 2. Original step 3 now carries number 4, step 4 carries 5, step 5 carries 6.
- **FAIL**: Any label is missing, altered, or out of order.
- **Edge case**: The `**Handle the verdict**` step contains a loop-back reference ("loop back to Phase 4"). Verify this text reads "Phase 4" (a name reference), not a step number. If it now reads "loop back to step N" with a number, that is an alteration of existing content — FAIL.

#### V-10.3: minor.md — Phase 4 existing steps intact

- **File**: `workflows/minor.md`
- **What to check**: Phase 4 `### Steps:` section must contain all seven original step labels in their original relative order:
  1. `**Confirm user approval**`
  2. `**Determine sub-PR execution plan**` (or equivalent — verify exact label from file)
  3. `**For each sub-PR**` (the loop step)
  4. `**After all sub-PRs are implemented**`
  5. `**Read both output documents.**`
  6. `**Present results to the user**`
  7. `**User checkpoint**`
- **PASS**: All seven labels present in order; a new step inserted between original step 4 and step 5. Original step 5 becomes 6, step 6 becomes 7, step 7 becomes 8.
- **FAIL**: Any label missing, altered, or out of order.

#### V-10.4: major.md — Phase 1.9 existing steps intact

- **File**: `workflows/major.md`
- **What to check**: Phase 1.9 `### Steps (only if activated):` section must contain all two original numbered step labels:
  1. `**Launch \`devops-engineer\` in SPEC REVIEW mode**`
  2. `**Read the infra spec review**`
- Additionally, the `**Track activation status**` paragraph that follows Phase 1.9 must still be present and unchanged.
- **PASS**: Both original numbered steps are present and intact. The `**Track activation status**` paragraph is present immediately after the last numbered step (now step 3 if a new step was inserted, otherwise step 2). It is not renumbered (it is prose, not a step).
- **FAIL**: Either original step is missing or altered. The `**Track activation status**` paragraph is missing, numbered, or moved outside Phase 1.9.

#### V-10.5: major.md — Phase 4 existing steps intact

- **File**: `workflows/major.md`
- **What to check**: Phase 4 `### Steps:` section must contain the six original step labels in order (steps 1–3 are a large loop structure, step 4 is the consolidation step, step 5 was "Present results", step 6 was "User checkpoint"):
  1. Step 1 — `**Confirm user approval**` (or equivalent)
  2. Step 2 — `**Plan sub-PR execution**` (or equivalent)
  3. Step 3 — the per-phase/per-sub-PR loop (containing sub-items a–g)
  4. Step 4 — `**After all sub-PRs are implemented**`
  5. Step 5 — `**Present results**`
  6. Step 6 — `**User checkpoint**`
- **PASS**: All six original labels present in order; new step inserted between original step 4 and step 5. Original step 5 becomes 6, step 6 becomes 7.
- **FAIL**: Any original label missing, altered, or reordered.
- **Edge case**: Step 3 contains sub-items (a through g). Verify that sub-item labels (e.g., `**Determine which agents implement this sub-PR**`, `**Launch SE in IMPLEMENTATION mode**`) are not altered. Spot-check sub-item `g` for the loop-back reference: it should reference "last sub-PR" in prose, not a step number.

---

### AC-9: Workflow document structure remains consistent

#### V-9.1: patch.md — Phase 4 new step heading level and dash style

- **File**: `workflows/patch.md`
- **What to check**: The new cross-review step in Phase 4 must use:
  - A step number followed by a period and a bold label: `5. **...**`
  - A hyphen-dash (`-`) between the label text and the launch description (not an em-dash `—`), matching the existing pattern in patch.md (e.g., current step 3: `**Launch \`software-engineer\` in IMPLEMENTATION mode**:`).
  - Agent sub-items using lowercase letters and a period: `a.`, `b.`
  - A `- Prompt: "..."` line under each agent sub-item.
- **PASS**: The new step matches all four formatting elements above.
- **FAIL**: Uses em-dash instead of hyphen-dash, or uses a different label style, or sub-items use numbers instead of letters, or prompt lines use a different prefix.

#### V-9.2: patch.md — Phase 5 new step heading level and sequential format

- **File**: `workflows/patch.md`
- **What to check**: The new QA final reflection step in Phase 5 must be a standalone numbered step (not a parallel sub-item), following the same step format as adjacent steps 1 and 2 in Phase 5. It must NOT use parallel sub-item lettering (`a.`, `b.`). It should use language signaling sequential launch, such as "After SE completes..." or equivalent.
- **PASS**: The new step is a single numbered step with one agent launch and sequential language.
- **FAIL**: The step uses parallel sub-item lettering, or it is formatted as a sub-item of step 2, or it lacks sequential language.

#### V-9.3: minor.md — Phase 4 new step uses em-dash

- **File**: `workflows/minor.md`
- **What to check**: The new cross-review step in Phase 4 must use an em-dash (`—`) in the step label (e.g., `5. **Cross-review round** — Launch two agents in parallel:`), matching the existing style in minor.md (e.g., Phase 2 step 3 which reads `**Cross-review round** — Launch agents in parallel (2 or 3):`).
- **PASS**: Em-dash is used in the step label.
- **FAIL**: Hyphen-dash (`-`) is used instead.

#### V-9.4: major.md — Phase 1.9 and Phase 4 new steps use em-dash

- **File**: `workflows/major.md`
- **What to check**: New cross-review steps in both Phase 1.9 and Phase 4 must use an em-dash (`—`) in the step label, consistent with the existing style in major.md.
- **PASS**: Both new steps use em-dash.
- **FAIL**: Either new step uses a hyphen-dash.

#### V-9.5: Agent label format in new parallel steps

- **File**: `workflows/patch.md`, `workflows/minor.md`, `workflows/major.md`
- **What to check**: In every new parallel cross-review step, agent sub-items must follow the pattern:
  `**\`agent-name\` in MODE mode (cross-review)**:`
  The mode in the label must be `IMPLEMENTATION mode (cross-review)` for Phase 4 steps, and `SPEC REVIEW mode (cross-review)` for the Phase 1.9 step.
  Cross-reference the existing Phase 2 pattern (e.g., patch.md step 3a: `**\`software-engineer\` in DISCOVERY mode (cross-review)**:`).
- **PASS**: Each agent sub-item in each new parallel step uses the correct mode name with the `(cross-review)` parenthetical.
- **FAIL**: Mode name is wrong, parenthetical is absent, or backticks around agent name are missing.

---

### Step Numbering Integrity Checks

#### V-NUM.1: patch.md Phase 4 — renumbering after step 4

- **File**: `workflows/patch.md`
- **What to check**: After the insertion of a new step 5 (cross-review), confirm:
  - New step is numbered `5.`
  - Original step 5 (`**Read both output documents.**`) is now numbered `6.`
  - Original step 6 (`**Present results to the user**`) is now numbered `7.`
  - Original step 7 (`**User checkpoint**`) is now numbered `8.`
  - No step in Phase 4 is numbered `5.` twice or skips a number.
- **PASS**: Numbers are sequential 1–8 with no gaps or duplicates.
- **FAIL**: Any number is duplicated, skipped, or still at its original pre-insertion value.

#### V-NUM.2: patch.md Phase 5 — renumbering after step 2

- **File**: `workflows/patch.md`
- **What to check**: After insertion of a new step 3 (QA reflection), confirm:
  - New step is numbered `3.`
  - Original step 3 (`**Read the quality gate report.**`) is now numbered `4.`
  - Original step 4 (`**Present results to the user**`) is now numbered `5.`
  - Original step 5 (`**Handle the verdict**`) is now numbered `6.`
  - No step in Phase 5 is numbered `3.` twice or skips a number.
- **PASS**: Numbers are sequential 1–6 with no gaps or duplicates.
- **FAIL**: Duplication, skip, or unchanged original numbering.

#### V-NUM.3: minor.md Phase 4 — renumbering after step 4

- **File**: `workflows/minor.md`
- **What to check**: After insertion of a new step 5 (cross-review), confirm:
  - New step is numbered `5.`
  - Original step 5 (`**Read both output documents.**`) is now numbered `6.`
  - Original step 6 (`**Present results to the user**`) is now numbered `7.`
  - Original step 7 (`**User checkpoint**`) is now numbered `8.`
- **PASS**: Numbers are sequential 1–8 with no gaps or duplicates.
- **FAIL**: Any number is duplicated, skipped, or unchanged.

#### V-NUM.4: major.md Phase 1.9 — new step 3 placement

- **File**: `workflows/major.md`
- **What to check**: The new cross-review step is numbered `3.` inside the `### Steps (only if activated):` section. The original step 1 and step 2 retain their numbers. The `**Track activation status**` paragraph that follows is NOT numbered (it remains a standalone bold paragraph, not `3. **Track activation status**`).
- **PASS**: New step 3 exists; original steps 1 and 2 unchanged; `**Track activation status**` is unnumbered prose.
- **FAIL**: `**Track activation status**` is numbered, or existing steps 1 or 2 are renumbered, or the new step is numbered other than `3.`

#### V-NUM.5: major.md Phase 4 — renumbering after step 4

- **File**: `workflows/major.md`
- **What to check**: After insertion of a new step 5 (cross-review), confirm:
  - New step is numbered `5.`
  - Original step 5 (`**Present results**`) is now numbered `6.`
  - Original step 6 (`**User checkpoint**`) is now numbered `7.`
- **PASS**: Numbers are sequential 1–7 with no gaps or duplicates.
- **FAIL**: Any number is duplicated, skipped, or unchanged.

---

### AC-1: Phase 4 cross-review exists in patch.md

#### V-1.1: New step exists between step 4 and step 6 in Phase 4

- **File**: `workflows/patch.md`
- **What to check**: A new step numbered `5.` must exist in Phase 4 between the `**After SE completes, launch \`qa-engineer\` in IMPLEMENTATION mode**` step (now step 4) and the `**Read both output documents.**` step (now step 6). The step label must indicate a cross-review round with parallel launch.
- **PASS**: Step 5 exists at this location with a label indicating parallel launch of SE and QA in cross-review.
- **FAIL**: No step 5 exists at this location, or step 5 is not a cross-review step.

#### V-1.2: Cross-review step contains both SE and QA sub-items

- **File**: `workflows/patch.md`
- **What to check**: The new step 5 in Phase 4 must contain exactly two lettered sub-items: one for `software-engineer` and one for `qa-engineer`.
- **PASS**: Sub-items `a.` and `b.` exist, one for each agent.
- **FAIL**: Only one sub-item exists, or sub-items cover agents other than SE and QA.

---

### AC-2: Phase 4 cross-review exists in minor.md

#### V-2.1: New step exists between step 4 and step 6 in Phase 4

- **File**: `workflows/minor.md`
- **What to check**: A new step numbered `5.` must exist in Phase 4 between the `**After all sub-PRs are implemented**` step (step 4) and the `**Read both output documents.**` step (now step 6).
- **PASS**: Step 5 exists at this location with a cross-review round label.
- **FAIL**: No step 5 at this location, or step 5 is not a cross-review step.

#### V-2.2: Cross-review step contains SE and QA sub-items

- **File**: `workflows/minor.md`
- **What to check**: Sub-items `a.` (SE) and `b.` (QA) are present.
- **PASS**: Both sub-items exist.
- **FAIL**: Missing sub-item or wrong agents.

---

### AC-3: Phase 4 cross-review exists in major.md

#### V-3.1: New step exists between step 4 and step 6 in Phase 4

- **File**: `workflows/major.md`
- **What to check**: A new step numbered `5.` must exist in Phase 4 between the `**After all sub-PRs are implemented**` step (step 4) and the `**Present results**` step (now step 6).
- **PASS**: Step 5 exists at this location with a cross-review round label.
- **FAIL**: No step 5 at this location.

#### V-3.2: Cross-review step contains SE and QA sub-items

- **File**: `workflows/major.md`
- **What to check**: Sub-items `a.` (SE) and `b.` (QA) are present.
- **PASS**: Both sub-items exist.
- **FAIL**: Missing sub-item.

#### V-3.3: Cross-review step contains SA and DevOps sub-items, marked conditional

- **File**: `workflows/major.md`
- **What to check**: Sub-items for `software-architect` and `devops-engineer` are present (labelled `c.` and `d.` or equivalent), each marked with `(only if activated)` or `(only if DevOps is active)` or equivalent conditional language consistent with the rest of major.md. Cross-reference Phase 2 step 3e in major.md for the exact conditional label pattern: `**\`devops-engineer\` cross-review** (only if activated):`.
- **PASS**: SA and DevOps sub-items exist with conditional labels that match the pattern used elsewhere in major.md.
- **FAIL**: SA and DevOps sub-items are missing, or they are present but lack conditional labels.

---

### AC-4: Phase 4 cross-review prompts match the Discovery/Planning pattern

For each new cross-review step, the prompt text for each agent sub-item is verified against five required elements.

#### V-4.1: SE prompt in patch.md Phase 4

- **File**: `workflows/patch.md`, Phase 4 new step 5, sub-item `a.`
- **Required elements** (all five must be present):
  1. Prompt opens with: `"You are completing a cross-review.`
  2. Reads the QA's document first: references `` `{SPEC_FOLDER}/04-test-report.md` ``
  3. Then reads its own document: references `` `{SPEC_FOLDER}/04-implementation-summary.md` ``
  4. Append instruction: `Append a '## Cross-Review Notes' section`
  5. Append target: `` `{SPEC_FOLDER}/04-implementation-summary.md` ``
- **PASS**: All five elements present.
- **FAIL**: Any element is absent, uses wrong file path, uses wrong section heading (e.g., `## SE Cross-Review` instead of `## Cross-Review Notes`), or opens with a different phrase.

#### V-4.2: QA prompt in patch.md Phase 4

- **File**: `workflows/patch.md`, Phase 4 new step 5, sub-item `b.`
- **Required elements**:
  1. `"You are completing a cross-review.`
  2. Reads `` `{SPEC_FOLDER}/04-implementation-summary.md` `` first
  3. Then reads `` `{SPEC_FOLDER}/04-test-report.md` ``
  4. `Append a '## Cross-Review Notes' section`
  5. Append target: `` `{SPEC_FOLDER}/04-test-report.md` ``
- **PASS**: All five elements present.
- **FAIL**: Any element absent or incorrect.

#### V-4.3: SE prompt in minor.md Phase 4

- **File**: `workflows/minor.md`, Phase 4 new step 5, sub-item `a.`
- **Required elements**: Same five as V-4.1 — same file paths, same phrases.
- **PASS**: All five elements present.
- **FAIL**: Any element absent, incorrect path, or uses wrong heading.

#### V-4.4: QA prompt in minor.md Phase 4

- **File**: `workflows/minor.md`, Phase 4 new step 5, sub-item `b.`
- **Required elements**: Same five as V-4.2.
- **PASS**: All five elements present.
- **FAIL**: Any element absent or incorrect.

#### V-4.5: SE prompt in major.md Phase 4

- **File**: `workflows/major.md`, Phase 4 new step 5, sub-item `a.`
- **Required elements**: Same five as V-4.1.
- **PASS**: All five elements present.
- **FAIL**: Any element absent or incorrect.

#### V-4.6: QA prompt in major.md Phase 4

- **File**: `workflows/major.md`, Phase 4 new step 5, sub-item `b.`
- **Required elements**: Same five as V-4.2.
- **PASS**: All five elements present.
- **FAIL**: Any element absent or incorrect.

#### V-4.7: SA prompt in major.md Phase 4

- **File**: `workflows/major.md`, Phase 4 new step 5, SA sub-item (c. or equivalent).
- **Required elements**:
  1. `"You are completing a cross-review.`
  2. Reads `` `{SPEC_FOLDER}/04-infra-summary.md` `` (the DevOps's Phase 4 document)
  3. Append instruction: `Append a '## Cross-Review Notes' section`
  4. Append target: `` `{SPEC_FOLDER}/04-implementation-summary.md` `` (SA appends to the consolidated implementation document, consistent with TL recommendation in 02-discovery-tl.md)
- **PASS**: All four elements present.
- **FAIL**: SA reads wrong document, appends to wrong document, or uses wrong section heading.
- **Note**: The TL discovery (section "Architectural Concerns #2") recommends SA reads `04-infra-summary.md` and appends to `04-implementation-summary.md`. Verify the SE followed this recommendation. If the SE chose a different target document, this check must be updated in the quality gate phase — but the heading `## Cross-Review Notes` and the opening phrase are non-negotiable.

#### V-4.8: DevOps prompt in major.md Phase 4

- **File**: `workflows/major.md`, Phase 4 new step 5, DevOps sub-item (d. or equivalent).
- **Required elements**:
  1. `"You are completing a cross-review.`
  2. Reads `` `{SPEC_FOLDER}/04-implementation-summary.md` `` (per TL recommendation)
  3. Append instruction: `Append a '## Cross-Review Notes' section`
  4. Append target: `` `{SPEC_FOLDER}/04-infra-summary.md` ``
- **PASS**: All four elements present.
- **FAIL**: Wrong file paths or wrong section heading.

#### V-4.9: SA prompt in major.md Phase 1.9

- **File**: `workflows/major.md`, Phase 1.9 new step 3, SA sub-item.
- **Required elements**:
  1. `"You are completing a cross-review.`
  2. Reads `` `{SPEC_FOLDER}/01-spec-infra-review.md` ``
  3. Append instruction: `Append a '## Cross-Review Notes' section`
  4. Append target: `` `{SPEC_FOLDER}/01-spec-arch-review.md` ``
- **PASS**: All four elements present.
- **FAIL**: Wrong file paths or wrong heading.

#### V-4.10: DevOps prompt in major.md Phase 1.9

- **File**: `workflows/major.md`, Phase 1.9 new step 3, DevOps sub-item.
- **Required elements**:
  1. `"You are completing a cross-review.`
  2. Reads `` `{SPEC_FOLDER}/01-spec-arch-review.md` `` (the Phase 1.7 primary deliverable)
  3. Append instruction: `Append a '## Cross-Review Notes' section`
  4. Append target: `` `{SPEC_FOLDER}/01-spec-infra-review.md` ``
- **PASS**: All four elements present.
- **FAIL**: Wrong file paths or wrong heading.

#### V-4.11: Document path format — {SPEC_FOLDER}/ prefix present in all new prompts

- **File**: `workflows/patch.md`, `workflows/minor.md`, `workflows/major.md`
- **What to check**: Every file reference inside a new prompt uses the `` `{SPEC_FOLDER}/[filename]` `` pattern. No bare filenames (e.g., `04-test-report.md` without the prefix), no relative paths (e.g., `./04-test-report.md`).
- **PASS**: All file references in all new prompts use the `{SPEC_FOLDER}/` prefix with backtick-quoted paths.
- **FAIL**: Any file reference lacks the prefix or uses a different path pattern.

---

### AC-5: Phase 4 cross-review prompts follow parallel launch pattern

#### V-5.1: patch.md Phase 4 cross-review is a single numbered step with lettered sub-items

- **File**: `workflows/patch.md`
- **What to check**: The new step 5 in Phase 4 must be structured as one numbered step (`5. **Cross-review round** - Launch two agents in parallel:`) with two lettered sub-items (`a.` and `b.`). It must NOT be two separate numbered steps (e.g., `5. **Launch software-engineer...` and `6. **Launch qa-engineer...`).
- **PASS**: One numbered step with lettered sub-items.
- **FAIL**: Two separate numbered steps, or any other non-parallel structure.

#### V-5.2: minor.md Phase 4 cross-review is a single numbered step with lettered sub-items

- **File**: `workflows/minor.md`
- **Same check as V-5.1**: One numbered step (step 5) with lettered sub-items.
- **PASS**: Single step with `a.` and `b.` sub-items.
- **FAIL**: Multiple numbered steps.

#### V-5.3: major.md Phase 4 cross-review is a single numbered step with lettered sub-items

- **File**: `workflows/major.md`
- **Same check as V-5.1**: One numbered step (step 5) with lettered sub-items (a, b for SE/QA, and c, d for SA/DevOps if those agents are present).
- **PASS**: Single step with lettered sub-items.
- **FAIL**: Multiple numbered steps.

#### V-5.4: major.md Phase 1.9 cross-review is a single numbered step with lettered sub-items (parallel)

- **File**: `workflows/major.md`
- **What to check**: The new step 3 in Phase 1.9 must be structured as one numbered step with two lettered sub-items (`a.` for SA and `b.` for DevOps). It must NOT be two sequential steps. This implements the TL's explicit recommendation (parallel, Option A).
- **PASS**: One numbered step with `a.` and `b.` sub-items.
- **FAIL**: Two separate sequential numbered steps, or sequential language like "After SA completes, launch DevOps".

---

### AC-6: Phase 5 patch QA final reflection step is added

#### V-6.1: New step 3 exists in patch.md Phase 5

- **File**: `workflows/patch.md`
- **What to check**: A new step numbered `3.` must exist in Phase 5 between the `**After QA completes, launch \`software-engineer\` in QUALITY GATE mode**` step (step 2) and the `**Read the quality gate report.**` step (now step 4).
- **PASS**: Step 3 exists at this location.
- **FAIL**: No step 3 at this location, or step 3 is the old "Read the quality gate report" step (which would mean the new step was not inserted).

#### V-6.2: New step 3 is sequential (not parallel) and launches qa-engineer

- **File**: `workflows/patch.md`
- **What to check**: Step 3 launches `qa-engineer` as a single agent (not a parallel sub-item) using language that makes the sequential dependency explicit (e.g., "After SE completes..." or "After the SE's test quality review is appended..."). There must be no lettered parallel sub-items under this step.
- **PASS**: Step launches one agent sequentially after the SE.
- **FAIL**: Step uses lettered sub-items, step launches SE and QA together, or step lacks sequential dependency language.

#### V-6.3: New step 3 prompt contains section heading `## QA Engineer - Final Reflection`

- **File**: `workflows/patch.md`, Phase 5 step 3 prompt
- **What to check**: The prompt instructs QA to append a section with the exact heading `## QA Engineer - Final Reflection` to `{SPEC_FOLDER}/05-quality-gate.md`.
- Required elements:
  1. `"You are completing a cross-review.` (or equivalent — note: this step may use "You are in QUALITY GATE mode" since QA is re-reading in quality gate context; verify against spec AC-6 wording which says "final QA reflection step" — the exact opening phrase is not mandated for this step the way it is for Phase 4 cross-reviews, but the step must reference reading `05-quality-gate.md` in full)
  2. Reads `{SPEC_FOLDER}/05-quality-gate.md` (including SE's test quality review section)
  3. `Append a '## QA Engineer - Final Reflection' section` — exact heading text
  4. Append target: `{SPEC_FOLDER}/05-quality-gate.md`
  5. States QA may acknowledge SE's findings and update verdict if warranted
- **PASS**: All five elements present.
- **FAIL**: Section heading uses different text (e.g., `## QA Final Reflection`, `## Final QA Reflection`), or QA is not instructed to read the SE's section, or the append target is wrong.
- **Edge case**: Verify the heading matches the spec's AC-6 verbatim: `## QA Engineer - Final Reflection`. Any variation is a FAIL.

---

### AC-7: Phase 1.7/1.9 SA↔DevOps cross-review exists in major.md (when DevOps is active)

#### V-7.1: New step 3 exists inside the `### Steps (only if activated):` block of Phase 1.9

- **File**: `workflows/major.md`
- **What to check**: A new step numbered `3.` must exist within the `### Steps (only if activated):` section of Phase 1.9, placed after the original step 2 (`**Read the infra spec review**`) and before the `**Track activation status**` paragraph.
- **PASS**: Step 3 exists in this position inside the activation-gated section.
- **FAIL**: Step 3 exists but is placed outside the `### Steps (only if activated):` block (e.g., after the `**Track activation status**` paragraph or as a new Phase 1.10 section), or step 3 does not exist.

#### V-7.2: Step 3 launches both SA and DevOps

- **File**: `workflows/major.md`
- **What to check**: The new Phase 1.9 step 3 contains sub-items for both `software-architect` and `devops-engineer`.
- **PASS**: Both agents are present as lettered sub-items.
- **FAIL**: Only one agent is present.

---

### AC-8: Phase 1.7/1.9 cross-review is skipped when DevOps is not active

#### V-8.1: Cross-review step is inside the `### Steps (only if activated):` block (not unconditional)

- **File**: `workflows/major.md`
- **What to check**: The new step 3 for the Phase 1.9 SA↔DevOps cross-review must be inside the existing `### Steps (only if activated):` section, not outside it. The `### Activation Check:` block at the top of Phase 1.9 already gates the entire `### Steps (only if activated):` section. Any step inside that section is automatically conditional. Verify that the new step is not placed after the `**Track activation status**` paragraph (which would put it outside the conditional block and make it run unconditionally).
- **PASS**: Step 3 is between step 2 and the `**Track activation status**` paragraph, inside the conditional block.
- **FAIL**: Step 3 appears after the `**Track activation status**` paragraph or outside the conditional block entirely.

#### V-8.2: No new unconditional SA↔DevOps cross-review section was added elsewhere

- **File**: `workflows/major.md`
- **What to check**: Confirm that no new section (e.g., `## Phase 1.95:` or a new heading) was added between Phase 1.9 and Phase 2 that would run the SA↔DevOps cross-review unconditionally.
- **PASS**: No new heading or unconditional cross-review block exists between Phase 1.9 and Phase 2.
- **FAIL**: A new unconditional cross-review section exists between Phase 1.9 and Phase 2.

---

## Regression Tests

### RT-1: Agent files unmodified

- **What it protects**: The spec explicitly scopes out agent file changes (agents/*.md). Modifying agent files is scope creep.
- **Files**: `agents/software-engineer.md`, `agents/qa-engineer.md`, `agents/software-architect.md`, `agents/devops-engineer.md`
- **Check**: Read the git diff or current content of each agent file and confirm no changes were made.
- **PASS**: All four agent files are identical to their pre-change content.
- **FAIL**: Any agent file has been modified.

### RT-2: Command file unmodified

- **What it protects**: The `/sisan` command entry point (`.claude/commands/sisan.md`) is not in scope.
- **File**: `.claude/commands/sisan.md`
- **Check**: Confirm no changes were made to this file.
- **PASS**: File is unchanged.
- **FAIL**: File was modified.

### RT-3: patch.md Phase 2 and Phase 3 cross-review steps unmodified

- **What it protects**: Existing cross-review steps in Phase 2 and Phase 3 of patch.md must remain intact as the canonical reference patterns.
- **File**: `workflows/patch.md`
- **Check**: Verify Phase 2 step 3 (parallel cross-review, SE and QA) and Phase 3 step 3 (parallel cross-review, SE and QA) are present and unchanged.
- **PASS**: Both existing cross-review steps are intact and unaltered.
- **FAIL**: Either step is missing, altered, or renumbered.

### RT-4: minor.md Phase 2 and Phase 3 cross-review steps unmodified

- **What it protects**: Same as RT-3 for minor.md.
- **File**: `workflows/minor.md`
- **Check**: Verify Phase 2 step 3 and Phase 3 step 3 cross-review rounds are intact.
- **PASS**: Both intact.
- **FAIL**: Either altered.

### RT-5: major.md Phase 2 and Phase 3 cross-review steps unmodified

- **What it protects**: Same as RT-3 for major.md, which has the most complex existing cross-reviews.
- **File**: `workflows/major.md`
- **Check**: Verify Phase 2 step 3 and Phase 3 step 3 cross-review rounds are intact, including all sub-items (SA, SE, QA, DevOps conditional items).
- **PASS**: Both intact.
- **FAIL**: Any sub-item missing or altered.

### RT-6: Phase 6 (Merge) steps unmodified in all files

- **What it protects**: Phase 6 is not in scope and must not be touched.
- **Files**: All three workflow files.
- **Check**: Verify Phase 6 content is present and unchanged in patch.md, minor.md, and major.md.
- **PASS**: Phase 6 intact in all three files.
- **FAIL**: Phase 6 content altered or missing in any file.

### RT-7: patch.md Phase 5 SE step 2 prompt unmodified

- **What it protects**: The existing SE test quality review step (step 2 in Phase 5) is the "partial Gap 2" noted in the spec. It must not be removed or altered — only a new step is added after it.
- **File**: `workflows/patch.md`
- **Check**: Phase 5 step 2 must still read: `**After QA completes, launch \`software-engineer\` in QUALITY GATE mode**` with the original prompt text (`"You are in QUALITY GATE mode (test review). Read the test report at \`{SPEC_FOLDER}/04-test-report.md\`..."`).
- **PASS**: Step 2 content unchanged.
- **FAIL**: Step 2 was altered, replaced, or merged with the new step.

---

## Edge Case Tests

### EC-1: patch.md Phase 5 loop-back reference intact

- **Scenario**: The original Phase 5 step 5 (`**Handle the verdict**`) contains a loop-back to "Phase 4", referenced by name. After renumbering (step 5 becomes step 6), the text inside this step must not have been accidentally changed to reference a step number.
- **File**: `workflows/patch.md`
- **Check**: In the new step 6 (`**Handle the verdict**`), verify the loop-back text still says "loop back to Phase 4" (or equivalent name-based reference), not "loop back to step 7" or any number.
- **PASS**: Loop-back uses phase name.
- **FAIL**: Loop-back now uses a step number.

### EC-2: major.md `Track activation status` paragraph not numbered or moved

- **Scenario**: The `**Track activation status**` paragraph in Phase 1.9 is a prose block, not a step. When a new step 3 is inserted, there is a risk it gets accidentally numbered as step 3 (shifting the paragraph into the numbered step sequence) or moved outside Phase 1.9.
- **File**: `workflows/major.md`
- **Check**: After the Phase 1.9 `### Steps (only if activated):` block (which now ends with the new step 3), the `**Track activation status**` paragraph must appear as a bold paragraph (not a numbered step), followed immediately by `---` (the horizontal rule separating phases).
- **PASS**: Paragraph is unnumbered, present, and followed by `---`.
- **FAIL**: Paragraph is numbered, absent, or appears in a different location.

### EC-3: major.md Phase 4 SA/DevOps sub-items — DevOps reads the right document

- **Scenario**: The TL discovery (AC concern #2) identifies that "SA context" in Phase 4 is vague. The recommended interpretation is that DevOps reads `04-implementation-summary.md` (not a non-existent SA-specific document). SA reads `04-infra-summary.md`.
- **File**: `workflows/major.md`
- **Check**: In Phase 4 step 5, the DevOps sub-item must reference reading `` `{SPEC_FOLDER}/04-implementation-summary.md` `` (or `04-infra-summary.md` if the SE chose the reverse pattern). The key negative check: the prompt must NOT reference a document that does not exist (e.g., `04-plan-sa.md` or `04-sa-summary.md`). Cross-reference with the SE's plan to confirm what document name was chosen.
- **PASS**: DevOps and SA prompts reference only documents that are already produced during Phase 4 (i.e., `04-implementation-summary.md`, `04-test-report.md`, `04-infra-summary.md`).
- **FAIL**: Prompt references a document name that does not exist as a Phase 4 output.

### EC-4: No duplicate `## Cross-Review Notes` sections caused by prompt wording

- **Scenario**: If a cross-review prompt instructs an agent to "append" without checking whether the section exists, running the workflow twice could create duplicate sections. While runtime behavior is out of scope, the prompt wording itself should not instruct the agent to unconditionally overwrite.
- **File**: All three workflow files, all new cross-review prompts.
- **Check**: Verify that new prompts use "Append" (not "Replace" or "Overwrite") for the `## Cross-Review Notes` section instruction. This matches the pattern of all existing cross-review prompts.
- **PASS**: All new prompts use "Append".
- **FAIL**: Any prompt uses language that implies replacement of an existing section.

### EC-5: Phase 1.9 cross-review step does not reference Phase 4 documents

- **Scenario**: The Phase 1.9 cross-review operates on spec-review documents (`01-spec-arch-review.md`, `01-spec-infra-review.md`). There is a risk of accidentally referencing Phase 4 implementation documents (`04-*`) in this step due to copy-paste from the Phase 4 cross-review.
- **File**: `workflows/major.md`, Phase 1.9 new step 3.
- **Check**: Verify that Phase 1.9 step 3 prompts reference only `01-spec-arch-review.md` and `01-spec-infra-review.md`. No `04-*` document should appear in Phase 1.9 step 3.
- **PASS**: Only `01-spec-arch-review.md` and `01-spec-infra-review.md` are referenced.
- **FAIL**: Any `04-*` file path appears in Phase 1.9 step 3.

### EC-6: Phase 4 cross-review steps do not reference spec-review documents

- **Scenario**: The inverse of EC-5 — Phase 4 cross-review steps should reference Phase 4 implementation output documents only, not spec-review documents.
- **Files**: All three workflow files, Phase 4 new step 5.
- **Check**: Verify that Phase 4 step 5 prompts in all three files reference only `04-implementation-summary.md`, `04-test-report.md`, and (for major.md) `04-infra-summary.md`. No `01-*` document should appear.
- **PASS**: Only `04-*` file paths in Phase 4 step 5.
- **FAIL**: Any `01-*` path appears in Phase 4 step 5.

---

## Test Data Requirements

No external test data, fixtures, or mocks are needed. All validation reads the workflow files directly.

**Reference anchors** (used during validation to confirm correct placement):

- `workflows/patch.md`: Before-state step labels for Phase 4 (steps 1–7) and Phase 5 (steps 1–5) are documented in the SE discovery (lines 14–16) and confirmed by direct file read in this planning phase.
- `workflows/minor.md`: Before-state step labels for Phase 4 (steps 1–7) documented in SE discovery (line 16).
- `workflows/major.md`: Before-state step labels for Phase 1.9 (steps 1–2 + prose paragraph) and Phase 4 (steps 1–6) documented in SE discovery (lines 17–18) and TL discovery.
- Established cross-review prompt template (from patch.md Phase 2 step 3a/3b): documented in both QA and SE discovery documents. This is the canonical comparison string for AC-4 content checks.

---

## Cross-Review Notes

### Alignment with SE and TL Discovery

Both the SE and TL discovery documents are fully aligned with this plan. The five insertion points identified by the SE match the five gaps identified in the spec and referenced in the QA discovery. The TL's architectural recommendations have been incorporated into specific validation checks:

1. **Parallel for Phase 1.9**: The TL explicitly recommends Option A (parallel SA↔DevOps in Phase 1.9). Check V-5.4 validates that the new step uses a parallel sub-item structure, not sequential language. This replaces the ambiguity flagged in the QA discovery (Risk 5) with a definitive pass/fail criterion.

2. **Document choices for major.md Phase 4 SA/DevOps**: The TL resolves the "relevant SA context" ambiguity by recommending SA reads `04-infra-summary.md` and DevOps reads `04-implementation-summary.md`. Checks V-4.7, V-4.8, and EC-3 operationalize this. If the SE chose a different (but still valid) document combination, EC-3 provides a non-prescriptive fallback (any existing Phase 4 document name is acceptable; any non-existent document name is a FAIL).

3. **Dash convention**: The TL reinforces the em-dash vs. hyphen-dash distinction from SE discovery. Checks V-9.1, V-9.3, V-9.4 explicitly verify this per file.

4. **No wait step in patch.md Phase 4**: The TL notes patch.md Phase 4 does not have an explicit wait step between SE and QA. Check V-5.1 does not require a separate wait step — it only requires the new cross-review step be a parallel step. The absence of a "Wait for both agents to complete" step before the cross-review is acceptable for patch.md.

### Divergence Resolved: Phase 1.9 parallel vs. sequential

The QA discovery (Risk 5) and SE discovery identified a conflict over whether Phase 1.9 SA↔DevOps should be parallel or sequential. The TL's recommendation (Option A, parallel) is adopted as the authoritative decision. Check V-5.4 enforces this: sequential language in that step is a FAIL.

### Items the SE Should Confirm Before Implementation

The following decisions were open in the SE discovery and are now resolved by this plan or the TL document:

1. Phase 1.9 ordering: parallel (V-5.4 enforces this)
2. Major Phase 4 DevOps reads `04-implementation-summary.md`, SA reads `04-infra-summary.md` (V-4.7, V-4.8 check this; EC-3 provides the fallback)
3. The QA Final Reflection section heading is exactly `## QA Engineer - Final Reflection` (V-6.3 enforces this verbatim)

---

## Cross-Review Notes (SE Implementation Plan — 03-plan-se.md)

### Overall Testability Assessment

The SE's implementation plan is highly testable. All five changes are precisely specified as literal text insertions with exact `old_string`/`new_string` pairs. Each insertion point is identified by surrounding context that is unique in the file, and the exact replacement text is given verbatim. This means every validation check in this plan has an unambiguous expected string to look for — there are no implementation choices left to inference during validation.

### Confirmations: Open Questions Now Closed

The three items flagged as "Items the SE Should Confirm Before Implementation" at the end of the original Cross-Review Notes above are all resolved by the SE's plan:

1. **Phase 1.9 parallel structure confirmed**: The SE's Change 3 uses lettered sub-items `a.` and `b.` under a single numbered step, with no sequential language. Check V-5.4 (requires parallel structure) will PASS with this implementation.

2. **Document targets for major.md Phase 4 SA/DevOps confirmed**: The SE's Change 4 has SA reading `04-infra-summary.md` and appending to `04-implementation-summary.md`, and DevOps reading `04-implementation-summary.md` and appending to `04-infra-summary.md`. This matches what V-4.7 and V-4.8 check for exactly. EC-3 (no non-existent document names) will also PASS.

3. **QA Final Reflection heading confirmed**: The SE's Change 5 uses the exact heading `## QA Engineer - Final Reflection`. Check V-6.3 will PASS.

### Conditional Label Wording: One Check Requires Adjustment

The SE's plan uses `(only if DevOps is active)` on sub-items c. and d. of the major.md Phase 4 cross-review step. The SE notes in the Convention Compliance section that `(only if activated)` may be preferred for strict consistency with the Phase 2 pattern, but states either is acceptable.

Check V-3.3 in this plan already accommodates this: it accepts `(only if activated)` or `(only if DevOps is active)` or equivalent conditional language. No change to V-3.3 is needed. However, during implementation validation, the verifier must confirm which exact phrase was used and cross-check it against the Phase 2 pattern in major.md. If the SE used `(only if DevOps is active)` and Phase 2 uses `(only if activated)`, this is an inconsistency worth flagging as a non-blocking observation (not a FAIL unless the spec mandates exact matching).

**Adjustment to V-3.3**: Tighten the pass/fail boundary. Accept `(only if DevOps is active)` as a PASS. Flag `(only if activated)` as a PASS with a note for consistency review. Any other phrasing (e.g., `(if DevOps active)`, no label at all) is a FAIL.

### Phase 5 QA Reflection Prompt Opener: Confirm Against V-6.3

The SE's Change 5 prompt opens with `"You are completing a final reflection."` — not `"You are completing a cross-review."` The SE explicitly notes this is intentional, matching the Phase 5 context.

Check V-6.3 accounts for this: element 1 already says `"You are completing a cross-review."` with a parenthetical noting the opener may differ for this step. The SE's choice of `"You are completing a final reflection."` is consistent with the spec (AC-6 does not mandate a specific opener) and with the SE's convention compliance note. No check needs to be changed; the verifier just needs to remember that the `"You are completing a final reflection."` opener is correct for this step and must not be flagged as a mismatch.

### PR Split: Two Separate File Reads Required for Validation

The SE splits the five changes across two PRs: PR 1 covers `patch.md` Phase 4 and `minor.md` Phase 4; PR 2 covers `major.md` Phase 1.9, `major.md` Phase 4, and `patch.md` Phase 5.

This means `patch.md` is modified in both PRs. The validation must be performed against the final merged state (after both PRs land), not after PR 1 alone. If validation is run after PR 1 only, the following checks will FAIL even though nothing is wrong: V-NUM.2, V-6.1, V-6.2, V-6.3, V-10.2, RT-7, EC-1. This is not a defect in the SE's plan — it is just an ordering constraint the verifier needs to account for.

No adjustments to the test plan are required, but the implementation report must confirm validation is run against the fully merged state.

### Renumbering Scope Matches Checks

The SE's Risk Mitigation section enumerates every renumbered step explicitly:
- patch.md Phase 4: 5→6, 6→7, 7→8
- patch.md Phase 5: 3→4, 4→5, 5→6
- minor.md Phase 4: 5→6, 6→7, 7→8
- major.md Phase 4: 5→6, 6→7
- major.md Phase 1.9: no renumbering (append only)

This maps exactly to checks V-NUM.1 through V-NUM.5. There are no renumbered steps the SE identifies that lack a corresponding check, and no check covers a renumbering the SE does not perform.

### SA Prompt in major.md Phase 4 — "own document" Ambiguity

In the SE's Change 4 (major.md Phase 4), the SA sub-item (c.) instructs the SA to "read your own implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`". The SA does not own `04-implementation-summary.md` — that document belongs to the SE. The phrasing "your own implementation summary" is slightly misleading because the SA co-appends to it; the SA does not author it.

This is not a functional defect (the SA is correctly directed to the right document), but it is a potential source of confusion for the executing agent. Check V-4.7 validates the document path, not the phrasing, so this does not require a check change. It is noted here as an observation for the SE's awareness. If the SE wishes to improve clarity, the phrase could read "read the consolidated implementation summary" instead of "your own implementation summary". This is non-blocking.

### No New Concerns

All five changes are purely additive. The SE's risk mitigation section addresses every risk identified in the SE discovery (Risks 1–8). The implementation plan contains no changes to agent files, command files, or Phase 6 content, which means regression checks RT-1, RT-2, and RT-6 are expected to PASS without any issues.
