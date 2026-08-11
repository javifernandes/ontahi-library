# An Entity at Work

An Entity declares a named kind of thing and the operations that belong to it.

```ts
export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: field.id(),
    name: field.nonEmptyString({ trim: true }),
  },
  operations: ({ self, commands, operation }) => ({
    list: operation({
      output: self.array(),
      run: () => commands.all().orderBy(list => list.name),
    }),
  }),
});
```

Entity-shaped contracts stay on the Entity: `self` for one materialized value, `self.array()` for
many, and—as the next chapters show—`self.one()` or `self.many()` for semantic targets. Later
examples import `graphSchema` as `O` only for standalone schema composition.

Fields describe its values. The exact, required `id: field.id()` field also declares the
conventional identity: Ontahí automatically gives TodoList a `refById` locator. For now, the
important addition is `list`: a typed operation whose implementation is a graph query.

> **Identity starts at the field.** The common case needs no parallel locator declaration. An
> Entity with `id: field.id()` can be addressed immediately with `TodoList.refById('list-research')`.
> Other id-shaped fields such as `ownerId` do not become identities by accident.

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

When the domain needs more than the convention, make only that difference explicit. An alternate
locator composes with `refById`:

```ts
export const Book = entity({
  name: 'Book',
  fields: {
    id: field.id(),
    slug: field.nonEmptyString({ trim: true }),
    title: field.nonEmptyString({ trim: true }),
  },
  locators: { refBySlug: 'slug' },
});
```

Book now has both `Book.refById(...)` and `Book.refBySlug(...)`; `refById` remains its default
identity. Composite identities or a different default identity are declared explicitly, where the
extra structure carries domain meaning.
