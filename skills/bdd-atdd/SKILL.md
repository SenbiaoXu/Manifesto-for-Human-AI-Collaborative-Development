---
name: bdd-atdd
description: Behavior and acceptance criteria driven development. Triggers when users request implementing features, fixing bugs, refactoring code, or adding/updating behaviors (i.e., any significant task involving source code modifications, with keywords such as "implement", "modify", "fix", "add", "refactor", "update", "change"). Does NOT trigger for pure information queries, documentation-only tasks, or when the user explicitly skips the BDD workflow.
---

# BDD-ATDD Skill

Enforces a structured code change workflow: define behaviors first (BDD), then write an acceptance test draft (ATDD draft) for user review, implement code, self-verify, and finally finalize the ATDD document based on actual verification results.

## Principles
- **Stop and ask when unclear**: If a user request has multiple interpretations, present them rather than silently choosing one. If boundary conditions are uncertain, ask for clarification
- **Only write necessary scenarios**: Each BDD scenario must correspond to a clear requirement. Do not add scenarios the user didn't request "just in case"
- **Only change what must be changed**: During implementation, only modify code related to the current BDD scenarios — do not take the opportunity to refactor adjacent code or "improve" style
- **Evidence serves human credibility judgment**: During verification, prioritize human-directly-verifiable evidence (screenshot comparisons, before/after mappings, actual output), not just executable evidence (test passed, command succeeded). When changes are human-visible, you must supplement with human-intuitively-verifiable comparative evidence

## Five-Phase Workflow

This workflow is **strictly sequential**. Do not proceed to the next phase until the current phase is complete.

### Phase 1: BDD Specification — Define Before Building

**Completion Criteria**: BDD document has been created, covering all happy path/boundary/error scenarios, and has received user confirmation.

1. **Analyze requirements**. If the user's request is unclear, clarify the scope first.
   Ask questions to understand: what are the inputs, outputs, boundary cases, and error conditions.
   **When to stop and ask**: If the requirement has more than one reasonable implementation approach, present the options to the user rather than deciding on your own.

2. **Derive a feature name from the requirements**, using kebab-case.
   - Extract core concepts (2-5 words)
   - All lowercase, spaces and special characters replaced with hyphens, no consecutive hyphens
   - Example: "Add email validation for login" → `email-validation-login`

3. **Locate or create the PROJECT_STATE directory**:
   - Look for `PROJECT_STATE/` in the project root
   - If it already exists, use it directly. If not, create `PROJECT_STATE/` with `BDD/` and `ATDD/` subdirectories

4. **Read the BDD template**, located at `references/bdd-template.md` in this skill's directory.

5. **Create `PROJECT_STATE/BDD/<feature-name>.md`**, containing:
   - A Feature title using the requirement name as the heading
   - A brief description
   - All scenarios: happy path, boundary cases, error conditions
   - Each scenario using Given/When/Then format
   - **Must use business/domain language** — language understandable by non-technical stakeholders; technical implementation details (CLI flags, API paths, data structures, etc.) are left for ATDD

   Note: If a file with a similar name already exists in the `PROJECT_STATE/BDD` directory, check whether the content is related first. If related but not an exact match, update the file to include the new scenarios rather than creating a new file.

6. **Submit the BDD document for user review** before proceeding.
   Wait for user confirmation or feedback. Adjust the document content as needed.

### Phase 2: ATDD Draft — Define Acceptance Criteria

**Completion Criteria**: ATDD draft has been created, with corresponding test case plans for each BDD scenario, and has received user confirmation.

1. **Only proceed after the user confirms the BDD document**.

2. **Read the ATDD template**, located at `references/atdd-template.md` in this skill's directory.

3. **Create the `PROJECT_STATE/ATDD/<feature-name>/` directory**.

4. **Write `PROJECT_STATE/ATDD/<feature-name>/acceptance.md` draft**, containing:
   - A reference to the corresponding BDD document path
   - Verification strategy (specific date, strategy method, verification approach)
   - For each BDD scenario: **planned** test cases, with expected Given/When/Then, planned test commands
   - Leave the **Result** field empty for each test case (to be filled after self-verification)
   - Leave the **Evidence** field empty for each test case (to be filled after self-verification)
   - Leave the summary table empty (to be filled after self-verification)

5. **Submit the ATDD draft for user review**.
   Wait for user confirmation that the acceptance criteria are reasonable. Adjust test case design as needed.

### Phase 3: Implementation — Write Code

**Completion Criteria**: All code changes corresponding to BDD scenarios have been implemented, with no remaining TODOs or placeholder implementations.

1. **Only start coding after the ATDD draft has been confirmed by the user**.
2. Implement code changes according to the BDD scenarios and acceptance criteria defined in the ATDD draft.
3. Always keep all BDD scenarios in mind — each scenario describes a behavior the code must satisfy.
4. If new scenarios are discovered during implementation, return to Phase 1. Add the new scenarios to the BDD document first, synchronize the ATDD draft, then proceed with implementation.
5. **Only change what must be changed**: Do not take the opportunity to refactor adjacent code, add "improvements", or change code style. Every line of change should be traceable to a BDD scenario.

### Phase 4: Self-Verification — Prove the Code Works

**Completion Criteria**: Each BDD scenario has a pass/fail conclusion with evidence, with no omissions and no evidence-less "passed" marks.

1. **For each BDD scenario**, verify that the implementation satisfies the scenario.
   Read `references/verification-guide.md` in this skill's directory for detailed strategies.

2. **Verification priority** (see the Evidence Credibility Assessment in `references/verification-guide.md`):
   - **Level S (target)**: Automated test passing **+** human-intuitively-verifiable comparative evidence (e.g., test passes + before/after screenshots)
   - **Level A**: Run existing tests or write and run new targeted tests → Record test commands and full output
   - **Level B**: Run verification commands, invoke tools (MCP/Skill), or execute scripts → Record commands and results
   - **Level C**: Observe runtime behavior (HTTP responses, file changes, process status) → Record observed outcomes
   - **Level D (last resort)**: Code review with justification — must explain why no executable evidence was obtainable and describe specific behavior, NOT just line number ranges
   - **Key**: When changes have a perceivable impact on humans (UI, API, data output, etc.), you **must** supplement with human-intuitively-verifiable evidence to elevate to Level S

3. **Record results**: For each scenario, mark pass/fail with evidence.

4. **Save non-text evidence**: For before/after screenshots, log files, and other human-intuitively-verifiable evidence,
   save them to the `PROJECT_STATE/ATDD/<feature-name>/` directory (e.g., `before-tc1.png`, `after-tc1.png`, `evidence-tc2.log`).
   When changes are human-visible, proactively collect both before and after comparative evidence, not just post-change evidence.

5. **If any scenario fails verification**:
   - Investigate the root cause
   - Fix the code
   - Re-verify that scenario (and any other scenarios the fix may affect)

6. **Do not skip any scenario**, and do not mark any scenario as passed without evidence.

### Phase 5: ATDD Finalization — Complete Acceptance Documentation

**Completion Criteria**: All empty fields in the ATDD document have been filled with actual results and evidence, the summary table is complete, and there are no inconsistencies.

1. **Only proceed after self-verification is complete and all scenarios have passed**.

2. **Update `PROJECT_STATE/ATDD/<feature-name>/acceptance.md`**, filling the draft's empty fields with actual results:
   - Fill in each test case's **Result** (Pass/Fail)
   - Fill in each test case's **Evidence** (test output, log snippets, etc.)
   - For non-text evidence files (screenshots, logs, etc.), reference them using relative paths in the evidence field: `See [evidence file](./evidence-tc1.png)`
   - Fill in the summary table: Scenario → Test Case → Result
   - Note any limitations in the remarks (e.g., skipped scenarios, conditions that could not be fully verified)

3. **Be honest and complete**. If a scenario genuinely cannot be verified, state so truthfully.

## Quick Reference — File Locations

```
PROJECT_STATE/
  BDD/
    <feature-name>.md                        # BDD specification document (Phase 1)
  ATDD/
    <feature-name>/
      acceptance.md                          # ATDD acceptance document (Phase 2 draft → Phase 5 finalized)
      before-tc1.png                         # Pre-change evidence (screenshot/data)
      after-tc1.png                          # Post-change evidence (screenshot/data)
      evidence-tc2.log                       # Other evidence file (logs, etc.)
```

## Reference Files

- `references/bdd-template.md` -- BDD document structure and examples. Read before Phase 1.
- `references/atdd-template.md` -- ATDD document structure and examples. Read before Phase 2.
- `references/verification-guide.md` -- Self-verification strategy guide. Read during Phase 4.
