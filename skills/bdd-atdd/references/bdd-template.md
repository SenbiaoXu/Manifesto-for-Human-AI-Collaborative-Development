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

1. Write scenarios from the user's perspective — describe behavior, not implementation details.
2. Cover all scenarios: happy path, boundary cases, and error conditions.
3. Each scenario should test only one behavior. If a scenario tests multiple things, split it into separate independent scenarios.
4. Use "And" to chain multiple preconditions or results within a single scenario.
5. Be specific in "Then" clauses — avoid vague assertions like "it works correctly"; instead say "the response status code is 200 and the response body contains a 'token' field".
6. Only use a "Background" section when multiple scenarios share the same precondition setup.

## Complete Example

```markdown
# Feature: User Login with Email Validation

## Description
Allow users to log in using their email address and password. The system validates the email format, checks credentials, and returns an authentication token upon success.

## Background
Given the user registration system is running
And the database contains registered users

## Scenario: Successful login with valid credentials
Given a user with email "alice@example.com" exists in the system
And the user's password is "SecurePass123!"
When the user submits a login request with email "alice@example.com" and password "SecurePass123!"
Then the response status code is 200
And the response body contains a valid JWT token
And the token has a 24-hour expiration

## Scenario: Login with invalid password
Given a user with email "alice@example.com" exists in the system
When the user submits a login request with email "alice@example.com" and password "WrongPassword"
Then the response status code is 401
And the response body contains the error message "Invalid credentials"
And no JWT token is returned

## Boundary Cases

### Scenario: Login with malformed email
When the user submits a login request with email "not-an-email" and password "AnyPassword"
Then the response status code is 400
And the response body contains the error message "Invalid email format"

### Scenario: SQL injection attack attempt
When the user submits a login request with email "admin@example.com' OR '1'='1" and password "anything"
Then the response status code is 400
And the input has been sanitized — no SQL injection occurred
And the response body contains the error message "Invalid email format"

## Error Conditions

### Scenario: Login when database is unreachable
Given the database connection is unavailable
When the user submits a login request with valid credentials
Then the response status code is 503
And the response body contains the error message "Service temporarily unavailable"
And the error has been logged for administrator review
```
