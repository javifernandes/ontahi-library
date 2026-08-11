# Queries

A \concept{Query} describes what to read from an Entity set. A Selection contributes membership;
the Query adds result shape, relations, order, and bounds. Building one does not execute it.

## Choose the set

Start from every instance or from a predicate:

```ts
const allTodos = Todo.all();

const openTodos = Todo.where(todo => todo.completed.eq(false));
```

A previously authored Selection is already a valid starting point:

```ts
const visibleTodos = Todo
  .selection(todo => todo.list.eq(TodoList.refById('list-research')))
  .and(todo => todo.completed.eq(false));

const query = visibleTodos.orderBy(todo => todo.title);
```

`where` can continue narrowing a Query. Each call combines with the existing membership:

```ts
const urgentOpenTodos = Todo
  .where(todo => todo.completed.eq(false))
  .where(todo => todo.priority.in(['high', 'critical']));
```

## Project the result

Without `select`, a Query returns the complete Entity shape. `select` declares an exact result:

```ts
const todoSummaries = Todo
  .all()
  .select(todo => ({
    id: todo.id,
    title: todo.title,
    completed: todo.completed,
  }));
```

Selections may be nested into semantic groups without changing the Entity:

```ts
const rows = Todo.all().select(todo => ({
  identity: { id: todo.id },
  content: { title: todo.title, completed: todo.completed },
}));
```

The inferred TypeScript result follows the projected shape.

## Include related Entities

Declared Relations are available from the Query proxy:

```ts
const todosWithList = Todo
  .all()
  .include(todo => ({
    list: todo.list.select(list => ({
      id: list.id,
      name: list.name,
    })),
  }));
```

Relation reads have their own shape operators:

```ts
const todosWithRecentAssignments = Todo
  .all()
  .include(todo => ({
    tagAssignments: todo.tagAssignments
      .orderBy(assignment => assignment.tagId)
      .limit(5),
  }));
```

A required Reference Field produces one related value; a nullable Reference Field produces one
value or `null`. `hasMany` produces an array. The Relation declaration carries that cardinality
into the Query result.

## Order and bound

A bare field orders ascending. Call `asc()` or `desc()` when the direction should be explicit:

```ts
const page = Todo
  .where(todo => todo.completed.eq(false))
  .orderBy(todo => todo.priority.desc())
  .orderBy(todo => todo.title.asc())
  .limit(50);
```

Order calls compose in declaration order. `limit` bounds the materialized result without changing
membership.

Name a reusable read without duplicating its shape:

```ts
const openTodoTitles = Todo
  .where(todo => todo.completed.eq(false))
  .select(todo => ({ id: todo.id, title: todo.title }))
  .orderBy(todo => todo.title)
  .named('open-todo-titles');
```

## Choose an execution

A Query bound to an application runtime exposes several interpretations:

| Method | Result |
| --- | --- |
| `run()` | every matching result as an array |
| `get()` | one matching result or `null` |
| `count()` | the number of matching Entities |
| `exists()` | whether at least one match exists |
| `stream()` | a stream of matching results |

```ts
const first = openTodos.orderBy(todo => todo.title).get();
const total = openTodos.count();
const anyUrgent = urgentOpenTodos.exists();
const stream = todoSummaries.stream();
```

These are runtime computations. An operation may return a Query directly when it only needs the
final result:

```ts
list: operation({
  input: self.many(),
  output: self.array(),
  run: todos => todos.orderBy(todo => todo.title).limit(100),
}),
```

A more complex operation can execute an intermediate Query and continue composing work. The
runtime remains responsible for interpreting the same Query language against its storage.
