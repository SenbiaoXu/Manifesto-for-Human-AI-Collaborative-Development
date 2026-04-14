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
**Given (Precondition)**: [Specific test setup — exact data, state, commands]
**When (Action)**: [Specific action performed — commands, API calls, tool invocations]
**Then (Expected Result)**: [Expected outcome — exact assertions, expected output]

**Test Command** (if applicable): [Specific command]
**Test Tool** (if applicable): [Tool name]

**Result**: Pass / Fail (Leave empty in draft phase)
**Evidence**: [Content confirming the result. When changes are human-visible (UI/API/data output, etc.), **must** include human-intuitively-verifiable comparative evidence (e.g., before/after screenshots, actual request/response examples, input/output mappings), not just test-passed information] (Leave empty in draft phase)

---

### Test Case 2: [Test Name]
...

## Summary

| BDD Scenario | Test Case | Result |
|---|---|---|
| [Scenario Name] | TC-1 | (Leave empty in draft phase) |
| ... | ... | ... |

**Total Scenarios**: [N]
**Passed**: [N] (Leave empty in draft phase)
**Failed**: [N] (Leave empty in draft phase)
**Remarks**: [Any limitations, skipped scenarios, or conditions that could not be fully verified]
```

## Non-Text Evidence

Non-text evidence (screenshots, data files, etc.) produced during self-verification should be saved to
the `PROJECT_STATE/ATDD/<feature-name>/` directory, using meaningful filenames.
When changes are human-visible, proactively collect before/after comparative evidence. Naming conventions:
- Before change: `before-tc1.png`, `before-tc2.json`
- After change: `after-tc1.png`, `after-tc2.json`
- General evidence: `evidence-tc3.log`

Reference them using relative paths in the acceptance.md evidence field:
```
Before: See [pre-change screenshot](./before-tc1.png)
After: See [post-change screenshot](./after-tc1.png)
```

Supported evidence file types:
- Screenshots: `.png`, `.jpg`, `.gif`
- Data files: `.json`, `.csv`, `.xml`
- Other: any file that demonstrates verification results

## Writing Guidelines

### Draft Phase (Phase 2)
1. The summary table **must** clearly indicate which BDD scenario each test case corresponds to.
2. Given/When/Then should describe the **planned** test setup, actions, and expected results in detail.
3. Include the test commands or tools **planned for use** (even if test code has not been written yet, describe the expected command format).
4. Leave result and evidence fields empty, marking them as "(Pending verification)".
5. Leave the result column in the summary table empty.

### Finalization Phase (Phase 5)
1. Fill in each test case's **Result** (Pass/Fail) based on actual self-verification outcomes.
2. Fill in each test case's **Evidence** (test output, log snippets/data, screenshots, etc.).
3. **Check human verifiability**: For each test case, review whether its evidence allows a person to judge correctness without reading code. When changes are human-visible (UI/API/data output, etc.), **must** include comparative evidence (before/after screenshots, actual request/response examples, input/output mappings, etc.) to reach evidence Level S.
4. For non-text evidence files, use relative path references in the evidence field (e.g., `See [screenshot](./evidence-tc1.png)`). Before/after comparative evidence should be named `before-<tc>.png` and `after-<tc>.png`.
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
**Given (Precondition)**: User alice@example.com has been inserted in the test database (password hash corresponding to "SecurePass123!")
**When (Action)**: POST /api/login with request body {"email": "alice@example.com", "password": "SecurePass123!"}
**Then (Expected Result)**: Response status code 200, response body contains "token" field, token is a valid JWT with 24-hour expiration

**Test Command**: `pytest tests/test_login.py::test_valid_login -v`

**Result**: Pass
**Evidence**: Test output shows "assert response.status_code == 200" assertion passed. JWT decoded with correct exp claim. See [test output screenshot](./evidence-tc1.png)

---

### Test Case 2: Malformed email is rejected
**Given (Precondition)**: No specific setup required
**When (Action)**: POST /api/login with request body {"email": "not-an-email", "password": "AnyPassword"}
**Then (Expected Result)**: Response status code 400, response body contains "Invalid email format"

**Test Command**: `pytest tests/test_login.py::test_malformed_email -v`

**Result**: Pass
**Evidence**: Validation layer rejected the request before the database query. See [test output screenshot](./evidence-tc2.png)

---

### Test Case 3: Database unreachable returns service unavailable error
**Given (Precondition)**: Database container stopped via `docker stop auth-db`
**When (Action)**: Send POST /api/login request with valid credentials
**Then (Expected Result)**: Response status code 503, error message "Service temporarily unavailable"

**Test Tool**: curl command-line tool

**Result**: Pass
**Evidence**: After stopping the database, request sent via curl. Received 503 response with correct error message body. Database restarted after testing. See [response screenshot](./evidence-tc3.png)

---

## Summary

| BDD Scenario | Test Case | Result |
|---|---|---|
| Successful login with valid credentials | TC-1 | Pass |
| Login with invalid email format | TC-2 | Pass |
| Login when service is unavailable | TC-3 | Pass |

**Total Scenarios**: 3
**Passed**: 3
**Failed**: 0
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
**Given (Precondition)**: User alice@example.com inserted in test database (password hash corresponding to "SecurePass123!")
**When (Action)**: POST /api/login with request body {"email": "alice@example.com", "password": "SecurePass123!"}
**Then (Expected Result)**: Response status code 200, response body contains "token" field, token is a valid JWT with 24-hour expiration

**Test Command** (Planned): `pytest tests/test_login.py::test_valid_login -v`

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 2: Malformed email is rejected
**Given (Precondition)**: No specific setup required
**When (Action)**: POST /api/login with request body {"email": "not-an-email", "password": "AnyPassword"}
**Then (Expected Result)**: Response status code 400, response body contains "Invalid email format"

**Test Command** (Planned): `pytest tests/test_login.py::test_malformed_email -v`

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 3: Database unreachable returns service unavailable error
**Given (Precondition)**: Database container stopped via `docker stop auth-db`
**When (Action)**: Send POST /api/login request with valid credentials
**Then (Expected Result)**: Response status code 503, error message "Service temporarily unavailable"

**Test Tool** (Planned): curl command-line tool

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

## Summary

| BDD Scenario | Test Case | Result |
|---|---|---|
| Successful login with valid credentials | TC-1 | (Pending verification) |
| Login with invalid email format | TC-2 | (Pending verification) |
| Login when service is unavailable | TC-3 | (Pending verification) |

**Total Scenarios**: 3
**Passed**: (Pending verification)
**Failed**: (Pending verification)
```
