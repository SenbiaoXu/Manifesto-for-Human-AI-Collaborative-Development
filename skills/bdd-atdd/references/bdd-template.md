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
When [a series of detailed actions]
Then [final expected result]
And [additional result]

## Scenario: [Another Scenario Name]
...

```

## Writing Guidelines

1. **You must analyze existing code before writing scenarios**. Given/When/Then must be based on code facts rather than guesses. Before writing any scenario, do the following:
   - Understand the **default state** (what is displayed by default, what the initial data is)
   - Map out the user's **actual operation path** (e.g., what needs to be clicked first, which tab to switch to, what steps are needed to reach the target area)
   - **Common mistake**: Assuming components are visible by default while ignoring navigation steps, skipping intermediate operation paths and going straight to describing results, imagining page layouts from thin air
   - **Correct approach**: Read the code first, figure out "what steps does the user actually need to take to see this component", then write Given/When/Then

2. **Given/When/Then must describe the complete interaction process in detail**. Since BDD is the only document describing functional behavior (ATDD will not repeat Given/When/Then), the interaction steps in scenarios must be detailed enough for people to clearly understand the complete operation path and expected results.
   - When should describe the **complete operation path**, not vague actions:
     - ✓ "The user enters alice@example.com in the email input field, enters SecurePass123! in the password input field, and clicks the login button"
     - ✗ "The user attempts to log in"
   - Then should describe **specifically perceivable behavioral outcomes**, covering key details:
     - ✓ "The page redirects to the user's homepage, the top navigation bar displays the user's avatar and username, and the login button is no longer displayed in the upper right corner"
     - ✗ "Login succeeds"
   - Given should include **specific data and state required by the scenario**:
     - ✓ "A user with email alice@example.com is registered in the system, with password SecurePass123!, and account status is activated"
     - ✗ "User is registered"

3. **Use business/domain language, avoid technical implementation language**. BDD documents are meant for human review and must use language understandable by non-technical stakeholders (product managers, business analysts, etc.). Technical implementation details (command parameters, API endpoint paths, data structure names, configuration key names, etc.) belong in ATDD, not BDD.
   - ✓ Good: "Launch browser in headless mode (in background)" — describes behavior, instantly understandable
   - ✗ Bad: "Launch browser with `--headless` flag" — `--headless` is a technical implementation detail, belongs in ATDD

4. Categorize scenarios from a business perspective.
5. Each scenario should test only one behavior. If a scenario tests multiple things, split it into separate independent scenarios.
6. Use "And" to chain multiple preconditions or results within a single scenario.
7. Only use a "Background" section when multiple scenarios share the same precondition setup.
8. **Scenarios should be as comprehensive as possible**: Cover all happy path, boundary, and error cases. Some scenarios cannot be automatically verified by the Agent (e.g., visual effects, cross-system verification, design mockup comparison, etc.) — these scenarios should still be written in BDD and verified by humans in ATDD.
   - Do not omit scenarios just because "the Agent can't verify them" — BDD describes expected behavior; verification method is ATDD's concern
   - Common human-only-verifiable scenarios: design mockup fidelity (color, spacing, layout), visual effects (animation smoothness), dependency on specific external databases
9. **BDD vs ATDD responsibility division**:
   - BDD answers "**what behavior is expected**" — describes the complete interaction process and expected results in detail (operation paths, data input, UI changes, state changes)
   - ATDD answers "**how to verify that behavior**" — defines specific technical verification methods through verification approaches and test commands
   - If a description requires technical background to understand, it likely belongs in ATDD, not BDD

## Complete Example

```markdown
# Feature: User Email Login

## Description
Allow users to log in using their email address and password. The system validates the email format and login credentials, returning an authentication token upon successful login.

## Background
Given the user registration system is running
And the user has opened the login page, which displays an email input field, a password input field, and a login button

## Scenario: Successful login with valid credentials
Given a user with email "alice@example.com" is registered in the system
And the user's password is "SecurePass123!" and account status is activated
When the user enters "alice@example.com" in the email input field
And enters "SecurePass123!" in the password input field
And clicks the login button
Then the page redirects to the user's homepage
And the top navigation bar displays the user's avatar and username "alice"
And the login button disappears, replaced by a "Logout" button
And the authentication token issued by the system is valid for 24 hours

### Scenario: Login with invalid email format
When the user enters "not-an-email" in the email input field
And enters any content in the password input field
And clicks the login button
Then the page does not redirect and remains on the login page
And a red prompt text "Invalid email format" appears below the email input field
And the login button returns to a clickable state

### Scenario: Login when service is unavailable
Given the database service that the system depends on is unavailable
When the user enters valid email "alice@example.com" in the email input field
And enters correct password "SecurePass123!" in the password input field
And clicks the login button
Then the page does not redirect and remains on the login page
And a yellow banner appears at the top of the page with the message "Service temporarily unavailable, please try again later"
And no internal error information or technical details are displayed
```
