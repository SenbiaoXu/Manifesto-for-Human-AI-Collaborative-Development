# Self-Verification Guide

## Purpose

This guide helps you decide how to verify each BDD scenario during Phase 4 (Self-Verification). The verification strategy depends on the project type, available test frameworks, and the nature of the scenario.

## Evidence Quality Hierarchy

Evidence falls into a clear quality spectrum. **Always prefer higher-quality evidence.**

| Level | Evidence Type | Quality | Examples |
|---|---|---|---|
| **A** | Executable output | Strong | Test runner output, command exit codes, program stdout/stderr, tool invocation results |
| **B** | Observable behavior | Good | HTTP responses, file system changes, process status, API call results |
| **C** | Artifact inspection | Acceptable | Build output, generated files, configuration snapshots |
| **D** | Code review only | Weak — last resort | Reading source code and reasoning about logic |

**Core principle**: Evidence must demonstrate that the code **actually behaves** as expected, not merely that the code **appears to be correct**. A passing test is always stronger than "I traced lines 42–58 and the logic looks right."

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
