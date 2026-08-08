# Entity and Identity

An Entity is a named kind of thing with stable identity.

```ts
export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: field.id(),
    name: field.nonEmptyString({ trim: true }),
  },
  locators: {
    refById: 'id',
  },
  identity: 'refById',
});
```

Fields describe values. A locator describes how an entity can be identified. `identity` selects
the default locator used when a caller supplies a plain ID or an entity record.

The declaration produces both a schema and ref factories:

```ts
const inbox = TodoList.refById('list-1');
```

A Ref is identity without loaded state:

```ts
{
  kind: 'entity-ref',
  entityName: 'TodoList',
  locator: { id: 'list-1' },
}
```

It says which TodoList. It does not claim that the list exists, contain its current fields, or read
storage.

## Composite identity

Identity can use more than one field:

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

```ts
const assignment = TodoTag.refByTodoAndTag('todo-1', 'tag-1');
```

Identity belongs to the semantic entity. Table names, indexes, and physical constraints belong to
the storage mapping and migration owned by the host.

## What the entity owns

The complete declaration may also contain:

- display and freshness metadata;
- relations;
- application dependencies through `uses`;
- graph and domain operations.

These are facets of one entity, not parallel schemas for each runtime. Ontahí projects the same
declaration toward storage, reflection, codegen, clients, and Explorer.
