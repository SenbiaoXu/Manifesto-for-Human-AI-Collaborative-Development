# ATDD Document Template

## Purpose
This template defines the structure of ATDD (Acceptance Test-Driven Development) documents. ATDD documents are divided into two phases:
- **Draft phase** (Phase 2): Before coding, plan acceptance test cases based on BDD scenarios for user review and confirmation.
- **Finalization phase** (Phase 5): After self-verification is complete, fill in the actual verification results and evidence.

ATDD documents directly correspond to and trace back to BDD scenarios.

## Document Structure

Every ATDD document **must** follow this structure:
```markdown
# [Feature Name]

## BDD Reference
[BDD document path, e.g., PROJECT_STATE/BDD/user-email-login.md]

## Verification Strategy
- Date: [Verification date]
- Strategy: [One or more: run test framework, invoke tools (MCP/skill), execute verification commands, write test scripts, build/run project...]
- Approach: [One or more: check test reports, tool call results, command/script execution results, project build artifacts, project runtime status...]

## Test Cases

### Test Case 1: [Test Name]
**Corresponding BDD Scenario**: [Scenario name, e.g., "Successful login with valid credentials"]
**Verifier**: Agent auto-verification / Human verification (for scenarios only verifiable by humans, e.g., visual effects, cross-system verification, design mockup comparison, etc.)

**Verification Approach**: [The thought process and procedure for obtaining evidence — how to verify the behavior of this scenario, including specific operation paths, verification methods, and observation points. If human verification, describe what the human needs to focus on]
**Test Command** (if applicable): [Specific command]
**Test Tool** (if applicable): [Tool name]

**Result**: Pass / Fail (Leave empty in draft phase)
**Evidence**: [Content confirming the result. When changes are human-visible (UI/API/data output, etc.), **must** include human-intuitively-verifiable evidence (e.g., post-change key screenshots, actual request/response examples, input/output mappings), not just test-passed information] (Leave empty in draft phase)

---

### Test Case 2: [Test Name]
...

## Summary

| BDD Scenario | Test Case | Verifier | Result |
|---|---|---|---|
| [Scenario Name] | TC-1 | Agent / Human | (Leave empty in draft phase) |
| ... | ... | ... | ... |

**Total Scenarios**: [N]
**Passed**: [N] (Leave empty in draft phase)
**Failed**: [N] (Leave empty in draft phase)
**Remarks**: [Any limitations, skipped scenarios, or conditions that could not be fully verified]
```

## Non-Text Evidence

Non-text evidence (screenshots, data files, etc.) produced during self-verification should be saved to
the `PROJECT_STATE/ATDD/<feature-name>/` directory, using meaningful filenames.
When changes are human-visible, proactively collect post-change key screenshots as evidence. Naming conventions:
- Screenshots: `tc1-homepage.jpeg`, `tc2-result.jpeg` (test case number + descriptive name)
- General evidence: `evidence-tc3.log`, `evidence-tc3.json`

Reference them using relative paths in the acceptance.md evidence field:
- **Image evidence** (`.jpeg`, `.png`, `.gif`) uses Markdown image syntax for direct preview:
  ```
  ![Homepage screenshot](./tc1-homepage.jpeg)
  ```
- **Non-image evidence** (`.log`, `.json`, `.csv`, etc.) uses regular link syntax:
  ```
  See [response data](./evidence-tc3.json)
  ```

Supported evidence file types:
- Screenshots: `.jpeg`, `.png`, `.gif`
- Data files: `.json`, `.csv`, `.xml`
- Other: any file that demonstrates verification results

## Writing Guidelines

### Draft Phase (Phase 2)
1. Each test case traces its source scenario via the "**Corresponding BDD Scenario**" field, without repeating Given/When/Then.
2. **Verification Approach** describes the thought process and procedure for obtaining evidence — specific operation paths, verification methods, and observation points.
3. Include the test commands or tools **planned for use** (even if test code has not been written yet, describe the expected command format).
4. Leave result and evidence fields empty, marking them as "(Pending verification)".
5. Leave the result column in the summary table empty.

### BDD vs ATDD Responsibility Division

BDD and ATDD answer different questions and should not repeat the same content:

- **BDD answers "what behavior is expected"** — describes the system's preconditions, operations, and expected results using Given/When/Then in business language
- **ATDD answers "how to verify that behavior"** — describes specific verification methods through verification approaches, test commands/tools, results, and evidence

ATDD traces BDD scenarios via the "Corresponding BDD Scenario" field, without repeating the description of functional behavior, focusing instead on the verification level.

**Example comparison** — how the same scenario appears in the two documents:
- In BDD: `Given a user with email alice@example.com is registered in the system` / `When the user submits a login request` / `Then login succeeds and the system returns an authentication token`
- In ATDD: `Corresponding BDD Scenario: Successful login with valid credentials` / `Verification Approach: Use pytest to send a POST request, assert status code 200 and that the response contains a token field`

### Finalization Phase (Phase 5)
1. Fill in each test case's **Result** (Pass/Fail) based on actual self-verification outcomes.
2. Fill in each test case's **Evidence** (test output, log snippets/data, screenshots, etc.).
3. **Check human verifiability**: For each test case, review whether its evidence allows a person to judge correctness without reading code. When changes are human-visible (UI/API/data output, etc.), **must** include key screenshots (post-change screenshots, actual request/response examples, input/output mappings, etc.) to reach evidence Level S.
4. For non-text evidence files, use relative path references in the evidence field. **Image evidence uses Markdown image syntax** (e.g., `![Homepage screenshot](./evidence-tc1.jpeg)`), non-image evidence uses link syntax (e.g., `See [log](./evidence-tc2.log)`). Screenshots should be named using the `tc<number>-<description>.jpeg` format.
5. Fill in the summary table.
6. Record results truthfully. If a scenario cannot be verified through automated testing, state this honestly and describe what was checked.
7. Note any limitations in the remarks.

## Complete Example (Finalized)

```markdown
# User Email Login

## BDD Reference
PROJECT_STATE/BDD/user-email-login.md

## Verification Strategy
- Date: 2026-04-09
- Strategy: Test framework - pytest (v8.1)
- Approach: Automated unit tests + API call result inspection

## Test Cases

### Test Case 1: Valid credentials login succeeds
**Corresponding BDD Scenario**: Successful login with valid credentials
**Verifier**: Agent auto-verification

**Verification Approach**: Use pytest to send a POST /api/login request (request body contains email and password), assert response status code 200 and that the response body contains a token field, decode JWT to verify the exp claim matches the 24-hour validity period
**Test Command**: `pytest tests/test_login.py::test_valid_login -v`

**Result**: Pass
**Evidence**: Test output shows "assert response.status_code == 200" assertion passed. JWT decoded with correct exp claim.
![Test output screenshot](./evidence-tc1.jpeg)

---

### Test Case 2: Malformed email is rejected
**Corresponding BDD Scenario**: Login with invalid email format
**Verifier**: Agent auto-verification

**Verification Approach**: Use pytest to send a POST /api/login request (with invalid email format), assert response status code 400 and that the response body contains the "Invalid email format" error message, confirm that the validation layer rejects before the database query
**Test Command**: `pytest tests/test_login.py::test_malformed_email -v`

**Result**: Pass
**Evidence**: Validation layer rejected the request before the database query.
![Test output screenshot](./evidence-tc2.jpeg)

---

### Test Case 3: Database unreachable returns service unavailable error
**Corresponding BDD Scenario**: Login when service is unavailable
**Verifier**: Agent auto-verification

**Verification Approach**: Simulate unavailable state by stopping the database container via docker stop, use curl to send a login request with valid credentials, check response status code 503 and whether the error message is "Service temporarily unavailable", restart the database after verification
**Test Tool**: curl, docker

**Result**: Pass
**Evidence**: After stopping the database, request sent via curl. Received 503 response with correct error message body. Database restarted after testing.
![Response screenshot](./evidence-tc3.jpeg)

---

### Test Case 4: Login page visual presentation is correct
**Corresponding BDD Scenario**: Successful login with valid credentials (visual verification portion)
**Verifier**: Human verification

**Verification Approach**: Check the visual presentation of the login page — input field alignment, button styles, error message color and position consistency with the design mockup. Agent provides page screenshot for human confirmation.
**Points for human attention**: Input field spacing, whether error message text color is red, overall layout consistency with design mockup

**Result**: Pending human confirmation
**Evidence**:
![Login page screenshot](./evidence-tc4-login-page.jpeg)

---

## Summary

| BDD Scenario | Test Case | Verifier | Result |
|---|---|---|---|
| Successful login with valid credentials | TC-1 | Agent | Pass |
| Login with invalid email format | TC-2 | Agent | Pass |
| Login when service is unavailable | TC-3 | Agent | Pass |
| Login page visual presentation is correct | TC-4 | Human | Pending human confirmation |

**Total Scenarios**: 4
**Passed**: 3
**Failed**: 0
**Pending human confirmation**: 1
```

## Complete Example (Draft State)

In the draft phase, result and evidence fields are left empty:

```markdown
# User Email Login

## BDD Reference
PROJECT_STATE/BDD/user-email-login.md

## Verification Strategy
- Date: 2026-04-09
- Strategy: Test framework - pytest (v8.1)
- Approach: Automated unit tests + API call result inspection

## Test Cases

### Test Case 1: Valid credentials login succeeds
**Corresponding BDD Scenario**: Successful login with valid credentials
**Verifier**: Agent auto-verification

**Verification Approach**: Use pytest to send a POST /api/login request (request body contains email and password), assert response status code 200 and that the response body contains a token field, decode JWT to verify the exp claim matches the 24-hour validity period
**Test Command** (Planned): `pytest tests/test_login.py::test_valid_login -v`

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 2: Malformed email is rejected
**Corresponding BDD Scenario**: Login with invalid email format
**Verifier**: Agent auto-verification

**Verification Approach**: Use pytest to send a POST /api/login request (with invalid email format), assert response status code 400 and that the response body contains the "Invalid email format" error message
**Test Command** (Planned): `pytest tests/test_login.py::test_malformed_email -v`

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 3: Database unreachable returns service unavailable error
**Corresponding BDD Scenario**: Login when service is unavailable
**Verifier**: Agent auto-verification

**Verification Approach**: Simulate unavailable state by stopping the database container via docker stop, use curl to send a login request with valid credentials, check response status code 503 and error message
**Test Tool** (Planned): curl, docker

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 4: Login page visual presentation is correct
**Corresponding BDD Scenario**: Successful login with valid credentials (visual verification portion)
**Verifier**: Human verification

**Verification Approach**: Check the visual presentation of the login page — input field alignment, button styles, error message color and position consistency with the design mockup. Agent provides page screenshot for human confirmation
**Points for human attention**: Input field spacing, whether error message text color is red, overall layout consistency with design mockup

**Result**: (Pending human confirmation)
**Evidence**: (To be collected)

---

## Summary

| BDD Scenario | Test Case | Verifier | Result |
|---|---|---|---|
| Successful login with valid credentials | TC-1 | Agent | (Pending verification) |
| Login with invalid email format | TC-2 | Agent | (Pending verification) |
| Login when service is unavailable | TC-3 | Agent | (Pending verification) |
| Login page visual presentation is correct | TC-4 | Human | (Pending human confirmation) |

**Total Scenarios**: 4
**Passed**: (Pending verification)
**Failed**: (Pending verification)
**Pending human confirmation**: 1
```
