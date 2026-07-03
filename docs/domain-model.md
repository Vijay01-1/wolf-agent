# Wolf-Agent Domain Model

## Request
Represents a user's objective.
Created by the CLI.
Consumed by the classifier.

---

## Workflow
Represents the execution strategy selected for a request.
Determines allowed tools, execution flow, and stopping conditions.

---

## Context
Represents the information supplied to the language model.
Contains only the data required for the current task.

---

## Action
Represents a decision produced by the language model.
Examples: read_file, search, edit_file, run_command.

---

## Tool
Represents a capability provided by the framework.
Tools perform deterministic work and return structured results.

---

## Result
Represents the output produced by a tool.
Used to update state and determine the next step.

---

## State
Represents the current execution status.
Tracks progress without requiring the language model to remember previous actions.