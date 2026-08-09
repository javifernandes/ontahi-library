# Create, Rename, Delete

Domain operations name the ways an entity may change.

```ts
operations: ({ self, commands, operation }) => ({
  create: operation({
    input: O.pick(self, ['id', 'name']).named('CreateTodoListInput'),
    output: self,
    run: input => commands.insertReturning(input, ['id', 'name']),
  }),

  rename: operation({
    input: O.object({
      list: O.selection(self, { cardinality: 'one' }),
      name: field.nonEmptyString({ trim: true }),
    }),
    output: self,
    run: ({ list, name }) =>
      commands.where(list).updateOneReturning({ name }, ['id', 'name']),
  }),

  delete: operation({
    input: O.object({
      list: O.selection(self, { cardinality: 'one' }),
    }),
    run: ({ list }) => commands.where(list).deleteOne(),
  }),
}),
```

The input schema is the public contract. The command is the storage-neutral effect. `create`,
`rename`, and `delete` are domain vocabulary available to every host.

## Use them from Node

```ts
import { TodoList } from './graph.js';

const created = await TodoList.create({
  id: crypto.randomUUID(),
  name: 'Research backlog',
});
if (!created.ok) throw new Error(`TodoList.create failed: ${created.kind}`);

const list = TodoList.refById(created.value.id);

const renamed = await TodoList.rename({
  list,
  name: 'Research queue',
});
if (!renamed.ok) throw new Error(`TodoList.rename failed: ${renamed.kind}`);

const deleted = await TodoList.delete({ list });
if (!deleted.ok) throw new Error(`TodoList.delete failed: ${deleted.kind}`);
```

The caller uses the entity, its locator, and its operations. It does not construct transport URLs,
write a storage query, or manually turn the Ref into a Selection.

The React projection comes later, once the browser boundary creates a reason to introduce hooks,
cache identity, and invalidation.
