# Operation Contracts and Failures

An operation's input schema says which values may enter. Its contracts say what must be true for
the operation to proceed. Its failures are expected domain answers, carried as data.

## Put shape in the input schema

`rename` accepts one TodoList and a non-empty name:

```ts
input: O.object({
  list: O.selection(self, { cardinality: 'one' }),
  name: field.nonEmptyString({ trim: true }),
}),
```

The same schema validates direct Node calls and transported calls. A blank name is an invalid
input, not a business decision hidden inside `run`.

## Put a precondition before the body

Suppose `Archive` is reserved for a list managed by the application:

```ts
operations: ({ self, commands, operation, app }) => ({
  rename: operation({
    input: O.object({
      list: O.selection(self, { cardinality: 'one' }),
      name: field.nonEmptyString({ trim: true }),
    }),
    output: self,
    contracts: {
      pre: ({ name }) =>
        name.toLowerCase() === 'archive'
          ? app.operation.createFailure(
              'reserved_list_name',
              'Archive is reserved for system use.',
            )
          : undefined,
    },
    run: ({ list, name }) =>
      commands.where(list).updateOneReturning({ name }, ['id', 'name']),
  }),
}),
```

The precondition receives validated semantic input. Returning a failure stops the operation before
its body runs. Returning nothing lets it continue.

A contract names a condition of the operation; it is not controller middleware and it does not
belong to a particular transport.

## Handle the answer from Node

```ts
const result = await TodoList.rename({
  list: 'A',
  name: 'Archive',
});

if (result.ok) {
  console.log(result.value.name);
} else if (
  result.kind === 'failed' &&
  result.failure.reason === 'reserved_list_name'
) {
  console.error(result.message);
}
```

The caller receives one invocation result. `reserved_list_name` is stable application meaning;
the message is its human-readable explanation.

## Show the same failure in React

```tsx
const rename = useOperation(TodoList.domain.rename);
const [error, setError] = useState<string>();

async function submitRename() {
  const result = await rename.executeAsync({ list, name });
  setError(result.ok ? undefined : result.message);
}
```

The hook adds execution state; it does not change the operation's failure model. The component may
show every failure message or branch on a specific reason when the UI has a specific recovery.

## Keep the failure boundary sharp

An invocation has four non-success shapes:

- `input_invalid`: the value did not satisfy the input schema; the operation did not start.
- `rejected`: a requirement such as authorization refused execution.
- `failed`: the operation reported an expected domain failure, from a contract or from `run`.
- `errored`: an unexpected defect or runtime failure occurred.

Use `app.operation.fail(...)` inside `run` when a domain answer only becomes known after reading
state or calling another capability. Use `contracts.pre` when a rule over validated input must hold
before the body. Keep authorization in `requires`, where the runtime can decide whether the
operation may run at all.

Operations also support `contracts.post` for checking their result. A postcondition is a check, not
a rollback policy; use it only with an execution boundary whose atomicity or compensation matches
the invariant.
