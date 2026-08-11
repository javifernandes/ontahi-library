# Browser Client and Projection

A \concept{Client Projection} preserves the public, executable portion of the application model
for another environment. The browser does not import the server application. It receives a projection of the
same model: browser-safe Entity schemas, identity and locator rules, relations, and the contracts
of operations exposed through a bridge.

Operation bodies, Capability implementations, storage, task executors, secrets, and server-only
operations stay on the server.

## Generate the browser surface

The host projects its application at build time:

```js
import {
  analyzeOntahiApplication,
  createFileSystemSourceLoader,
  renderGeneratedClientEntityModule,
} from '@ontahi/codegen';

const application = analyzeOntahiApplication({
  graphApiPath: './src/graph.ts',
  sourceLoader: createFileSystemSourceLoader({ rootDir: process.cwd() }),
});

if (application.diagnostics.length > 0) {
  throw new Error(JSON.stringify(application.diagnostics, null, 2));
}

const source = renderGeneratedClientEntityModule({
  entities: application.clientEntities,
});
```

`application.clientEntities` is intentionally smaller than the analyzed application. The
generated module carries enough meaning for code such as this to remain typed and semantic in the
browser:

```ts
TodoItem.refById('todo-write-outline');
TodoItem.selection(todo => todo.completed.eq(false));
TodoItem.domain.list.input;
TodoItem.domain.complete.output;
```

The generated file is deterministic and checked for drift; application code imports it but does
not edit it. Output paths, formatting, import aliases, and build integration remain host choices.

> [!MARGIN] **Projection is not a shared server bundle.** A conventional shared-types package may
> reproduce request and response shapes while losing identity, Selection semantics, and operation
> metadata. Ontahí projects those links from the application declaration and omits the executable
> server implementation.

## Compose the browser runtime once

The React host supplies the available execution paths at its root:

```tsx
const bridge = createFetchOperationBridgeAdapter({
  mountPath: '/runtime/ontahi',
});

<OntahiGraphProvider
  runtime={browserRuntime}
  graphExecutor={browserGraphExecutor}
  operationBridgeAdapters={[bridge]}
>
  <App />
</OntahiGraphProvider>
```

`graphExecutor` interprets browser-authorized Data Graph Queries and Commands directly. The bridge
carries domain-operation intentions to the server. An application may configure either path or
both; components consume the projected Entity instead of choosing an HTTP endpoint or database
client on every call.

## A Selection becomes an observation

The UI can author the same membership value used from Node:

```tsx
const openTodos = useMemo(
  () => TodoItem.selection(todo => todo.completed.eq(false)),
  [],
);

const todos = useOperationQuery(TodoItem.domain.list, openTodos);
const complete = useOperation(TodoItem.domain.complete);
```

`useOperationQuery` does three model-aware things:

1. it normalizes the Selection against the operation input contract;
2. it derives a stable observation key from the operation and that input;
3. it uses the operation output contract to place returned Entities in the graph cache.

For `TodoItem.list(openTodos)`, the declared bridge metadata produces an observation shaped like:

```ts
['TodoItem', 'list', openTodos]
```

That key identifies one observed result set. It is not the identity of any TodoItem inside it. Another
Selection produces another observation even when the two sets overlap.

## Cache snapshots by Entity identity

Suppose two operation results contain the same TodoItem:

```text
TodoItem.list(open)       -> [TodoItem A, TodoItem B]
TodoItem.list(inResearch) -> [TodoItem B, TodoItem C]
```

The client cache does not preserve four unrelated object copies. The declared output shape tells
it that these values are Entities. It replaces each snapshot in the stored result with its
canonical Ref and stores one current record per Entity identity:

```text
open       -> [Ref A, Ref B]
inResearch -> [Ref B, Ref C]

Ref B -> current TodoItem B snapshot
```

When React reads either observation, Ontahí resolves those Refs back into materialized values. An
update learned through one operation can therefore be visible anywhere that the same identity is
used. Declared alternate locators can resolve to that same canonical record.

This cache is why the projected output contract matters at runtime. An opaque JSON result can be
cached as a result; an Entity result can participate in graph identity.

> [!MARGIN] **Identity is semantic, not JavaScript reference equality.** Components receive
> materialized snapshots suitable for ordinary rendering. Ontahí unifies them because they refer
> to the same Entity, not because every read returns the same object instance.

## Reconcile results, then invalidate observations

Mutation hooks use the same metadata. Consider the two declarations already used by the TodoItem UI:

```ts
list: operation({
  input: self.many(),
  output: self.array(),
  bridge: { query: [(todos: unknown) => todos] },
  run: todos => todos.orderBy(todo => todo.title),
}),

complete: operation({
  input: O.object({
    todos: self.many(),
  }),
  bridge: { invalidate: [['TodoItem']] },
  run: ({ todos }) => todos.update({ completed: true }),
}),
```

After a successful operation, Ontahí first reconciles any returned Entity snapshots into the
identity cache. It then invalidates observations declared by the operation. `['TodoItem']` is a prefix,
so the current `complete` contract marks every TodoItem observation stale, including both lists above.

```tsx
await complete.executeAsync({ todos: openTodos });
```

The component does not know which transport ran the operation, manufacture cache keys, or copy the
result into each filtered list. The operation declares the semantic area it may have changed; the
browser runtime handles the current cache implementation.

Entity-wide invalidation is deliberately coarse. Ontahí also has graph-aware machinery for
invalidating observations that contain specific Refs, but the exact `clientCache` declaration
surface is advanced and still moving. Prefer the operation's stable query and invalidation
metadata until a narrower contract is justified.

> [!MARGIN] **TanStack Query is the current React observer.** It schedules requests, exposes loading
> state, and refetches invalidated observations. Query keys and normalized Entity records are
> derived from Ontahí contracts; they are not intended to become a second domain model authored in
> components.

## Durable completion is the invalidation boundary

A durable invocation succeeds when the runtime accepts a run. Its data changes may happen later in
a worker, so invalidating at acceptance could refetch the old state.

`useDurableOperation` observes the accepted `TaskRunRef`. It invalidates the operation's declared
observations only after the snapshot reaches `completed`; `failed` and `cancelled` runs do not
pretend that the intended change happened.

The invariant across ordinary and durable operations is the same: the projected contract tells
the browser what was invoked, what kind of value came back, and which observations may now be
stale. Codegen carries that meaning across the process boundary; the client runtime interprets it.
