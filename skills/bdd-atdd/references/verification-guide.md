# Self-Verification Guide

## Purpose

This guide helps you decide how to verify each BDD scenario during Phase 4 (Self-Verification). The verification strategy depends on the project type, available test frameworks, and the nature of the scenario.

## Evidence Credibility Assessment

Evidence quality is not determined by how automated the acquisition method is, but by **whether a human can quickly and independently judge the credibility of the result**.
High-quality evidence must satisfy two dimensions:
1. **Machine-executable verification**: Proves code actually behaves as expected (tests pass, commands succeed, etc.)
2. **Human-directly-verifiable**: Allows a person to intuitively confirm the result is correct without reading code (key screenshots, actual output, etc.)

### Evidence Quality Hierarchy

| Level | Evidence Type | Human Verifiability | Description |
|---|---|---|---|
| **S** | Multi-perspective corroboration | Highest | Automated test passing **+** human-intuitively-perceivable evidence. E.g., E2E test passes **+** post-change key screenshots; unit test passes **+** input/output mapping |
| **A** | Executable output | High | Test runner output, command exit codes, tool invocation results. But humans cannot intuitively judge whether the test itself is valid |
| **B** | Observable behavior | Good | HTTP responses, file system changes, process status, API call results |
| **C** | Artifact inspection | Acceptable | Build output, generated files, configuration snapshots |
| **D** | Code review only | Weak — last resort | Reading source code and reasoning about logic |

**Core principles**:
- **Evidence serves human credibility judgment**: When choosing evidence types, always ask yourself — "Can the user, seeing this evidence, judge whether the result is correct without reading code?" If not, supplement with human-perceivable evidence
- **Level A is not good enough**: A passing test only proves "a test passed," not "the right thing was tested." When changes have visible impact on humans (UI, API behavior, output format, etc.), you **must** aim for Level S — provide human-directly-verifiable key screenshots
- **Evidence must prove actual code behavior matches expectations**: Not merely that the code looks correct. "I traced lines 42-58 and the logic looks right" is never acceptable evidence

### When Code Review Evidence is Acceptable

Code review (Level D) is **only** acceptable when:
- No test framework exists AND no verification command or tool can be invoked AND no script can be written
- The scenario is purely declarative (e.g., configuration file structure, static resource placement)

If you must use code review evidence, you **must** explain:
1. Why stronger evidence could not be obtained
2. What specific behavior you verified and how you traced it

## Decision Tree

For each BDD scenario, evaluate in the following order:

1. **Is there an existing test that covers this scenario?**
   - Yes → Run the existing test. Record the **full test output** (command + result) as evidence.
   - No → Proceed to step 2.

2. **Can this scenario be tested automatically?**
   - Yes → Write a targeted test. Run it. Record the test command and its output.
   - No → Proceed to step 3.

3. **Can this scenario be verified by running a command, tool, or script?**
   - Yes → Run it. Capture the output or observable result as evidence.
   - No → Proceed to step 4.

4. **Can this scenario be verified by observing runtime behavior?**
   - Yes → Run the program, invoke the API, or trigger the action. Capture the response or observable state.
   - No → Proceed to step 5.

5. **Code review only** (Level D — last resort).
   - Record what you checked and why stronger evidence was unavailable.
   - If even code review is infeasible → Mark the scenario as "Cannot verify" and explain why.

6. **[Required after all steps] Does the change have a visible/perceivable impact on humans?**
   - Yes → **Must** additionally collect human-intuitively-verifiable evidence (key screenshots, actual output examples, etc.) to elevate evidence to Level S. See "Human-Verifiable Evidence by Change Type" below
   - No → Current evidence is sufficient

### Human-Verifiable Evidence by Change Type

When changes have a perceivable impact on humans, you **must** additionally provide human-intuitively-verifiable evidence. Different types of changes require different human-verifiable evidence:

| Change Type | Required Human-Verifiable Evidence | Rationale |
|---|---|---|
| UI/Page changes | Post-change key screenshots | Humans can instantly judge whether UI changed as expected, verifying the test actually tests the right thing |
| API behavior changes | Actual request/response examples | Show the specific request and complete response body, not just "test passed" |
| Data/Logic changes | Input/output mapping examples | Before: input X → output Y; After: input X → output Z |
| Style/Appearance changes | Post-change rendered screenshots | Post-change rendered result screenshots |
| Config/Structure changes | Changed files | git diff output or changed file content |
| Build/Deploy changes | Build artifacts or runtime status screenshots | Service started successfully, page accessible, etc. |

**Why human-verifiable evidence is needed**: Agent-written test cases may have the following issues —
- Test assertions are too weak, not actually verifying key behavior (e.g., `assert true`)
- Tests verify the wrong thing (e.g., checking a button exists but not its text)
- Tests don't match the requirement (requirement says change color, test only checks element exists)

Human-verifiable evidence lets users **judge whether the test is effective without reading code** — this is the ultimate safeguard for evidence credibility.

## Detecting Test Frameworks

Check whether the project declares specific test frameworks or has directories characteristic of test suites.

## Targeted Testing

If a test framework is available, verify specific scenarios without running the full test suite.
1. Create test files in the project's test directory (following existing naming conventions).
2. Test names should correspond to BDD scenarios (e.g., `test_valid_login_returns_jwt`).
3. Test structure: set up Given, execute When, assert Then.
4. Run the test and capture the output.
5. Record the exact test command in the ATDD document.

If no test framework is available, consider the following strategies in order:
1. Whether suitable verification commands are available (e.g., `curl`, `docker`, build commands)
2. Whether suitable tools (MCP/Skill) can be used for verification
3. Whether a temporary script can be written for verification (e.g., a shell script, Python one-liner)

## Execution-Based Verification Strategies

### Test Execution
1. Run the test command (e.g., `pytest tests/test_login.py::test_valid_login -v`).
2. Capture the full output including pass/fail status.
3. Record: `Verified by test: [command] → [pass/fail with output summary]`

### Command / Tool Invocation
1. Execute the relevant command or invoke the relevant tool.
2. Capture exit code, stdout, stderr, or the tool's response.
3. Record: `Verified by running [command] → [exit code, output snippet]`

### Runtime Behavior Observation
1. Start the program or service if needed.
2. Trigger the scenario's action (e.g., send an HTTP request, call a function).
3. Capture the response or observable state change.
4. Record: `Verified by [action] → [observed result, e.g., "HTTP 200 with token in body"]`

### Build / Compilation Verification
1. Run the build or compilation command.
2. Check for errors or warnings related to the changed code.
3. Record: `Verified by build: [command] → [success/failure, relevant output]`

## Code Review Verification (Last Resort Only)

**Use this only when no execution-based method is available.** Code review does not prove behavior — it only shows intent.

### When You Must Use Code Review
1. Read the relevant source code.
2. Identify the specific logic path for the scenario's inputs.
3. Verify that each branch and condition leads to the expected outcome.
4. Record with required justification:
   ```
   Verified by code review (no executable verification available):
   Reason no stronger evidence: [explanation]
   Logic traced: [describe the specific behavior path, NOT just file:line ranges]
   Key assertions: [what specific conditions/branches were confirmed]
   ```

### Anti-Patterns — What NOT to Record as Evidence
- `Verified by tracing code path: file.js:10 through file.js:35` — Line ranges are NOT evidence.
- `Code appears correct` — Vague and unverifiable.
- `Function X handles this case` — Claims without demonstration.

### What Good Code Review Evidence Looks Like
- `Function validateEmail() rejects input "not-an-email" because the regex pattern [pattern] does not match — the regex is defined at config.js:15 and applied at validator.js:42. No test framework is available and the validation logic is a pure function with no I/O.`
  Note: Even in this case, it includes a **reason** for not running a test and describes **specific behavior**, not just line numbers.

## Handling Verification Failures

If a scenario fails verification:

1. Honestly record the failure in the ATDD document.
2. Investigate the root cause.
3. Fix the code.
4. Re-verify the scenario.
5. Update the ATDD document with the new results.

**Do not skip any scenario, and do not mark any scenario as passed without evidence.**

## Handling Scenarios That Cannot Be Verified

Some scenarios cannot be verified in the current environment (e.g., they require specific hardware, external services, or production data). In such cases:

1. Mark the scenario as "Not Verified" in the ATDD document.
2. Explain why it cannot be verified.
3. Describe how it should ideally be verified (the ideal test setup).
4. Record any partial verification that was possible.

## Handling Human-Only-Verifiable Scenarios

Some scenarios are inherently only verifiable by humans (e.g., visual effects, cross-system verification, design mockup comparison, etc.), where the Agent cannot automatically judge pass/fail. For such scenarios:

1. **Label as "Verifier: Human verification" in the ATDD**, clearly distinguishing between Agent-verifiable and human-only-verifiable scenarios.
2. **In the verification approach, describe what the human needs to focus on**: List specific points the human needs to check (e.g., "input field spacing", "error message color", "button position").
3. **Agent provides supporting evidence**: Collect key screenshots, render results, etc. as much as possible for human reference.
4. **Mark the result as "Pending human confirmation"**: The Agent should not mark the result as passed or failed on its own.
5. **Count "Pending human confirmation" separately in the summary table** to ensure nothing is missed.

### Typical Human-Only-Verifiable Scenarios

| Scenario Type | Description | Points for Human Attention |
|---|---|---|
| Design mockup comparison | Whether page rendering matches the design mockup | Color, spacing, typography, layout |
| Visual effects | Animation smoothness, natural interaction | Animation duration, transition effects, responsiveness |
| Cross-system verification | Requires integration with specific systems | Specific databases, specific user permissions, etc. |
| Audio/Video | Media content quality | Audio quality, visual quality, synchronization |
