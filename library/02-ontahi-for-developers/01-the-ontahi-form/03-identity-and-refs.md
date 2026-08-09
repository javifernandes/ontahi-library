# Identity and Refs

A Ref identifies an entity without loading it.

```ts
import { TodoList } from './graph.js';

const list = TodoList.refById(listId);
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

const listId = process.argv[2];
if (!listId) throw new Error('Usage: rename-list <list-id>');

const list = TodoList.refById(listId);
const result = await TodoList.rename({
  list,
  name: 'Research backlog',
});

if (!result.ok) throw new Error(`TodoList.rename failed: ${result.kind}`);
console.log(result.value);
```

The operation boundary promotes the Ref to a one-member Selection. Application code does not need
to import `Selection` or repeat the entity just to target one known member.

The distinction still matters: a Ref answers “which entity?”; a Selection describes membership
and may later represent predicates, unions, or many members.

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

The locator carries both identity fields, and the same Ref-to-Selection promotion applies.
