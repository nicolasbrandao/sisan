# Major Workflow

This workflow handles large features, new systems, cross-service changes, and infrastructure-level work (4+ PRs, 15+ files, 1+ services). It uses up to 7 agents: PM, Software Architect, Tech Lead, Software Engineer, QA Engineer, and optionally UI/UX Specialist and DevOps Engineer.

**Context variables** (provided by the orchestrator from Phase 0):
- `SPEC_FOLDER`: Path to the spec folder (e.g., `./specs/003-new-auth-system/`)
- `USER_REQUEST`: The user's request with clarifications
- `TRIAGE_RESULT`: PM's triage classification and rationale
- `TIER`: `major`
- `BRANCH_NAME`: The epic branch for this workflow (e.g., `epic/003-new-auth-system`)
- `BASE_BRANCH`: The branch the final PR targets (e.g., `main`)
- `CYCLE_CONTEXT`: Multi-cycle context if applicable
- `BRANCHING_STRATEGY`: The full branching plan from triage

---

## Phase 1: Specification

**Goal**: Create a comprehensive spec that addresses cross-service concerns, infrastructure needs, and UI changes.

### Steps:

1. **Launch the `pm` agent in SPECIFICATION mode**:
   - Prompt: "You are in SPECIFICATION mode. Create a detailed specification for the following request. Explore the codebase to understand the affected area, identify risks, and define precise acceptance criteria. Write the spec to `{SPEC_FOLDER}/01-spec.md`. The workflow tier is `major` — this is a large feature or cross-service change. Be thorough with ALL 'Affected Areas' subsections: (1) 'UI' — describe any UI changes clearly, as this determines whether a UI/UX Specialist will be activated. (2) 'Infrastructure' — describe any CI/CD, deployment, monitoring, or scaling changes, as this determines whether a DevOps Engineer will be activated. (3) 'Cross-Service' — describe any inter-service communication, shared data, or API contract changes, as this informs the Software Architect's review depth. Mark any section as 'None' if not applicable. Also detail the delivery strategy — how many PRs are recommended, what each covers, and any ordering dependencies between them. Request: {USER_REQUEST}. Triage context: {TRIAGE_RESULT}"

2. **Read the spec**: After the PM agent completes, read `{SPEC_FOLDER}/01-spec.md`.

3. **Present the spec to the user**:
   - Show: title, type, priority, summary, acceptance criteria count, PR decomposition strategy.
   - Highlight activation flags:
     - **UI changes detected**: UI/UX Specialist will be activated (or "No UI changes — UI/UX Specialist will be skipped")
     - **Infrastructure changes detected**: DevOps Engineer will be activated (or "No infra changes — DevOps Engineer will be skipped")
     - **Cross-service scope**: Software Architect will review (always active in major)
   - If there are open questions, ask the user to answer them.

4. **User checkpoint**: Ask the user to approve the spec or request changes.

---

## Phase 1.5: UI/UX Spec Review (CONDITIONAL)

**Goal**: Augment the spec with UI-specific acceptance criteria, accessibility requirements, and design system guidance.

### Activation Check:

1. **Read the spec** `{SPEC_FOLDER}/01-spec.md`.
2. **Check `Affected Areas > UI`**: If empty/"None"/"N/A" → SKIP. Otherwise → ACTIVATE.

### Steps (only if activated):

1. **Launch `ui-ux-specialist` in SPEC REVIEW mode**:
   - Prompt: "You are in SPEC REVIEW mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`. Explore the codebase to understand the existing design system, component library, CSS methodology, accessibility patterns, and responsive breakpoints. Augment the spec with UI-specific acceptance criteria, accessibility requirements, responsive behavior expectations, and design system components to reuse. Write your review to `{SPEC_FOLDER}/01-spec-ui-review.md`."

2. **Read the UI spec review**. Present to user. **User checkpoint**.

---

## Phase 1.7: Architecture Spec Review (ALWAYS ACTIVE)

**Goal**: The Software Architect maps the system topology, identifies affected services, defines data ownership, and flags cross-service concerns before discovery begins. This establishes the system context for all agents.

### Steps:

1. **Launch `software-architect` in SPEC REVIEW mode**:
   - Prompt: "You are in SPEC REVIEW mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ' and the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}. Explore the codebase to map the current system topology — services, databases, message queues, API gateways, shared libraries. Identify affected services, data ownership boundaries, API contracts that need to change or be created, cross-service data flow, and backward compatibility concerns. Draft initial Architecture Decision Records (ADRs) for significant decisions. Recommend a PR phasing strategy based on service dependencies. Write your review to `{SPEC_FOLDER}/01-spec-arch-review.md`."

2. **Read the architecture spec review**: After the SA completes, read `{SPEC_FOLDER}/01-spec-arch-review.md`.

3. **Present findings to user**:
   - System topology summary and affected services
   - Data ownership implications
   - API contracts that will change or be created
   - Backward compatibility assessment
   - ADR drafts (decisions being made)
   - Recommended PR phasing
   - **If SA recommends re-scope** (e.g., "split into two epics"): present prominently.

4. **User checkpoint**: Ask the user to approve the system architecture assessment. This shapes all subsequent phases.

---

## Phase 1.9: DevOps Spec Review (CONDITIONAL)

**Goal**: Identify infrastructure requirements early — deployment strategy, CI/CD changes, environment config, monitoring needs.

### Activation Check:

1. **Read the spec** `{SPEC_FOLDER}/01-spec.md`.
2. **Check `Affected Areas > Infrastructure`**: If empty/"None"/"N/A" → SKIP. Otherwise → ACTIVATE.

### Steps (only if activated):

1. **Launch `devops-engineer` in SPEC REVIEW mode**:
   - Prompt: "You are in SPEC REVIEW mode. Read the specification at `{SPEC_FOLDER}/01-spec.md` and the architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`. Explore the codebase to understand the current CI/CD pipelines, deployment topology, Infrastructure as Code, monitoring, and environment configuration. Identify infrastructure requirements, deployment strategy needs, environment configuration needs, and monitoring requirements. Write your review to `{SPEC_FOLDER}/01-spec-infra-review.md`."

2. **Read the infra spec review**. Present to user combined with Phase 1.7 results if both present. **User checkpoint**.

3. **Cross-review round** — Launch two agents in parallel:

   a. **`software-architect` in SPEC REVIEW mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the DevOps Engineer's infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`. Then read your own architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/01-spec-arch-review.md` with your observations about whether the architecture decisions are feasible from an infrastructure perspective, any conflicts between architectural choices and infrastructure constraints, and any concerns."

   b. **`devops-engineer` in SPEC REVIEW mode (cross-review)**:
   - Prompt: "You are completing a cross-review. Read the Software Architect's architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`. Then read your own infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`. Append a '## Cross-Review Notes' section to your document at `{SPEC_FOLDER}/01-spec-infra-review.md` with your observations about infrastructure feasibility gaps or confirmations in the SA's architecture, any deployment complexity the architecture creates, and any concerns."

**Track activation status**: Remember which conditional agents (UI/UX, DevOps) are active. This determines their participation in all subsequent phases.

---

## Phase 2: Discovery

**Goal**: All agents explore the codebase independently. SA maps the system landscape. DevOps maps infrastructure. SE and QA explore their respective domains. Then cross-reviews, then TL reviews everything.

### Steps:

1. **Launch agents in parallel** (3-5 agents depending on conditional activations):

   a. **`software-engineer` in DISCOVERY mode**:
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ', the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}, and the architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`{if DevOps active: ', and the infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`'}. Explore the codebase to understand the affected code paths, identify root causes or insertion points, map dependencies, and document conventions. In major workflows, pay special attention to the SA's API contracts and service boundaries — your implementation must conform to them. Write your findings to `{SPEC_FOLDER}/02-discovery-se.md`."

   b. **`qa-engineer` in DISCOVERY mode**:
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ', the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}, and the architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`{if DevOps active: ', and the infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`'}. Explore the test infrastructure, identify existing test coverage, document test patterns and conventions, and identify coverage gaps. For major workflows, also identify cross-service integration test opportunities based on the SA's architecture review. Write your findings to `{SPEC_FOLDER}/02-discovery-qa.md`."

   c. **`software-architect` in DISCOVERY mode**:
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md` and your architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`. Map the service dependency graph, inventory API contracts, trace end-to-end data flows, identify integration patterns, audit shared state, and analyze failure modes. Write your findings to `{SPEC_FOLDER}/02-discovery-sa.md`."

   d. **`ui-ux-specialist` in DISCOVERY mode** (only if activated):
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md` and your spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`. Map the existing UI architecture of the affected area — component tree, styling patterns, accessibility state, and reusable elements. Write your findings to `{SPEC_FOLDER}/02-discovery-ui.md`."

   e. **`devops-engineer` in DISCOVERY mode** (only if activated):
   - Prompt: "You are in DISCOVERY mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`, the architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`, and your infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`. Map the CI/CD pipelines, deployment topology, IaC inventory, environment configuration, monitoring, and scaling configuration. Write your findings to `{SPEC_FOLDER}/02-discovery-devops.md`."

2. **Wait for all agents to complete.**

3. **Cross-review round** — Launch agents in parallel (3-5):

   a. **`software-engineer` cross-review**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's discovery at `{SPEC_FOLDER}/02-discovery-qa.md` and the Software Architect's discovery at `{SPEC_FOLDER}/02-discovery-sa.md`. Then read your own discovery at `{SPEC_FOLDER}/02-discovery-se.md`. Append a '## Cross-Review Notes' section with observations about QA and SA findings, system-level implications of your code-level findings, and any alignment or conflicts."

   b. **`qa-engineer` cross-review**:
   - Prompt: "You are completing a cross-review. Read the Software Engineer's discovery at `{SPEC_FOLDER}/02-discovery-se.md`. Then read your own discovery at `{SPEC_FOLDER}/02-discovery-qa.md`. Append a '## Cross-Review Notes' section with observations about SE findings, alignment or conflicts, and suggestions."

   c. **`software-architect` cross-review**:
   - Prompt: "You are completing a cross-review. Read the SE's discovery at `{SPEC_FOLDER}/02-discovery-se.md`{if DevOps active: ' and the DevOps discovery at `{SPEC_FOLDER}/02-discovery-devops.md`'}. Then read your own discovery at `{SPEC_FOLDER}/02-discovery-sa.md`. Append a '## Cross-Review Notes' section with observations about system-level implications of the SE's findings{if DevOps active: ' and architectural alignment of the DevOps findings'}."

   d. **`ui-ux-specialist` cross-review** (only if activated):
   - Prompt: "You are completing a cross-review. Read the SE's discovery at `{SPEC_FOLDER}/02-discovery-se.md`. Then read your own discovery at `{SPEC_FOLDER}/02-discovery-ui.md`. Append a '## Cross-Review Notes' section with observations about UI implications of the SE's findings."

   e. **`devops-engineer` cross-review** (only if activated):
   - Prompt: "You are completing a cross-review. Read the SE's discovery at `{SPEC_FOLDER}/02-discovery-se.md` and the SA's discovery at `{SPEC_FOLDER}/02-discovery-sa.md`. Then read your own discovery at `{SPEC_FOLDER}/02-discovery-devops.md`. Append a '## Cross-Review Notes' section with observations about deployment implications of the SE's findings and infrastructure feasibility of the SA's architecture."

4. **Wait for cross-reviews to complete.**

5. **Launch `tech-lead` in DISCOVERY REVIEW mode** (sequential — after cross-reviews):
   - Prompt: "You are in DISCOVERY REVIEW mode. Read the specification at `{SPEC_FOLDER}/01-spec.md`, the architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`{if UI/UX active: ', the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}{if DevOps active: ', the infrastructure spec review at `{SPEC_FOLDER}/01-spec-infra-review.md`'}, the SE discovery at `{SPEC_FOLDER}/02-discovery-se.md`, the QA discovery at `{SPEC_FOLDER}/02-discovery-qa.md`, the SA discovery at `{SPEC_FOLDER}/02-discovery-sa.md`{if UI/UX active: ', the UI/UX discovery at `{SPEC_FOLDER}/02-discovery-ui.md`'}{if DevOps active: ', the DevOps discovery at `{SPEC_FOLDER}/02-discovery-devops.md`'}. Assess the architectural implications from a code-level perspective. Note: the Software Architect handles system-level architecture; your focus is code-level architecture within each service. Write your findings to `{SPEC_FOLDER}/02-discovery-tl.md`."

6. **Read all discovery documents.**

7. **Present summary to user**: Key findings from each agent, cross-review highlights, SA system concerns, DevOps infra concerns, TL code-level concerns. If SA or TL recommends re-scope, present prominently.

8. **User checkpoint**.

---

## Phase 3: Planning

**Goal**: All agents create plans. SA defines API contracts and renders a system-level verdict. TL validates the code-level approach and renders a code-level verdict. Both can block implementation independently.

### Steps:

1. **Launch agents in parallel** (3-4 agents):

   a. **`software-engineer` in PLANNING mode**:
   - Prompt: "You are in PLANNING mode. Read all documents: specification at `{SPEC_FOLDER}/01-spec.md`, all spec reviews (UI, architecture, infra — whichever exist), all discovery documents (SE, QA, SA, TL, UI/UX, DevOps — whichever exist). Create a detailed implementation plan. Pay special attention to: (1) the SA's architecture review and API contracts — your implementation must conform to them, (2) the TL's recommendations, (3) the SA's recommended PR phasing. In major workflows, infra code (Dockerfiles, CI configs, IaC) is implemented by the DevOps Engineer, not you. Include a PR Strategy section specifying sub-PRs and their ordering. Write your plan to `{SPEC_FOLDER}/03-plan-se.md`."

   b. **`qa-engineer` in PLANNING mode**:
   - Prompt: "You are in PLANNING mode. Read all documents: specification, all spec reviews, all discovery documents. Create a detailed test plan mapping every acceptance criterion to specific test cases. For major workflows, include cross-service integration test cases based on the SA's integration test requirements. Write your plan to `{SPEC_FOLDER}/03-plan-qa.md`."

   c. **`software-architect` in PLANNING mode**:
   - Prompt: "You are in PLANNING mode. Read all documents: specification, all spec reviews, all discovery documents. Define API contract specifications (schemas, error codes, versioning), service interaction design, data migration strategy (if needed), backward compatibility plan, and integration test requirements. Finalize Architecture Decision Records (ADRs). Write your plan to `{SPEC_FOLDER}/03-plan-sa.md`. After the SE's plan is available, you will review it for system-level compliance and render your verdict."

   d. **`devops-engineer` in PLANNING mode** (only if activated):
   - Prompt: "You are in PLANNING mode. Read all documents: specification, all spec reviews, all discovery documents. Plan infrastructure changes: file-by-file changes, deployment strategy, environment configuration, CI/CD modifications, monitoring plan, rollback strategy. Specify which infra changes go in which PRs and dependencies on the SE's PRs. Write your plan to `{SPEC_FOLDER}/03-plan-devops.md`. After the SE's plan is available, you will review it for deployment feasibility."

2. **Wait for all agents to complete.**

3. **Cross-review round** — Launch agents in parallel (3-5):

   a. **`software-engineer` cross-review**:
   - Prompt: "You are completing a cross-review. Read the QA Engineer's test plan at `{SPEC_FOLDER}/03-plan-qa.md`. Then read your own plan at `{SPEC_FOLDER}/03-plan-se.md`. Append a '## Cross-Review Notes' section with comments on whether your implementation supports QA's test cases, interface contracts, and any adjustments."

   b. **`qa-engineer` cross-review**:
   - Prompt: "You are completing a cross-review. Read the SE's plan at `{SPEC_FOLDER}/03-plan-se.md`. Then read your own plan at `{SPEC_FOLDER}/03-plan-qa.md`. Append a '## Cross-Review Notes' section with comments on testability, implementation dependencies, and any adjustments."

   c. **`ui-ux-specialist` plan review** (only if activated):
   - Prompt: "You are reviewing the SE's plan for UI compliance. Read the SE's plan at `{SPEC_FOLDER}/03-plan-se.md` and your spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`. Assess whether the plan adequately addresses your UI acceptance criteria, accessibility requirements, and design system recommendations. Write your review to `{SPEC_FOLDER}/03-plan-ui-review.md`."

   d. **`software-architect` plan review** (reviews SE's plan for system compliance):
   - Prompt: "You are reviewing the SE's plan for system-level compliance. Read the SE's plan at `{SPEC_FOLDER}/03-plan-se.md` and your own plan at `{SPEC_FOLDER}/03-plan-sa.md`. Assess: does the SE's approach honor your API contracts, service boundaries, and integration patterns? Append '## SE Plan Review Notes' to your plan at `{SPEC_FOLDER}/03-plan-sa.md`. Then render your system-level verdict by appending the '## Verdict' section."

   e. **`devops-engineer` plan review** (only if activated):
   - Prompt: "You are reviewing the SE's plan for deployment feasibility. Read the SE's plan at `{SPEC_FOLDER}/03-plan-se.md` and your own plan at `{SPEC_FOLDER}/03-plan-devops.md`. Assess: can the SE's changes be deployed with the planned strategy? Any infrastructure dependencies not yet planned? Append '## SE Plan Review Notes' to your plan at `{SPEC_FOLDER}/03-plan-devops.md`."

4. **Wait for cross-reviews to complete.**

5. **Launch `tech-lead` in PLANNING REVIEW mode** (sequential — after cross-reviews):
   - Prompt: "You are in PLANNING REVIEW mode. Read all documents: specification, all spec reviews, all discovery documents, the SE plan at `{SPEC_FOLDER}/03-plan-se.md` (with cross-review notes), the QA plan at `{SPEC_FOLDER}/03-plan-qa.md` (with cross-review notes), the SA plan at `{SPEC_FOLDER}/03-plan-sa.md` (with SE review notes and verdict){if UI/UX active: ', the UI/UX plan review at `{SPEC_FOLDER}/03-plan-ui-review.md`'}{if DevOps active: ', the DevOps plan at `{SPEC_FOLDER}/03-plan-devops.md` (with SE review notes)'}. Evaluate the SE's approach from a code-level architecture perspective. Note: the SA handles system-level concerns; your focus is code-level quality. Reference the SA's API contracts when evaluating the SE's approach. Write your review to `{SPEC_FOLDER}/03-plan-tl.md`."

6. **Read the SA's verdict** (in `03-plan-sa.md`) and **TL's verdict** (in `03-plan-tl.md`).

7. **Handle dual verdicts** — use the most restrictive:

   a. **Both APPROVED**: Proceed to step 8.

   b. **SA: APPROVED WITH CHANGES** (system-level changes required):
      - Present SA's required changes to user.
      - If user agrees: re-launch SE to revise plan addressing system-level issues. Re-run SA review.
      - If user overrides: note override, proceed.

   c. **TL: APPROVED WITH CHANGES** (code-level changes required):
      - Present TL's required changes to user.
      - If user agrees: re-launch SE to revise plan addressing code-level issues. Re-run TL review.
      - If user overrides: note override, proceed.

   d. **Either REJECTED**:
      - Present rejection reasons and recommended path forward.
      - Ask user: "(1) SE revises the plan, (2) Override and proceed anyway, (3) Stop here."
      - If revise: re-launch SE with feedback from the rejecting agent. Re-run that agent's review.
      - If override: warn, note, proceed.
      - If stop: halt.

   e. **Both have issues**: Address the most restrictive first (REJECTED > WITH CHANGES > APPROVED). Both must be satisfied before proceeding.

8. **Present all plans to user**:
   - SE plan: approach, PRs, files per PR, implementation order
   - QA plan: test strategy, test cases, integration tests
   - SA plan: API contracts, ADRs, system verdict
   - TL review: code-level verdict
   - DevOps plan (if active): infra changes, deployment strategy, rollback plan
   - UI/UX review (if active): UI criteria coverage

9. **User checkpoint**: Approve all plans before implementation.

---

## Phase 4: Implementation

**Goal**: SE implements application code, DevOps implements infrastructure code, QA implements tests. Sub-PRs are grouped into phases with ordering dependencies.

### Steps:

1. **Confirm user approval**.

2. **Determine sub-PR structure and phasing**:
   - Read the SE's plan PR Strategy section and the SA's recommended PR phasing.
   - Map sub-PRs into phases (from SA's plan): Phase A (foundation), Phase B (services), Phase C (integration), Phase D (infrastructure).
   - Determine which PRs can be parallel within a phase vs. must be sequential.

3. **For each phase** (iterate sequentially — Phase A → B → C → D):

   **For each sub-PR within the phase** (sequential or parallel per SA's guidance):

   a. **Create the sub-branch**: `{BRANCH_NAME}/pr-{N}-{description}` from the epic branch.

   b. **Determine implementer**:
      - **Application PRs**: Launch `software-engineer` in IMPLEMENTATION mode.
      - **Infrastructure PRs**: Launch `devops-engineer` in IMPLEMENTATION mode (if active).
      - **Mixed PRs**: Launch SE first (application code), then DevOps (infra code), same branch.

   c. **Launch SE in IMPLEMENTATION mode** (for application PRs):
      - Prompt: "You are in IMPLEMENTATION mode. Read your plan at `{SPEC_FOLDER}/03-plan-se.md`, the specification at `{SPEC_FOLDER}/01-spec.md`, and the SA's API contracts in `{SPEC_FOLDER}/03-plan-sa.md`. This is PR {N} of {total} (Phase {phase}). Implement ONLY the scope for this PR: {PR scope}. Your implementation MUST conform to the SA's API contract specifications. Infra code (Dockerfiles, CI configs, IaC) is NOT your responsibility. Write your implementation summary to `{SPEC_FOLDER}/04-implementation-summary.md` (append a section for PR {N})."

   d. **Launch DevOps in IMPLEMENTATION mode** (for infra PRs, only if activated):
      - Prompt: "You are in IMPLEMENTATION mode. Read your plan at `{SPEC_FOLDER}/03-plan-devops.md` and the specification at `{SPEC_FOLDER}/01-spec.md`. This is PR {N} of {total} (Phase {phase}). Implement ONLY the infrastructure scope for this PR: {PR scope}. Write your infrastructure summary to `{SPEC_FOLDER}/04-infra-summary.md` (append a section for PR {N})."

   e. **Launch QA in IMPLEMENTATION mode** (after SE/DevOps complete, or parallel if safe):
      - Prompt: "You are in IMPLEMENTATION mode. Read your test plan at `{SPEC_FOLDER}/03-plan-qa.md`, the specification at `{SPEC_FOLDER}/01-spec.md`, and the implementation summaries at `{SPEC_FOLDER}/04-implementation-summary.md`{if DevOps active: ' and `{SPEC_FOLDER}/04-infra-summary.md`'}. This is PR {N} of {total}. Implement ONLY the tests for this PR scope: {PR scope}. For cross-service integration test cases, reference the SA's integration test requirements. Run the test suite after writing tests. Write your test report to `{SPEC_FOLDER}/04-test-report.md` (append a section for PR {N})."

   f. **Commit changes** to the sub-branch.

   g. **Intermediate checkpoint** (if not last sub-PR): Present results, ask user to proceed.

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

7. **User checkpoint**.

---

## Phase 5: Quality Gates

**Goal**: Final validation by all agents. QA validates acceptance criteria, SE reviews tests, TL reviews code architecture, SA validates system integration, UI/UX validates design, DevOps validates deployment readiness.

### Steps:

1. **Launch `qa-engineer` in QUALITY GATE mode** (first — writes base report):
   - Prompt: "You are in QUALITY GATE mode. This is the final validation. Read the specification at `{SPEC_FOLDER}/01-spec.md`{if UI/UX active: ', the UI spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`'}, the architecture spec review at `{SPEC_FOLDER}/01-spec-arch-review.md`, the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`{if DevOps active: ', the infra summary at `{SPEC_FOLDER}/04-infra-summary.md`'}, and the test report at `{SPEC_FOLDER}/04-test-report.md`. Review the actual code changes. Run the full test suite. Validate every acceptance criterion. For cross-service integration scenarios, reference the SA's integration test requirements. Write the quality gate report to `{SPEC_FOLDER}/05-quality-gate.md`."

2. **After QA completes, launch remaining reviewers in parallel** (3-5 agents):

   a. **`software-engineer` in QUALITY GATE mode** (test review):
   - Prompt: "You are in QUALITY GATE mode (test review). Read the test report at `{SPEC_FOLDER}/04-test-report.md` and review the actual test code. Assess test correctness, coverage adequacy, and test quality. Append your '## Software Engineer - Test Quality Review' section to `{SPEC_FOLDER}/05-quality-gate.md`."

   b. **`tech-lead` in QUALITY GATE mode** (architecture review):
   - Prompt: "You are in QUALITY GATE mode (architecture review). Read the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`, the SE's plan at `{SPEC_FOLDER}/03-plan-se.md`, your plan review at `{SPEC_FOLDER}/03-plan-tl.md`, and the SA's plan at `{SPEC_FOLDER}/03-plan-sa.md`. Review the actual code changes. Assess plan adherence, code-level architectural quality, and convention compliance. Note: the SA handles system-level integration; your focus is code-level quality within each service. Append your '## Tech Lead - Architecture Review' section to `{SPEC_FOLDER}/05-quality-gate.md`."

   c. **`software-architect` in QUALITY GATE mode** (system integration review):
   - Prompt: "You are in QUALITY GATE mode (system integration review). Read your plan at `{SPEC_FOLDER}/03-plan-sa.md`, the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`{if DevOps active: ', the infra summary at `{SPEC_FOLDER}/04-infra-summary.md`'}, and the test report at `{SPEC_FOLDER}/04-test-report.md`. Review the actual code changes — especially API endpoints, event handlers, service clients, and integration code. Validate API contract compliance, cross-service data flow, backward compatibility, and ADR accuracy. Append your '## Software Architect - System Integration Review' section to `{SPEC_FOLDER}/05-quality-gate.md`."

   d. **`ui-ux-specialist` in QUALITY GATE mode** (only if activated):
   - Prompt: "You are in QUALITY GATE mode (design review). Read your spec review at `{SPEC_FOLDER}/01-spec-ui-review.md`, the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`, and the test report at `{SPEC_FOLDER}/04-test-report.md`. Review the actual UI code changes. Validate design system compliance, accessibility, and responsive behavior. Append your '## UI/UX Specialist - Design Review' section to `{SPEC_FOLDER}/05-quality-gate.md`."

   e. **`devops-engineer` in QUALITY GATE mode** (only if activated):
   - Prompt: "You are in QUALITY GATE mode (deployment readiness review). Read your plan at `{SPEC_FOLDER}/03-plan-devops.md`, the infra summary at `{SPEC_FOLDER}/04-infra-summary.md`, and the implementation summary at `{SPEC_FOLDER}/04-implementation-summary.md`. Review the actual infrastructure code changes. Validate CI/CD pipeline, infrastructure correctness, deployment strategy, environment configuration, monitoring, and rollback plan. Append your '## DevOps Engineer - Deployment Readiness Review' section to `{SPEC_FOLDER}/05-quality-gate.md`."

3. **Read the full quality gate report**.

4. **Present results to user**:
   - QA verdict + acceptance criteria results
   - SE test quality assessment
   - TL code architecture verdict
   - SA system integration verdict
   - UI/UX design verdict (if active)
   - DevOps deployment readiness verdict (if active)
   - **Overall recommendation**: consolidated from all verdicts.

5. **Handle the verdict** (most restrictive across all reviewers):

   a. **All APPROVED/ADEQUATE**: Proceed to Phase 6.

   b. **Any APPROVED WITH CONDITIONS or NEEDS IMPROVEMENT (non-blocking)**:
      - Present all conditions. Ask user:
        - Address now (loop to Phase 4 with specific fixes)
        - Accept and proceed to merge (create follow-up tasks)
        - Stop

   c. **QA REJECTED or any NEEDS IMPROVEMENT (blocking)**:
      - Present required fixes. Ask user:
        - Fix (loop to Phase 4)
        - Stop
        - Override (warn about bypassing quality gates)

---

## Phase 6: Merge

**Goal**: Create sub-PRs targeting the epic branch, then a final PR from epic branch to main.

### Steps:

1. **Generate the PR summary**: Create `{SPEC_FOLDER}/06-pr-summary.md`:

   ```markdown
   # PR Summary

   ## Title
   [Short descriptive title from the spec]

   ## Summary
   [2-3 sentences from the spec summary]

   ## Changes Made
   [From 04-implementation-summary.md — consolidated across all sub-PRs]

   ## Infrastructure Changes
   [From 04-infra-summary.md — if DevOps was active, otherwise "N/A"]

   ## Tests
   [From 04-test-report.md — summary of tests written and results]

   ## Quality Gate
   [From 05-quality-gate.md — all verdicts]
   - QA Verdict: [verdict]
   - SE Test Review: [verdict]
   - TL Architecture Review: [verdict]
   - SA System Integration Review: [verdict]
   - UI/UX Design Review: [verdict or "N/A"]
   - DevOps Deployment Readiness: [verdict or "N/A"]

   ## Architecture Decision Records
   [List of ADRs from 03-plan-sa.md]

   ## Acceptance Criteria Status
   [PASS/FAIL per criterion]

   ## Deployment Plan
   [From 03-plan-devops.md — deployment strategy and rollback plan, or "Standard deployment"]

   ## Branch Info
   - **Epic branch**: `{BRANCH_NAME}`
   - **Target branch**: `{BASE_BRANCH}`
   - **Sub-PRs**: [list with phase grouping]

   ## Spec Folder
   `{SPEC_FOLDER}`
   ```

2. **Handle sub-PR branches** (phased):
   - For each sub-branch `{BRANCH_NAME}/pr-N-{description}`:
     - Push the sub-branch to remote with `-u` flag.
     - Create a PR targeting the epic branch `{BRANCH_NAME}`:
       - `gh pr create --base {BRANCH_NAME} --title "{PR title}" --body "{PR description}"`
     - Present the PR link to the user.
   - Group PR links by phase for clarity.

3. **Handle the epic branch** (final PR):
   - Ask user how to proceed:
     - **Option A**: Create the final PR now (epic branch → BASE_BRANCH).
     - **Option B**: Wait for sub-PRs to be reviewed/merged first.
   - Use `gh pr create --base {BASE_BRANCH} --title "{spec title}" --body "$(cat {SPEC_FOLDER}/06-pr-summary.md)"`

4. **Stage and commit the spec folder**:
   - Stage the spec folder (audit trail + ADRs).
   - Commit: `docs: add spec audit trail and ADRs for {id}-{slug}`
   - Push.

5. **Present results**: All PR links, phase grouping, deployment plan summary, remaining cycles.

6. **Mark all todos complete.**

### Branching Rules

- **Major (single-phase)**: `epic/{id}-{slug}` → PR to `{BASE_BRANCH}` (default: `main`)
- **Major (multi-phase)**: Epic branch `epic/{id}-{slug}`. Sub-branches `epic/{id}-{slug}/pr-N-{desc}` → PRs to `epic/{id}-{slug}`. Final PR: `epic/{id}-{slug}` → `{BASE_BRANCH}`.
- **NEVER** push directly to `main`, `master`, or any shared branch unless the user explicitly requests it.

---

## Error Recovery

- **Agent fails or returns incomplete output**: Inform the user. Offer to re-launch or proceed manually.
- **SA rejects the plan**: Expected workflow. SE revises plan, SA re-reviews. User can override.
- **TL rejects the plan**: Expected workflow. SE revises plan, TL re-reviews. User can override.
- **Both SA and TL reject**: Both issues must be addressed. SE revises addressing both sets of feedback.
- **Tests fail during implementation**: Document in test report. Discuss at checkpoint.
- **Quality gate rejects**: Loop back to Phase 4 with specific fixes.
- **Cross-service integration issues**: SA's quality gate review catches these. May require changes to multiple sub-PRs.
- **Infrastructure issues**: DevOps's quality gate review catches these. May require infra PR revisions.
- **User wants to stop mid-workflow**: All documents are preserved. User can resume later.
- **DevOps finds deployment blocker late**: Present to user. If blocking, loop back. May require SA re-assessment.
