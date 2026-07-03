# Wolf-Agent Workflows

## Purpose

Wolf-Agent is built around **workflows**, not prompts.

Instead of letting the language model freely decide how every request should be handled, the framework first classifies the user's request into a predefined workflow.

Each workflow has:

- a single responsibility
- a limited set of tools
- a predictable execution path
- clear stopping conditions

This architecture keeps the agent deterministic, reduces unnecessary LLM calls, minimizes token usage, and makes the system suitable for lightweight local models running on CPU-only hardware.

---

# Why Workflows?

Large language models are excellent at reasoning, but they are poor at managing execution.

Without constraints they tend to:

- repeatedly call the same tools
- perform unnecessary searches
- over-plan simple tasks
- consume excessive context
- forget the original objective

Wolf-Agent solves this by allowing the framework—not the model—to control execution.

The language model is responsible for reasoning.

The framework is responsible for orchestration.

---

# Workflow Overview

Wolf-Agent currently supports four workflows.

| Workflow | Purpose |
|----------|---------|
| ANALYZE | Understand and explain code without making changes |
| COMMAND | Execute terminal commands safely |
| PATCH | Perform a small localized code modification |
| REFACTOR | Perform complex multi-file engineering tasks |

---

# ANALYZE

## Purpose

Understand an existing codebase.

The objective is to answer questions, explain code, review architecture, identify problems, or provide recommendations.

The workflow never modifies project files.

---

## Typical Requests

- Explain this file.
- Review this module.
- Find possible bugs.
- Suggest improvements.
- Explain the architecture.
- Compare two implementations.
- Find security issues.

---

## Allowed Tools

- read_file
- read_chunk
- search
- list_files
- git_diff (optional)

---

## Forbidden Tools

- edit_file
- write_file
- planner
- shell
- memory_write

---

## Execution Flow

User Request

↓

Read relevant files

↓

Reason over collected context

↓

Generate response

↓

Finish

---

## Stop Conditions

The workflow ends immediately when enough information has been collected to answer the user's request.

No additional exploration should occur after the objective has been satisfied.

---

# COMMAND

## Purpose

Execute terminal commands without modifying project files.

This workflow is isolated from editing operations.

---

## Typical Requests

- Run pytest
- Run npm test
- Run cargo build
- Check git status
- Execute a Python script

---

## Allowed Tools

- shell
- git

---

## Forbidden Tools

- planner
- read_file
- edit_file
- write_file
- memory_write

---

## Execution Flow

User Request

↓

Validate command

↓

Execute

↓

Collect output

↓

Summarize

↓

Finish

---

## Stop Conditions

The workflow finishes immediately after command execution and result summarization.

---

# PATCH

## Purpose

Perform a small, localized code modification.

This workflow is optimized for speed and intentionally bypasses planning.

---

## Typical Requests

- Fix typo
- Rename variable
- Add logging
- Update timeout
- Change constant
- Fix import
- Update README

---

## Characteristics

- Single file
- Small scope
- Low risk
- No planning
- Fast execution

---

## Allowed Tools

- read_file
- read_chunk
- edit_file
- verify

---

## Forbidden Tools

- planner
- shell
- memory_write

---

## Execution Flow

Read file

↓

Generate patch

↓

Apply modification

↓

Verify

↓

Finish

---

## Stop Conditions

The workflow ends after verification succeeds.

If verification fails, retry within the current file only.

---

# REFACTOR

## Purpose

Perform complex engineering work requiring multiple coordinated modifications.

This is the only workflow allowed to invoke the planner.

---

## Typical Requests

- Implement new feature
- Split large modules
- Multi-file refactor
- Architectural redesign
- Framework migration
- Large bug fix

---

## Characteristics

- Multiple files
- Planning required
- Incremental execution
- Verification required

---

## Allowed Tools

All project tools.

---

## Execution Flow

Understand request

↓

Planner

↓

Execute plan step

↓

Verify

↓

Continue until complete

↓

Finish

---

## Stop Conditions

The workflow completes when every planned task has been successfully executed and verified.

---

# Workflow Selection

The classifier determines the appropriate workflow before any tool execution begins.

General mapping:

Explanation or review

→ ANALYZE

Terminal execution

→ COMMAND

Small localized edit

→ PATCH

Large engineering task

→ REFACTOR

The classifier should prefer the simplest workflow capable of completing the request.

---

# Design Principles

Every workflow follows the same engineering principles.

## Single Responsibility

Each workflow exists for exactly one purpose.

---

## Minimal Tool Access

Only expose the tools required for that workflow.

Unused capabilities increase complexity and reduce reliability.

---

## Python Controls Execution

The framework controls execution.

The language model performs reasoning.

The model should never control the overall execution loop.

---

## Answer As Soon As Possible

The objective is not to maximize tool usage.

The objective is to satisfy the user's request using the smallest number of actions.

---

## Prefer Deterministic Logic

Whenever a task can be solved reliably by Python, it should not be delegated to the language model.

---

## Keep Context Small

Only provide the information required for the current workflow.

Avoid carrying unnecessary history or unrelated repository context.

---

## Planner Is Expensive

Planning is a specialized capability.

Only REFACTOR is allowed to invoke the planner.

---

## Simplicity Before Intelligence

A predictable workflow is more valuable than an overly autonomous workflow.

The framework should reduce unnecessary reasoning rather than expecting the language model to solve every problem.