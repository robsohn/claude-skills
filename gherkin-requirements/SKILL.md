---
name: gherkin-requirements
description: Generate functional requirements for mobile applications using pure Gherkin syntax (Feature, Background, Scenario, Given/When/Then). Use this skill when creating or updating feature documentation, writing business-readable scenarios, or generating acceptance criteria for mobile app features. Outputs structured markdown files with user stories, acceptance criteria, and Gherkin scenarios covering happy paths, validation errors, and security edge cases.
---

# Gherkin Requirements Generator

Generate business-readable functional requirements using pure Gherkin syntax for mobile applications.

## Core Principles

**Business Language Only**

- Write scenarios in business terms that stakeholders understand
- Never mention: UI elements (buttons, screens, ViewControllers), API calls, HTTP codes, database details, technical implementation
- Focus on user actions and business outcomes

**One Business Rule Per Scenario**

- Each scenario tests exactly one business rule or behavior
- Split complex logic into multiple scenarios
- Keep scenarios focused and independently testable

**Constraint: And Chains**

- Maximum 5-6 consecutive `And` statements in any Given/When/Then block
- Split into separate scenarios when setup becomes longer
- Use Background section for common preconditions

## Output Structure

### For New Features

Generate complete feature documentation:

```gherkin
Feature: [Feature Name]

[User Story]
As a [role]
I want to [action]
So that [benefit]

Acceptance Criteria:
* [Criterion 1]
* [Criterion 2]
* [Criterion 3]

Background:
  Given [common precondition shared by all scenarios]
  And [another common precondition]

Scenario: [Descriptive scenario name]
  Given [precondition]
  And [additional context]
  When [action]
  Then [expected outcome]
  And [additional outcome]

Scenario: [Another scenario]
  ...
```

### For Updating Existing Features

Be flexible:

- Add new scenarios to existing feature files
- Update existing scenarios when requirements change
- Preserve user story and acceptance criteria unless explicitly asked to change them

## File Organization

- Each feature in its own file: `docs/features/[feature-name].md`
- Ask for filename if not provided by user
- Use kebab-case for filenames: `events-summary.md`, `meal-tracking.md`

## Scenario Coverage

Include scenarios for:

1. **Happy Path**: Primary success scenario
2. **Alternative Paths**: Valid variations of user behavior
3. **Validation Errors**: Invalid input, missing data, format errors
4. **Edge Cases**: Boundary conditions, empty states, first/last items
5. **Security**: Unauthorized access, expired sessions, permission checks

## Background Section

Use Background for:

- Common authentication/authorization state
- Shared data setup appearing in 3+ scenarios
- Standard preconditions (e.g., "Given I'm logged in")

Example:

```gherkin
Background:
  Given I'm logged in as a registered user
  And I have events configured in my account
```

## Scenario Naming

Use descriptive names that tell a story:

- ✅ "Display events summary with countdown"
- ✅ "Handle events happening today"
- ✅ "Reject future dates for past events"
- ❌ "Test event display"
- ❌ "Scenario 1"

## Data Tables

Use data tables for:

- Expected lists or sequences
- Multiple test cases with same structure
- Complex data comparisons

Example:

```gherkin
Then I should see events in this order:
  | Position | Event   | Date       |
  | 1        | Event B | 2024-02-15 |
  | 2        | Event A | 2024-03-01 |
  | 3        | Event C | 2024-04-10 |
```

## Examples of Business Language

**Good** (Business Language):

- "Given I have 3 events scheduled"
- "When I view my upcoming events"
- "Then I should see events sorted by date"
- "And the countdown should show '5 weeks, 1 day'"

**Bad** (Technical Language):

- "Given the database has 3 event records"
- "When I call GET /api/events"
- "Then the EventsViewController should display"
- "And the response code should be 200"

## Validation Error Scenarios

Structure validation scenarios clearly:

```gherkin
Scenario: Reject event without name
  Given I'm creating a new event
  When I submit without entering a name
  Then I should see error "Event name is required"
  And the event should not be created

Scenario: Reject past date for new event
  Given I'm creating a new event
  And today's date is "2024-01-15"
  When I enter event date as "2024-01-10"
  And I submit the form
  Then I should see error "Event date cannot be in the past"
  And the event should not be created
```

## Security Scenarios

Include security considerations:

```gherkin
Scenario: Prevent access to other users' events
  Given I'm logged in as User A
  And User B has event "Private Marathon"
  When I attempt to view User B's events
  Then I should not see "Private Marathon"
  And I should only see my own events

Scenario: Require authentication to view events
  Given I'm not logged in
  When I attempt to view events
  Then I should be prompted to log in
  And no event data should be displayed
```

## Workflow

1. **Understand the feature**: Clarify what the feature does and who uses it
2. **Identify business rules**: Extract distinct rules that need testing
3. **Draft user story**: Write As a/I want/So that format
4. **List acceptance criteria**: Bullet points of must-have functionality
5. **Create Background**: Extract common preconditions if 3+ scenarios share them
6. **Write scenarios**: One per business rule, descriptive names, business language
7. **Review coverage**: Ensure happy path, validations, edge cases, security
8. **Check And chains**: Split scenarios if any Given/When/Then has 6+ Ands
9. **Save to file**: Use provided filename or ask for one
