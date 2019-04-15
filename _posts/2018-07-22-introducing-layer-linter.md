---
layout: post
title: Meet Layer Linter
description: "A tool for imposing architectural constraints on your Python projects."
image: lint.jpg
featured: false
weight: 2
tags: [python, architecture]
---
Python is a wonderful language that is a joy to develop in. But I've found that
projects written in Python can easily grow into an unmaintainable mess. Keeping a code base maintainable, particularly when it's large and
complex, is difficult.

A common strategy is to organise it into smaller, decoupled
subpackages. But that's easier said than done. Circular dependencies between these subpackages
have a nasty habit of creeping in. Over time, what you worked hard to separate
 creeps inexorably together.

Part of the problem is that Python has no formal way of declaring, and enforcing,
a dependency flow. That's why I wrote [Layer Linter](https://github.com/seddonym/layer_linter).

# What Layer Linter does

Layer Linter is a tool that helps impose a structure on your Python project, based on its 
internal dependency flows. It analyses which modules are importing which, and
checks this conforms to a *contract* defined by you.

In this contract, you describe an ordered list of *layers*. Each layer is just a subpackage or module
within your codebase. The contract stipulates that any code within a layer lower down the list
must not import, even indirectly, anything from a higher up layer.

For example, you could decide to structure your project using three layers:

- ``interfaces`` (highest level)
- ``domain``
- ``data`` (lowest level)

Once you've created the contract, you run Layer Linter's command line tool to see if anything is not adhering
to your architecture. Here's what the output might look like:

<pre class="console">
$ layer-lint myproject
<strong>
============
Layer Linter
============

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

<strong>1. myproject.data.userrecord imports myproject.domain.user:</strong>

    myproject.data.userrecord <-
    myproject.domain.user

<strong>2. myproject.domain.user imports myproject.interfaces.api:</strong>

    myproject.domain.user <-
    myproject.common.apitools <-
    myproject.interfaces.api
</div>
</pre>

In this report, Layer Linter is telling you that you're not
adhering to your architecture in two places. First, the `data` layer
(which is the lowest level) is importing from `domain` (which is a mid level layer).
Second, `domain` is importing (via a package not listed in the contract) something
from `interfaces` (the highest layer).

If you're serious about preventing violations of the contract, you can add
the `layer-lint` command to your automated test / continuous integration run. (You can
see an example of this in the [Layer Linter repo](https://github.com/seddonym/layer_linter/blob/master/tox.ini).)

# Usage

To use Layer Linter, first you need to define your layers. You do this in a ``layers.yml``
file that looks something like this:

<pre>
My contract:
    packages:
        myproject
    layers:
        - high_level
        - medium_level
        - low_level
</pre>

You then run:

<pre>
$ layer-lint myproject
</pre>

The report will tell you whether you're following your contract.

You can define other architectural styles too, such as a more
modular layered style which I call the '[Rocky River]({% link _posts/2018-09-16-rocky-river-pattern.md %})'. It also supports multiple
contracts. See more detailed information about how to use Layer Linter in [the docs](https://layer-linter.readthedocs.io/en/latest/usage.html).

(*Note:* Since this post was written, Layer Linter has been superseded by
[Import Linter](https://import-linter.readthedocs.io), which does everything Layer Linter does, but with more features
and a slightly different API. There is a guide to the minor API differences
[here](https://layer-linter.readthedocs.io/en/latest/migrating-to-import-linter.html).)


# Further information

- [Layer Linter documentation](https://layer-linter.readthedocs.io).
- The Layers pattern in Chapter 2 of <em>Pattern-Oriented Software Architecuture Vol. 1</em> (Buschmann, Meunier, Rohnert, Sommerlad and Stal, 1996).
- [My talk on layers and the Rocky River pattern]({% link _talks/2017-12-12-rocky-river.md %}).
- [Encapsulated Django]({% link _talks/2015-11-16-encapsulated-django.md %}).



