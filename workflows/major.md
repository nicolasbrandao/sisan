# Major Workflow

Large features, new systems, cross-service changes (4+ PRs, 15+ files, 1+ services). Up to 7 agents: PM, Software Architect, Tech Lead, SE, QA, optionally UI/UX, optionally DevOps, optionally Database Architect.

**Context variables** (from Phase 0):
- `SPEC_FOLDER` · `USER_REQUEST` · `TRIAGE_RESULT` · `TIER` (= `major`) · `BRANCH_NAME` · `BASE_BRANCH` · `CYCLE_CONTEXT` · `BRANCHING_STRATEGY`

**Standing instruction for every agent prompt below**: include the directive `Do not restate the spec or prior docs. Reference section IDs only. Lead with results.`

---

## Phase 1: Specification

**Inputs Manifest**: `USER_REQUEST`, `TRIAGE_RESULT`

### Steps

1. Launch `pm` SPECIFICATION (`{MODEL_STANDARD}`):
   - Prompt: "SPECIFICATION mode. Tier: major. Be thorough on ALL `Affected Areas` subsections — UI (triggers UI/UX), Database (triggers DB Architect), Infrastructure (triggers DevOps), Cross-Service (informs SA depth). Mark `None` if not applicable. Detail delivery strategy. Request: {USER_REQUEST}. Triage: {TRIAGE_RESULT}. Write to `{SPEC_FOLDER}/01-spec.md`."

2. Read `01-spec.md`.

3. Present summary. Highlight activation flags: **UI active?** **DB active?** **DevOps active?** **SA always active.**

4. **User checkpoint**.

---

## Phase 1.5: UI/UX Spec Review (CONDITIONAL)

**Activation**: spec `Affected Areas > UI` non-empty.

**Inputs Manifest**: `01-spec.md`

1. Launch `ui-ux-specialist` SPEC REVIEW (`{MODEL_STANDARD}`) → `01-spec-ui-review.md`.
2. Read, present, **user checkpoint**.

---

## Phase 1.6: Database Spec Review (CONDITIONAL)

**Activation**: spec `Affected Areas > Database` non-empty.

**Inputs Manifest**: `01-spec.md` (+ UI review if active)

1. Launch `database-architect` SPEC REVIEW (`{MODEL_STANDARD}`) → `01-spec-db-review.md`.
2. Read, present, **user checkpoint**.

---

## Phase 1.7: Architecture Spec Review (ALWAYS ACTIVE)

**Inputs Manifest**: `01-spec.md` (+ UI review if active, + DB review if active)

1. Launch `software-architect` SPEC REVIEW (`{MODEL_THOROUGH}`):
   - Prompt: "SPEC REVIEW mode. Map system topology. Identify affected services, data ownership, contract changes, cross-service flow, BC concerns. Draft initial ADRs. Recommend PR phasing. Write to `{SPEC_FOLDER}/01-spec-arch-review.md`."

2. Read. Present (topology summary, affected services, data ownership, contracts, BC, ADR drafts, PR phasing). If SA recommends re-scope: highlight.

3. **User checkpoint** — this shapes all subsequent phases.

---

## Phase 1.9: DevOps Spec Review (CONDITIONAL)

**Activation**: spec `Affected Areas > Infrastructure` non-empty.

**Inputs Manifest**: `01-spec.md`, `01-spec-arch-review.md`

### Steps (only if activated)

1. Launch `devops-engineer` SPEC REVIEW (`{MODEL_STANDARD}`) → `01-spec-infra-review.md`.

2. Read, present (combined with Phase 1.7 results), **user checkpoint**.

3. Cross-review round in parallel (`{MODEL_FAST}`):

   a. SA reads infra review; appends `## Cross-Review Notes` to `01-spec-arch-review.md`.
   b. DevOps reads SA review; appends `## Cross-Review Notes` to `01-spec-infra-review.md`.

**Track active states (UI, DB, DevOps).**

---

## Phase 2: Discovery

**Inputs Manifest**
- SE / QA: `01-spec.md`, all spec reviews (UI, arch, DB, infra) that are active
- SA: `01-spec.md`, `01-spec-arch-review.md` (+ DB review if active)
- UI/UX (active only): `01-spec.md`, `01-spec-ui-review.md`
- DevOps (active only): `01-spec.md`, `01-spec-arch-review.md`, `01-spec-infra-review.md`
- DB Architect (active only): `01-spec.md`, `01-spec-db-review.md`, `01-spec-arch-review.md` (+ infra review if active)
- TL (sequential, after cross-reviews): all of the above + all discovery docs + cross-review notes

### Steps

1. Launch in parallel (`{MODEL_STANDARD}` for SE/QA/UI/DevOps/DBA, `{MODEL_THOROUGH}` for SA, 3–6 agents):

   a. `software-engineer` DISCOVERY → `02-discovery-se.md` (must conform to SA's contracts)
   b. `qa-engineer` DISCOVERY → `02-discovery-qa.md`
   c. `software-architect` DISCOVERY → `02-discovery-sa.md`
   d. `ui-ux-specialist` DISCOVERY (if UI active) → `02-discovery-ui.md`
   e. `devops-engineer` DISCOVERY (if DevOps active) → `02-discovery-devops.md`
   f. `database-architect` DISCOVERY (if DB active) → `02-discovery-db.md`

2. Cross-review round in parallel (`{MODEL_FAST}`, 3–6):

   a. SE reads QA + SA discovery; appends to `02-discovery-se.md`.
   b. QA reads SE discovery; appends to `02-discovery-qa.md`.
   c. SA reads SE (+ DevOps if active, + DB if active); appends to `02-discovery-sa.md`.
   d. (UI active) UI reads SE; appends to `02-discovery-ui.md`.
   e. (DevOps active) DevOps reads SE + SA; appends to `02-discovery-devops.md`.
   f. (DB active) DB reads SE + SA; appends to `02-discovery-db.md`.

3. Launch `tech-lead` DISCOVERY REVIEW sequentially (`{MODEL_STANDARD}`) → `02-discovery-tl.md`.

4. Read all discovery docs.

5. Present summary (SA system findings, DevOps infra findings, DB findings if active, TL code-level concerns, cross-review highlights). Highlight any re-scope recommendation.

6. **User checkpoint**.

---

## Phase 3: Planning

**Inputs Manifest**
- SE / QA / DB / DevOps: `01-spec.md`, all spec reviews active, all discovery docs available
- SA: same as above
- TL (sequential, after cross-reviews): all of the above + all plans + all cross-review notes

### Steps

1. Launch in parallel (`{MODEL_STANDARD}` for SE/QA/DevOps/DBA, `{MODEL_THOROUGH}` for SA, 3–5 agents):

   a. `software-engineer` PLANNING. Conform to SA's API contracts. Address TL recommendations. Include PR Strategy. Note: infra code is DevOps's; entity/migration code is DB Architect's. → `03-plan-se.md`
   b. `qa-engineer` PLANNING. Map all AC + cross-service integration tests per SA's requirements. → `03-plan-qa.md`
   c. `software-architect` PLANNING. Define API contracts, interaction patterns, migration strategy, ADRs. Will review SE plan after it's available. → `03-plan-sa.md`
   d. `devops-engineer` PLANNING (if active). → `03-plan-devops.md`
   e. `database-architect` PLANNING (if active). → `03-plan-db.md`

2. Cross-review round in parallel (`{MODEL_FAST}`, 3–6):

   a. SE reads QA plan; appends `## Cross-Review Notes` to `03-plan-se.md`.
   b. QA reads SE plan; appends to `03-plan-qa.md`.
   c. (UI active) UI reviews SE plan against `01-spec-ui-review.md` → `03-plan-ui-review.md`.
   d. SA reviews SE plan for system compliance; appends `## SE Plan Review Notes` + `## Verdict` to `03-plan-sa.md`.
   e. (DevOps active) DevOps reviews SE plan; appends `## SE Plan Review Notes` to `03-plan-devops.md`.
   f. (DB active) DB Architect reviews SE plan; appends `## SE Plan Review Notes` + `## Cross-Review Notes` to `03-plan-db.md`.

3. Launch `tech-lead` PLANNING REVIEW sequentially (`{MODEL_STANDARD}`) → `03-plan-tl.md`. Code-level verdict.

4. Read SA verdict (`03-plan-sa.md` §Verdict) and TL verdict (`03-plan-tl.md` §Verdict).

5. Handle dual verdicts (most restrictive wins):
   - **Both APPROVED**: proceed.
   - **SA APPROVED WITH CHANGES**: SE revises (system-level); SA re-reviews.
   - **TL APPROVED WITH CHANGES**: SE revises (code-level); TL re-reviews.
   - **Either REJECTED**: ask user — revise | override (warn) | stop.
   - **Both have issues**: address most restrictive first; both must be satisfied.

6. Present all plans (SE, QA, SA, TL, DevOps if active, UI if active, DB if active).

7. **User checkpoint** — **CRITICAL**: do NOT start without explicit approval.

---

## Phase 4: Implementation

**Inputs Manifest** (per sub-PR)
- SE: `01-spec.md`, `03-plan-se.md`, `03-plan-sa.md` (API contracts), scaffold/RED summary as appropriate
- DevOps: `01-spec.md`, `03-plan-devops.md`, scope description
- DB Architect: `01-spec.md`, `03-plan-db.md`, `01-spec-db-review.md`, `03-plan-sa.md`, `04-implementation-summary.md`
- QA: `03-plan-qa.md`, `01-spec.md`, all impl summaries that exist

### Steps

1. Confirm user approval.

2. Determine sub-PR structure and phasing from `03-plan-se.md` PR Strategy + `03-plan-sa.md` PR phasing. Map to phases A (Foundation) → B (Services) → C (Integration) → D (Infra). Determine within-phase parallelism.

3. **For each phase A→D, for each sub-PR within the phase**:

   a. Create sub-branch `{BRANCH_NAME}/pr-{N}-{description}` from `{BRANCH_NAME}`.

   b. Determine implementer:
      - **App PR**: SE
      - **Infra PR** (if DevOps active): DevOps
      - **Mixed PR**: SE first, then DevOps (same branch)
      - **DB-layer PR** (if DB active): DB Architect
      - **Mixed with DB**: SE first, then DB Architect (same branch)

   c. Launch SE IMPLEMENTATION (for app PRs) (`{MODEL_STANDARD}`):
      - Prompt: "IMPLEMENTATION mode. PR {N}/{total}, Phase {phase}, scope: {PR scope}. Conform to SA's API contracts. Infra/DB layer code is NOT yours. Append PR section to `04-implementation-summary.md`."

   d. Launch DevOps IMPLEMENTATION (for infra PRs, if active) (`{MODEL_STANDARD}`):
      - Prompt: "IMPLEMENTATION mode. PR {N}/{total}, Phase {phase}, scope: {PR scope}. Append PR section to `04-infra-summary.md`."

   e. Launch DB Architect IMPLEMENTATION (for DB-layer PRs, if active) (`{MODEL_STANDARD}`):
      - Prompt: "IMPLEMENTATION mode. PR {N}/{total}, Phase {phase}, scope: {PR scope}. Append PR section to `04-db-summary.md`."

   f. Launch QA IMPLEMENTATION (after SE/DevOps/DB complete or in parallel if safe) (`{MODEL_STANDARD}`):
      - Prompt: "IMPLEMENTATION mode. PR {N}/{total}, scope: {PR scope}. Reference SA's integration test requirements for cross-service cases. Run suite. Append PR section to `04-test-report.md`."

   g. Commit. Message: `feat: {PR description} (specs/{id}-{slug}, PR {N}/{total})`.

   h. Intermediate checkpoint (if not last sub-PR).

4. After all sub-PRs:
   - Ensure `04-implementation-summary.md`, `04-test-report.md`, and `04-infra-summary.md` (if DevOps active) and `04-db-summary.md` (if DB active) are consolidated.

5. Cross-review round in parallel (`{MODEL_FAST}`, 2 or 4):

   a. SE reads `04-test-report.md`; appends to `04-implementation-summary.md`.
   b. QA reads `04-implementation-summary.md`; appends to `04-test-report.md`.
   c. (DevOps active) SA reads infra summary; appends architecture-alignment notes to `04-implementation-summary.md`.
   d. (DevOps active) DevOps reads `04-implementation-summary.md`; appends deployment notes to `04-infra-summary.md`.

6. Present per-phase summary, per-PR summary, consolidated totals.

7. **User checkpoint**.

---

## Phase 5: Quality Gates

**Inputs Manifest** (each reviewer reads what their domain requires)

### Steps

1. Launch in parallel (`{MODEL_STANDARD}`):

   a. `qa-engineer` QUALITY GATE (writes base): `01-spec.md`, all spec reviews active, `04-implementation-summary.md`, `04-tdd-red-report.md` (if exists), `04-test-report.md`, `04-infra-summary.md` if DevOps active. Reference SA integration requirements for cross-service. Write `05-quality-gate.md`.
   b. `software-engineer` QUALITY GATE (test review): `04-tdd-red-report.md` (if exists), `04-test-report.md`, test files. Write `05-quality-gate-se.md`.

2. After both complete, launch remaining reviewers in parallel (`{MODEL_STANDARD}` for TL/UI/DevOps, `{MODEL_THOROUGH}` for SA, 2–4 agents):

   a. `tech-lead` QUALITY GATE (architecture, code-level): `04-implementation-summary.md`, `03-plan-se.md`, `03-plan-tl.md`, `03-plan-sa.md`, modified files. Append `## Tech Lead — Architecture Review` to `05-quality-gate.md`.
   b. `software-architect` QUALITY GATE (system integration): `03-plan-sa.md`, `04-implementation-summary.md`, `04-infra-summary.md` if DevOps active, `04-test-report.md`, integration code. Append `## Software Architect — System Integration Review` to `05-quality-gate.md`.
   c. `ui-ux-specialist` QUALITY GATE (if UI active): `01-spec-ui-review.md`, `04-implementation-summary.md`, UI code. Append `## UI/UX Specialist — Design Review` to `05-quality-gate.md`.
   d. `devops-engineer` QUALITY GATE (if active): `03-plan-devops.md`, `04-infra-summary.md`, `04-implementation-summary.md`, infra code. Append `## DevOps Engineer — Deployment Readiness Review` to `05-quality-gate.md`.

3. After all reviewers complete, append SE test review section to `05-quality-gate.md` (copy from `05-quality-gate-se.md`), then append `## QA Engineer — Final Reflection` (`{MODEL_FAST}`).

4. Read full quality gate report.

5. Present consolidated verdict (most restrictive across reviewers).

6. Handle verdict:
   - **All APPROVED/ADEQUATE**: → Phase 6.
   - **WITH CONDITIONS / non-blocking NEEDS IMPROVEMENT**: present; ask user — address now | accept and merge | stop.
   - **REJECTED / blocking NEEDS IMPROVEMENT**: present; ask user — fix (loop to Phase 4) | stop | override (warn).

---

## Phase 6: Merge

### Steps

1. **Assemble PR summary** (deterministic — extract, do not paraphrase, no agent invocation):

   Create `{SPEC_FOLDER}/06-pr-summary.md`:

   ```markdown
   # [spec title]

   ## Summary
   [Copy spec §Summary verbatim]

   ## Changes
   [Copy 04-implementation-summary.md §Changes — consolidated across sub-PRs]

   ## Infrastructure Changes
   [Copy 04-infra-summary.md §Changes if DevOps active, else "N/A"]

   ## Tests
   [Copy 04-test-report.md §Run + §AC Coverage]

   ## Quality Gate
   - **QA Verdict:** [from §Verdict]
   - **SE Test Review:** [verdict line]
   - **TL Architecture:** [verdict line]
   - **SA System Integration:** [verdict line]
   - **UI/UX Design:** [verdict line | N/A]
   - **DevOps Deployment Readiness:** [verdict line | N/A]

   ## ADRs
   [List ADR titles from 03-plan-sa.md §ADRs]

   ## AC Status
   [Copy 05-quality-gate.md §AC Validation table]

   ## Deployment Plan
   [Copy 03-plan-devops.md §Deployment Strategy summary if active, else "Standard deployment"]

   ## Branch
   `{BRANCH_NAME}` (epic) → `{BASE_BRANCH}`
   **Sub-PRs:** [list with phase grouping]

   ## Spec Folder
   `{SPEC_FOLDER}`
   ```

   Target: ≤180 lines. No re-narration; no new commentary.

2. **Handle sub-PR branches** (phased):
   For each `{BRANCH_NAME}/pr-N-{description}`:
   - Push: `git push -u origin {sub-branch}`.
   - Create PR: `gh pr create --base {BRANCH_NAME} --title "{PR title}" --body "{PR description}"`.
   - **GitHub merge checkpoint** — wait for user:

     ```
     ⏸ GitHub Merge Required
     Sub-PR #{N} created: {PR URL}
     Reply: "merged" / "done" → next · "changes requested" → handle review
     ```

     **If "changes requested"**: loop until merge.
     a. Launch `software-engineer` PR REVIEW RESPONSE mode. Append to `06-pr-review-response-{N}.md`.
     b. Read, present, address comments.
     c. Return to checkpoint.

   Group PR links by phase.

3. **Handle the epic branch** (final PR):
   - Ask user — Option A (create final PR now) or Option B (wait for sub-PRs to merge first).
   - `gh pr create --base {BASE_BRANCH} --title "{spec title}" --body "$(cat {SPEC_FOLDER}/06-pr-summary.md)"`.
   - **GitHub merge checkpoint** for final PR — same loop as sub-PRs.

4. Stage and commit the spec folder (audit trail + ADRs): `docs: add spec audit trail and ADRs for {id}-{slug}`. Push.

5. Present results: PR links by phase, deployment plan summary, remaining cycles.

6. **Mark all todos complete.**

---

### Branching Rules

- **Major (single-phase)**: `epic/{id}-{slug}` → PR to `{BASE_BRANCH}`.
- **Major (multi-phase)**: epic branch `epic/{id}-{slug}`. Sub-branches `epic/{id}-{slug}/pr-N-{desc}` → PRs to `epic/{id}-{slug}`. Final PR: `epic/{id}-{slug}` → `{BASE_BRANCH}`.
- **NEVER** push directly to shared branches unless explicitly requested.

---

## Error Recovery

- **Agent fails**: inform user; offer re-launch or manual.
- **SA rejects plan**: SE revises; SA re-reviews. Override via user.
- **TL rejects plan**: SE revises; TL re-reviews. Override via user.
- **Both reject**: address both sets of feedback before re-review.
- **Tests fail**: document; discuss at checkpoint.
- **Quality gate rejects**: loop to Phase 4 with specific fixes.
- **Cross-service integration issues**: SA quality gate catches these; may require multi-PR changes.
- **Infrastructure issues**: DevOps quality gate catches these; may require infra PR revisions.
- **DevOps finds blocker late**: present; if blocking, loop back; may require SA re-assessment.
- **User wants to stop**: spec folder preserved; resume later.
