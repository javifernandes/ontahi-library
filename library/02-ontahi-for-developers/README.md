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
8. Durable Operations

### Part III — Carrying the Model

9. Capabilities and Runtime Composition
10. Storage
11. Transport

### Part IV — Projecting the Application

12. React Runtime: Cache and Invalidation
13. Reflection and Explorer
14. The Host Boundary

The sequence is provisional until the canonical-surface inventory has been exercised through the
first three chapters. The distinctions are not provisional: Application, Entity, Ref, Relation,
Selection, Query, Command, Domain Operation, Capability, and Runtime each carry a different job.

## Writing posture

Lead with executable form. Use the application first from ordinary Node code. Introduce the React
projection only when the browser boundary creates pressure for it. Explain an abstraction when the
code creates pressure for its name. Prefer one complete example over a catalog of disconnected
features.

React support stays transparent on the main path. Cache behavior and invalidation mechanics belong
to the advanced projection section. The main path uses only canonical APIs; transitional APIs do
not become doctrine by appearing in a book.
