# Relations

A \concept{Relation} names how Entities are connected. The most common relation is declared where
the connection actually lives: in the referencing field.

```ts
export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: field.id(),
    name: field.nonEmptyString({ trim: true }),
  },
});

export const Todo = entity({
  name: 'Todo',
  fields: {
    id: field.id(),
    list: field.ref(TodoList),
    title: field.nonEmptyString({ trim: true }),
    completed: field.boolean(),
  },
});
```

`Todo.list` is both a field whose value is a TodoList \concept{Ref} and the `belongsTo` relation
from Todo to TodoList. There is no separate `listId` field and no second declaration that restores
its lost meaning.

The physical representation belongs to storage. A relational adapter may conventionally store
`Todo.list` in `todos.list_id`; the Entity, its operations, and its clients continue to speak in
Refs.

## Use the reference without loading the relation

```ts
const research = TodoList.refById('list-research');

await Todo.insert({
  id: 'todo-map-runtime',
  list: research,
  title: 'Map the runtime',
  completed: false,
}).run();

const todos = await Todo.where(todo => todo.list.eq(research)).run();
```

The `list` value identifies one TodoList. Neither statement requires its current fields. Ontahí
lowers the Ref only when the chosen runtime reaches its storage boundary.

A Ref is also a valid membership criterion for its own Entity:

```ts
const lists = await TodoList.where(research).run();
```

## Materialize the relation when the result needs it

Without an include, each Todo contains a Ref at `list`. An include materializes the target at that
same path:

```ts
const todosWithLists = await Todo
  .all()
  .include(todo => ({
    list: todo.list.select(list => ({
      id: list.id,
      name: list.name,
    })),
  }))
  .run();
```

For a required `field.ref`, `list` becomes the selected TodoList value. A
`field.nullable(field.ref(TodoList))` produces `TodoList | null` after inclusion.

## Navigate through the relation

An operation can traverse the same declared edge in the opposite direction:

```ts
uses: {
  entities: () => ({ TodoList }),
},
operations: ({ self, commands, operation, entities }) => ({
  listForList: operation({
    input: O.object({
      list: TodoList.one(),
    }),
    output: self.array(),
    exposure: 'bridge',
    bridge: { query: [(input: unknown) => input] },
    run: ({ list }) =>
      commands
        .relatedTo(entities.TodoList.where(list), { through: 'list' })
        .orderBy(todo => todo.title),
  }),
}),
```

From Node, the operation accepts a Ref directly:

```ts
const result = await Todo.listForList({
  list: TodoList.refById('list-research'),
});

if (!result.ok) throw new Error(`Todo.listForList failed: ${result.kind}`);

for (const todo of result.value) console.log(todo.title);
```

The browser projection preserves the same input:

```tsx
export const TodosForList = ({ listId }: { listId: string }) => {
  const todos = useOperationQuery(Todo.domain.listForList, {
    list: TodoList.refById(listId),
  });

  return (
    <ul>
      {todos.data?.map(todo => <li key={todo.id}>{todo.title}</li>)}
    </ul>
  );
};
```

## Declare the inverse collection

A plural inverse is not stored as an Entity field. It points back through the field that owns the
references:

```ts
export const TodoTag = entity({
  name: 'TodoTag',
  fields: {
    todo: field.ref(Todo),
    tag: field.ref(Tag),
  },
});

// In Todo's declaration
relations: () => ({
  tagAssignments: relation.inverse(TodoTag.fields.todo),
}),
```

`TodoTag.fields.todo` already supplies the related Entity, the reference value, and the join
evidence. The inverse declaration adds only the domain name and `hasMany` cardinality.

An association remains an Entity when it has identity, fields, or behavior of its own. Ontahí does
not hide it inside an implicit many-to-many mechanism.

## Break declaration cycles explicitly

When a reference field must target an Entity declared later, give Ontahí its semantic contract:

```ts
const TodoRef = entity.ref('Todo', { fields: todoFields });

export const TodoTag = entity({
  name: 'TodoTag',
  fields: {
    todo: field.ref(TodoRef),
    tag: field.ref(Tag),
  },
});
```

`ontahi()` resolves deferred references against the complete Entity catalog before storage or
operations are bound. Prefer the direct Entity when declaration order already permits it.

## Keep explicit relations for exceptional mappings

The lower-level declarations remain available for legacy scalar fields, composite physical keys,
and mappings that cannot be derived from a reference field:

```ts
fields: {
  legacyListId: field.id(),
},
relations: {
  list: relation.belongsTo(TodoList, { via: 'legacyListId' }),
},
```

Use `field.ref` for the ordinary semantic case. Use explicit `belongsTo` and `hasMany` when the
exception itself is information the model needs to preserve.
