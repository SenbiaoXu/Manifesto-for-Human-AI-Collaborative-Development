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
[BDD document path, e.g., PROJECT_STATE/BDD/user-login-email-validation.md]

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
**Evidence**: [Content confirming the result — test output, log snippets, screenshot references, etc.] (Leave empty in draft phase)

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
the `PROJECT_STATE/ATDD/<feature-name>/` directory, using meaningful filenames (e.g., `evidence-tc1.png`, `evidence-tc2.log`).

Reference them using relative paths in the acceptance.md evidence field:
```
See [evidence screenshot](./evidence-tc1.png)
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
3. For non-text evidence files, use relative path references in the evidence field (e.g., `See [screenshot](./evidence-tc1.png)`).
4. Fill in the summary table.
5. Record results truthfully. If a scenario cannot be verified through automated testing, state this honestly and describe what was checked.
6. Note any limitations in the remarks.

## Complete Example (Finalized)

```markdown
# User Login with Email Validation

## BDD Reference
PROJECT_STATE/BDD/user-login-email-validation.md

## Verification Strategy
- Date: 2026-04-09
- Strategy: Test framework - pytest (v8.1)
- Approach: Automated unit tests + API call result inspection

## Test Cases

### Test Case 1: Valid credentials return JWT token
**Given (Precondition)**: User alice@example.com has been inserted in the test database (password hash corresponding to "SecurePass123!")
**When (Action)**: POST /api/login with request body {"email": "alice@example.com", "password": "SecurePass123!"}
**Then (Expected Result)**: Response status code 200, response body contains "token" field, token is a valid JWT with 24-hour expiration

**Test Command**: `pytest tests/test_login.py::test_valid_login -v`

**Result**: Pass
**Evidence**: Test output shows "assert response.status_code == 200" assertion passed. JWT decoded with correct exp claim. See [test output screenshot](./evidence-tc1.png)

---

### Test Case 2: Invalid password returns 401
**Given (Precondition)**: User alice@example.com has been inserted in the test database
**When (Action)**: POST /api/login with request body {"email": "alice@example.com", "password": "WrongPassword"}
**Then (Expected Result)**: Response status code 401, response body contains "Invalid credentials", no token field

**Test Command**: `pytest tests/test_login.py::test_invalid_password -v`

**Result**: Pass
**Evidence**: Test output confirms status code 401 and error message matches.

---

### Test Case 3: Malformed email returns 400
**Given (Precondition)**: No specific setup required
**When (Action)**: POST /api/login with request body {"email": "not-an-email", "password": "AnyPassword"}
**Then (Expected Result)**: Response status code 400, response body contains "Invalid email format"

**Test Command**: `pytest tests/test_login.py::test_malformed_email -v`

**Result**: Pass
**Evidence**: Validation layer rejected the request before the database query.

---

### Test Case 4: SQL injection is sanitized
**Given (Precondition)**: Test database populated with standard data
**When (Action)**: POST /api/login with request body {"email": "admin@example.com' OR '1'='1", "password": "anything"}
**Then (Expected Result)**: Response status code 400, no database query executed with raw input

**Test Command**: `pytest tests/test_login.py::test_sql_injection_sanitized -v`
**Test Tool**: Database query log analysis

**Result**: Pass
**Evidence**: Parameterized queries used. Input was rejected by email format validation before reaching the database layer. See [query log](./evidence-tc4.log)

---

### Test Case 5: Database unreachable returns 503
**Given (Precondition)**: Database container stopped via `docker stop auth-db`
**When (Action)**: Send POST /api/login request with valid credentials
**Then (Expected Result)**: Response status code 503, error message "Service temporarily unavailable"

**Test Tool**: curl command-line tool

**Result**: Pass
**Evidence**: After stopping the database, request sent via curl. Received 503 response with correct error message body. Database restarted after testing. See [response screenshot](./evidence-tc5.png)

---

## Summary

| BDD Scenario | Test Case | Result |
|---|---|---|
| Successful login with valid credentials | TC-1 | Pass |
| Login with invalid password | TC-2 | Pass |
| Login with malformed email | TC-3 | Pass |
| SQL injection attack attempt | TC-4 | Pass |
| Login when database is unreachable | TC-5 | Pass |

**Total Scenarios**: 5
**Passed**: 5
**Failed**: 0
```

## Complete Example (Draft State)

In the draft phase, result and evidence fields are left empty:

```markdown
# User Login with Email Validation

## BDD Reference
PROJECT_STATE/BDD/user-login-email-validation.md

## Verification Strategy
- Date: 2026-04-09
- Strategy: Test framework - pytest (v8.1)
- Approach: Automated unit tests + API call result inspection

## Test Cases

### Test Case 1: Valid credentials return JWT token
**Given (Precondition)**: User alice@example.com inserted in test database (password hash corresponding to "SecurePass123!")
**When (Action)**: POST /api/login with request body {"email": "alice@example.com", "password": "SecurePass123!"}
**Then (Expected Result)**: Response status code 200, response body contains "token" field, token is a valid JWT with 24-hour expiration

**Test Command** (Planned): `pytest tests/test_login.py::test_valid_login -v`

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 2: Invalid password returns 401
**Given (Precondition)**: User alice@example.com inserted in test database
**When (Action)**: POST /api/login with request body {"email": "alice@example.com", "password": "WrongPassword"}
**Then (Expected Result)**: Response status code 401, response body contains "Invalid credentials", no token field

**Test Command** (Planned): `pytest tests/test_login.py::test_invalid_password -v`

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 3: Malformed email returns 400
**Given (Precondition)**: No specific setup required
**When (Action)**: POST /api/login with request body {"email": "not-an-email", "password": "AnyPassword"}
**Then (Expected Result)**: Response status code 400, response body contains "Invalid email format"

**Test Command** (Planned): `pytest tests/test_login.py::test_malformed_email -v`

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 4: SQL injection is sanitized
**Given (Precondition)**: Test database populated with standard data
**When (Action)**: POST /api/login with request body {"email": "admin@example.com' OR '1'='1", "password": "anything"}
**Then (Expected Result)**: Response status code 400, no database query executed with raw input

**Test Command** (Planned): `pytest tests/test_login.py::test_sql_injection_sanitized -v`

**Result**: (Pending verification)
**Evidence**: (Pending verification)

---

### Test Case 5: Database unreachable returns 503
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
| Login with invalid password | TC-2 | (Pending verification) |
| Login with malformed email | TC-3 | (Pending verification) |
| SQL injection attack attempt | TC-4 | (Pending verification) |
| Login when database is unreachable | TC-5 | (Pending verification) |

**Total Scenarios**: 5
**Passed**: (Pending verification)
**Failed**: (Pending verification)
```
