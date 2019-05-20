---
layout: post
title: Meet Import Linter
description: >
    Python is a wonderful language that is a joy to develop in. But I've found that
    projects written in Python can easily grow into an unmaintainable mess. Keeping a code base maintainable, particularly when it's large and
    complex, is difficult.
image: depgraph.png
featured: true
weight: 2
tags: [python, architecture]
---

A common strategy is to organise it into smaller, decoupled
subpackages. But that's easier said than done. Circular dependencies between these subpackages
have a nasty habit of creeping in. Over time, what you worked hard to separate
 creeps inexorably together.

Part of the problem is that Python has no formal way of declaring, and enforcing,
a dependency flow. That's why I wrote [Import Linter](https://github.com/seddonym/import-linter).

(*Note: This is an adaptation of [an earlier blog post of mine]({% link _posts/2018-07-22-introducing-layer-linter.md %}),
adapted for the newer version of the tool.*)

# What Import Linter does

Import Linter is a tool that helps impose a structure on your Python project, based on its 
dependency flows. It analyses which modules are importing which, and checks this conforms to a *contract* defined by you.

Each contracts must be of [a particular type](https://import-linter.readthedocs.io/en/latest/contract_types.html),
depending on what sort of constraints you wish to impose. For example, an *independence contract* checks that there are
no imports between a given list of subpackages. If one of the built in contract types doesn't meet your needs, you
may create a [custom contract type](https://import-linter.readthedocs.io/en/latest/custom_contract_types.html) instead.

# Usage

To use Import Linter, first you need to write your contract. You do this in a ``.importlinter``
file that looks something like this:

<pre>
[importlinter]
root_package = mypackage

[importlinter:contract:1]
name = My independence contract
type = independence
modules =
    mypackage.foo
    mypackage.bar
    mypackage.baz
</pre>

You then run:

<pre>
$ lint-imports
</pre>

The report will tell you whether you're following your contract. If you're using
[continuous integration](https://en.wikipedia.org/wiki/Continuous_integration),
you can add the ``lint-imports`` command to your test pipeline to ensure that everyone in your team adheres to the
architectural rules.

# Example: Layer contracts

A powerful contract type is the *layers contract*. This allows you to impose a classic 'Layered Architecture' on
your project. 

In this contract, you describe an ordered list of *layers*. Each layer is just a subpackage or module
within your codebase. The contract stipulates that any code within a layer lower down the list
must not import, even indirectly, anything from a higher up layer.

For example, you could decide to structure your project using three layers:

- ``interfaces`` (highest level)
- ``domain``
- ``data`` (lowest level)

This contract would be defined in your ``.importlinter`` file as follows:

<pre>
[importlinter]
root_package = mypackage

[importlinter:contract:1]
name = My three-tier layers contract
type = layers
layers=
    high
    medium
    low
containers=
    mypackage
</pre>

You can then run Import Linter's command line tool to see if anything is not adhering
to your architecture. Here's what the output might look like:

<pre class="console">
$ lint-imports
<strong>
=============
Import Linter
=============

---------
Contracts
---------
</strong>
Three tier architecture <strong class='error'>BROKEN</strong>
<div class='error'><strong>
Contracts: 0 kept, 1 broken.

----------------
Broken contracts
----------------


Three tier architecture
-----------------------
</strong>

<strong>myproject.data.userrecord is not allowed to import myproject.domain.user:</strong>

    myproject.data.userrecord -> myproject.domain.user (l. 5)

<strong>myproject.domain.user is not allowed to import myproject.interfaces.api:</strong>

    myproject.domain.user -> myproject.common.apitools (l. 1)
    myproject.common.apitools -> myproject.interfaces.api (l. 10)
</div>
</pre>

In this report, Import Linter is telling you that you're not
adhering to your architecture in two places. First, the `data` layer
(which is the lowest level) is importing from `domain` (which is a mid level layer).
Second, `domain` is importing (via a package not listed in the contract) something
from `interfaces` (the highest layer).

## Other architectural styles

You can define other architectural styles with layers contracts. One example is a more modular layered style which I
call the '[Rocky River]({% link _posts/2018-09-16-rocky-river-pattern.md %})'. Another is
[ports and adapters](https://herbertograca.com/2017/09/14/ports-adapters-architecture/), a.k.a 'hexagonal architecture'.

For more detailed information about how to use Import Linter, see [the docs](https://import-linter.readthedocs.io).

If you have any feedback, or questions about using Import Linter, I'd love to hear from you - just [get in touch]({% link contact.html %}).

# Further information

- [Import Linter documentation](https://import-linter.readthedocs.io).
- The Layers pattern in Chapter 2 of <em>Pattern-Oriented Software Architecuture Vol. 1</em> (Buschmann, Meunier, Rohnert, Sommerlad and Stal, 1996).
- [My talk on layers and the Rocky River pattern]({% link _talks/2017-12-12-rocky-river.md %}).
- [Encapsulated Django]({% link _talks/2015-11-16-encapsulated-django.md %}).



