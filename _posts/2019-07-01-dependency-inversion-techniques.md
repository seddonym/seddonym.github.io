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

The three techniques we'll be looking at are:

1. *Dependency Injection.*
2. *Registry* - which comes in two forms: *Configuration Registry* and *Subscriber Registry*.
3. *Monkey Patching.*

{% include tips/open.html %}    
  <p>If you haven't read <a href="{% link _posts/2019-04-15-why-dependency-inversion.md %}">my previous post on dependency inversion</a>,
  you might want to first.</p>
{% include tips/close.html %}

# Hello, world

In the examples that follow, we'll be playing with a very simple function:

{% highlight python %}
# hello_world.py

def hello_world():
    print("Hello, world.")
{% endhighlight %}

This function is called from a top level function like so:

{% highlight python %}
# main.py

from hello_world import hello_world

if __name__ == "__main__":
    hello_world()
{% endhighlight %}

``hello_world`` has one dependency that is of interest to us: the built in function ``print``. We can draw a diagram
of these dependencies like this:

(Diagram of main -> hello world depending on print.)

We'll be breaking the dependency of ``hello_world`` on ``print``, making it possible to swap in an
alternative function, ``print_twice``:

{% highlight python %}
# print_twice.py

def print_twice(text):
    print(text)
    print(text)
{% endhighlight %}

# Technique One: Dependency Injection

Dependency Injection is where a piece of code allows the calling code to control its dependencies.
The simplest way to implement this is as a function argument.

This is trivial to achieve this in our example system: we adjust the ``hello_world`` function so it receives the
output function as an argument: 

{% highlight python %}
# hello_world.py

def hello_world(output_function):
    output_function("Hello, world.")
{% endhighlight %}

The top level code then passes in the ``print`` function via the argument:

{% highlight python %}
# main.py
import hello_world

if __name__ == "__main__":
    hello_world.hello_world(output_function=print)
{% endhighlight %}

Or, if it wanted to print everything twice, it could inject a different dependency:

{% highlight python %}
# main.py
from hello_world import hello_world
from print_twice import print_twice

if __name__ == "__main__":
    hello_world.hello_world(output_function=print_twice)
{% endhighlight %}

In this example, we're injecting a callable, but other implementations could expect a class, an instance or even a module.

With very little code, we have moved the dependency out of ``hello_world``, into the top level function:

(Diagram of main -> hello_world, main -> print)

# Technique Two: Registry

A registry is a store that one piece of code reads from to decide how to behave, which may be
written to by other parts of the system. Registries require a bit more machinery that dependency injection. 

They take two forms: *Configuration* and *Subscriber*:

## The Configuration Registry

A configuration registry gets populated once, and only once. A piece of code uses one
to allow its behaviour to be configured from outside.

A simple way to achieve this is to get the dependencies from a dictionary that is writeable to from outside the module:

{% highlight python %}
# hello_world.py


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

As with dependecy injection, the dependency has been shifted to the outskirts of the system.

(Same graph as above)

In a real world system, we might want a slightly more sophisticated config system
(making it immutable for example, is a good idea). But at heart, any key-value store
will do.

## The Subscriber Registry

In contrast to a configuration registry, which should only be populated once, a
subscriber registry may be populated an arbitrary number of times by different parts
of the system.

A piece of code exposes a mechanism for adding some configuration to the registry. This time,
instead of being a key-value store, it's a list. This list is populated, typically
 at startup, by other components scattered throughout the system. When the time is right,
 the code works through each item one by one.
 
Let's develop our ultra-trivial example to use this pattern. Instead of saying "Hello, world", we want
to greet an arbitrary number of people: "Hello, John.", "Hello, Martha.", etc. Other parts of the system should be able to add people to the list of
those we should greet.

{% highlight python %}
# hello_people.py

people = []

def hello_people():
    for person in people:
        print(f"Hello, {person}.")
{% endhighlight %}
{% highlight python %}
# john.py

import hello_people

hello_people.people.append("John")
{% endhighlight %}
{% highlight python %}
# martha.py

import hello_people

hello_people.people.append("Martha")
{% endhighlight %}

A diagram of this system would be:
 
(Pic of john and martha both pointing to hello people)

### Subscribing to events

A very common reason for using a subscriber registry is to allow other parts of a system to react to events
that happen one place, without that place directly calling them. This is often solved by the [Observer Pattern](https://sourcemaking.com/design_patterns/observer),
a.k.a. publish-subscribe, a.k.a. pub/sub. This can be implemented in much the same way as above, except
instead of adding strings to a list, we add callables:
 
{% highlight python %}
# hello_world.py

subscribers = []

def hello_world():
    print("Hello, world.")
    for subscriber in subscribers:
        subscriber()
{% endhighlight %}

{% highlight python %}
# log.py

import hello_world

def write_to_log():
    ...
   
hello_world.subscribers.append(write_to_log)
{% endhighlight %}

# Technique Three: Monkey Patching

Monkey patching is the technique of dynamically manipulating code that is called elsewhere. This is usually unwise,
but I'm including it for completeness.

If our ``hello_world`` function doesn't implement any hooks for injecting its output function, we can monkey patch the
built in ``print`` function with our own, custom defined one:

{% highlight python %}
# main.py

import hello_world 
from print_twice import print_twice

hello_world.print = print_twice

if __name__ == "__main__":
    hello_world.hello_world()
{% endhighlight %}

Monkey patching can take other forms, too. There may be a class that is defined in one part of the code base
that you can then manipulate to your heart's content elsewhere - changing attributes, swapping in other methods,
etc.

# Choosing a technique

Given these three techniques, which should you choose, and when?

## When to use monkey patching

Code that abuses the Python's dynamic power can be extremely
difficult to understand or maintain. The problem is, that if you are reading monkey patched code, you have no way of
knowing that it is being manipulated elsewhere to do something differently.

Monkey patching should be reserved for desperate times, where you don't have the ability to change the code you're
patching, and it's really, truly impractical to do anything else. Even then, you should try to find some other way.

Instead of monkey patching, it's much better to use one of the other dependency inversion techniques.
These expose an API that formally provides the hooks that other code can use to change behaviour, which is easier
to reason about and predict.

A legitimate exception is in tests, where you can make use of ``unittest.mock.patch``. This is monkey patching but it's
a pragmatic way to manipulate dependencies when testing code. (Personally, I think patching is a bit of a
code smell, even in tests - though practically speaking it's often more practical to do it than to refactor
the code under test.)  

## When to use dependency injection

If your dependencies need to change at runtime, you'll need dependency injection. Its alternative, the registry,
is best kept immutable. You don't want to be changing what's in a registry, except at application start up.

A good example of a Python library that uses dependency injection for this reason is the
[``json``](https://docs.python.org/3/library/json.html) module from the standard library.
``json.dumps``, which serializes a Python object to a JSON string, allows you to pass in a custom encoder class, if the
default encoding doesn't support what you're trying to serialize.

Even if you don't need dependencies to change, dependency injection is also a good technique if you want a really simple way
of overriding dependencies, and don't want the extra machinery of configuration.

However, if you are having to inject the same dependency a lot, you might find your code becomes rather unwieldy and
repetitive. This can also happen if you only need the dependency quite deep in the call stack, and are having to pass
it around a lot of functions.

## When to use registries

Registries are a good choice if the dependency can be fixed at start up time. While you may always use dependency injection
instead, the registry is a good way to keep configuration separate from the control flow code. If there is already a
configuration system in place (e.g. if you're using a framework that has a way of providing global configuration) then
there's very little extra machinery to set up.

Use a configuration registry when you need something configured to a single value; use a subscriber registry for an
arbitrary number of values. You may also use a configuration registry in place of a subscriber registry for configuring,
say, a list - if you prefer linking things up in a configuration file, rather than scattered throughout the application.

# Conclusion

I hope these examples have shown how easy it is to manipulate dependencies. The examples are
as simple as I could think of.

---
## A word on interfaces

Before we look at these techniques, let's reflect for a moment on the importance of *interfaces*.

In certain strictly typed object oriented languages such as Java, an interface is a way of defining a set of available
class methods without actually implementing them. Once you've defined an interface, you can *implement* that interface
on a variety of classes. You can then know that any class that implements that interface can be interacted with in
a particular way, using those defined methods.

In Python, we can be somewhat looser with our definition. We can approximate Java-style interfaces by creating classes
that have certain methods that just raise ``NotImplementedError``. However, you can also think of any function as having
an interface. Although there is no formal way to declare it, if two functions have the same signatures, then they share
an interface. You can even think of modules as sharing interfaces with other modules, if they contain functions with
the same names and signatures.

When you invert dependencies, a defined interface is crucial. It's what enables the code to know how to interact
with something *without knowing specifically what it is*. (In Python, this is sometimes known as 'duck typing'.)

In the examples that follow we'll explore this c:

{% highlight python %}
def hello_world():
    print("Hello, world.")
{% endhighlight %}

``hello_world`` has one dependency that we're going to invert: the built in ``print`` function. In this context,
``print``'s interface is 

a simple function
as a swappable dependency. It's a function that takes a single argument, which it will print.
At first, we can use the builtin ``print`` function to do this, as it supports that interface.

The function we'll be swapping is called ``print_twice``:

{% highlight python %}
# print_twice.py

def print_twice(text):
    print(text)
    print(text)
{% endhighlight %}

This function provides the same interface, which is why we can swap it in.