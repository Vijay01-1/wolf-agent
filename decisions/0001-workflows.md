Decision

We use workflows instead of one autonomous loop.

Context

The previous implementation allowed unrestricted tool
usage, causing unnecessary LLM calls and context growth.

Alternatives

One autonomous executor.

Separate executors.

Chosen Solution

Workflow architecture.

Consequences

Simpler execution.

Less flexibility.

Much lower token usage.