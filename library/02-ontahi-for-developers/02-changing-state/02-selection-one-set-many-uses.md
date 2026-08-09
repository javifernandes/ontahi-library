# Selection: One Set, Many Uses

A Selection is a value that describes which entities belong to a set. It does not read storage,
contain loaded entities, or require those entities to be present in memory. Reads and operations
can consume that same value.

```ts
operations: ({ self, commands, operation }) => ({
  list: operation({
    input: O.selection(self, { cardinality: 'many' }),
    output: O.array(self),
    bridge: { query: [(todos: unknown) => todos] },
    run: todos => commands.where(todos).orderBy(todo => todo.title),
  }),

  complete: operation({
    input: O.object({
      todos: O.selection(self, { cardinality: 'many' }),
    }),
    bridge: { invalidate: [['Todo']] },
    run: ({ todos }) => todos.update({ completed: true }),
  }),
}),
```

`list` reads a Selection. `complete` changes one. Neither operation needs to know whether its
members came from a filter, checked rows, or another operation.

## Author a criterion from Node

Build membership from the Entity itself:

```ts
import { Todo } from './graph.js';

const openResearchTodos = Todo
  .selection(todo => todo.listId.eq('list-research'))
  .and(todo => todo.completed.eq(false));

const result = await Todo.list(openResearchTodos);
if (!result.ok) throw new Error(`Todo.list failed: ${result.kind}`);

for (const todo of result.value) {
  console.log(todo.title);
}
```

`Todo.selection(...)` produces a serializable membership value. `and`, `or`, and `not` compose it
without executing it. At this point no entity has been loaded and no storage has been consulted.

The same value can target the mutation:

```ts
const completed = await Todo.complete({ todos: openResearchTodos });
if (!completed.ok) throw new Error(`Todo.complete failed: ${completed.kind}`);
```

## Let the UI edit the same criterion

The React filter authors the same Selection through the generated Entity:

```tsx
import { useOperation, useOperationQuery } from '@ontahi/react/graph';
import { useMemo, useState } from 'react';
import { Todo } from './generated/client-entities.js';

type StatusFilter = 'all' | 'open' | 'completed';

export const Todos = ({ listId }: { listId: string }) => {
  const [status, setStatus] = useState<StatusFilter>('all');
  const visibleTodoSelection = useMemo(() => {
    const inList = Todo.selection(todo => todo.listId.eq(listId));
    return status === 'all'
      ? inList
      : inList.and(todo => todo.completed.eq(status === 'completed'));
  }, [listId, status]);

  const todos = useOperationQuery(Todo.domain.list, visibleTodoSelection);
  const complete = useOperation(Todo.domain.complete);

  return (
    <section>
      <nav>
        {(['all', 'open', 'completed'] as const).map(value => (
          <button key={value} onClick={() => setStatus(value)}>{value}</button>
        ))}
      </nav>

      <ul>
        {todos.data?.map(todo => <li key={todo.id}>{todo.title}</li>)}
      </ul>

      <button
        disabled={!todos.data?.length || complete.isExecuting}
        onClick={() => void complete.executeAsync({ todos: visibleTodoSelection })}
      >
        Complete visible
      </button>
    </section>
  );
};
```

The filter is not translated into a request DTO or rebuilt in the operation. It edits the same
membership language used in Node and consumed by both operations.

## Selected rows are a Selection too

When the UI already has explicit identities, pass them directly:

```ts
await complete.executeAsync({ todos: checkedTodoIds });
```

Ontahi turns those IDs into entity Refs and a reference-defined Selection. Predicate-defined and
reference-defined sets share the same operation contract.

A Selection becomes active only when something interprets it. The next chapter separates the Query
that observes its members from the Command that changes them.
