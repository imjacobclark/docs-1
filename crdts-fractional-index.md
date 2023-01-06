---
layout: docs
title: Fractional Indexing
---

Handling user defined ordering efficiently is a common problem in relational databases. When you cast that into a multi-writer scenario, a few more problems pop up.

We've outlined some solutions here:
- [[blog/fractional-indexing.md]]

And community members have started implementing proof of concepts:
- https://gist.github.com/Azarattum/0071f6dea0d2813c0b164b8d34ac2a1f

Which we'll eventually be rolling directly into `cr-sqlite`.