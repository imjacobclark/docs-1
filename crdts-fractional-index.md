---
layout: docs
title: Fractional Indexing
---

Handling user defined ordering efficiently is a common problem in relational databases which is often solved through [factional indexing](https://madebyevan.com/algos/crdt-fractional-indexing/). When you cast that into a multi-writer scenario, however, a few more problems pop up.

We've outlined what fractional indexing is and how to go about it in a distributed system here:
- [[blog/fractional-indexing]]

And community members have started implementing proof of concepts:
- https://gist.github.com/Azarattum/0071f6dea0d2813c0b164b8d34ac2a1f

Which we'll eventually be rolling directly into `cr-sqlite`.