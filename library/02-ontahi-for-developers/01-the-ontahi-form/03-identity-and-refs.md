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
