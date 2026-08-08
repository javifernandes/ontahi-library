# An Entity at Work

An Entity declares a named kind of thing and the operations that belong to it.

```ts
export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: field.id(),
    name: field.nonEmptyString({ trim: true }),
  },
  locators: { refById: 'id' },
  identity: 'refById',
  domainOperationDefaults: {
    authority: 'server',
    exposure: 'bridge',
    layer: 'todos',
  },
  operations: ({ self, commands, operation }) => ({
    list: operation({
      output: graphSchema.array(self),
      bridge: { query: [() => 'all'] },
      run: () => commands.all().orderBy(list => list.name),
    }),
  }),
});
```

Fields describe its values. Locators and identity will matter in the next chapter. For now, the
important addition is `list`: a typed operation whose implementation is a graph query.

## Use it from Node

`ontahi()` binds the declaration to the application's storage and runtime. Ordinary Node code can
invoke the resulting operation directly:

```ts
import { TodoApplication } from './graph.js';

const lists = TodoApplication.graph.entities.TodoList;
const result = await TodoApplication.invokeOperation(lists.domain.list, undefined);

if (!result.ok) throw new Error(`TodoList.list failed: ${result.kind}`);

for (const list of result.value) {
  console.log(`${list.name} (${list.id})`);
}
```

There is no HTTP request, React component, or storage-specific query here. The application executes
the operation through the runtime selected at composition.

## Use the same operation from React

Codegen projects the browser-safe side of the same entity. The React hook consumes that projection:

```tsx
const lists = useOperationQuery(TodoList.domain.list);

if (lists.isLoading) return <p>Loading lists…</p>;
if (!lists.data) return null;

return (
  <ul>
    {lists.data.map(list => (
      <li key={list.id}>{list.name}</li>
    ))}
  </ul>
);
```

React is one caller of the Ontahí application. It receives typed data and lifecycle state; the
entity and operation remain independent of React. Cache mechanics come later.

## What the declaration owns

An entity may also declare:

- display and freshness metadata;
- refs and relations;
- application dependencies through `uses`;
- graph and domain operations.

These are facets of one semantic entity, not parallel schemas for Node, React, storage, reflection,
or Explorer.
