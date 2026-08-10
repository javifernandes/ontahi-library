# Further Directions

Ontahí already makes domain meaning portable across Node, React, storage, transport, durable work,
code generation, and reflection. That is the important base. The next directions are not a list of
features to add to a framework; they ask what else can interpret the same semantic application.

The surfaces below are directions, not promises made by the current packages. Some have working
pieces. Others still need a durable contract before they deserve an API.

## Two concrete next moves

Two lines are concrete enough to shape now:

- [**Model-Backed Operations**](02-model-backed-operations.md) keeps one typed operation while the
  runtime may execute it through code, a model, an external system, or a composition.
- [**Selection as an Editable Language**](03-selection-as-an-editable-language.md) treats filters,
  expressions, structural editors, and code as projections of the same Selection AST.

## Wider system directions

The same base opens several system-level lines:

- [**Continuous Execution and First-Class Events**](04-continuous-execution-and-first-class-events.md)
  separates observation streams, operation invocations, and facts that happened.
- [**Semantic Operational Policy**](05-semantic-operational-policy.md) makes authorization,
  rollout, limits, retries, telemetry, and budgets inspectable parts of execution.
- [**A Topology of Graphs**](06-a-topology-of-graphs.md) carries Entities, Refs, Selections, and
  operations across explicit runtime and storage segments.
- [**More Adapters, Same Contracts**](07-more-adapters-same-contracts.md) uses new storage,
  transport, task, and UI technologies to test the portability of the contracts.
- [**Living Entities**](08-living-entities.md) is the farther horizon: safely evolving the model
  itself at runtime.

## What the base makes possible

The ambition is not to become a small framework with many plugins. Ontahí treats domain meaning as
portable, reflected program data. That lets storage engines, transports, workers, UI projections,
tools, policies, and eventually model-backed executors participate in one application without each
reconstructing what the application means.

The current framework is useful before all of these directions exist. Its value is also that they
can be pursued as extensions of one semantic base instead of as disconnected infrastructure
features.
