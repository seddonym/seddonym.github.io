---
layout: post
title: What is dependency inversion?
description: "Dependency Inversion is a powerful but underused programming technique. Find out what what it is, and when you should use it."
image: upside-down.jpg
featured: true
weight: 5
tags: [architecture, factoring]
---
Take two components within a software system. Let's call them **A** and **B**,
and let's say that **A** depends on **B**. We can draw this as an arrow from **A** to **B**.

(diagram of A -> B)

Dependency inversion is simply reorganizing the code so that **B** depends on **A** instead:

(diagram of A <- B)

I should probably define my terms. By *component*, I just mean any part of a software system,
at any level of detail: a function, a class, a source file, a collection of source files, or indeed an entire
runnable service.

When I say **A** *depends on* **B**, I mean that *A* could be broken if *B* was changed in some way.

There are several different patterns for *how* you reverse that arrow. One such pattern is dependency injection.
I will cover some of these patterns in a future blog post, but for now I'll focus on why you would want to invert
dependencies, and how to decide in which direction the arrows should point.

# Why dependency inversion is important

Most programmers recognise the need to structure their code bases by separating concerns. Rather than have one enormous
file with code in, we will split things apart. We might organize things by feature area (the authentication system,
the shopping cart, the blog) or by level of detail (the user interface, the business logic, the database), or both.
But if the only organizing factor of a code base is separation of concerns, it will inevitably end up looking like this:

(diagram of web of dependencies)

The problem with a system like this is that, because of the web of dependencies, it is not a collection of smaller
subsystems. It is a single, large system - and the larger a system, the more difficult it is to understand. And systems
that are difficult to understand are difficult to work with.

This is naturally what happens to software if you don't think about dependencies. This is because in the real world
things *are* a messy, interconnected web of connections. It's natural that as we build functionality, we realise
that one component needs to know about another component. Later on, that other component needs to know about the first
component, and thus a circular dependency is born.

There is only one way to fight this natural tendency for things to get into a mess: dependency inversion. Once you
realise that it is possible to invert the direction of *any* dependency, the web of
connections can be tamed.


# Avoiding complications

Perhaps the reason why dependency inversion is underused is because it is not usually the obvious thing to do. If there
is a real world concept in which the dependency works in one direction, it can feel strange to invert it.

The [Zen of Python](https://en.wikipedia.org/wiki/Zen_of_Python) states:

    Simple is better than complex.

But also that

    Complex is better than complicated.

I think of dependency inversion as an example of choosing the complex over the complicated. If we don't use it when
it's needed, our efforts to create a simple system will tangle into complications. Inverting dependencies allows us
to trade a little simplicity, but allowing us to avoid a system that is complicated to work with.

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

