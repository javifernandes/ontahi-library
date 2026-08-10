# Queries and Commands

A Selection answers **which entities** belong to a set. A Query describes **what to read** from
that set. A Command describes **what to change**.

The distinction matters because a Selection is useful before any entity object exists. It can be
created from predicates or Refs, passed across an operation boundary, and interpreted later by the
application runtime.

## Observe a Selection

The `list` operation turns its input Selection into a Query:

```ts
list: operation({
  input: self.many(),
  output: self.array(),
  run: todos =>
    commands
      .where(todos)
      .orderBy(todo => todo.title)
      .limit(100),
}),
```

`todos` carries membership. `where`, `orderBy`, and `limit` build the read. Returning that Query
lets the configured runtime interpret it and produce the operation result.

The Selection itself remains unchanged. Another Query may reuse it with a different order, limit,
or projection.

## Change the same Selection

The `complete` operation interprets the same kind of value as the target of a Command:

```ts
complete: operation({
  input: O.object({
    todos: self.many(),
  }),
  run: ({ todos }) => commands.where(todos).update({ completed: true }),
}),
```

The Command does not need the result of `list`. It receives the Selection directly and updates the
entities that belong to it when the Command runs.

## One value, two executions

From Node, author the set once:

```ts
import { Todo } from './graph.js';

const openResearchTodos = Todo
  .selection(todo => todo.listId.eq('list-research'))
  .and(todo => todo.completed.eq(false));

const listed = await Todo.list(openResearchTodos);
if (!listed.ok) throw new Error(`Todo.list failed: ${listed.kind}`);

for (const todo of listed.value) {
  console.log(todo.title);
}

const completed = await Todo.complete({ todos: openResearchTodos });
if (!completed.ok) throw new Error(`Todo.complete failed: ${completed.kind}`);
```

`Todo.list` observes the Selection. `Todo.complete` changes it. The second operation does not
receive `listed.value`; it receives the original membership description.

## The same boundary in React

React uses the same generated Selection and the same two operations:

```tsx
const visibleTodos = useMemo(
  () =>
    Todo
      .selection(todo => todo.listId.eq(listId))
      .and(todo => todo.completed.eq(false)),
  [listId],
);

const todos = useOperationQuery(Todo.domain.list, visibleTodos);
const complete = useOperation(Todo.domain.complete);

return (
  <button
    disabled={!todos.data?.length || complete.isExecuting}
    onClick={() => void complete.executeAsync({ todos: visibleTodos })}
  >
    Complete visible
  </button>
);
```

The hook observes operation lifecycle; it does not turn the Selection into a client-side query
object. The generated Selection crosses the bridge as semantic input, and the server runtime
interprets it.

## Membership is not a snapshot

The Query and the Command execute at different moments. If membership changes between them, the
Command acts on the entities that match when it runs—not on a hidden copy of the rows previously
rendered.

When exact identities matter, construct the Selection from Refs instead. Predicate-defined and
reference-defined sets keep the same Query and Command contracts.
