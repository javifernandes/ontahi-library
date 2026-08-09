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

## Read through the relation

Give Todo access to TodoList and add a read for one list:

```ts
uses: {
  entities: () => ({ TodoList }),
},
operations: ({ self, commands, operation, entities }) => ({
  listForList: operation({
    input: O.object({
      list: O.selection(TodoList, { cardinality: 'one' }),
    }),
    output: O.array(self),
    exposure: 'bridge',
    bridge: { query: [(input: unknown) => input] },
    run: ({ list }) =>
      commands
        .relatedTo(entities.TodoList.where(list), { through: 'list' })
        .orderBy(todo => todo.title),
  }),
}),
```

`entities.TodoList.where(list)` is the selected source. `relatedTo(..., { through: 'list' })`
navigates the declared `Todo.list` relation in reverse and returns only its Todos.

From Node, pass the TodoList Ref directly:

```ts
import { Todo, TodoList } from './graph.js';

const list = TodoList.refById('list-research');
const result = await Todo.listForList({ list });

if (!result.ok) throw new Error(`Todo.listForList failed: ${result.kind}`);

for (const todo of result.value) {
  console.log(todo.title);
}
```

The React projection uses the same Ref and returns the same records:

```tsx
import { useOperationQuery } from '@ontahi/react/graph';
import { Todo, TodoList } from './generated/client-entities.js';

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

The component does not fetch all Todos and filter them locally. The relation shapes the read before
the selected runtime executes it.

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
