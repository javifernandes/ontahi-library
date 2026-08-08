# Relations and Topology

A Relation names how entities are connected.

```ts
export const Todo = entity({
  name: 'Todo',
  fields: {
    id: field.id(),
    listId: field.id(),
    title: field.nonEmptyString({ trim: true }),
    completed: field.boolean(),
  },
  locators: { refById: 'id' },
  identity: 'refById',
  relations: {
    list: relation.belongsTo(TodoList, { via: 'listId' }),
    tagAssignments: relation.hasMany(TodoTag, { via: 'todoId' }),
  },
});
```

`Todo.list` says that each Todo belongs to a TodoList through `listId`.
`Todo.tagAssignments` says that TodoTag rows refer back to a Todo through `todoId`.

The names express domain topology. `via` supplies storage evidence without placing table or column
names in the entity.

## Associations stay visible

Todo and Tag have a many-to-many association. Ontahí models its identity explicitly:

```text
Todo -> TodoTag -> Tag
```

```ts
export const TodoTag = entity({
  name: 'TodoTag',
  fields: {
    todoId: field.id(),
    tagId: field.id(),
  },
  locators: {
    refByTodoAndTag: ['todoId', 'tagId'],
  },
  identity: 'refByTodoAndTag',
  relations: {
    tag: relation.belongsTo(Tag, { via: 'tagId' }),
  },
});
```

The association is a semantic entity because it has identity and can receive behavior. It is not a
hidden storage detail.

## Cyclic declarations

When two declarations cannot import each other directly, use a semantic reference:

```ts
const PendingInvite = entity.ref('PendingInvite');

const Book = entity({
  name: 'Book',
  fields: bookFields,
  relations: {
    pendingInvites: relation.hasMany(PendingInvite, { via: 'bookId' }),
  },
});
```

`ontahi()` resolves the reference against the complete entity catalog before binding storage or
operations. Prefer direct declarations when there is no cycle.

Relations currently define topology, navigation, reflection, and mapping evidence. Relation-owned
identity and behavior beyond an explicit association entity remain outside the canonical model.
