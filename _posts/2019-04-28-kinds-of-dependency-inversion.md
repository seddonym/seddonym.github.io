---
layout: post
title: Three and a half methods of dependency inversion 
description: >
    Dependency inversion can sound complicated or even impossible, but there are just a few
    simple approaches. 
image: upside-down.jpg
featured: true
weight: 1
tags: [architecture, factoring, dependency-inversion]
---
In [What is dependency inversion]({% link _posts/2019-04-15-why-dependency-inversion.md %}), I describe the important
technique of avoiding circular dependencies using dependency inversion - but I didn't say how to do it. It's actually
not that complicated. In this post I'll take you through three (and a half) approaches that you can choose between.

Here, then, are the different methods, and when you might want to use them:

1. **Configuration**: When you want the dependency to be declared in a single place, and applies globally throughout
   your application.
2. **Registry**: When you want multiple dependencies to be declared in different places, scattered across your application.
3. **Dependency injection**: When you want the dependency to vary based on when the code is called.
3½. **Monkey patching**: When you want, or need, to hack.

I code mainly in Python, which is a very dynamic language that gives you a lot of flexibility. I've tried to describe
these approaches in such a way that they can be applied in different languages, though some techniques may not be
possible in every language. Throughout this post I'm going to be using the term **component**, by which I mean simply
a part of some software system. Each method is focused on changing the direction of dependency between two or more
such components.

I'm also going to refer to **interfaces**, by which I mean not any specific language construct, but just the API used
between two components. The best example of this is from the object oriented paradigm, where an interface
defines a set of methods without implementing any behaviour. Classes that implement this interface must then supply
the methods described by that interface.

(Consider just talking about a global registry vs custom registry).

# 1. Configuration

**Configuration** is the most method I use most often. It involves the following steps:making it possible to configure a component from
  outside itself.

(pic of A -> B)

Let's say component *A* depends on *B* for something, but we want to invert this using configuration. We can apply
the following steps:

1. First, we need a 'configuration api'. This is just some kind of key-value store that is populated at the top level
of the application, and that components inside the application can read from. If you're already working within a
framework, chances are you probably already have this.
2. Next, define a new component *I*, which provides the same interface as *A* uses to interact with *B*. This new component
 can either be a subcomponent of *A*, or sit upstream of it.
3. Then make *A* use the configuration api to get the concrete implementation of *I* it should use,
instead of directly using *B*.
4. Finally, configure *B* to be the concrete implementation of *I*.
 
This diagram shows the complete picture. A no longer knows about B but does know about the interface it uses. B implements
that interface (and so knows about A). The application's configuration framework wraps the whole thing - a config api
that components know about and the actual configuration, downstream of everything else.

(pic of configuration -> B -> A(I) -> config api) 
 
This is easier to understand by example. Let's say 
 
 TODO example

# 2. Registry

Using a registry is 

# 3. Dependency injection

# 3½. Monkey patching

# Conclusion 
