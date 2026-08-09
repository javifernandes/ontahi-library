# Ontahí for Developers

Model the domain once. Let runtimes carry it.

This book teaches one way to build an Ontahí application: the way expressed by the framework's
current semantic declarations, executable examples, and runtime contracts.

It does not begin with the history of Ontahí. It does not translate every concept into the
vocabulary of another technology. It starts with the application and follows its model wherever
that model is interpreted.

## Shape

### Part I — The Ontahí Form

1. [One Application](01-the-ontahi-form/01-one-application.md)
2. [An Entity at Work](01-the-ontahi-form/02-entity-and-identity.md)
3. [Identity and Refs](01-the-ontahi-form/03-identity-and-refs.md)
4. [Relations and Topology](01-the-ontahi-form/04-relations-and-topology.md)

### Part II — Changing State

5. [Create, Rename, Delete](02-changing-state/01-create-rename-delete.md)
6. [Selection: One Set, Many Uses](02-changing-state/02-selection-one-set-many-uses.md)
7. [Queries and Commands](02-changing-state/03-queries-and-commands.md)
8. [Operation Contracts and Failures](02-changing-state/04-operation-contracts-and-failures.md)
9. [Durable Operations](02-changing-state/05-durable-operations.md)

### Part III — Carrying the Model

10. [Capabilities and Runtime Composition](03-carrying-the-model/01-capabilities-and-runtime-composition.md)
11. [Storage Adapters](03-carrying-the-model/02-storage-adapters.md)
12. [Transport, Bridges, and HTTP Ingress](03-carrying-the-model/03-transport-bridges-and-http-ingress.md)

### Part IV — Projecting the Application

13. [Browser Projection Internals](04-projecting-the-application/01-browser-projection-internals.md)
14. [Reflection, Explorer, and Host Responsibilities](04-projecting-the-application/02-reflection-explorer-and-host-responsibilities.md)

The sequence follows the canonical-surface inventory. Its distinctions are deliberate:
Application, Entity, Ref, Relation, Selection, Query, Command, Domain Operation, Capability,
Ingress, and Runtime each carry a different job.

## Executable spine

The code builds progressively: a fence may omit imports or defaults introduced earlier, but its
public identifiers and call shapes match the current packages unless the text labels a surface as
draft or transitional. The complete
[`todo-express` application](https://github.com/javifernandes/bookops/tree/main/ontahi/examples/todo-express)
checks the canonical path through code generation, server and browser typechecking, and integration
tests. Its server declarations live in
[`src/todo.ts`](https://github.com/javifernandes/bookops/blob/main/ontahi/examples/todo-express/src/todo.ts),
its React use in
[`client/src/App.tsx`](https://github.com/javifernandes/bookops/blob/main/ontahi/examples/todo-express/client/src/App.tsx),
and its end-to-end assertions in
[`test/application.test.ts`](https://github.com/javifernandes/bookops/blob/main/ontahi/examples/todo-express/test/application.test.ts).

## Writing posture

Lead with executable form. Use the application first from ordinary Node code. Introduce the React
projection only when the browser boundary creates pressure for it. Explain an abstraction when the
code creates pressure for its name. Prefer one complete example over a catalog of disconnected
features.

Keep the main narrative in Ontahí's own vocabulary. A short margin note may contrast a familiar
transport or framework pattern when that contrast makes the pressure behind an Ontahí abstraction
concrete; it must not turn into a second tutorial.

React support stays transparent on the main path. Cache behavior and invalidation mechanics belong
to the advanced projection section. The main path uses canonical APIs. When no semantic
replacement exists, a required draft surface is labeled explicitly rather than becoming doctrine
by appearing in a book.

Use diagrams sparingly where runtime distribution, execution paths, or lifecycle boundaries are
harder to see in prose. They clarify the model; they do not decorate every chapter.
