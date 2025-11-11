---
layout: talk
title: Crabs in Snakes! Writing compiled Python modules in Rust
featured: true
weight: 3
location: PyCon Italia 2024, Florence
tags: [python, "domain driven design", modelling]
image: crabsinsnakes.jpg
description:
  Python is great for productivity and readability. Rust is great if you want things to run really fast.
  Isn't it a shame we can't use both in the same program?
  It turns out we can! Learn how, with the help of some clever libraries, we can write compiled Python modules in Rust.
youtube_embed_url: https://www.youtube.com/embed/L844NAJ24QI?si=ugPZEvSjtQVRl3XN
slides_link: https://docs.google.com/presentation/d/1HNTdnFNu0SsT7jS4x0SANKGlCPGIYMadC12G-mP35-4/edit?usp=sharing
---

In this talk, you’ll learn how to get started with using Rust directly from Python.

We’ll start by introducing Rust itself. It’s a very different language to Python with a whole new way of thinking.
Along the way, we’ll meet Cargo, Rust’s excellent package manager.

Then we’ll see how we can plumb Rust code into Python as a native extension module, and distribute it as a wheel.
We’ll do this using Py03 and Maturin.

You don’t need to know anything about Rust, extension modules or packaging - this will give you the fundamentals.
By the end, you will know how to get started putting the power of Rust inside your Python programs.
