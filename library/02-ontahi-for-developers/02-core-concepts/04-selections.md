# Selections

A \concept{Selection} is a value that describes which instances of one Entity belong to a set.
It does not load records or require those records to exist in memory. A runtime interprets it only
when a Query, Command, or Operation consumes it.

## Describe membership

Start a Selection from the Entity:

```ts
const open = Todo.selection(todo => todo.completed.eq(false));
const inResearch = Todo.selection(todo =>
  todo.list.eq(TodoList.refById('list-research')),
);
const urgent = Todo.selection(todo => todo.priority.in(['high', 'critical']));
const unassigned = Todo.selection(todo => todo.assigneeId.isNull());
const stale = Todo.selection(todo => todo.createdAt.lt('2026-07-01T00:00:00Z'));
```

Field references expose the predicates supported by the Selection language:

| Predicate | Membership test |
| --- | --- |
| `field.eq(value)` | the field equals one value |
| `field.in(values)` | the field belongs to a list of values |
| `field.isNull()` | the field has no value |
| `field.lt(value)` / `lte` | the field is below a boundary |
| `field.gt(value)` / `gte` | the field is above a boundary |

These methods build a portable expression. None of the examples above has queried storage.

## Compose sets

Selections form a small algebra through `and`, `or`, and `not`:

```ts
const visible = open.and(inResearch);
const needsAttention = urgent.or(stale);
const active = Todo
  .selection(todo => todo.archived.eq(true))
  .not();

const actionable = visible.and(needsAttention).and(active);
```

An operand may also be written inline when it is used once:

```ts
const recentlyCreatedOpenTodos = open.and(todo =>
  todo.createdAt.gte('2026-08-01T00:00:00Z'),
);
```

Name a reusable criterion when the name carries application meaning:

```ts
const triageQueue = actionable.named('triage-queue');
const portableAst = triageQueue.toJSON();
```

The serialized form preserves the Entity root and membership expression. That is what lets the
same criterion cross a client/server boundary or become editable by another tool.

## Identities can describe membership too

An operation input declared as `self.many()` accepts a predicate-defined Selection, explicit
identities, or materialized records:

```ts
await Todo.complete({ todos: triageQueue });
await Todo.complete({ todos: ['todo-23', 'todo-42'] });
await Todo.complete({ todos: checkedTodoRecords });
```

Ontahí normalizes each form into the same semantic target. The operation does not need
`completeById`, `completeSelected`, and `completeByFilter` variants.

## One Selection, different interpretations

A Query can observe the set:

```ts
const nextTen = triageQueue
  .orderBy(todo => todo.createdAt)
  .limit(10);
```

A Command can change exactly the same set:

```ts
const completeTriageQueue = triageQueue.update({ completed: true });
```

The Selection remains membership. `orderBy` and `limit` add read shape; `update` chooses a write
interpretation. The next two chapters describe those operator surfaces directly.

## The UI can author the same value

A filter model can construct the Selection used by both a read and an operation:

```tsx
const visibleTodos = useMemo(() => {
  const inList = Todo.selection(todo =>
    todo.list.eq(TodoList.refById(listId)),
  );
  return status === 'all'
    ? inList
    : inList.and(todo => todo.completed.eq(status === 'completed'));
}, [listId, status]);

const todos = useOperationQuery(Todo.domain.list, visibleTodos);
const complete = useOperation(Todo.domain.complete);

await complete.executeAsync({ todos: visibleTodos });
```

The UI does not translate its filter into a separate request model. It authors the same
Selection language used by Node and interpreted by the operation runtime.
