# Minor Workflow

Small features, enhancements, refactors, new endpoints, new UI components (1–3 PRs, 5–15 files, single service). 5 agents: PM, Tech Lead, SE, QA, optionally UI/UX, optionally Database Architect.

**Context variables** (from Phase 0):
- `SPEC_FOLDER` · `USER_REQUEST` · `TRIAGE_RESULT` · `TIER` (= `minor`) · `BRANCH_NAME` · `BASE_BRANCH` · `CYCLE_CONTEXT` · `BRANCHING_STRATEGY`

**Standing instruction for every agent prompt below**: include the directive `Do not restate the spec or prior docs. Reference section IDs only. Lead with results.`

---

## Phase 1: Specification

**Inputs Manifest**
- PM (specification): `USER_REQUEST`, `TRIAGE_RESULT`

### Steps

1. Launch `pm` in SPECIFICATION mode (`{MODEL_STANDARD}`):
   - Prompt: "SPECIFICATION mode. Tier: minor. Be thorough on `Affected Areas > UI` (triggers UI/UX) and `Affected Areas > Database` (triggers DB Architect) — describe or mark `None`. Detail delivery strategy (PR count, what each covers). Request: {USER_REQUEST}. Triage: {TRIAGE_RESULT}. Write to `{SPEC_FOLDER}/01-spec.md`."

2. Read `01-spec.md`.

3. Present summary (title, type, priority, AC count, PR strategy, open questions). Highlight: **UI active?** **DB active?**

4. **User checkpoint**.

---

## Phase 1.5: UI/UX Spec Review (CONDITIONAL)

**Activation**: spec `Affected Areas > UI` non-empty (not "None"/"N/A"/empty). Otherwise SKIP.

**Inputs Manifest**: `01-spec.md`

### Steps (only if activated)

1. Launch `ui-ux-specialist` in SPEC REVIEW mode (`{MODEL_STANDARD}`). Write to `01-spec-ui-review.md`.

2. Read `01-spec-ui-review.md`. Present: UI AC, accessibility, components to reuse.

3. **User checkpoint**.

**Track UI active state.**

---

## Phase 1.6: Database Spec Review (CONDITIONAL)

**Activation**: spec `Affected Areas > Database` non-empty. Otherwise SKIP.

**Inputs Manifest**: `01-spec.md` (+ `01-spec-ui-review.md` if UI active)

### Steps (only if activated)

1. Launch `database-architect` in SPEC REVIEW mode (`{MODEL_STANDARD}`). Write to `01-spec-db-review.md`.

2. Read `01-spec-db-review.md`. Present: schema requirements, migration safety, index strategy.

3. **User checkpoint**.

**Track DB active state.**

---

## Phase 2: Discovery

**Inputs Manifest** (each agent reads only what's listed)
- SE / QA: `01-spec.md` (+ UI review if active, + DB review if active)
- UI/UX (active only): `01-spec.md`, `01-spec-ui-review.md`
- DB Architect (active only): `01-spec.md`, `01-spec-db-review.md` (+ UI review if active)
- TL (sequential, after cross-reviews): `01-spec.md`, all spec reviews, all discovery docs

### Steps

1. Launch in parallel (`{MODEL_STANDARD}`, 2–4 agents):

   a. `software-engineer` DISCOVERY → `02-discovery-se.md`
   b. `qa-engineer` DISCOVERY → `02-discovery-qa.md`
   c. `ui-ux-specialist` DISCOVERY (if UI active) → `02-discovery-ui.md`
   d. `database-architect` DISCOVERY (if DB active) → `02-discovery-db.md`

2. Cross-review round in parallel (`{MODEL_FAST}`):

   a. SE reads QA's; appends `## Cross-Review Notes` to `02-discovery-se.md`.
   b. QA reads SE's; appends `## Cross-Review Notes` to `02-discovery-qa.md`.
   c. (if UI active) UI reads SE's; appends `## Cross-Review Notes` to `02-discovery-ui.md`.
   d. (if DB active) DB reads SE's; appends `## Cross-Review Notes` to `02-discovery-db.md`.

3. Launch `tech-lead` DISCOVERY REVIEW sequentially (`{MODEL_STANDARD}`) → `02-discovery-tl.md`.

4. Read all discovery docs.

5. Present summary (key findings per agent, cross-review highlights, TL recommendations). If TL recommends re-triage: highlight prominently.

6. **User checkpoint**.

---

## Phase 3: Planning

**Inputs Manifest**
- SE: `01-spec.md`, all spec reviews, all discovery docs (own + others available)
- QA: same as SE
- DB Architect (active only): same
- TL (sequential, after cross-reviews): all of the above + all plans + all cross-review notes

### Steps

1. Launch in parallel (`{MODEL_STANDARD}`, 2–3 agents):

   a. `software-engineer` PLANNING. Address TL recommendations from discovery review. Include PR Strategy. → `03-plan-se.md`
   b. `qa-engineer` PLANNING. Map every AC (incl. UI AC if active) to test cases. → `03-plan-qa.md`
   c. `database-architect` PLANNING (if DB active). → `03-plan-db.md`

2. Cross-review round in parallel (`{MODEL_FAST}`, 2–4):

   a. SE reads QA plan (and DB plan if active); appends `## Cross-Review Notes` to `03-plan-se.md`.
   b. QA reads SE plan; appends `## Cross-Review Notes` to `03-plan-qa.md`.
   c. (if UI active) `ui-ux-specialist` reviews SE plan against `01-spec-ui-review.md` → `03-plan-ui-review.md`.
   d. (if DB active) DB Architect reads SE plan; appends `## SE Plan Review Notes` to `03-plan-db.md`.

3. Launch `tech-lead` PLANNING REVIEW sequentially (`{MODEL_STANDARD}`) → `03-plan-tl.md`. Verdict can block.

4. Read `03-plan-tl.md`. Handle verdict:

   - **APPROVED**: proceed.
   - **APPROVED WITH CHANGES**: present required changes; ask user → revise (re-launch SE, then re-run TL review) or override.
   - **REJECTED**: present rejection + path forward; ask user → revise | override (warn) | stop.

5. Present all plans (SE approach, PR count; QA tests, AC coverage; TL verdict; UI plan review if active; DB plan if active).

6. **User checkpoint** — **CRITICAL**: do NOT start implementation without explicit approval.

---

## Phase 4: Implementation

**Inputs Manifest** (per sub-PR)
- SE / QA / DB: `01-spec.md`, all relevant spec reviews, own plan, scaffold/RED summary as appropriate

### Steps

1. Confirm user approval.

2. Determine sub-PR structure from `03-plan-se.md` PR Strategy.
   - 1 PR → behaves like patch workflow (single branch).
   - 2–3 PRs → iterate sequentially.

3. **For each sub-PR** (sequential):

   a. Create sub-branch `{BRANCH_NAME}/pr-{N}-{description}` from `{BRANCH_NAME}`. (Single PR: skip — work on `{BRANCH_NAME}` directly.)

   b. Launch `software-engineer` IMPLEMENTATION mode, **SCAFFOLD PHASE** (`{MODEL_STANDARD}`):
      - Prompt: "SCAFFOLD PHASE. {Multi-PR: 'PR {N}/{total}, scope: {PR scope}.'} Stubs only — no business logic. Project must compile. Append PR section to `04-scaffold-summary.md`."

   c. (DB active only) Launch `database-architect` IMPLEMENTATION (`{MODEL_STANDARD}`) after SE scaffold:
      - Prompt: "IMPLEMENTATION mode. {Multi-PR: 'PR {N}/{total}.'} DB layer files only (entities, migrations, raw queries, ORM config). Append PR section to `04-db-summary.md`."

   d. Launch `qa-engineer` IMPLEMENTATION mode, **TDD-RED PHASE** (`{MODEL_STANDARD}`):
      - Prompt: "TDD-RED PHASE. {Multi-PR: 'PR {N}/{total}, scope: {PR scope}.'} Write all planned tests against scaffolds. Run suite. Verify RED state. Append PR section to `04-tdd-red-report.md`."

   e. Launch `software-engineer` IMPLEMENTATION mode, **TDD-GREEN PHASE** (`{MODEL_STANDARD}`):
      - Prompt: "TDD-GREEN PHASE. {Multi-PR: 'PR {N}/{total}.'} Implement business logic. Goal: every RED → GREEN, no regressions. Run full suite. Append PR section to `04-implementation-summary.md`."

   f. Commit changes. Message: `feat: {PR description} (specs/{id}-{slug}, PR {N}/{total})`.

   g. Intermediate checkpoint (if not last sub-PR): present results, ask user to continue.

4. After all sub-PRs:
   - Ensure `04-implementation-summary.md`, `04-test-report.md`, and `04-db-summary.md` (if active) are consolidated.

5. Cross-review round in parallel (`{MODEL_FAST}`):

   a. SE reads `04-test-report.md`; appends `## Cross-Review Notes` to `04-implementation-summary.md`.
   b. QA reads `04-implementation-summary.md`; appends `## Cross-Review Notes` to `04-test-report.md`.

6. Read both. Present per-PR summary + consolidated totals.

7. **User checkpoint**.

---

## Phase 5: Quality Gates

**Inputs Manifest** (each reviewer reads what their domain requires; full list per agent below)

### Steps

1. Launch in parallel (`{MODEL_STANDARD}`):

   a. `qa-engineer` QUALITY GATE (writes base report): `01-spec.md`, all spec reviews, `04-implementation-summary.md`, `04-tdd-red-report.md`, `04-test-report.md`, modified files. Write `05-quality-gate.md`.
   b. `software-engineer` QUALITY GATE (test review): `04-tdd-red-report.md`, `04-test-report.md`, test files. Write `05-quality-gate-se.md`.

2. After both complete, launch remaining reviewers in parallel (`{MODEL_STANDARD}`, 1–3 agents):

   a. `tech-lead` QUALITY GATE (architecture): `04-implementation-summary.md`, `03-plan-se.md`, `03-plan-tl.md`, modified files. Append `## Tech Lead — Architecture Review` to `05-quality-gate.md`.
   b. `ui-ux-specialist` QUALITY GATE (if UI active): `01-spec-ui-review.md`, `04-implementation-summary.md`, UI code changes. Append `## UI/UX Specialist — Design Review` to `05-quality-gate.md`.
   c. `database-architect` QUALITY GATE (if DB active): `01-spec-db-review.md`, `03-plan-db.md`, `04-db-summary.md`, DB layer files. Append `## Database Architect — Schema & Query Review` to `05-quality-gate.md`.

3. After all reviewers complete, append SE test review section to `05-quality-gate.md` (copy from `05-quality-gate-se.md`).

4. Read full quality gate report.

5. Present consolidated verdict (most restrictive across reviewers): QA verdict, AC results, SE test review, TL architecture, UI/UX (if active), DB Architect (if active).

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

   ## Tests
   [Copy 04-test-report.md §Run + §AC Coverage]

   ## Quality Gate
   - **QA Verdict:** [from 05-quality-gate.md §Verdict]
   - **SE Test Review:** [verdict line]
   - **TL Architecture:** [verdict line]
   - **UI/UX Design:** [verdict line | N/A]
   - **DB Schema & Query:** [verdict line | N/A]

   ## AC Status
   [Copy 05-quality-gate.md §AC Validation table]

   ## Branch
   `{BRANCH_NAME}` → `{BASE_BRANCH}`
   **Sub-PRs:** [list, if multi-PR]

   ## Spec Folder
   `{SPEC_FOLDER}`
   ```

   Target: ≤120 lines. No re-narration; no new commentary.

2. **Handle sub-PR branches** (if multi-PR):
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
     a. Launch `software-engineer` PR REVIEW RESPONSE mode (fetch via `gh pr view {N} --comments` and `gh pr diff {N}`; address each comment; commit and push). Append to `06-pr-review-response-{N}.md`.
     b. Read response. Present comments + resolutions.
     c. If clarification needed: ask user, re-launch SE.
     d. Return to checkpoint.

3. **Handle the feature branch** (single PR or final PR):
   - Single PR: push `{BRANCH_NAME}`, create PR targeting `{BASE_BRANCH}`.
   - Multi-PR: ask user — Option A (create final PR now) or Option B (wait for sub-PRs to merge first).
   - `gh pr create --base {BASE_BRANCH} --title "{spec title}" --body "$(cat {SPEC_FOLDER}/06-pr-summary.md)"`.
   - **GitHub merge checkpoint** for final PR — same loop as sub-PRs.

4. Stage and commit the spec folder: `docs: add spec audit trail for {id}-{slug}`. Push.

5. Present results: PR links, branch mapping, totals. If multi-cycle: remind user of remaining cycles.

6. **Mark all todos complete.**

---

### Branching Rules

- **Minor (single PR)**: `minor/{id}-{slug}` → PR to `{BASE_BRANCH}`.
- **Minor (multi-PR)**: feature branch `minor/{id}-{slug}` stays. Sub-branches `minor/{id}-{slug}/pr-N-{desc}` → PRs to `minor/{id}-{slug}`. Final PR: `minor/{id}-{slug}` → `{BASE_BRANCH}`.
- **NEVER** push directly to shared branches unless explicitly requested.

---

## Error Recovery

- **Agent fails**: inform user; offer re-launch or manual.
- **TL rejects plan**: expected; SE revises, TL re-reviews. User can override.
- **Tests fail**: document; discuss at checkpoint.
- **Quality gate rejects**: loop to Phase 4 with specific fixes.
- **Sub-PR has issues mid-stream**: address that PR; later PRs may need re-implementation if earlier ones change.
- **User wants to stop**: spec folder preserved; resume later.
- **UI/UX finds issues late in quality gate**: present to user; loop if blocking, follow-up if not.
