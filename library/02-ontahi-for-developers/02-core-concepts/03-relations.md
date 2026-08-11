# Relations

A \concept{Relation} names how Entities are connected. The most common relation is declared where
the connection actually lives: in the referencing field.

```ts
export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: f.id(),
    name: f.nonEmptyString({ trim: true }),
  },
});

export const TodoItem = entity({
  name: 'TodoItem',
  fields: {
    id: f.id(),
    list: f.ref(TodoList),
    title: f.nonEmptyString({ trim: true }),
    completed: f.boolean(),
  },
});
```

`TodoItem.list` is both a field whose value is a TodoList \concept{Ref} and the `belongsTo` relation
from TodoItem to TodoList. There is no separate `listId` field and no second declaration that restores
its lost meaning.

The physical representation belongs to storage. A relational adapter may conventionally store
`TodoItem.list` in `todo_items.list_id`; the Entity, its operations, and its clients continue to speak in
Refs.

## Use the reference without loading the relation

```ts
const research = TodoList.refById('list-research');

await TodoItem.insert({
  id: 'todo-map-runtime',
  list: research,
  title: 'Map the runtime',
  completed: false,
}).run();

const todos = await TodoItem.where(todo => todo.list.eq(research)).run();
```

The `list` value identifies one TodoList. Neither statement requires its current fields. Ontahí
lowers the Ref only when the chosen runtime reaches its storage boundary.

A Ref is also a valid membership criterion for its own Entity:

```ts
const lists = await TodoList.where(research).run();
```

## Materialize the relation when the result needs it

Without an include, each TodoItem contains a Ref at `list`. An include materializes the target at that
same path:

```ts
const todosWithLists = await TodoItem
  .all()
  .include(todo => ({
    list: todo.list.select(list => ({
      id: list.id,
      name: list.name,
    })),
  }))
  .run();
```

For a required `f.ref`, `list` becomes the selected TodoList value. A
`f.nullable(f.ref(TodoList))` produces `TodoList | null` after inclusion.

## Navigate through the relation

An operation can traverse the same declared edge in the opposite direction:

```ts
operations: ({ self, commands, operation }) => ({
  itemsForList: operation({
    input: O.object({
      list: TodoList.one(),
    }),
    output: self.array(),
    exposure: 'bridge',
    bridge: { query: [(input: unknown) => input] },
    run: ({ list }) => commands.relatedTo(list).orderBy(todo => todo.title),
  }),
}),
```

`list` already carries the source Entity and its selection criterion. Because `TodoItem.list` is
the only relation connecting TodoItem and TodoList, `relatedTo(list)` can infer the traversal. The
operation does not reconstruct a query or repeat the relation name.

If two relations connect the same pair of Entities, the edge is genuinely ambiguous. Name it only
then:

```ts
commands.relatedTo(user, { through: 'createdBy' });
```

From Node, the operation accepts a Ref directly:

```ts
const result = await TodoItem.itemsForList({
  list: TodoList.refById('list-research'),
});

if (!result.ok) throw new Error(`TodoItem.itemsForList failed: ${result.kind}`);

for (const todo of result.value) console.log(todo.title);
```

The browser projection preserves the same input:

```tsx
export const ItemsForList = ({ listId }: { listId: string }) => {
  const todos = useOperationQuery(TodoItem.domain.itemsForList, {
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
    todo: f.ref(TodoItem),
    tag: f.ref(Tag),
  },
});

// In TodoItem's declaration
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
const TodoItemRef = entity.ref('TodoItem', { fields: todoItemFields });

export const TodoTag = entity({
  name: 'TodoTag',
  fields: {
    todo: f.ref(TodoItemRef),
    tag: f.ref(Tag),
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
  legacyListId: f.id(),
},
relations: {
  list: relation.belongsTo(TodoList, { via: 'legacyListId' }),
},
```

Use `f.ref` for the ordinary semantic case. Use explicit `belongsTo` and `hasMany` when the
exception itself is information the model needs to preserve.
