---
layout: post
title: Code Review Checklist
description: A set of probing questions I use when reviewing a fellow developer's work.
image: checklist.jpg
weight: 9
tags: ["ways of working"]
---

Code reviews are important. A second pair of eyes on a developer's work, before
it gets released, kills two birds with one stone: not only does it improve
quality, it also ensures that system knowledge is shared.

But in my experience reviews tend to be narrowly focused. We stay in the details,
concerned mainly with correctness and (perhaps) style.

Reviewing code is about more than just hunting for bugs, or being a glorified
[linter](https://en.wikipedia.org/wiki/Lint_(software)){:target="_blank"}. It's about thinking
around the subject, asking questions that the original developer might have
forgotten about because they were so focused on getting the work done.

Here, then, is a checklist of exploratory questions I use when doing a review.
I've found it useful to categorise them into four stages: the stages that
a feature passes through as it is developed.

- Define
  - Why are we doing this?
  - Is the scope clear?
  - How should the system behave in different scenarios?
  - Have any edge cases been missed?
- Implement
  - Do you fully understand what has been built?
  - How good is the end user's experience?
  - Is the code high quality?
  - What's the test coverage like?
  - Is there any impact on local development?
  - How will this scale?
- Validate
  - Are you confident that it actually works?
  - Is there a plan for QA?
  - Are any scenarios untested?
  - What other parts of the system should we regression test?
- Release
  - Is there a plan for how this will be released?
  - Is there any risk of data loss?
  - How will the system behave during the release?

Let me know if you find it useful - and if you think I should add anything else to the list!
