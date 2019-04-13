---
layout: post
title: Python and Dependency Inversion
description: "Dependency inversion is a powerful programming technique. Find out what it is, and when you should apply it in your Python projects."
image: upside-down.jpg
featured: true
weight: 5
tags: [python, architecture, factoring]
---

Most programmers recognise the value of organised structure in all but the smallest of software projects. In Python,
we can easily split code across modules and organise these modules into nested subpackages. It's a simple, powerful way
of structuring source code.

And how do we choose to split things? By the *separation of concerns*. This separation can take different forms. We
might organize things by feature area (the authentication system, the shopping cart, the blog) or by level of detail
(the user interface, the business logic, the database), or both.

When we do this, we tend to be aiming at modularity. We know that systems can get complicated to work with, so
separating concerns like this makes it easier to work on smaller bits of the system in isolation.

Except for some reason it doesn't. In practice, working on one module turns out to relate to another part of the system,
which relates to another, which relates back to the original module. Pretty soon your head hurts and you need to have
a lie down. What's going wrong?

## Separation of concerns is not enough

The sad fact is, if the only organizing factor of a code base is separation of concerns, a code base will not be
modular after all. Instead, separate parts will tangle together, and it will become increasingly difficult to work with.

Pretty quickly, our efforts to organise what goes into each module are undermined by *the relationships between those
modules*.

This is naturally what happens to software if you don't think about relationships. This is because in the real world
things *are* a messy, interconnected web of connections. It's natural that as we build functionality, we realise
that one module needs to know about another. Later on, that other module needs to know about the first.

The problem with a system like this is that, because of the web of relationships, it is not a collection of smaller
subsystems. Instead, it is a single, large system - and the larger a system, the more difficult it is to understand and
work with.

## Dependency graphs

These relationships between modules can be thought of as *dependencies*.

In Python, the most obvious dependency is if one module imports another module. Another common form is when a function
in one module expects a particular type of object as an argument.

We can represent the dependencies of any project as a [directed graph](https://en.wikipedia.org/wiki/Directed_graph),
which is simply a network of items with arrows drawn between them. The arrows are usually drawn towards the module
being depended on: so if there is an arrow from A to B, we say A depends on B. We call such a representation a
*dependency graph*.

(dependency graph)

## Circular dependencies

Let's look at a graph of a system whose web of relationships is out of control:

(tangled dependency graph)

Here we see a tangled mess of connections. One can trace the reason for this tangle back to one problem:
it has cycles between its items. These cycles are what we call circular dependencies.

## The Directed Acyclic Graph

There is a specific name for a Directed Graph that has no cycles: a
[Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph), or DAG. In other
words, you could not start at one point and follow the arrows around until you returned to that same point.

Here's an example of one:

(pic of DAG)

If you want a modular, maintainable code base, then this is the overall shape that you're striving for.

Notic the clear direction of flow: the arrows are all pointing downward.
You can think of it like a river, with the water all flowing downhill. Using this metaphor, we can talk about modules
that are being imported as 'upstream', while modules doing the importing as 'downstream'. This is a useful turn of
phrase, especially if the dependencies are via several intermediate modules. 

# Enter dependency inversion

Circular dependencies are the enemy of modularity. With them, our subsystems are coupled in both directions, which
means they aren't separate subsystems at all. We need to avoid letting them creep into our code bases. But how?

Take two modules, **A** and **B**, and let's say that **A** depends on **B**.

(diagram of A -> B)

Now, let's say we realise that **B** needs something from **A**. We begin to implement it, but then, alas! We realise
we are about to introduce a circular dependency:

(diagram of A <-> B)

There's only one way to avoid the cycle. We pick an arrow, and then refactor the code so that the direction of the
 arrow is reversed. Once that's done, the circular dependency is gone.

(diagram of A -> B, two arrows)

This refactoring is dependency inversion. 

In practice, this can sometimes feel impossible. Surely there is no way to reverse the direction of the arrow
merely by refactoring? But I have good news. It is never impossible. I promise. You can *always* avoid circular
dependencies by inverting them. It's not always the most obvious way to write code, but it can make your code base
significantly easier to work with.

There are several different techniques for *how* you reverse that arrow. One such technique is called
'dependency injection'. I will cover some of these techniques in part two of this series, but in this post I'll focus
on the bigger picture.

# Your code base's overall shape

We've just looked at two modules in isolation. It's all very well avoiding cycles in a simple example like this, but
how on earch are we meant to achieve this across a code base with hundreds or thousands of modules? What would that
even look like?

# Achieving DAGs in practice

Organising a code base's dependencies as a DAG needs both design and process.

## Designing your project's dependencies

Deciding on the relationships between modules in your code base is an act of design - that's why it's called
software architecture.

### Level-specific graphs

The first thing to realise is that if your system is large, designing a dependency graph between every module within it
will be much too detailed to be useful. Instead, you should only consider modules all at one level.

(Top level graph)

In this example, we are only depicting the first level modules ( the immediate children of the top level package). All
dependencies between deeper descendants have been summarized into dependencies between their first level ancestors.
So, the arrow from `mypackage.green` to `mypackage.blue` could be a result an import of `mypackage.green.foo` by
`mypackage.blue.bar`.

Level-specific graphs like this occur throughout the system: `mypackage.green` might also have a graph of its own:

(graph for mypackage.green)

The first thing you need to decide is which of these graphs you care about. How far you take it is up to you: some
subpackages may be small enough that they don't need as much management to prevent mess.  You can choose the parts
of your system that are in danger of getting overcomplicated, and aim to make them DAGs. If you're not sure, you
should usually just pick one to begin with: the first level graph.

### Dependency ordering

Once you've chosen which graph to focus on, it's time to define its structure. The simplest way to do this is to decide
on a dependency order. This is simply a list of modules, from downstream to upstream. Here's an example:

```
- mypackage.green
- mypackage.blue
- mypackage.orange
- mypackage.yellow
- mypackage.purple
- mypackage.red
```

This list describes a code base in which each modules may import from lower down the list, but not higher up. If you
follow this rule, you'll have a DAG.

If you want to add a further constraint, you could also allow multiple items per line:

```
- mypackage.green
- mypackage.blue
- mypackage.orange, mypackage.yellow
- mypackage.purple
- mypackage.red
```

In this example, items on the same line must be independent (i.e neither can import from each other).

Although DAGs can have a more complex structure, this is a simple way of designing a code base free from
cycles (at least at the level of the list).

---

## 

There is only one way to fight this natural tendency for things to get into a mess: dependency inversion. In its
simplest form

Take two components within a software system. Let's call them **A** and **B**,
and let's say that **A** depends on **B**. We can draw this as an arrow from **A** to **B**.

(diagram of A -> B)

Dependency inversion is simply reorganizing the code so that **B** depends on **A** instead:

(diagram of A <- B)

I should probably define my terms. By *component*, I just mean any part of a software system,
at any level of detail: a function, a class, a source file, a collection of source files, or indeed an entire
runnable service.

When I say **A** *depends on* **B**, I mean that **A** could be broken if **B** was changed in some way.

There are several different patterns for *how* you reverse that arrow. One such pattern is dependency injection.
I will cover some of these patterns in a future blog post, but for now I'll focus on why you would want to invert
dependencies, and how to decide in which direction the arrows should point.

# Why dependency inversion is important






# How should your dependencies look?

Armed with the knowledge that dependencies may be inverted, what do you do next? How should your dependencies look?

There are two good options: the first is a bit simpler to implement, the second is harder but has greater benefits.
We'll look at each in turn.

## Option one: DAG


This is a great shape for a code base. In programming parlance, we might simply say, 'there are no circular
dependencies'.

How would you apply dependency inversion to get your code base in this shape?

First, you need to decide the level at which you wish this shape to apply. For example, you may wish to have a DAG at the
top level, but not mind too much if there are circular dependencies within each component. Or you may also want
certain components to have their own DAGS inside. You don't need to apply it everywhere in order
to get the benefits, just apply it to the parts of your system that are in danger of getting overcomplicated.

Next, decide on the order which the dependencies should flow. This could be simply an ordered list of each component.
Items at the top of the list may depend upon items lower down, but not the other way around.
For example, if you want to structure your system as a traditional
[three-tier architecture](https://en.wikipedia.org/wiki/Multitier_architecture#Three-tier_architecture),
your list might be:

1. User interface component
2. Business logic component
3. Data component

This list could be a lot longer, particularly if your top level structure is feature-based instead. You could also
have some items at the same level, in which case they should be independent of each other.

Once you have a list, it's time to identify the dependencies in your system which break this structure. This could be
a manual process, or depending on the language you're using there may be static analysis tools to help.

Now you can use dependency inversion to invert the dependencies and gradually reduce the cycles in your
dependency graph.

It's important that everyone working on the code base is signed up this approach, otherwise cycles will inevitably
creep in again. One way of stopping that is to add automated checks to prevent this. (For example, Python developers
can use [Import Linter](https://import-linter.readthedocs.io)).

## Option two: Hexagonal Architecture

If you want to take dependency inversion further, you should consider a style of organizing your code base known,
among other things, as *Hexagonal Architecture*. Broadly speaking, this style is based on the last of the five
[SOLID principles](https://en.wikipedia.org/wiki/SOLID): 'the Dependency Inversion Principle'. This can be summarized
as:

<div class='rule'>
    Details should depend on abstractions.
</div>

Truly counterintuitive. We're used to higher level code depending on lower level code. This turns it on its head and
says that lower level code (the details) should depend on the high level code (policies, business logic etc.)

Hexagonal Architecture puts high level code upstream of the lower level code, and attempts to decouple things as much
as possible. Ports and adaptors.

(dependency graph with business logic plugging into ports, then application, then adaptors)

The advantage of this approach is that your business logic is isolated from implementation details. This means you
can test your most critical code quickly with unit tests by using in-memory adaptors when you test.

A second benefit is that it encourages you to focus on your conceptual models, without getting too dragged into
implementation details. This is particularly good if you are working within a domain where the business logic itself
is complex.

Notice still a DAG, but one with a more specific structure.


# Conclusion: complex is better than complicated

When structuring projects, it's not enough just to separate concerns: you must control the dependency flow, otherwise
things will get messy and difficult to work with. Whichever shape you choose for your system's dependency graph,
you will sometimes need to make the dependency flow in the opposite direction to what comes naturally.
That is dependency inversion.

If there is a real world concept in which the dependency works in one direction, it can feel strange to invert it.

The [Zen of Python](https://en.wikipedia.org/wiki/Zen_of_Python) states:

    Simple is better than complex.

But also that

    Complex is better than complicated.

I think of dependency inversion as an example of choosing the complex over the complicated. If we don't use it when
it's needed, our efforts to create a simple system will tangle into complications. Inverting dependencies allows us
to trade a little simplicity, but allowing us to avoid a system that is complicated to work with.