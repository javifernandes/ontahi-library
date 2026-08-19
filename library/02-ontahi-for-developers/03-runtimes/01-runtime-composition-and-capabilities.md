# Runtime Composition and Capabilities

A \concept{Runtime} interprets Ontahí's semantic declarations in a concrete environment.
An Entity may also need work that does not belong to its graph: send a notification, call an
external service, or apply a host policy. A \concept{Capability} names that need. The
application supplies the implementation.

> [!MARGIN] **A draft, low-level surface.** Today `capabilities` is typed resource injection. It can
> carry almost anything, which makes it useful, but the injected resources remain opaque to
> reflection and composition. This API may change as Ontahí reifies recurring concepts instead of
> turning arbitrary dependency injection into the final model.

## Declare the need

`TodoList` needs to announce a newly created list:

```ts
type TodoCapabilities = OntahiCapabilities & {
  runtime: {
    notifications: {
      todoListCreated(input: {
        listId: string;
        name: string;
      }): Effect.Effect<void>;
    };
  };
};

export const TodoList = entity({
  name: 'TodoList',
  fields: {
    id: f.id(),
    name: f.nonEmptyString({ trim: true }),
  },
  uses: {
    capabilities: {} as TodoCapabilities,
  },
  // operations follow
});
```

`uses.capabilities` is currently a TypeScript witness for the application surface this Entity
expects. `notifications` is vocabulary of this application; Ontahí does not prescribe a universal
notification service.

## Use it inside the operation

The operation coordinates the graph command and the external effect:

```ts
operations: ({ self, commands, operation, app }) => ({
  create: operation({
    input: O.pick(self, ['id', 'name']).named('CreateTodoListInput'),
    output: self,
    run: input =>
      Effect.gen(function* () {
        const created = yield* commands
          .insertReturning(input, ['id', 'name'])
          .run();

        yield* app.runtime.notifications.todoListCreated({
          listId: created.id,
          name: created.name,
        });

        return created;
      }),
  }),
}),
```

The Entity decides **when** notification belongs in `create`. It does not know whether the host
sends email, publishes an event, writes an audit record, or does nothing.

The Query or Command still owns graph execution. The Capability owns work outside that graph. The
operation is where both become one application intention.

## Supply the implementation once

The host implements the Capability at the application root:

```ts
const todoNotifications = adaptEffectMethods<
  TodoCapabilities['runtime']['notifications']
>({
  todoListCreated: ({ listId, name }) =>
    console.info(`[todo] created list ${listId}: ${name}`),
});

export const TodoApplication = ontahi({
  storage: defaultStorage,
  tasks: inProcessTasks(),
  capabilities: {
    runtime: {
      notifications: todoNotifications,
    },
  },
  entities: [TodoList, TodoItem, Tag],
});
```

`ontahi(...)` is the composition boundary. It combines semantic declarations with the concrete
storage, task runtime, and application Capabilities that will interpret them in this process.

The operation consumes one uniform Effect, but a host implementation may be synchronous,
asynchronous, or already return an Effect. `adaptEffectMethods(...)` normalizes that resource once,
at the host boundary, and preserves lazy execution. The implementation itself does not need
`Effect.sync(...)` ceremony.

## Call the composed Entity from Node

```ts
import { TodoList } from './graph.js';

const result = await TodoList.create({
  id: crypto.randomUUID(),
  name: 'Reading queue',
});

if (!result.ok) throw new Error(result.message);
console.log(result.value.name);
```

The caller invokes one operation. The bound Entity already belongs to `TodoApplication`, so the
operation receives the notification implementation selected by that application.

## React sees the operation, not the Capability

```tsx
const createList = useOperation(TodoList.domain.create);

await createList.executeAsync({
  id: crypto.randomUUID(),
  name: 'Reading queue',
});
```

The browser does not import `todoNotifications`, receive its credentials, or choose an
implementation. It invokes the projected operation; the server-side application supplies the
Capability when that operation runs.

## Replace the host, preserve the Entity

A test can record the same effect without changing `TodoList`:

```ts
const recorded: Array<{ listId: string; name: string }> = [];

const notifications = adaptEffectMethods<
  TodoCapabilities['runtime']['notifications']
>({
  todoListCreated: input => recorded.push(input),
});

const TestApplication = ontahi({
  storage: createInMemoryDataGraphStorage(),
  capabilities: { runtime: { notifications } },
  entities: [TodoList, TodoItem, Tag],
});
```

Production may bind a different implementation under the same contract. The Entity and operation
remain unchanged because their dependency is semantic, not provider-specific.

## Keep the boundary honest

A Capability does not make multiple effects atomic. In the example, persistence happens before
notification. If both must succeed or recover together, the operation and host need a transaction,
an outbox, compensation, or durable coordination that matches that guarantee.

Today `uses.capabilities` is a typed dependency witness over low-level injected resources. Ontahí
does not yet reflect these dependencies or validate the complete Capability graph at composition
time. Keep the declared surface narrow: it documents and types what the Entity uses, but the host
remains responsible for supplying it correctly. Repeated semantic needs may eventually deserve
their own declared model rather than another arbitrary object path.

Authentication is one such need that now has its own model. Use the invocation Principal rather
than introducing a generic `currentUser` Capability; see
[Authentication and Principals](04-authentication-and-principals.md).
