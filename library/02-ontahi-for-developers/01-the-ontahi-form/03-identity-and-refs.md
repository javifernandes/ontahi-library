# Identity and Refs

A Ref is a value that references one particular instance of an Entity.

```ts
import { TodoList } from './graph.js';

const listA = TodoList.refById('A');
```

`listA` is not a TodoList record or a snapshot of its fields. It models the semantic identity of
that instance as ordinary data:

```ts
{
  kind: 'entity-ref',
  entityName: 'TodoList',
  locator: { id: 'A' },
}
```

Like a Promise lets application code speak about a value before that value is available, a Ref
lets application code describe work involving an Entity instance without first deciding how to
find it, load it, or transport it. The Ref can cross operation boundaries and become part of a
Selection while Ontahí's runtime decides when its identity must be interpreted.

The Ref does not claim that the instance exists or contain its current fields. It carries enough
meaning to refer to it.

## Act on the Ref

An operation that accepts one TodoList declares that target as a Selection:

```ts
rename: operation({
  input: O.object({
    list: O.selection(self, { cardinality: 'one' }),
    name: field.nonEmptyString({ trim: true }),
  }),
  output: self,
  run: ({ list, name }) =>
    commands.where(list).updateOneReturning({ name }, ['id', 'name']),
}),
```

Call it with the Ref directly:

```ts
import { TodoList } from './graph.js';

const listA = TodoList.refById('A');
const result = await TodoList.rename({
  list: listA,
  name: 'Research backlog',
});

if (!result.ok) throw new Error(`TodoList.rename failed: ${result.kind}`);
console.log(result.value);
```

The operation receives a semantic target, not a caller-owned snapshot. The Ref preserves which
TodoList the caller means until the runtime interprets that identity.

## Composite identity

Some identities need more than one field:

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
  operations: ({ self, commands, operation }) => ({
    remove: operation({
      input: O.object({
        assignment: O.selection(self, { cardinality: 'one' }),
      }),
      run: ({ assignment }) => commands.where(assignment).deleteOne(),
    }),
  }),
});
```

Given records returned by real operations, the association needs no synthetic join-table ID:

```ts
const assignment = TodoTag.refByTodoAndTag(todo.id, urgentTag.id);
const result = await TodoTag.remove({ assignment });

if (!result.ok) throw new Error(`TodoTag.remove failed: ${result.kind}`);
```

The locator carries both identity fields, so the association can cross an operation boundary by
semantic identity.

## The object does not cross the boundary

A React screen may first read and render complete TodoList records:

```tsx
const lists = useOperationQuery(TodoList.domain.list);
const rename = useOperation(TodoList.domain.rename);

return lists.data?.map(list => (
  <button
    key={list.id}
    onClick={() =>
      void rename.executeAsync({
        list,
        name: 'Research queue',
      })
    }
  >
    Rename {list.name}
  </button>
));
```

At the call site, `list` is a materialized object. It may feel like ordinary object-oriented code:
the UI has a TodoList and acts on that TodoList. But the complete object is not sent back to the
server. Because TodoList declares its identity, the generated boundary derives a Ref from
`list.id` and transports the semantic target instead.

The server operation interprets that target in its own runtime. It never treats the fields held by
the browser as the current or authoritative TodoList.

## One operation, many ways to name its target

An operation becomes more useful when it does not reduce its target to an `entityId` parameter.
The same `complete` operation can receive identities typed by hand, records selected in the UI, or
a criterion that has not been evaluated yet:

```tsx
const complete = useOperation(Todo.domain.complete);

await complete.executeAsync({
  todos: ['23'],
});

await complete.executeAsync({
  todos: selectedTodos,
});

await complete.executeAsync({
  todos: Todo.selection(todo => todo.listId.eq('list-research')),
});
```

The first call means “complete Todo 23.” The second means “complete these Todos selected in the
UI.” The third means “complete every Todo in the research list.” No preliminary read is required:
the criterion remains a lazy description until the operation runs.

The implementation of `complete` is identical in all three cases. Ontahí normalizes each target to
the operation's semantic input and delegates its interpretation to the runtime.
