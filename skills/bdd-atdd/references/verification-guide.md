# Self-Verification Guide

## Purpose

This guide helps you decide how to verify each BDD scenario during Phase 4 (Self-Verification). The verification strategy depends on the project type, available test frameworks, and the nature of the scenario.

## Decision Tree

For each BDD scenario, evaluate in the following order:

1. **Is there an existing test that covers this scenario?**
   - Yes → Run the existing test. Record the output as evidence.
   - No → Proceed to step 2.

2. **Can this scenario be tested automatically?**
   - Yes → Write a test. Run the test. Record the output.
   - No → Proceed to step 3.

3. **Can this scenario be verified by reading code and tracing logic?**
   - Yes → Perform a code review verification. Record what was checked.
   - No → Mark the scenario as "Cannot verify" and explain why.

## Detecting Test Frameworks
Check whether the project declares specific test frameworks or has directories characteristic of test suites.

## Targeted Testing
If a test framework is available, verify specific scenarios without running the full test suite.
1. Create test files in the project's test directory (following existing naming conventions).
2. Test names should correspond to BDD scenarios (e.g., `test_valid_login_returns_jwt`).
3. Test structure: set up Given, execute When, assert Then.
4. Run the test and capture the output.
5. Record the exact test command in the ATDD document.

If no test framework is available, consider the following strategies:
1. Whether suitable verification commands are available
2. Whether suitable tools (MCP/Skill) can be used for verification
3. Whether a temporary script can be written for verification

## Code Review Verification Strategy

### Code Path Tracing
1. Read the relevant source code.
2. Trace the execution path for the scenario's inputs.
3. Verify that each branch and condition leads to the expected outcome.
4. Record: `Verified by tracing code path: [file]:[line number] through [file]:[line number]`

### Output Inspection
1. Run the program with the scenario's inputs.
2. Capture standard output / standard error or API responses.
3. Compare against expected results.
4. Record: `Verified by running [command], output matches [expected pattern]`

### Configuration Review
1. Read configuration files related to the scenario.
2. Verify that configuration items produce the expected behavior.
3. Record: `Verified by reviewing configuration at [path]: [key]=[value] matches expected behavior`

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
