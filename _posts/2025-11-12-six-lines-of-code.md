---
layout: post
title: Six lines of code to prevent Python spaghetti 
description: "Python projects have a habit of getting tangled. The latest Import Linter contract type can help."
image: no-cycles.jpg
featured: true
weight: 1
tags: [python, architecture]
---

This week I released the latest version of [Import Linter](https://import-linter.readthedocs.io/). It includes my favourite
feature in years: the [`acyclic_siblings` contract type](https://import-linter.readthedocs.io/en/stable/contract_types.html#acyclic-siblings).
Unlike the other contract types, which require design thought to get up and running, this contract is something you can drop
into a new code base and it'll stop you in your tracks at the first sign of spaghetti code.

Here's a sneak preview of what those six lines of code might look like:[^1]

```text
[importlinter]
root_package = mypackage

[importlinter:contract:my-contract]
name = No cycles in mypackage
type = acyclic_siblings
ancestors = mypackage
```

## What the contract checks for

So what does the `acyclic_siblings` contract actually do?

It begins by looking at the `ancestor` specified in the contract (in this case `mypackage`) and checks no dependency cycles
exist between its children. It then drills down to each child and performs the same check for its children, and so on,
down the generations.

This is much easier to explain with a diagram. Take this Python codebase: the boxes are modules
(nested within other modules) and the arrows are imports between them.

{% include content_illustration.html image="six-lines-of-code/acyclic-siblings.svg" alt="A Python codebase with arrows drawn between modules" %}

Now if you look carefully, there are actually no dependency cycles between individual Python files. Nowhere is there
a chain of imports that eventually leads back to where it started. Indeed, when such a cycle exists, Python
often raises an exception when we try to import a module featured in the cycle. You may have seen this kind of
thing before:

```text
Traceback (most recent call last):
  File ".../blue.py", line 1, in <module>
    from mypackage import green
  File ".../green.py", line 1, in <module>
    from mypackage import yellow
  File ".../yellow.py", line 1, in <module>
    from mypackage import blue
  File ".../blue.py", line 6, in <module>
    def foo(b: green.SomeType):
                 ^^^^^^^^^
AttributeError: partially initialized module 'mypackage.green' has no attribute 'SomeType'
(most likely due to a circular import)
```

This kind of circular import is particularly disruptive, and as a result they tend to be a lot rarer. But what we care
about here is a much more common kind of circular dependency, which we get if we consider imports between a
subpackages _and their descendants_ (i.e. every module inside them). Defined in this way, it should be easy to see in
the diagram that a cycle exists between `mypackage`'s children: `blue` depends on `green`, which depends on `red`,
which depends on `yellow`, which depends on `blue`. It's this cycle that the contract will complain about.

This isn't the only dependency cycle, though. As I mentioned, the linter also drills down into deeper generations and
performs the same check there. In this case, there's also a dependency cycle between the children of `mypackage.blue`.
Have a look at the diagram and confirm this for yourself.

So if we run Import Linter on this code base we will get two errors: one for the children of `mypackage`, one for the
children of `mypackage.blue`:

```
----------------
Broken contracts
----------------

No cycles in mypackage
----------------------

No cycles are allowed in mypackage.
It could be made acyclic by removing 1 dependency:

- .yellow -> .blue (1 import)

No cycles are allowed in mypackage.blue.
It could be made acyclic by removing 1 dependency:

- .gamma -> .alpha (1 import)
```

It's possible to control this behaviour a little more (have a look at
[the docs](https://import-linter.readthedocs.io/en/stable/contract_types.html#acyclic-siblings) if you like), but those
are the basics.

## Why do I like this feature so much?

Dependency cycles are, for me, the ultimate enemy of maintainable code bases: their tangled strands it difficult to make
changes, or understand small parts in isolation.[^2]

A great way of tackling this is using _layering_. Each subpackage is assigned a position from top to bottom, and
packages lower down aren't allowed to import from those higher up. Here's an example contract from Import Linter itself:

```text
[importlinter:contract:layers]
name = Layered architecture
type = layers
exhaustive = true
containers = importlinter
layers =
    cli
    api
    contracts
    configuration
    adapters
    application
    domain
```

This prevents, for example, anything in the `importlinter.domain` package from importing anything in
`importlinter.application` (because it's above `domain` in the list). That `exhaustive = true` line ensures that
whenever a new subpackage of `importlinter` is added, it must explicitly be listed as a layer.

Not that there is anything wrong with explicitly identifying layers. I'd take a layered subpackage over merely an
acyclic one any day, because there are design insights in the way layers have been chosen.  

But where `acyclic_siblings` contracts shine is in their coverage. We can stick one in an empty Python package, right
at the beginning before tangling has had a chance to take root, and it'll keep everyone honest. That's why I also have
this contract, alongside the `layers` contract:

```text
[importlinter:contract:acyclic]
name = All packages are acyclic
type = acyclic_siblings
ancestors = importlinter
```

# Practical considerations

If you would like to start using this contract yourself, you might run into a couple of issues.

## Introducing a contract into an already tangled code base

Most of the time we don't get to start from scratch. Our code base is already tangled, and it's getting more tangled
every day. What can we do in this case?

Fortunately, Import Linter provides an escape hatch that allows you to burn down technical debt: the `ignore_imports`
contract option. If you're running into problems, you can specify problematic imports to be ignored, meaning that the
contract will pass with what you currently have and only fail if new problems are introduced. It looks something like
this:

```text
[importlinter:contract:acyclic]
name = All packages are acyclic
type = acyclic_siblings
ancestors = importlinter
ignore_imports =
    mypackage.blue.one -> mypackage.green.two
    mypackage.blue.one.alpha -> mypackage.blue.two.beta
```

You can then treat the list of ignored imports as a burndown, which you gradually work to pay off.

## Contract failure

It's all very well turning on a tool to forbid cycles -- but what if you need to introduce a change that
causes the contract to fail? Maybe you _do_ need a cycle after all...?

No, you don't. There's _always, always_ a way to reorganize code so a cycle is avoided. Different
approaches are appropriate at different times, but for more guidance take a look
at [Three Techniques for Inverting Control, in Python](http://127.0.0.1:4000/2019/08/03/ioc-techniques/).

# In summary

Python packages are best kept acyclic. That doesn't only apply to cycles between individual modules (the one Python
itself picks up on), but also between subpackages.

Now we have a way to enforce it. The `acyclic_siblings` contract allows us to impose this constraint with
minimal effort.

Introducing such a contract to an existing codebase is, I concede, likely to be a bit of work. But if you're serious
about getting dependency cycles under control, the sooner it's done, the better.

For new projects, however, it's a no-brainer. I'll be including those six lines of code on every new Python package I
work on from now on. Will you?

# Credits

- [Photo by Steve DiMatteo](https://www.pexels.com/photo/no-bicycle-signage-hanging-on-a-wooden-pole-13033981/]).

# Footnotes

[^1]: If you've no idea what Import Linter is, you might want to read the [overview](https://import-linter.readthedocs.io/en/stable/readme.html#overview) before reading on.
[^2]: More on this in [What is Inversion of Control and why does it matter?](/2019/04/15/inversion-of-control/).