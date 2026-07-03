# Wolf-Agent Architecture

## Vision

Wolf-Agent is a lightweight local coding agent designed for software engineering tasks.

Its goal is not to imitate large commercial coding assistants by relying on massive models or cloud infrastructure.

Instead, Wolf-Agent focuses on **building a reliable execution framework** that enables small local language models to perform practical software engineering tasks efficiently.

The language model provides reasoning.

The framework provides structure.

This separation allows the system to remain fast, predictable, understandable, and resource efficient.

---

# Mission

Build a coding agent that:

- understands software projects
- explains existing code
- performs safe code modifications
- executes development commands
- supports multi-step engineering tasks
- runs efficiently on local hardware

The architecture should prioritize correctness, maintainability, and deterministic behavior over unnecessary autonomy.

---

# Core Philosophy

The intelligence of Wolf-Agent does not come from the language model alone.

It comes from the collaboration between deterministic software components and the language model.

The framework should solve every problem that does not require natural language reasoning.

The language model should only solve problems that require reasoning or code generation.

---

# Design Principles

## Framework First

The framework owns execution.

The language model assists execution.

Execution flow, tool access, state management, and stopping conditions are always controlled by Python.

---

## Simplicity Over Complexity

Every component should have a single responsibility.

Large modules that perform multiple unrelated tasks should be avoided.

If a component becomes difficult to understand, it should be divided into smaller modules.

---

## Deterministic Whenever Possible

If Python can reliably solve a problem, Python should solve it.

Examples include:

- workflow classification (when possible)
- duplicate tool detection
- state management
- caching
- context construction
- repository indexing
- stopping conditions

The language model should never be responsible for deterministic system behavior.

---

## Minimize Language Model Usage

Every LLM request has a cost.

Even when running locally, each request consumes:

- CPU
- memory
- inference time
- context window

The architecture should minimize unnecessary reasoning.

---

## Small Context, Better Reasoning

Large prompts reduce the quality of small language models.

Wolf-Agent intentionally provides only the minimum context required for the current task.

This improves reasoning quality while reducing latency.

---

## Workflow Driven

Every request belongs to exactly one workflow.

A workflow determines:

- execution strategy
- available tools
- context construction
- stopping conditions

This prevents unnecessary exploration and simplifies execution.

---

# High-Level Architecture

```
                   User
                     │
                     ▼
             Intent Classifier
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
     Workflow Selection     Reject Invalid
          │
          ▼
      Workflow Policy
          │
          ▼
    Context Construction
          │
          ▼
      Language Model
          │
          ▼
      Tool Decision
          │
          ▼
      Tool Executor
          │
          ▼
      Result Analysis
          │
          ▼
      Answer Gate
          │
      ┌───┴────┐
      │        │
      ▼        ▼
 Continue    Finish
```

---

# Request Lifecycle

Every request follows the same lifecycle.

## 1. Receive Request

The user submits a request.

Example:

> Explain executor.py

---

## 2. Classify Workflow

The classifier determines which workflow should handle the request.

Possible workflows:

- ANALYZE
- COMMAND
- PATCH
- REFACTOR

Only one workflow may be active.

---

## 3. Apply Workflow Policy

Each workflow exposes a different set of capabilities.

Example:

ANALYZE

Allowed:

- read
- search

Forbidden:

- edit
- shell

The language model cannot access tools outside the current workflow.

---

## 4. Build Context

Before contacting the language model, the framework prepares the smallest useful context.

This may include:

- user request
- workflow
- relevant files
- recent tool outputs
- execution state

Unnecessary information is excluded.

---

## 5. Language Model Reasoning

The language model performs reasoning based on the supplied context.

It may request:

- file reads
- searches
- edits
- command execution

depending on workflow permissions.

---

## 6. Execute Tool

Python validates the request before executing the tool.

The framework is responsible for:

- permission checks
- duplicate detection
- parameter validation
- execution safety

---

## 7. Evaluate Progress

After every tool execution, the framework evaluates whether enough information has been collected.

Instead of asking:

"What tool should we use next?"

Wolf-Agent asks:

"Can we answer the user's request now?"

---

## 8. Finish

When the objective has been satisfied, the workflow terminates immediately.

No unnecessary exploration should occur after completion.

---

# Components

## CLI

Receives user requests and initializes execution.

Responsible only for user interaction.

---

## Classifier

Maps user requests to workflows.

Should prefer deterministic classification whenever possible.

---

## Workflow Manager

Loads the selected workflow.

Applies workflow policy.

Coordinates execution.

---

## Context Manager

Builds minimal prompts.

Tracks execution state.

Prevents unnecessary context growth.

---

## Tool Registry

Defines available tools.

Controls permissions.

Provides a consistent execution interface.

---

## Tool Executor

Executes approved tools.

Handles validation.

Captures output.

Returns structured results.

---

## Language Model Client

Communicates with the configured language model.

Responsible only for inference.

No business logic belongs here.

---

## Planner

Generates execution plans.

Only available to the REFACTOR workflow.

---

## Verifier

Checks whether generated modifications satisfy the original request.

Acts as a quality gate before completion.

---

## Memory

Stores useful long-term information.

Memory should improve future execution without polluting current context.

---

# Responsibility Separation

Python is responsible for:

- execution
- workflows
- context
- permissions
- validation
- caching
- indexing
- stopping conditions

The language model is responsible for:

- reasoning
- explanation
- planning
- code generation
- summarization

This separation keeps the system predictable and maintainable.

---

# Out of Scope

The following features are intentionally excluded from the initial architecture.

- distributed agents
- autonomous internet browsing
- vector databases
- cloud-only services
- autonomous background execution
- GUI applications
- plugin ecosystems

These may be added in future versions if they support the project's goals.

---

# Future Evolution

The architecture is intentionally modular.

Future improvements should be introduced as independent components rather than increasing the complexity of existing modules.

Examples include:

- repository indexing
- semantic search
- context summarization
- execution caching
- parallel analysis
- test generation
- benchmark framework

Every addition should preserve the core principles defined in this document.

---

# Guiding Principle

A good coding agent is not the one with the largest language model.

A good coding agent is the one that gives the language model exactly the information it needs, exactly when it needs it, and no more.

The framework exists to make small models perform beyond their apparent limitations through good software engineering rather than larger model size.