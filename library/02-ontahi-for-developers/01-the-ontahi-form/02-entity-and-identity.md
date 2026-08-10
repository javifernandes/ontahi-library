# An Entity at Work

An Entity declares a named kind of thing and the operations that belong to it. The book imports
`graphSchema` as `O` to keep schema expressions compact.

```ts
import { graphSchema as O } from '@ontahi/core/data-graph';

export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: field.id(),
    name: field.nonEmptyString({ trim: true }),
  },
  locators: { refById: 'id' },
  identity: 'refById',
  operations: ({ self, commands, operation }) => ({
    list: operation({
      output: O.array(self),
      run: () => commands.all().orderBy(list => list.name),
    }),
  }),
});
```

Fields describe its values. Locators and identity will matter in the next chapter. For now, the
important addition is `list`: a typed operation whose implementation is a graph query.

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
that operation into the bridge and give repeated reads one identity:

```ts
list: operation({
  exposure: 'bridge',
  output: O.array(self),
  bridge: { query: [() => 'all'] },
  run: () => commands.all().orderBy(list => list.name),
}),
```

Codegen emits the browser-safe entity. React uses that projection directly:

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
