---
layout: post
title: Techniques for Dependency Inversion in Python
description: >
    In <a href="/2019/04/15/why-dependency-inversion/">my previous post</a> we learned how
    Dependency Inversion can help modularise code. But how do we do this in practice?
image: di-techniques.jpg
featured: true
weight: 1
tags: [python, architecture, factoring, dependency-inversion]
---

Dependency Inversion is less complicated than it sounds. In Python, you may use three different techniques:

1. *Dependency Injection.*
2. *Registry* - which comes in two forms: *Configuration Registry* and *Subscriber Registry*.
3. *Monkey Patching.*

{% include tips/open.html %}    
  <p>If you haven't read <a href="{% link _posts/2019-04-15-why-dependency-inversion.md %}">my previous post on dependency inversion</a>,
  you might want to first.</p>
{% include tips/close.html %}

## The Dependency Inversion Principle

The first two techniques follow an important software design principle known as
**[The Dependency Inversion Principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle)**, which states
that "details should depend upon abstractions". What does this mean?

### Details and abstractions

Consider these classes:

{% highlight python %}
class Animal:
    def speak(self):
        raise NotImplementedError


class Cat(Animal):
    def speak(self):
        print("Meow.")


class Dog(Animal):
    def speak(self):
        print("Woof.")
{% endhighlight %}

``Animal`` (the abstraction) is inherited by ``Cat`` and ``Dog`` (the details). ``Animal.speak`` is declared,
but not implemented. The two subclasses implement the ``speak`` method, each in their own way.

This relationship of classes is often drawn like this:

{% include content_illustration.html image="di-techniques/animal-cat-dog.png" alt="Diagram of Cat and Dog subclassing Animal" %}

Because the details implement a shared interface, we can interact with either class without knowing which one it is:

{% highlight python %}
def make_animal_speak(animal):
    animal.speak()

    
make_animal_speak(Cat())
make_animal_speak(Dog())
{% endhighlight %}

The ``make_animal_speak`` function need not know anything about cats or dogs; all it has to know is that
it's an animal, and so it understands how to interact with it. (Interacting with objects without knowing
their specific type, only their interface, is known as *polymorphism*.)

Of course, in Python we don't actually *need* the base class:

{% highlight python %}
class Cat:
    def speak(self):
        print("Meow.")


class Dog:
    def speak(self):
        print("Woof.")
{% endhighlight %}

Even if ``Cat`` and ``Dog`` don't inherit ``Animal``, they can still be passed to ``make_animal_speak`` and things
will work just fine. (This informal ability to interact with an object without it explicitly declaring an interface
is known as *duck typing*.)

We aren't limited to classes. You can use functions:

{% highlight python %}
def notify_by_email(customer, event):
    ...

def notify_by_text_message(customer, event):
    ...
    
for notify in (notify_by_email, notify_by_text_message):
    notify(customer, event)
{% endhighlight %}

You can even use Python modules:

{% highlight python %}
import email
import text_message

for notification_method in (email, text_message):
    notification_method.notify(customer, event)
{% endhighlight %}

However the shared interface is manifested - be it in a formal, object oriented manner, or more implicitly, we can
generalise the separation between the interface and the implementation as follows:

{% include content_illustration.html image="di-techniques/interface-implementation.png" alt="Diagram of implementation inheriting abstract interface" %}

## Hoisting the implementation

Separating the interface from the implementation will allow you to apply the Dependency Inversion Principle. Take the
following two packages. ``A`` depends on ``B``, but we don't want it to.

{% include content_illustration.html image="di-techniques/A-B.png" alt="A pointing to B" %}

Our first step is to identify the interface that ``B`` presents to ``A``. This, we leave in place as a dependency of ``A``.
However, we move ``B``'s implementation of that interface up, so ``A`` no longer depends on it. Finally, we need some
orchestration code that knows about ``A`` and ``B``, and does the final linking of them together.

This can be drawn like this:

{% include content_illustration.html image="di-techniques/di-pattern.png" alt="main pointing to A and B, A pointing to <<B>>, B pointing (open arrow) to <<B>>" %}

If this feels somewhat abstract, fear not! The techniques below will show you how to do it with real code.

## Technique One: Dependency Injection

Dependency Injection is where a piece of code allows the calling code to control its dependencies.
The simplest way to implement this is as a function argument.

Take the following function:

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

{% include content_illustration.html image="di-techniques/main-hw-print.png" alt="Main pointing to hello_world pointing to print" %}

We'll be breaking the dependency of ``hello_world`` on ``print``, making it possible to swap in an
alternative function, ``print_twice``, which shares the same interface.

{% highlight python %}
# print_twice.py

def print_twice(text):
    print(text)
    print(text)
{% endhighlight %}

Let's adjust ``hello_world`` so it supports dependency injection. All we do is allow it to receive the
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

{% include content_illustration.html image="di-techniques/main-hw-print-output.png" alt="Main pointing to hello_world and print, hello_world pointing to <<output>>, print pointing (open arrow) to <<output>>." %}

## Technique Two: Registry

A registry is a store that one piece of code reads from to decide how to behave, which may be
written to by other parts of the system. Registries require a bit more machinery that dependency injection. 

They take two forms: *Configuration* and *Subscriber*:

### The Configuration Registry

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

As with dependency injection, the dependency has been shifted to the outskirts of the system.

{% include content_illustration.html image="di-techniques/configuration-registry.png" alt="Configuration registry" %}

In a real world system, we might want a slightly more sophisticated config system
(making it immutable for example, is a good idea). But at heart, any key-value store
will do.

### The Subscriber Registry

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
 
{% include content_illustration.html image="di-techniques/subscriber-registry.png" alt="Subscriber registry" %}


#### Subscribing to events

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

## Technique Three: Monkey Patching

Monkey patching is the technique of dynamically manipulating code that is called elsewhere. This is usually unwise,
but I'm including it for completeness.

If our ``hello_world`` function doesn't implement any hooks for injecting its output function, we can monkey patch the
built in ``print`` function with something different:

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

## Choosing a technique

Given these three techniques, which should you choose, and when?

### When to use monkey patching

Code that abuses the Python's dynamic power can be extremely
difficult to understand or maintain. The problem is that if you are reading monkey patched code, you have no way of
knowing that it is being manipulated elsewhere to do something differently.

Monkey patching should be reserved for desperate times, where you don't have the ability to change the code you're
patching, and it's really, truly impractical to do anything else.

Instead of monkey patching, it's much better to use one of the other dependency inversion techniques.
These expose an API that formally provides the hooks that other code can use to change behaviour, which is easier
to reason about and predict.

A legitimate exception is in tests, where you can make use of ``unittest.mock.patch``. This is monkey patching, but it's
a pragmatic way to manipulate dependencies when testing code. Even then, some people view testing like this as
a code smell.

### When to use dependency injection

If your dependencies change at runtime, you'll need dependency injection. Its alternative, the registry,
is best kept immutable. You don't want to be changing what's in a registry, except at application start up.

[``json.dumps``](https://docs.python.org/3/library/json.html) is a good example from the standard library which uses
dependency injection. It serializes a Python object to a JSON string, but if the default encoding doesn't support what
you're trying to serialize, it allows you to pass in a custom encoder class.

Even if you don't need dependencies to change, dependency injection is a good technique if you want a really simple way
of overriding dependencies, and don't want the extra machinery of configuration.

However, if you are having to inject the same dependency a lot, you might find your code becomes rather unwieldy and
repetitive. This can also happen if you only need the dependency quite deep in the call stack, and are having to pass
it around a lot of functions.

### When to use registries

Registries are a good choice if the dependency can be fixed at start up time. While you may always use dependency injection
instead, the registry is a good way to keep configuration separate from the control flow code. If there is already a
configuration system in place (e.g. if you're using a framework that has a way of providing global configuration) then
there's very little extra machinery to set up.

Use a configuration registry when you need something configured to a single value. A good example of this is Django's ORM,
which provides a Python API around different database engines. The ORM does not depend on any one database engine; instead,
you [configure your project to use a particular database engine](https://docs.djangoproject.com/en/2.2/ref/settings/#databases)
via Django's configuration system.  
 
Use a subscriber registry for an arbitrary number of values. Another example from Django is
[the admin site](https://docs.djangoproject.com/en/2.2/ref/contrib/admin/), which allows you to register different
database tables with it, exposing a CRUD interface in the UI. As mentioned before, subscriber registries are also good
for pub/sub.

Configuration registries can be used in place of subscriber registries for configuring,
say, a list - if you prefer linking things up in a configuration file, rather than scattered throughout the application.

## Conclusion

I hope these examples, which were as simple as I could think of, have shown how easy it is to invert or manipulate dependencies.
While it's not always the most obvious way to structure things, it can be achieved with very little extra code.

In the real world, you may prefer to employ these techniques with a bit more structure. I often choose classes rather
than functions as the swappable dependencies, as they allow you to declare the interface (i.e. the methods that may be
called on the object) in a more formal way. Dependency injection, too, has more sophisticated implementations, and
there are even some third party frameworks available.

Whichever approaches you take, the important thing to remember is that the structure of dependencies in a software package is
crucial to how easy it will be to understand and change. Following the path of least resistance can result in dependencies
 being structured in ways that are, in fact, harder to work with. These techniques give you the power to make
decisions about this structure. Use them wisely!