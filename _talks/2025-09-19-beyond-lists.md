---
layout: talk
title: Beyond Lists
featured: true
weight: 2
location: PyCon UK 2025, Manchester
tags: [python, design]
image: beyond-lists.jpg
description:
  Lists are everywhere in Python. But are they always the best tool for the job?
youtube_embed_url: https://www.youtube.com/embed/3N8qs0qWZoY?si=xHpXxOHhRZvDkRh-
slides_link: https://docs.google.com/presentation/d/1skGK9XNQzZMOlTnho-pq_VPmc37YDq-dc2LWqmFdiGU/edit?slide=id.p#slide=id.p
---

When writing Python, we often to need to work with a collection of objects. And so often we reach for the humble list. But lists, while they can get the job done, are often not the best choice.

Do you really need that collection to change? If you don't, picking a tuple instead will bring unexpected benefits. Or is the order of the elements important? If not, then using a set will open up a world of new possibilities. And if your set doesn't need to change, then consider a frozenset - Python's most underrated built-in data structure.

Equipped with these data structures, we'll close by looking at the subtleties of how to type-annotate functions, and why on earth it is better to use `Set` (with a capital 'S') for an argument type, but `set` for the return type.

It's time to go beyond lists!
