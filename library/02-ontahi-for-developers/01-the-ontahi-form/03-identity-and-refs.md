# Identity and Refs

A Ref identifies an entity without loading it.

```ts
const lists = TodoApplication.graph.entities.TodoList;
const list = lists.refById(listId);
```

`listId` can come from a route parameter, a job payload, or a previous operation result. The ref
contains only semantic identity:

```ts
{
  kind: 'entity-ref',
  entityName: 'TodoList',
  locator: { id: listId },
}
```

It does not claim that the list exists or contain its current fields.

## Act on the ref from Node

An operation that accepts one TodoList declares that target as a Selection:

```ts
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
```

Ordinary Node code turns the ref into a one-member Selection and invokes the operation:

```ts
import { Selection } from '@ontahi/core/data-graph';
import { TodoApplication } from './graph.js';

const listId = process.argv[2];
if (!listId) throw new Error('Usage: rename-list <list-id>');

const lists = TodoApplication.graph.entities.TodoList;
const list = lists.refById(listId);

const result = await TodoApplication.invokeOperation(lists.domain.rename, {
  list: Selection.references(lists, [list]),
  name: 'Research backlog',
});

if (!result.ok) throw new Error(`TodoList.rename failed: ${result.kind}`);
console.log(result.value);
```

The Ref answers “which entity?”. The Selection answers “which members?”. The operation owns what
renaming means.

## The transparent React form

The client accepts the same ref directly and normalizes it to the operation's one-member Selection:

```ts
const renameList = useOperation(TodoList.domain.rename);

await renameList.executeAsync({
  list: TodoList.refById(listId),
  name: 'Research backlog',
});
```

The default identity also permits a plain ID or an entity record. An explicit ref keeps the domain
identity visible and continues to work when a locator is composite.

## Composite identity

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
});
```

Given records returned by real operations, the association ref needs no synthetic join-table ID:

```ts
const assignment = TodoTag.refByTodoAndTag(todo.id, urgentTag.id);
const assignments = Selection.references(TodoTag, [assignment]);
```

That Selection can be passed to any operation that targets TodoTag assignments.
