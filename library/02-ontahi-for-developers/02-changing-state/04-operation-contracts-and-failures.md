# Operation Contracts and Failures

An operation's input schema says which values may enter. Put every condition knowable from those
values in that schema. Reserve executable preconditions for facts that must be resolved when the
operation runs.

## Make static rules visible

`Archive` is a reserved TodoList name. That is a property of the Entity field, not a dynamic
precondition:

```ts
export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: field.id(),
    name: field.nonEmptyString({
      trim: true,
      exclude: {
        values: ['archive'],
        caseInsensitive: true,
      },
      messages: {
        exclude: 'Archive is reserved for system use.',
      },
    }),
  },
  operations: ({ self, commands, operation }) => ({
    rename: operation({
      input: O.object({
        list: O.selection(self, { cardinality: 'one' }),
        name: self.fields.name,
      }),
      output: self,
      run: ({ list, name }) =>
        commands.where(list).updateOneReturning({ name }, ['id', 'name']),
    }),
  }),
});
```

`exclude` is reflected schema data. The server validates it, codegen preserves it, and a UI can
inspect the same values and message before invoking the operation. Nothing needs to execute to
discover that `Archive` is outside the accepted set. `rename` reuses `self.fields.name`; inputs
derived with `O.pick(...)` inherit it too.

## Anticipate the rule in React

The generated operation carries its input schema:

```tsx
import { safeParseGraphSchema } from '@ontahi/core/data-graph';

const rename = useOperation(TodoList.domain.rename);
const nameValidation = safeParseGraphSchema(
  TodoList.domain.rename.input.fields.name,
  name,
);

return (
  <div>
    <button
      disabled={!nameValidation.success || rename.isExecuting}
      onClick={() => rename.execute({ list, name })}
    >
      Rename
    </button>

    {!nameValidation.success && <p>{nameValidation.issues[0]?.message}</p>}
  </div>
);
```

The component consumes the contract; it does not duplicate `archive` as UI knowledge. Another UI
may render the reflected constraint differently while preserving the same admissible input.

## Keep preconditions dynamic

Uniqueness depends on current application state. A `create` operation may query for an existing
TodoList before running its command:

```ts
operations: ({ self, commands, operation, app }) => ({
  create: operation({
    input: O.pick(self, ['id', 'name']).named('CreateTodoListInput'),
    output: self,
    contracts: {
      pre: ({ name }) =>
        Effect.gen(function* () {
          const existing = yield* commands
            .where(list => list.name.eq(name))
            .select(list => ({ id: list.id }))
            .limit(1)
            .run();

          return existing.length > 0
            ? app.operation.createFailure(
                'todo_list_name_taken',
                'A TodoList already uses that name.',
              )
            : undefined;
        }),
    },
    run: input => commands.insertReturning(input, ['id', 'name']),
  }),
}),
```

This precondition cannot be decided from the input alone. It resolves graph state immediately
before the operation body. Returning a failure stops the body; returning nothing lets it continue.

Executable checks are necessarily less transparent than schema constraints. Reflection can expose
that the operation has a precondition, but it cannot predict the result without executing the
required reads.

## Keep the two failures distinct

```ts
const invalid = await TodoList.rename({
  list: 'A',
  name: 'Archive',
});

if (!invalid.ok && invalid.kind === 'input_invalid') {
  console.error(invalid.issues[0]?.message);
}

const duplicate = await TodoList.create({
  id: crypto.randomUUID(),
  name: 'Research',
});

if (
  !duplicate.ok &&
  duplicate.kind === 'failed' &&
  duplicate.failure.reason === 'todo_list_name_taken'
) {
  console.error(duplicate.message);
}
```

`input_invalid` means the value was outside the declared input set and the operation did not
start. `failed` means execution began and produced an expected domain answer.

## A precondition is not a uniqueness invariant

Two concurrent `create` operations can both observe that a name is available. A precondition makes
the dynamic rule and its failure explicit, but it does not make the read and insert atomic by
itself.

True uniqueness wants to live as a declarative Entity invariant interpreted by storage. Ontahí
does not yet expose that settled model. Until it does, the database constraint remains the final
authority, and the operation must translate its violation into the same expected domain failure.

The remaining invocation shapes keep their existing meaning:

- `rejected`: a requirement such as authorization refused execution.
- `errored`: an unexpected defect or runtime failure occurred.

Use `app.operation.fail(...)` inside `run` when an expected domain answer only becomes known during
the body. Keep authorization in `requires`. Use `contracts.post` for checking a result, remembering
that a postcondition is a check—not a rollback policy.
