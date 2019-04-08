---
layout: post
title: Python and Dependency Inversion
description: "Dependency Inversion is a powerful programming technique. Find out what it is, and when you should apply it in your Python projects."
image: upside-down.jpg
featured: true
weight: 5
tags: [python, architecture, factoring]
---

Most programmers recognise the value of an organised structure in all but the smallest of software projects. In Python,
we can easily split code across modules and organise these modules into nested subpackages. It's a simple, powerful way
of structuring source code.

And how do we choose to split things? By the *separation of concerns*. This separation can take different forms. We
might organize things by feature area (the authentication system, the shopping cart, the blog) or by level of detail
(the user interface, the business logic, the database), or both.

When we do this, we're aiming at modularity. We know that systems can get complicated to work with, so splitting up
like this makes it easier to work on smaller bits of the system in isolation.

Except for some reason it doesn't. In practice, working on one module turns out to relate to another part of the system,
which relates to another, which relates back to the original module. Pretty soon your head hurts and you need to have
a lie down.

## Separation of Concerns is not enough

The sad fact is, if the only organizing factor of a code base is separation of concerns, a code base it will end up
looking like this:

(diagram of web of dependencies)

This is a dependency graph of a Python project. The arrows depict dependencies between modules. These dependencies
are so easy to create in Python. Need something from another part of the system? No problem, a simple `import`
statement makes it available.

Pretty quickly, our efforts to organise what goes into each module are undermined by *the relationships between those
modules*.

The problem with a system like this is that, because of the web of dependencies, it is not a collection of smaller
subsystems. It is not, after all, modularized. Instead, it is a single, large system - and the larger a system, the
more difficult it is to understand and work with.

This is naturally what happens to software if you don't think about dependencies. This is because in the real world
things *are* a messy, interconnected web of connections. It's natural that as we build functionality, we realise
that one module needs to know about another. Later on, that other module needs to know about the first, and thus a
circular dependency is born.

## Enter dependency inversion

Circular dependencies are the enemy of modularity. With them, our subsystems are coupled in both directions, which
means they aren't separate subsystems at all. We need to avoid letting them creep into our code bases. But how?

Take two modules, **A** and **B**, and let's say that **A** depends on **B**. We can draw this as an arrow from
**A** to **B**.

(diagram of A -> B)

Now, let's say we realise that **B** needs something from **A**. We begin to implement it, but then, alas! We realise
we are about to introduce a circular dependency:

(diagram of A <-> B)

We now have two simple choices. We just have to pick an arrow, and then refactor the code so that the direction of the
 arrow is reversed. We have removed the circular dependency.

(diagram of A -> B, two arrows)

This reorganization the code is dependency inversion. 

In practice, this can sometimes feel impossible. Surely there is no way to reverse the direction of the arrow
merely by refactoring? But I have good news. It is never impossible. I promise. You can *always* avoid circular
dependencies by inverting them. It's not always the most obvious way to write code, but it can make your code base
significantly easier to work with.

There are several different techniques for *how* you reverse that arrow. One such technique is called
'dependency injection'. I will cover some of these techniques in a future blog post, but for now I'll focus on the
 bigger picture.

---

## How not to avoid circular dependencies

(inline imports example)
t's tempting to view a circular dependency as annoying niggles that need to be worked around. The classic hack
is to move the import statement within the body of the function/method that is triggering

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

**DAG** stands for [Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph). This is a
mathematical structure consisting of points, and arrows connecting those points, where there are no cycles. In other
words, you could not start at one point and follow the arrows around until you returned to that same point. Here's a
picture of one:

(diagram of DAG)

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