---
name: implement-plan
description: Execute a validated implementation plan using specialized agents
---

# Implement Plan Command

This command orchestrates the implementation of a validated plan by coordinating specialized agents.

## Prerequisites

- A plan must be finalized and **explicitly confirmed by the developer**
- The plan should include clear implementation steps and file changes

## Execution Steps

1. **Confirm plan approval**
   - Do NOT proceed without explicit developer confirmation
   - If uncertain, ask: "Is the plan approved and ready for implementation?"

2. **Invoke drupal-implementer**
   - Pass the validated plan from the current session context
   - The implementer will create/modify files according to the plan
   - The implementer will run phpcs after writing PHP

3. **Invoke drupal-code-reviewer**
   - Review all modified files
   - Check for coding standards violations
   - Check for security issues
   - If issues found, implementer must fix them before proceeding

4. **Invoke test-runner**
   - Run tests related to modified code
   - If tests fail, implementer must fix them before proceeding

5. **Report completion**
   - Summarize what was implemented
   - List files created/modified
   - Report test results
   - Suggest next steps (e.g., manual testing, config export)

## Example Usage

User says: "implement the plan"

You respond:
1. Confirm: "The plan is approved. Proceeding with implementation."
2. Spawn drupal-implementer with plan context
3. After implementer finishes, spawn drupal-code-reviewer
4. After reviewer finishes, spawn test-runner
5. Report summary

## Safety

- Never start implementation without plan approval
- If code review finds issues, fix them before running tests
- If tests fail, fix them before declaring implementation complete

## TODO

- Project-specific implementation workflows may vary
- Additional agents may be needed (e.g., deployment, documentation)
