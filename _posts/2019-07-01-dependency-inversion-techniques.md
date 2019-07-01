---
layout: post
title: Techniques for Dependency Inversion in Python
description: >
    In my previous post we learned how Dependency Inversion can help modularise
    code. But how do we do this in practice? Here are three distinct techniques
    you can use in Python.
image: upside-down.jpg
featured: true
weight: 1
tags: [python, architecture, factoring, dependency-inversion]
---

The three techniques are:

1. *Dependency Injection.*
2. *Registry.* This comes in two forms, *Configuration Registry* and *Subscriber Registry*.
3. *Monkey Patching.*

{% include tips/open.html %}    
  <p>If you haven't read <a href="">my previous post</a>, you might want to first.</p>
{% include tips/close.html %}

# A word on interfaces

Before we look at these techniques, it's important to consider the importance of *interfaces*.

An interface is a defined way of interacting with a piece of code. In Python, the best example is a function
signature: when we read the signature, we know what arguments we can pass in. There are informal means of defining
interfaces too. You may create a class that doesn't have all its methods implemented, which you expect client code
to subclass and implement those methods. You could even define an interface at the module level.
The important thing is that the way in some code is expected to interact with some other code is clearly defined.

When you invert dependencies, a defined interface is crucial. It's what enables the code to know how to interact
with something *without knowing specifically what it is*. In the examples that follow we're choosing a simple function
as a swappable dependency. It's a function that takes a single argument, which it will print.
At first, we can use the builtin ``print`` function to do this, as it supports that interface.

The function we'll be swapping is called ``print_twice``:

{% highlight python %}
# print_twice.py

def print_twice(text):
    print(text)
    print(text)
{% endhighlight %}

This function also supports the same interface, which is why we can swap it in.

# Technique One: Dependency Injection

Dependency Injection is a way of allowing the calling code to control the dependencies of the code it calls.
The simplest way to implement this is as a function argument.

Let's look at how we might implement this on the most trivial of functions:

{% highlight python %}
def hello_world():
    print("Hello, world.")
{% endhighlight %}

The ``hello_world`` function can be adjusted to receive its output function as an argument: 

{% highlight python %}
def hello_world(output_function):
    output_function("Hello, world.")
{% endhighlight %}

The calling code could then look like this:

{% highlight python %}
# main.py
import hello_world

if __name__ == "__main__":
    hello_world.hello_world(output_function=print)
{% endhighlight %}

Or, if it wanted to print everything twice, it could inject a different output function:

{% highlight python %}
# main.py
import hello_world


def print_twice(text):
    print(text)
    print(text)


if __name__ == "__main__":
    hello_world.hello_world(output_function=print_twice)
{% endhighlight %}

The parameter passed in doesn't need to be a callable - it could be a class, an instance, even a module.

# Technique Two: Registry

Registries involve slightly more machinery. The basic idea of a registry
is a store that one component reads from to decide how to behave, and which can be
written to by other components.

Registries take two forms: *Configuration* and *Subscriber*.

## The Configuration Registry

A configuration registry gets populated once, and only once. A component uses one
to allow its behaviour to be configured from outside.

Let's say that instead of calling ``print`` directly, we want to make it possible
to configure which function is called to output the message. We could do it like this:

{% highlight python %}
config = {}

def hello_world():
    output_function = config["OUTPUT_FUNCTION"]
    output_function("Hello, world.")
{% endhighlight %}

To complete the picture, here's how it could be configured externally:

{% highlight python %}
# main.py
import hello_world

hello_world.config["OUTPUT_FUNCTION"] = print

if __name__ == "__main__":
    hello_world.hello_world()

{% endhighlight %}

We could draw this system as follows:

(Dep graph with main pointing to print and hello world separately)

In a real world system, we might want a slightly more sophisticated config system
(making it immutable for example, is a good idea). But at heart, any key-value store
will do.

## The Subscriber Registry

In contrast to a configuration registry, which should only be populated once, a
subscriber registry may be populated an arbitrary number of times by different parts
of the system.

A component exposes a mechanism for adding some configuration to the registry. This time,
instead of being a key-value store, it's a list. This list is populated, typically
 at startup, by other components scattered throughout the system. When the time is right,
 the component works through each item one by one.
 
Let's develop our ultra-trivial example to use this pattern. Instead of saying "Hello, world", we want
to greet an arbitrary number of people. Other parts of the system should be able to add people to the list of
those we should greet.

{% highlight python %}
# hello_people.py

people = []

def hello_people():
    for person in people:
        print(f"Hello, {person}.")

# john.py
import hello_people

hello_people.people.append("John")

# martha.py
import hello_people

hello_people.people.append("Martha")
{% endhighlight %}

A diagram of this system would be:
 
(Pic of john and martha both pointing to hello people)

### Subscribing to events

The classic example of this is the [Observer Pattern](https://sourcemaking.com/design_patterns/observer),
a.k.a. publish-subscribe, a.k.a. pub/sub. This can be implemented in much the same way as above, except
instead of adding strings to a list, we add callables:
 
{% highlight python %}
# hello_world.py
subscribers = []

def hello_world():
    print("Hello, world.")
    for subscriber in subscribers:
        subscriber()

# log.py
import hello_people

def write_to_log():
    ...
   
hello_people.subscribers.append(write_to_log)

{% endhighlight %}

# Technique Three: Monkey Patching

Monkey patching is the technique of dynamically manipulating code that is called elsewhere. This is usually a
horrible thing to do, but I'm including it for completeness.

If our ``hello_world`` function didn't implement any hooks for injecting its output function, we
could instead use that library to patch in our own:

{% highlight python %}
def hello_world():
    print("Hello, world.")
{% endhighlight %}

The calling code could then monkey patch the built in ``print`` function with our own, custom defined one:

{% highlight python %}
# main.py
import hello_world

def print_twice(text):
    print(text)
    print(text)

hello_world.print = print_twice

if __name__ == "__main__":
    hello_world.hello_world()
{% endhighlight %}

Monkey patching can take other forms, too. There may be a class that is defined in one part of the code base
that you can then manipulate to your heart's content elsewhere - changing attributes, swapping in other methods,
etc.

## Why monkey patch?

Usually you shouldn't monkey patch. Code that abuses the Python's dynamic power can be extremely difficult to understand
or maintain. The problem is, that if you read the code in ``hello_world`` you have no way of knowing that code elsewhere
is manipulating it to do something differently.

Instead of monkey patching, it's much, much better to use one of the other dependency inversion techniques.
These expose an API that formally provides the hooks that other code can use to change behaviour, which is easier
to reason about and predict.

Monkey patching should be reserved for desperate times, where you don't have the ability to change the code you're
patching, and it's really, truly impractical to do anything else. Even then, you should try to find some other way.

A legitimate exception is in tests, where you can make use of ``unittest.mock.patch``. This is monkey patching but it's
a pragmatic way to manipulate dependencies when testing code. (Personally, I think patching is a bit of a
code smell, even in tests - though practically speaking it's often more practical to do it than to refactor
the code under test.)  

# Choosing a technique

Registries are a good way of inverting dependencies when...

Typically, a configuration registry is good for things which only need to be configured once

while a subscriber registry is good for y. There are also situations where you can choose either.

TODO

This approach is somewhat similar to dependency injection, but with two key differences:

1. The machinery is simpler (you don't need to implement a configuration system).
2. The dependency is injected at run time, rather than at start up. What this means is that you could have
two different parts of the system that call ``hello_world`` with a different ``output_function``. A configuration
registry doesn't allow that.