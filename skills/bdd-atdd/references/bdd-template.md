# BDD Document Template

## Purpose
This template defines the structure of BDD (Behavior-Driven Development) specification documents. BDD documents capture the expected behavior of a feature **before** code is written,
using the Given/When/Then format.

## Document Structure
Every BDD document **must** follow this structure:

```markdown
# Feature: [Feature Name]

## Description
[1-3 sentences describing what the feature does and why it exists]

## Background (Optional)
[Shared preconditions that apply to all scenarios in this feature]

Given [shared precondition]

## Scenario: [Scenario Name]
Given [precondition]
And [additional precondition]
When [action]
Then [expected result]
And [additional result]

## Scenario: [Another Scenario Name]
...

## Boundary Cases

### Scenario: [Boundary Case Name]
Given [boundary case precondition]
When [boundary case action]
Then [boundary case expected result]

## Error Conditions

### Scenario: [Error Scenario Name]
Given [error precondition]
When [action that triggers the error]
Then [expected error handling]
```

## Writing Guidelines

1. **Use business/domain language, avoid technical implementation language**. BDD documents are meant for human review and must use language understandable by non-technical stakeholders (product managers, business analysts, etc.). Technical implementation details (CLI flags, API endpoint paths, data structure names, configuration keys, etc.) belong in ATDD, not BDD.
   - ✓ Good: "Launch browser in headless mode (in background)" — describes behavior, instantly understandable
   - ✗ Bad: "Launch browser with `--headless` flag" — `--headless` is a technical implementation detail, belongs in ATDD
   - ✓ Good: "After the user submits a login request, the system validates credentials and returns an authentication token" — describes behavior
   - ✗ Bad: "POST /api/login returns JWT token" — API path and data structure are technical details, belongs in ATDD
2. Cover all scenarios: happy path, boundary cases, and error conditions.
3. Each scenario should test only one behavior. If a scenario tests multiple things, split it into separate independent scenarios.
4. Use "And" to chain multiple preconditions or results within a single scenario.
5. In "Then" clauses, describe **observable behavioral outcomes** — avoid vague assertions like "it works correctly"; instead describe specific perceivable behavior changes (e.g., "the system returns a success response", "the user is redirected to the homepage", "the page displays an error message"). Exact assertion values and verification methods are defined in ATDD.
6. Only use a "Background" section when multiple scenarios share the same precondition setup.
7. **BDD vs ATDD language boundary**:
   - BDD answers "**what behavior is expected**" — uses business language to describe inputs, actions, and expected outcomes
   - ATDD answers "**how to verify that behavior**" — uses technical language to define specific test steps, commands, assertions, and verification methods
   - If a description requires technical background to understand, it likely belongs in ATDD, not BDD

## Complete Example

```markdown
# Feature: User Email Login

## Description
Allow users to log in using their email address and password. The system validates the email format and login credentials, returning an authentication token upon successful login.

## Background
Given the user registration system is running

## Scenario: Successful login with valid credentials
Given a user with email "alice@example.com" is registered in the system
And the user's password is "SecurePass123!"
When the user submits a login request with this email and password
Then the login succeeds and the system returns an authentication token
And the token is valid for 24 hours

## Boundary Cases

### Scenario: Login with invalid email format
When the user submits a login request with an invalid email address "not-an-email"
Then the system rejects the login and displays "Invalid email format"

## Error Conditions

### Scenario: Login when service is unavailable
Given the database that the login service depends on is unavailable
When the user submits a login request with valid credentials
Then the system displays "Service temporarily unavailable, please try again later"
```
