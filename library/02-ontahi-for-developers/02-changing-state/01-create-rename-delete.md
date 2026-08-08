# Create, Rename, Delete

Domain operations name the ways an entity may change.

```ts
operations: ({ self, commands, operation }) => ({
  create: operation({
    input: graphSchema.pick(self, ['id', 'name']).named('CreateTodoListInput'),
    output: self,
    bridge: { invalidate: [['TodoList']] },
    run: input => commands.insertReturning(input, ['id', 'name']),
  }),

  rename: operation({
    input: graphSchema.object({
      list: graphSchema.selection(self, { cardinality: 'one' }),
      name: field.nonEmptyString({ trim: true }),
    }),
    output: self,
    bridge: { invalidate: [['TodoList']] },
    run: ({ list, name }) =>
      commands.where(list).updateOneReturning({ name }, ['id', 'name']),
  }),

  delete: operation({
    input: graphSchema.object({
      list: graphSchema.selection(self, { cardinality: 'one' }),
    }),
    bridge: { invalidate: [['TodoList']] },
    run: ({ list }) => commands.where(list).deleteOne(),
  }),
}),
```

The input schema is the public contract. The command is the storage-neutral effect. `create`,
`rename`, and `delete` are domain vocabulary available to every host.

## Call them from React

```ts
const createList = useOperation(TodoList.domain.create);
const renameList = useOperation(TodoList.domain.rename);
const deleteList = useOperation(TodoList.domain.delete);

const id = crypto.randomUUID();

await createList.executeAsync({ id, name: 'Research backlog' });
await renameList.executeAsync({
  list: TodoList.refById(id),
  name: 'Research queue',
});
await deleteList.executeAsync({
  list: TodoList.refById(id),
});
```

The component does not construct URLs or duplicate input and output types. The operation projection
already carries them.

`bridge.invalidate` declares that a successful mutation makes TodoList queries stale. The React
runtime uses that declaration to refresh `useOperationQuery(TodoList.domain.list)`. The cache model
is important, but it is not required to use the operation; it gets its own advanced chapter.
