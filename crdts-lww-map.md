---
layout: docs
title: Last Write Wins Map
---

A row in a table can be thought of as a map. Column names are the keys and cell values are the values. Tables that are upgraded to CRRs are modeled as [Last Write Wins](./concept-lww) Maps by default.

In this setup, each cell in a row is a [last write wins register](https://lars.hupel.info/topics/crdt/07-deletion/#last-write-wins).

# Alternatives

- Columns as [counter CRDTs](./crdts-counter)
- Columns as [RGAs or mergable text documents](./crdts-rga)
- Columns to store a user defined ordering via [fractional indexing](./crdts-fractional-index)