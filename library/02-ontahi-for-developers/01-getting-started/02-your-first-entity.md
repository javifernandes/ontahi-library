# Your First Entity

An \concept{Entity} declares a named kind of thing and the operations that belong to it.

```ts
import { field as f } from '@ontahi/core/data-graph';
import { entity } from '@ontahi/core/entity';

export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: f.id(),
    name: f.nonEmptyString({ trim: true }),
  },
  operations: ({ self, commands, operation }) => ({
    list: operation({
      output: self.array(),
      run: () => commands.all().orderBy(list => list.name),
    }),
  }),
});
```

This guide imports the Field factory as `f`. The short alias keeps Entity declarations compact;
the model still reflects each value as a Field.

Entity-shaped contracts stay on the Entity: `self` for one materialized value, `self.array()` for
many, and—as the next chapters show—`self.one()` or `self.many()` for semantic targets. Later
examples import `graphSchema` as `O` only for standalone schema composition.

Fields describe its values. `list` is a typed operation whose implementation is a graph query.
Part II opens this declaration and explains Entity identity, locators, relations, and operations
individually.

## Use it from Node

Import the entity from the module that composes the application. At that point it is bound to the
selected runtime and its operations can be called directly:

```ts
import { TodoList } from './graph.js';

const result = await TodoList.list();

if (!result.ok) throw new Error(`TodoList.list failed: ${result.kind}`);

for (const list of result.value) {
  console.log(`${list.name} (${list.id})`);
}
```

There is no HTTP request, React component, or storage-specific query here. `TodoList.list()` runs
through the application runtime selected at composition.

## Project the same operation to React

Operations are server-only by default. To make `list` available to a generated browser client, opt
that operation into the bridge:

```ts
list: operation({
  exposure: 'bridge',
  output: self.array(),
  run: () => commands.all().orderBy(list => list.name),
}),
```

Because `list` has no input, its operation identity is already a complete observation key. More
specific query metadata is only needed when an operation's input creates distinct observations.

## Generate the client projection

A Node-only application does not need codegen. Add it when code in another environment—such as a
browser—needs to use the application model:

```sh
pnpm add --save-exact @ontahi/react@alpha
pnpm add @tanstack/react-query
pnpm add --save-exact --save-dev @ontahi/codegen@alpha
```

For the conventional `src/graph.ts` composition root, no generation script is necessary:

```json
{
  "scripts": {
    "codegen": "ontahi-codegen",
    "codegen:check": "ontahi-codegen --check"
  }
}
```

```sh
pnpm codegen
```

The command generates `src/generated/client-entities.ts`. It analyzes the same application but
projects only browser-safe schemas and bridge-exposed operation contracts. Implementations,
Capabilities, storage, secrets, and server-only operations do not cross that boundary.

> [!MARGIN] **More than a transport specification.** The pressure resembles generating an OpenAPI
> document from an HTTP API, but the result is not merely a route and payload description. Ontahí
> emits an executable semantic projection that preserves Entity identity, relations, Selections,
> and operation contracts independently of the transport carrying them.

React uses that generated projection directly:

```tsx
import { useOperationQuery } from '@ontahi/react/graph';
import { TodoList } from './generated/client-entities.js';

export const TodoLists = () => {
  const lists = useOperationQuery(TodoList.domain.list);

  if (lists.isLoading) return <p>Loading lists…</p>;
  if (lists.isError) return <p>Could not load lists.</p>;

  return (
    <ul>
      {lists.data?.map(list => <li key={list.id}>{list.name}</li>)}
    </ul>
  );
};
```

The hook adds browser lifecycle state; it does not redefine the entity or operation. Query-cache
mechanics remain outside this first use.

## One semantic declaration

The entity is the shared source for identity, graph behavior, operations, reflection, storage
adapters, and generated projections. Those surfaces interpret one declaration; they are not
parallel schemas.
