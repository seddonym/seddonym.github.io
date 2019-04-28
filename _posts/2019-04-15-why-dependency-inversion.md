---
layout: post
title: What is dependency inversion and why does it matter?
description: >
    Dependency inversion is a powerful technique for improving the modularity of software. Without it,
    systems will naturally become tangled and difficult to understand.
image: upside-down.jpg
featured: true
weight: 1
tags: [architecture, factoring, dependency-inversion]
---

A simple example of dependency inversion: let's say there is a function that is declared in a part of a system named **A**. If some code in **B**
(another part of the system) calls that function, we say that **B** depends on **A**. We can draw this as an arrow
from **B** to **A**: the arrow points to the thing that is being depended on.

{% include content_illustration.html image="why-di/b_to_a.png" alt="An arrow pointing from B to A" %}

If, through refactoring, we reverse the direction of this arrow so that **A** depends on **B** instead, then we can say
that we have 'inverted the dependency'.

{% include content_illustration.html image="why-di/a_to_b.png" alt="An arrow pointing from A to B" %}

This, in its simplest form, is dependency inversion: reversing the direction of a dependency between two different parts
of a software system. It's one of the most useful tools in a programmer's belt, but
we often neglect it. To those unfamiliar with the idea, it may seem a bizarre or even impossible thing to do. But for
most systems, it's crucial if we want avoid our code getting into a mess.

{% include tips/open.html %}    
  <p>My definition of dependency inversion is a bit looser than the kind described by the
  well known design principle the
  <a href="https://en.wikipedia.org/wiki/Dependency_inversion_principle">Dependency Inversion Principle</a>,
  which is more prescriptive about the directions of dependencies. I'll write more about the DIP in a future
  blog post.</p>
{% include tips/close.html %}

# Striving for modularity

Software gets complicated easily. Every programmer has experienced tangled, difficult-to-work with code.
Here's a diagram of such a system:

{% include content_illustration.html image="why-di/big.png" alt="A single complicated system" %}

Perhaps not such a helpful diagram, but some systems can feel like this to work with: a forbidding mass
of code that feels impossible to wrap one's head around.

A common approach to tackling such complexity is to break up the system into smaller, more manageable parts.
By separating it into simpler subsystems, the aim is to reduce complexity and allow us to think more clearly
about each one in turn.

{% include content_illustration.html image="why-di/modular.png" alt="A system composed of small simple modules" %}

We call this quality of a system its *modularity*, and we can refer to these subsystems as *modules*.

# Separation of concerns

Most of us recognise the value of modularity, and put effort into organising our code into smaller parts. We have to
decide what goes into which part, and the way we do this is by the *separation of concerns*.

This separation can take different forms. We might organize things by feature area
(the authentication system, the shopping cart, the blog) or by level of detail
(the user interface, the business logic, the database), or both.

When we do this, we tend to be aiming at modularity. Except for some reason, the system remains complicated.
In practice, working on one module turns out to relate to another part of the system,
which relates to another, which relates back to the original one. Soon our heads hurt and we need to have
a lie down. What's going wrong?

## Separation of concerns is not enough

The sad fact is, if the only organizing factor of code is separation of concerns, a system will not be
modular after all. Instead, separate parts will tangle together.

Pretty quickly, our efforts to organise what goes into each module are undermined by the relationships between those
modules.

This is naturally what happens to software if you don't think about relationships. This is because in the real world
things *are* a messy, interconnected web. As we build functionality, we realise that one module needs to know about
another. Later on, that other module needs to know about the first. Soon, everything knows about everything else.

{% include content_illustration.html image="why-di/complicated-modular.png" alt="A complicated system with lots of arrows between the modules" %}


The problem with software like this is that, because of the web of relationships, it is not a collection of smaller
subsystems. Instead, it is a single, large system - and large systems tend to be more complicated than smaller ones.

# Improving modularity through decoupling

The crucial problem here is that the modules, while appearing separate, are *tightly coupled* by their dependencies
upon one other. Let's take two modules as an example:

{% include content_illustration.html image="why-di/a_b_cycle.png" alt="Arrows pointing in both directions between A and B" %}

In this diagram we see that **A** depends on **B**, but **B** also depends upon **A**. It's a
circular dependency. As a result, these two modules are in fact no less complicated than a single module.
How can we improve things?

It's time for a spot of dependency inversion. If we reverse one of the arrows, then we eliminate the circular dependency
and limit the flow of dependencies to a single direction:

{% include content_illustration.html image="why-di/a_b_acyclic.png" alt="Two arrows, both pointing from B to A" %}

Now that **A** has no knowledge of **B**, we think about **A** in isolation. We've just reduced our mental overhead,
and made the system more modular.

The technique remains useful for larger groups of modules. For example, three modules may depend upon each other, in
a cycle:

{% include content_illustration.html image="why-di/abc_cycle.png" alt="Arrows pointing from A to B to C, and back to A" %}

In this case, we can invert one of the dependencies, gaining us a single direction of flow:

{% include content_illustration.html image="why-di/abc_acyclic.png" alt="Arrows pointing from A to B, A to C and B to C" %}

Again, dependency inversion has come to the rescue.

# Dependency inversion in practice

In practice, inverting a dependency can sometimes feel impossible. Surely there is no way to reverse the direction of
the arrow merely by refactoring? But I have good news. As far as I know, it is never impossible.
You should *always* be able to avoid circular dependencies through some form of inversion. It's not always the most obvious way
to write code, but it can make your code base significantly easier to work with.

There are several different techniques for *how* you reverse that arrow. One such technique that is often
 talked about is dependency injection. I will cover some of these techniques in part two of this series.

There is also more to be said about how to apply this approach across the wider code base: if the system consists of
more than a handful of files, where do we start? Again, I'll cover this later in the series.

# Conclusion: complex is better than complicated

If you want to avoid your code getting into a mess, it's not enough merely to separate concerns. You must control the
relationships between those concerns. In order to gain the benefits of a more modular system,
you will sometimes need to use dependency inversion to make your dependencies flow in the opposite direction to what
comes naturally.

The [Zen of Python](https://en.wikipedia.org/wiki/Zen_of_Python) states:

    Simple is better than complex.

But also that

    Complex is better than complicated.

I think of dependency inversion as an example of choosing the complex over the complicated. If we don't use it when
it's needed, our efforts to create a simple system will tangle into complications. Inverting dependencies allows us,
at the cost of a small amount of complexity, to make our systems less complicated.