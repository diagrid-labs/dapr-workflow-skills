# Dapr Workflow Skills

This repository contains skill definitions that can be used with Claude Code to build Dapr Workflow applications.

## Prerequisites

- Claude Code

## How to use this

- Clone this repo
- Open a terminal and navigate to the cloned repo
- Start Claude code
- Use a prompt to build a Dapr workflow application (see the example below)

## Prompt example

Create a plan to build a Dapr workflow app in .NET. The workflow automates the onboarding process of a new employee. The first activity is employee registration, which creates a new employeeId in a data store. Then 4 activities are called in parallel:
  1. AddEmployeeToInternalCommsTool
  2. AddEmployeeToBenefitsProgram
  3. UpdateOrgChart
  4. SendWelcomePackage

The input records for the 4 parallel activities include the employeeId. The workflow output should include the employeeId.