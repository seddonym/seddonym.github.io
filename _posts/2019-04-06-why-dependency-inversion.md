---
layout: post
title: What is dependency inversion and why does it matter?
description: "Dependency inversion is a powerful programming technique. Find out what it is, and why it is important "
    "to most software projects."
image: upside-down.jpg
featured: true
weight: 1
tags: [architecture, factoring, dependency-inversion]
---

Dependency inversion is the practice of reversing the direction of a dependency between two different parts of a
software system.

A simple example: let's say there is a function that is declared in a part of a system named **A**. If some code in **B**
(another part of the system) calls that function, we can say that **B** depends on **A**. We can draw it as an arrow
from **B** to **A**: the arrow points to the thing that is being depended on:

(diagram of A <- B)

If, for some reason, changed the direction of this arrow, so that **A** depends on **B** instead, then we could say that
we have 'inverted the dependency'.

(A -> B)

That, in its simplest form, is dependency inversion. It's one of the most useful tools in a programmer's belt, but
we often neglect it. To those unfamiliar with the idea, it may seem a bizarre, or even impossible thing to do. In this
post, I'll try to convince you of its importance.

# Striving for modularity

The complexity of a software system can easily get out of hand. Every programmer has experienced code
that has become tangled and difficult to work with. Systems like this are hard to think about:

(big system with complicated in middle)

Perhaps the most common approach to tackling this is to break up the system into smaller, more manageable parts.
By separating the system into simpler subsystems, the aim is to reduce complexity and allow us to think more clearly
about each one. We call this quality of a system its *modularity*, and we can refer to these subsystems as *modules*.

(smaller modules with simple in middle)

# Separation of concerns

We all recognise the value of splitting systems up in this way. And how do we choose to split things? By the
*separation of concerns*.

This separation can take different forms. We might organize things by feature area
(the authentication system, the shopping cart, the blog) or by level of detail
(the user interface, the business logic, the database), or both.

When we do this, we tend to be aiming at modularity. Except for some reason, the system remains complicated.
In practice, working on one module turns out to relate to another part of the system,
which relates to another, which relates back to the original one. Pretty soon your head hurts and you need to have
a lie down. What's going wrong?

The sad fact is, if the only organizing factor of a code base is separation of concerns, a code base will not be
modular after all. Instead, separate parts will tangle together, and it will become increasingly difficult to work with.

Pretty quickly, our efforts to organise what goes into each modules are undermined by *the relationships between those
modules*.

This is naturally what happens to software if you don't think about relationships. This is because in the real world
things *are* a messy, interconnected web of connections. It's natural that as we build functionality, we realise
that one module needs to know about another. Later on, that other module needs to know about the first.

(Diagram of modules with relationships, and complicated in them)

The problem with a system like this is that, because of the web of relationships, it is not a collection of smaller
subsystems. Instead, it is a single, large system - and the larger a system, the more difficult it is to understand and
work with.

# Improving modularity through decoupling

The crucial problem here is that the modules, while appearing separate, are *tightly coupled* by their dependencies
upon one other. Let's take two modules as an example:

(A <-> B)

In this diagram we see that **A** depends on **B**, but **B** also depends upon **A**. This is known as a
*circular dependency*. This circular dependency means that these two modules are in fact a single module. How can we
improve things?

It's time for a spot of dependency inversion. If we reverse one of the arrows, then we eliminate the circular dependency
and limit the flow of dependencies to a single direction:

(A <- B)

Now that **A** has no knowledge of **B**, we can work on **A** without needing to know about **B** either. We've just
reduced our mental overhead, and made the system more modular.

The technique remains useful for larger groups of modules. For example, three modules may depend upon each other, in
cycle:

(A <- B <- C -> A)

In this case, we can invert one of the dependencies, gaining us a single dependency flow:

(single flow)

Again, dependency inversion has come to the rescue, and helped us modularise our system.

# Dependency inversion in practice

In practice, inverting a dependency can sometimes feel impossible. Surely there is no way to reverse the direction of
the arrow merely by refactoring? But I have good news. It is never impossible. I promise. You can *always* avoid circular
dependencies by inverting them. It's not always the most obvious way to write code, but it can make your code base
significantly easier to work with.

There are several different techniques for *how* you reverse that arrow. One such technique is called
'dependency injection'. I will cover some of these techniques in part two of this series.

There is also more to be said about how to apply this approach across the wider code base: if the system consists of
more than a handful of files, where do we start? Again, I'll cover this later in the series.

# Conclusion: complex is better than complicated

When structuring projects, it's not enough just to separate concerns: you must control the relationships between those
concerns, otherwise things will get into a mess. In order to gain the benefits of a modular system,
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