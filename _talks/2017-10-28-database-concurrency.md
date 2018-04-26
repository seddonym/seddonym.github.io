---
layout: talk
title: Handling Database Concurrency with Django
location: PyCon UK
tags: [django]
image: database-concurrency.jpg
description: How concurrency is handled by Django, and by the database.
youtube_embed_url: https://www.youtube.com/embed/5g9fwYcF3r8
---
It's easy to forget that your Django site may end up accessing the database several times at once. This can be perilous!

In this talk, I explore Django's concurrency handling in detail. You'll learn key concepts, common pitfalls
and gain a solid foundation in how to write code that is concurrency-safe. This talk has plenty of tips that
are relevant to all Python developers that work with databases.

N.B. This is a shorter version of [a talk I gave at the Django Meetup]({% link _talks/2016-11-15-database-gotchas.md %}) the previous year.

[Link to slides](http://slides.com/davidseddon/db-concurrency)
