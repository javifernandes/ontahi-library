# Evolution

Biology gives us a useful warning.

Living systems survive because they *change*. But they do not change arbitrarily. A living thing cannot adapt by becoming anything at all. It can only adapt through the possibilities made available by its own structure.

A tree does not become a river because water is useful.

A cell does not become a stone because stability is useful.

> Life evolves by changing while preserving enough *continuity* to remain itself.

This is the first lesson software can borrow from biology. Not that software is literally alive in the same way an organism is alive. It is not. A program does not breathe, heal, desire, or reproduce by itself.

But software systems do develop a kind of organized continuity. They maintain boundaries. They exchange matter, energy, and meaning with an environment. They remember prior changes. They absorb pressures from users, data, operators, organizations, laws, tools, and adjacent systems. They can become more capable, more brittle, more expressive, or more confused.

They can also *lose themselves*.

## Adaptation

Adaptation is often misunderstood as *speed*.

The adaptable system is imagined as the one that can change quickly, accept every new requirement, bend toward every request, and remain pleasant while doing so.

But this is not adaptation. It is *compliance*.

A system that accepts every pressure directly does not adapt. It deforms.

In biology, adaptation is constrained by form. The organism does not merely receive the environment. It interprets the environment through its own organization. A wing, an eye, a root, a shell, a nervous system: each makes certain responses possible and others impossible.

Software has *form* too.

Its form is not only code. It is also data model, vocabulary, runtime, ownership, permissions, workflows, conventions, tests, interfaces, deployment habits, and the stories people tell about what the system is.

When a new pressure arrives, the system can respond only through this form.

If the form is too rigid, the pressure cracks it.

If the form is too vague, the pressure passes through without becoming knowledge.

> Architecture lives between those failures.
>
> It gives change somewhere to go.

## Boundaries

Every living system has a boundary.

The boundary is not simply a wall. A wall separates. A living boundary *selects*.

A cell membrane does not isolate the cell from the world. It decides what may enter, what must leave, and under what conditions exchange can occur. Without that boundary, the cell cannot be open. It can only dissolve.

Software boundaries work the same way.

An API is a boundary. A schema is a boundary. A permission rule is a boundary. A user interface is a boundary. A module boundary is a boundary. A workflow is a boundary between possible actions and impossible ones.

Bad boundaries make change expensive in two opposite ways.

Some boundaries are too hard. They force every new idea to arrive as an exception, a duplicate path, or a parallel system.

Some boundaries are too soft. They allow everything to know everything else. At first this feels flexible. Later it becomes impossible to move any part without disturbing the whole.

> A good boundary does not prevent change.
>
> It metabolizes change.
>
> It lets the system receive the outside world without becoming indistinguishable from it.

> [!NOTE]
> This is why "openness" is not the opposite of structure. A system can be open only because it has a form capable of receiving what comes from outside.

## Metabolism

Biological systems do not merely collect inputs.

They transform what they receive into something the organism can use.

Food becomes tissue. Light becomes energy. Experience becomes behavior. The outside world is not copied into the organism. It is metabolized.

Software systems need the same discipline.

A support ticket should not become a flag by default.

A new workflow should not become a one-off branch by default.

A customer-specific request should not become a customer-specific ontology by default.

Sometimes the local answer is correct. Not every input deserves to reshape the system. But some inputs reveal that the system has been *missing a concept*. They show that the old model can no longer digest what reality is giving it.

> [!CULTURAL]
> The same pattern appears in social history. A society can absorb a pressure as a local practice, or it can be forced to reorganize its institutions around it. In Jared Diamond's *Guns, Germs, and Steel*, geography and material conditions are not just background. They become pressures that shape what a society can metabolize. Revolutions are moments when the old model can no longer digest reality without changing its categories.

This is when architecture becomes active.

It asks:

Is this pressure local?

Is this pressure evidence?

Is this an exception, or have we discovered a dimension?

> The work is not to say yes to every change.
>
> The work is to understand what kind of change has arrived.

## Mutation

Evolution needs variation.

Without variation, there is nothing to select. Without selection, variation becomes noise.

Software teams create variation constantly. They try a new workflow. They add a field. They introduce a naming convention. They split a module. They move a responsibility. They create an internal tool. They change the order of a screen. They write down an idea that had only been implicit.

Most variations should not become architecture.

They should remain experiments long enough to teach something.

This is hard because teams often confuse implementation with understanding. Once a feature exists, it begins to feel inevitable. Once a table exists, people begin to speak as if the table names the world. Once a workflow exists, users adapt to it, and the adaptation looks like confirmation.

> But a first implementation is not a discovery.
>
> It is a probe.

The system learns from what happens next.

Does the idea repeat?

Does it connect to other ideas?

Does it reduce strain elsewhere?

Does it make the vocabulary clearer?

Does it make future changes cheaper?

If not, the mutation may still have been useful. It taught the system what not to become.

## Selection

In nature, selection is not moral. It does not choose what is best in some absolute sense. It preserves what can continue under the conditions that exist.

Software selection is more complicated because the environment is partly human.

A design survives because users can understand it. Because operators can keep it running. Because the team can maintain it. Because the data can support it. Because the business still needs it. Because adjacent systems can depend on it. Because the language around it remains useful.

This means software evolution is never only technical.

> Technical correctness is one selection pressure among many.

A perfect abstraction can die if no one can explain it.

A mediocre abstraction can survive if too many workflows depend on it.

A fragile implementation can become central because it arrived at the right historical moment.

Architecture cannot ignore this. A system is not maintained by code alone. It is maintained by a *community of understanding* around the code.

When that understanding decays, the system becomes harder to change even if the code still runs.

> [!NOTE]
> Peter Naur's [*Programming as Theory Building*](https://pages.cs.wisc.edu/~remzi/Naur.pdf) makes this point sharply. The program text is not the whole system. The theory held by the programmers is part of what makes the program changeable. When that theory disappears, the code may still execute, but the system has lost part of its capacity to evolve.

## Continuity

The deepest lesson of evolution is that identity is not sameness.

An organism changes its cells and remains itself. A language changes its words and remains itself. A city changes its buildings and remains itself. A software system changes its code, schema, interface, deployment model, and users, and may still remain itself.

But not every continuity is healthy.

Some systems remain themselves only by refusing to learn.

Others change so often that nothing can be trusted to endure.

The interesting question is not whether a system preserves its current shape.

It should not.

> The question is whether the system preserves a living relation between what it has been and what it is becoming.

This is why the next problem is identity.

Before we can ask how a system changes, we must ask what it means for the system to remain itself.
