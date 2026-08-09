# Storage and Transport Adapters

Entities describe identity, relations, Selections, and operations. They do not know whether a Query
runs over an object in memory or a PostgreSQL table, nor whether an operation arrives from Node,
Express, or another transport.

The application keeps those two choices independent:

- **storage** interprets graph reads and changes;
- **transport** carries public operation invocations to the composed application.

## Start in memory

The smallest host needs no external infrastructure:

```ts
const storage = createInMemoryDataGraphStorage();

export const TodoApplication = ontahi({
  storage,
  tasks: inProcessTasks(),
  capabilities: { runtime: { notifications: todoNotifications } },
  entities: [TodoList, Todo, Tag, TodoTag],
});
```

When `ontahi(...)` composes the application, the storage receives the registered Entities. It can
then execute their Queries and Commands and expose the same Entity catalog to reflection.

The in-memory dataset lives with this process. It is ideal for examples and tests; it does not
claim persistence across restarts or coordination between processes.

## Carry the same model with PostgreSQL

Replace the storage, not the Entities or their operations:

```ts
const storage = createPostgresDataGraphStorage({
  pool: new Pool({ connectionString: process.env.DATABASE_URL }),
});
```

By default, the adapter derives plural snake-case tables and snake-case columns from the Entity
catalog: `TodoList` maps to `todo_lists`, and `listId` maps to `list_id`. A physical exception is a
focused override:

```ts
const storage = createPostgresDataGraphStorage({
  pool,
  overrides: {
    TodoList: {
      table: 'lists',
      columns: { name: 'label' },
    },
  },
});
```

All unspecified names keep the convention. There is no second complete Entity schema to maintain.

The adapter compiles the same Selections, Queries, Commands, relation paths, projections, ordering,
and cardinality rules into parameterized SQL. `TodoList.rename(...)` is still the operation; storage
changes how its graph command is carried out.

> [!MARGIN] **The host still owns schema history.** Ontahí can map the current semantic model to
> physical names. It does not infer the deployment history required to reach that schema. SQL
> migrations, indexes, constraints, connections, and transaction boundaries remain host choices.

## Mount the composed application

Transport begins after `TodoApplication` exists:

```ts
const server = express();

server.use(
  ontahiExpress(TodoApplication, {
    explorer: { indexFile },
  }),
);
```

One mount exposes the application surfaces that have an HTTP representation:

- `POST /operations` invokes public operations and answers permission checks;
- `GET /operations/tasks/:taskId/:runId` observes durable runs;
- `GET /application` returns reflected application metadata;
- `/explorer/*` serves inspection endpoints when Explorer is enabled.

The adapter does not create `renameTodoList`, `completeTodo`, or provider-specific domain routes.
It resolves the operation already declared by the Entity, validates its transported input, invokes
it through the application, and serializes the canonical result.

Host routes can live beside that mount:

```ts
server.get('/health', (_request, response) => {
  response.json({ storage: TodoApplication.storage.kind });
});
```

Express still owns process lifecycle, middleware ordering, authentication context, logging, static
assets, and error reporting. Ontahí owns operation meaning and invocation semantics.

## Change one boundary at a time

| Change | What remains unchanged |
| --- | --- |
| In-memory → PostgreSQL | Entities, Selections, operations, Node callers, and browser projection |
| Express → another transport | Storage, operation contracts, execution, and results |
| Enable Explorer | The application model and the storage used to read reflected Entity data |

The Next.js adapter carries the same invocation protocol through an App Router handler. The
Supabase adapter compiles the same graph plans through PostgREST and can persist task runs. Its
current application-storage assembly remains lower-level than the PostgreSQL path, so it belongs in
production adapter reference rather than the main form.

The invariant is the useful part: changing where state lives must not redefine the operation, and
changing how an invocation travels must not redefine the application.
