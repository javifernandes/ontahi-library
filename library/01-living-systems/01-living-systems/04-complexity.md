# Complexity

Complexity begins when no single *observer* can hold the whole system at once.

This does not mean the system is large.

A large system can be simple if its parts are repetitive, its *boundaries* are stable, and its behavior can be understood by looking at one piece at a time.

A small system can be complex if every part changes the *meaning* of every other part.

Complexity is not the number of things.

It is the number of *relations* that matter.

> Complexity is what happens when understanding stops being local.

This is why complexity often arrives quietly. A system can remain small in code and become large in consequence. One flag affects one workflow, which affects one permission rule, which affects one report, which affects one operational habit, which changes what users believe the system promises.

> [!NOTE]
> This is close to the butterfly effect, but the architectural point is not drama. It is sensitivity. A small change can become large when it travels through relations the system does not yet know how to see.

Nothing looks dramatic from any single point.

But the whole has changed.

## Relations

A system is not made only of *components*.

It is made of *relations* between components.

A body relates sleep to hunger, hunger to energy, energy to attention, attention to mood, mood to the way a person meets the world.

A city relates transport to work, work to time, time to care, care to politics, politics to what people believe is possible.

A software system is not different in kind. It only makes some of these relations executable.

The component is visible.

The relation is often *invisible*.

This is why complexity hides. We can count files, endpoints, tables, services, and screens. It is harder to count the assumptions that pass between them.

When a system is young, many relations live in the heads of the people who built it. They are not yet architecture. They are memory.

Over time, those relations either become *explicit* enough to be shared, or they become traps.

## Local Understanding

The first sign of complexity is that *local understanding* stops being enough.

A developer can understand the function and still misunderstand the change.

An operator can understand the alert and still misunderstand the incident.

A designer can understand the screen and still misunderstand the workflow.

A user can understand the action and still misunderstand its consequences.

Each local view is true.

Each local view is *incomplete*.

> Complexity does not begin when people are wrong.
>
> It begins when people can be locally right and systemically wrong.

This is one reason complex systems feel unfair. The mistake is not always inside the part where the failure appears. A report breaks because a naming convention changed. A permission bug appears because an onboarding workflow learned a shortcut. A queue stalls because a retry policy was correct in isolation and destructive in combination.

The system has more *behavior* than any one part declares.

## Coupling

*Coupling* is not always bad.

Systems need things to depend on other things. A system with no coupling has no shape. It cannot coordinate. It cannot remember. It cannot promise anything.

The problem is not coupling.

The problem is coupling that cannot be *named*.

*Named coupling* can be designed around. It can be tested, documented, monitored, negotiated, or made explicit in an interface.

*Unnamed coupling* becomes superstition.

People learn not to touch a file.

They learn to deploy in a certain order without knowing why.

They learn that a field is "probably unused" but dangerous.

They learn that a workflow must remain strange because too many things depend on its strangeness.

> [!NOTE]
> This is another place where symptoms matter. A strange ritual may be irrational as an explanation and rational as a trace. It may preserve knowledge of a relation the system has failed to name.

Complexity becomes dangerous when the system depends on relations that the system cannot *describe*.

## Compression

No one can understand a complex system by remembering everything.

Understanding requires *compression*.

We give names to patterns. We draw boundaries. We create diagrams. We define invariants. We build dashboards. We write tests. We create types, protocols, services, modules, and stories.

Each of these is a *compression* of reality.

> [!NOTE]
> Freud's idea of condensation in dreams is useful here. A single image can carry several wishes, conflicts, memories, and substitutions at once. Architectural abstractions do something similar: they gather many relations into one name. This gives thought a handle, but it also means the name may hide more than it shows.

Each helps us ignore detail safely.

But compression can also *lie*.

A diagram can make a relation look simpler than it is.

A type can name a concept too early.

A service boundary can hide a dependency instead of clarifying it.

A metric can make the system look healthy by measuring what is easy to count.

A story can keep coherence by leaving out the contradiction that matters.

Good architecture is not the elimination of compression.

It is the discipline of choosing compressions that remain *correctable*.

## Coordination

Complexity is not only *technical*.

It is also *social*.

As a system grows, no one changes the system directly. People change parts of the system through roles, permissions, habits, reviews, meetings, documents, alerts, dashboards, releases, migrations, and conversations.

The architecture of the software and the architecture of *coordination* become entangled.

Sometimes the code is simple and the coordination is complex.

Sometimes the coordination is simple because the code has absorbed the complexity.

Sometimes both are complex, and the system survives only because people have learned informal practices that the architecture does not yet understand.

This is fragile.

Not because informal practice is bad.

Informal practice is often where understanding is born.

It is fragile because knowledge that cannot *move* cannot scale.

## Observability

Complex systems need ways to *see themselves*.

Without observability, a system can only discover itself through failure.

*Observability* is not just logs, metrics, and traces. Those matter, but they are not enough.

A system is observable when its behavior can be connected back to the *concepts* that explain it.

Can we see which workflow produced this state?

Can we see which boundary allowed this change?

Can we see which invariant was preserved, weakened, or violated?

Can we see whether this failure is local noise or evidence of a missing concept?

The goal is not total visibility.

Total visibility is another fantasy of control.

The goal is enough visibility for the system to *learn* from what happens to it.

> Observability is the memory of complexity becoming available to thought.

## The Cost of Change

Complexity matters because it changes the *cost of change*.

In a simple system, the cost of a change is close to the cost of editing the part.

In a complex system, the cost of a change includes the cost of discovering what else the change *means*.

This *discovery cost* can dominate everything.

A field takes one minute to add and three weeks to understand.

A workflow takes one day to implement and six months to unwind.

A boundary takes one meeting to name and years to make real.

A concept takes one sentence to say and a long time to earn.

This is why architecture cannot be judged only by the elegance of its current shape.

It must be judged by the kinds of *discovery* it makes possible.

## The Next Question

Evolution shows that systems change.

Identity asks how they remain continuous.

Complexity shows why change becomes expensive.

The next problem is the cost of change itself.

Because once a system contains more relations than any one person can hold, every change becomes partly an act of *interpretation*.

> [!NOTE]
> This is also a linguistic problem. A change does not only alter behavior; it can alter the language by which the system is understood. A new name can recombine old meanings. An old word can begin to carry a new function. In this abstract sense, language is the first human-computer interface: the way humans ask a system what it is, what it means, and what can be done with it.

The question is no longer only what should change.

The question is what it will cost the system to *understand* the change.
