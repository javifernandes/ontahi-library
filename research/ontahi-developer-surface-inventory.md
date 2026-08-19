# Ontahí Developer Surface Inventory

Status: working inventory
Evidence baseline: Ontahí `0.1.0-alpha.7`, including Plan 131 Relationship Semantics

This inventory decides what the first edition of *Ontahí for Developers* may teach as the normal
way to build an application. It is evidence for the book, not a chapter in the book.

The classifications mean:

- **Canonical**: the main path. Examples should introduce this surface without qualification.
- **Advanced**: a real supported surface that belongs after the main model is understood.
- **Transitional**: compatibility, migration, or low-level construction machinery. Do not teach it
  as the Ontahí form.

## The canonical chain

```text
Application
  composes Entities + Capabilities + Runtimes

Entity
  declares identity + Relations + Operations

Selection
  names a set of an Entity

Query / Command
  observes or changes a Selection

Domain Operation
  adds intention + policy + coordination + effects

Runtime
  interprets the declarations
```

This is the shortest complete account of the current framework.

## Public authoring surface

| Concept | Classification | Canonical form | Evidence | Boundary |
| --- | --- | --- | --- | --- |
| Application | Canonical | `ontahi({ storage, tasks, capabilities, entities })` | `@ontahi/core/runtime/server`, Todo and BookOps composition roots | The single composition root. |
| Entity | Canonical | `entity({ name, fields, locators, identity, relations, uses, operations })` | `@ontahi/core/entity`, every unified BookOps entity | The semantic declaration and operation module are one form. |
| Field/schema | Canonical | `field.*` and `graphSchema.*`; the book imports the latter as `O` | `@ontahi/core/data-graph`, reflected inputs and outputs | Runtime-neutral validation and reflection vocabulary. Statically knowable admissibility—including excluded string values—belongs here, not in executable operation preconditions. |
| Ref | Canonical | Entity locators and their ref factories | core ref tests, client and server input lowering, Todo explicit refs | Identifies one entity; an operation boundary may promote it to a singleton Selection. |
| Relation | Canonical | Reference Fields, `relation.inverse(...)`, and `relation.manyToMany(...)`; read through relation-aware Queries | Todo direct tags relation, relation-root runtime tests, BookOps cyclic refs | Owns topology, cardinality, nullability, navigation, and structural relationship behavior. Associations with attributes or lifecycle remain ordinary Entities. |
| Selection | Canonical | `Entity.selection(predicate)`; `Selection.all`, `none`, `references`; Boolean composition | Todo Node/React filters, core algebra tests, and all graph runtimes | A serializable description of membership, shared by UI filters, reads, and operation targets. |
| Operation entity target | Canonical | `entity.one()` or `entity.many()`; pass a Selection, Ref, identity scalar, identity-bearing record, or—when targeting many—an array of those values | Todo operations, reflection, input normalization, and Explorer tests | The entity-facing API keeps Selection schema machinery out of declarations; the runner receives a Selection and materialized records cross the boundary only by identity. |
| Query | Canonical | `commands.where(...)` or `commands.relatedTo(...)`, then shape with `select`, `include`, `orderBy`, and `limit` | Todo, BookOps, in-memory/Postgres/Supabase conformance | Shapes and executes a read over a selection or declared relation. |
| Command | Canonical | `insert*`, `upsert*`, `selection.update`, `selection.delete` | Todo and graph runtime tests | Performs ubiquitous persistence behavior without inventing a domain endpoint. |
| Relationship Command | Canonical | Entity-bound Ref facades (`student.course.assign(course)`, `course.students.add(student)`) and Selection-level `relationshipSet(...).add/remove` | Core relationship semantics and adapter conformance tests; Todo tag assignment | Preserves structural intent, normalizes inverse directions, accepts Ref- or Selection-valued endpoints, and returns the exact Relationship Delta. |
| Domain operation | Canonical | Declare with `operation(...)`; invoke a bound operation as `Entity.operation(input)` | Todo and BookOps operations | Names intention and owns policy, coordination, effects, or a stable use case. |
| Durable operation | Canonical, second pass | `operation({ durable: {...}, run })`; start with the bound operation and observe its `TaskRunRef` | Todo `completeAll`, task runtime, React lifecycle hook, and workflow tests | Start acceptance, progress snapshots, final output, and failure are separate lifecycle values. Polling is the current portable observation baseline; durability guarantees come from the configured executor and task storage. |
| Capabilities | Canonical need; draft low-level API | `uses.capabilities` plus root `capabilities` | Todo notification Capability, BookOps exercises and notification capabilities | Typed resource injection connects Entities to host implementations, but the resources are opaque and may later become more semantic declarations. Sync, async, and Effect providers normalize once at the host boundary. Dependencies are not yet reflected or checked for completeness at composition time. |
| Cross-entity dependency | Canonical | `uses.entities`, resolved through the application catalog | Todo and unified BookOps entities | Makes semantic dependencies explicit and independent of registration order. |
| Storage | Canonical port | one `storage` supplied to `ontahi()`; in-memory and PostgreSQL bind directly to the registered Entity catalog | Todo storage switch, in-memory/PostgreSQL conformance, BookOps Supabase binding | Ontahí owns execution semantics. Conventional PostgreSQL mapping avoids a second schema declaration; the host owns persistence, migrations, physical exceptions, and transactions. |
| Operation invocation transport | Canonical adapter boundary | mount a composed application through `ontahiExpress(...)` or a provider route handler | Express Todo, Next.js BookOps, shared invocation dispatcher | A generic bridge carries operation identity and opaque input through the common protocol; it does not redefine operations or create one endpoint per intention. |
| Relationship Command transport | Canonical, bounded write boundary | versioned graph command plus explicit `graphCommand.policies` | Express Todo, remote graph-command tests, PostgreSQL and Supabase integration tests | Carries structural relationship changes without exposing storage topology. It is default-deny and does not imply generic remote CRUD. |
| HTTP ingress | Canonical operation metadata; low-level host binding | `ingress.http({ method, route, provider, channel })`, reflected routes, provider registry, and shared dispatcher | BookOps GitHub push and installation webhooks, core ingress router tests | External providers authenticate and normalize requests before dispatch. Provider/router composition, delivery context, deduplication, and resource binding remain in motion. |
| Browser projection | Canonical when a browser exists | codegen output consumed by `@ontahi/react`; author the same Refs and Selections from generated entities | Todo Vite client and BookOps generated entities | Browser-safe projection and client input types come from the application declaration. |
| Reflection / Explorer | Canonical inspection surface | reflect the application and mount Explorer | Express example and BookOps Explorer | Inspection consumes the same application model; it is not a second registry. |

## Advanced surfaces

These are real parts of Ontahí, but they should appear only where their pressure is visible.

| Surface | Why it is advanced |
| --- | --- |
| `entity.ref(name, contract?)` | Solves cyclic or import-sensitive semantic references. Direct entity references remain clearer when cycles are absent. |
| `operationGroup(...)` | Bounds very large operation families for TypeScript and codegen. Ordinary entities should return an operation record directly. |
| Named runtime values and cache identities | Connect operation caching and invalidation to derived identities; they are not entity locators. |
| Graph-direct browser operations | Useful when ubiquitous graph behavior can execute safely in the browser runtime. Relationship Commands now have a bounded remote path; all authority remains server- or data-boundary-owned. |
| Requirements, concerns, layers, effect intents, and telemetry | Cross-cutting runtime composition is supported, but each deserves a focused chapter or reference rather than entering the first entity example. |
| Lower-level runtime constructors | Needed by adapter authors and unusual hosts, not by the normal application author. |
| Provider-specific ingress and durable metadata | Real operation metadata whose meaning depends on the selected host/runtime. |
| Generated application analysis IR | A build-tooling boundary, not the semantic language application developers author directly. |

## Transitional surfaces

| Surface | Canonical replacement or direction |
| --- | --- |
| `entityModule(...)`, `entityModuleWithCapabilities(...)` | Use the unified `entity({...})` declaration. The module APIs remain migration machinery. |
| `relationModule(...)`, `relationModuleWithCapabilities(...)` | Use Relation declarations plus canonical Relationship Commands for structural behavior. Use an Entity for an association with identity or lifecycle, and an Operation for domain coordination. |
| Global `architecture(...)` registration and `getArchitecture()` | Compose once with `ontahi(...)`; consume the returned application and app facade. |
| Manual `defineGraphApi(...)` entity registries | Use `ontahi({ entities })`; reflection, storage, codegen, and execution must observe the same catalog. |
| `defineOntahiApplication(...)` as app authoring | Treat as lower-level compatibility/construction. `ontahi(...)` is the application root. |
| Split `input` plus `inputRefs` operation contracts | The intended model is one semantic input tree containing scalars, values, refs, and selections. Document current ref ergonomics only where unavoidable. |
| Same-name server/browser schema witnesses | Use codegen projections derived from the registered semantic entity. |
| Named CRUD wrappers with no added meaning | Use selection/query/command primitives unless the operation adds policy, invariant, coordination, effects, or stable domain intention. |

## Runtime and adapter inventory

| Implementation | Interprets | Framework guarantee | Host responsibility | First-edition placement |
| --- | --- | --- | --- | --- |
| In-memory core runtime | Graph reads, commands, relations, reflection | Reference behavior without external infrastructure | Seed/lifecycle and acceptance of process-local durability | Main path |
| `@ontahi/postgres` | Graph plans as parameterized SQL | Direct PostgreSQL execution and conformance with the reference runtime | Pool, schema, migrations, mappings, transactions | Primary persistence chapter |
| `@ontahi/supabase` | Graph plans through Supabase/PostgREST; task-run storage | Production adapter behind core contracts | Client creation, mappings, RLS, credentials, request scope | Production adapter appendix until its BookOps seams are fully separated |
| `@ontahi/runtime-express` | Operation, reflection, task, and Explorer HTTP endpoints | Thin transport over the composed application | Express lifecycle, JSON, auth middleware, static assets, logging | Main path |
| `@ontahi/runtime-nextjs` | Next.js actions/routes and invocation transport | Next-specific adaptation of core contracts | App Router composition, request identity, deployment behavior | Adapter chapter |
| Core HTTP ingress runtime | Reflected method/path/provider/channel routes into operation invocations | Accepted/ignored/rejected provider outcomes and dispatch through the canonical operation dispatcher | Raw bodies, signatures, provider decoding, secrets, delivery policy, route mounting | Transport chapter, explicitly low-level |
| `@ontahi/runtime-vercel-workflows` | Durable task execution | Ontahí task lifecycle over Vercel Workflow | Registries, generated entrypoints, stores, deployment | Advanced durable chapter |
| `@ontahi/react` | Browser graph context, hooks, invocation, cache integration | Typed consumption of generated/reflected operations | Query client, host providers, UI behavior | Main browser chapter |
| `@ontahi/explorer-react` | Reflected application and entity data | Reusable inspection and operation UI | Mounting, routes, access control, host theme | Inspection chapter |
| `@ontahi/codegen` | Source declarations into target projections | Deterministic application analysis and browser/runtime artifacts | Build invocation and committed/generated-file policy | Browser/build boundary chapter |
| `@ontahi/opentelemetry` | Core telemetry port | Ontahí span semantics and attributes | SDK, resources, processors, exporter | Advanced operations reference |

## Decisions

### PQL is not current public vocabulary

The repository exposes no `@ontahi/pql` package, `PQL` entrypoint, or singular contract carrying
that name. The current public language is more precise when named by its parts:

- a **Selection** describes membership;
- a **Query** shapes and reads it;
- a **Command** changes it;
- a runtime compiles or interprets those values for its provider.

The first edition will not use PQL as a proper noun. The name can return later if Ontahí defines a
real textual, structural, or package-level language boundary.

### The book teaches one application root

`ontahi(...)` is the only application-composition form in the main narrative. Lower-level graph,
architecture, and application constructors belong to adapter/reference material.

### Bound entities are the ordinary server API

Import entities from the application composition module and call their operations directly:
`TodoList.list()` and `TodoList.rename({ list, name })`. The lower-level application invocation API
remains runtime machinery, not the main authoring form.

When an operation expects a Selection, its public input accepts an existing Selection, an Entity
Ref, an identity scalar, or an identity-bearing materialized record. Many-member inputs accept
arrays of those values. Bound Node entities and generated clients share this normalization: the
runner receives a Selection, and materialized records cross the boundary only by identity. This
removes ceremony without erasing the semantic distinction.

Predicate membership is authored as `Entity.selection(predicate)` from a bound Node entity or its
generated browser projection. The resulting value is the same Selection AST whether it drives a
React filter, a Node read, or an operation target.

### Graph behavior is ubiquitous; domain operations must earn their name

Reading, selecting, inserting, updating, and deleting are graph capabilities. A named domain
operation exists when the name preserves domain intention or adds policy, validation, authority,
coordination, durable execution, effects, or a stable public contract.

## Known gaps: do not write them as settled behavior

1. Generic remote insert, update, upsert, and delete do not yet have the bounded write-policy
   algebra already available to Relationship Commands.
2. Relation predicates and relational free-text search remain incomplete.
3. Capability injection works, but it is a draft low-level resource API and `uses.capabilities`
   remains a typed witness. Dependency reflection and composition-time completeness checks do not
   yet exist; recurring concepts risk remaining opaque if arbitrary injection becomes the final
   model.
4. `TaskRun` remains application-owned in BookOps instead of being supplied by the durable runtime.
5. Named and saved selections, selection-language editing, and durable membership snapshots remain
   later concepts.
6. Authorization and relationship policies are not yet one settled Ontahí model.
7. Entity-level uniqueness is not yet a declarative invariant interpreted consistently by storage,
   operation failures, reflection, and generated clients. A query-based precondition cannot supply
   the required atomicity on its own.
8. HTTP ingress metadata is reflected and executable, but provider/resource binding, raw-request
   adaptation, delivery context, and deduplication are still host-level. The current Todo ingress
   declaration is discoverable but is not automatically mounted by `ontahiExpress(...)`.

## Chapter contract

Every chapter in the first edition must:

1. introduce one distinction through executable code;
2. use canonical APIs in its primary example;
3. state the framework/adapter/host boundary when one appears;
4. link to or derive from tested source;
5. omit extraction history and comparisons with other frameworks;
6. move unsettled behavior into a clearly marked limit, never a speculative promise.
