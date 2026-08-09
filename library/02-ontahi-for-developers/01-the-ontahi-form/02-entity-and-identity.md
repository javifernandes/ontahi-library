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

The browser will later receive a safe projection of this same operation. The entity itself is not
a React abstraction.

## One semantic declaration

The entity is the shared source for identity, graph behavior, operations, reflection, storage
adapters, and generated projections. Those surfaces interpret one declaration; they are not
parallel schemas.
