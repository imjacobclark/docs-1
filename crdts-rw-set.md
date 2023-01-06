---
layout: docs
title: Remove-Wins Set
---

A table in a database can be thought of as a set of rows. Tables that are upgraded to [CRRs](./concept-crr) in cr-sqlite are modeled as Remove-Wins Sets by default. I.e., when merging tables from other databases, if a row was deleted then the delete trumps all other changes to that row.

Each row within a table is modeled as a [lww map](./crdts-lww-map) by default.

# Considerations

- Primary keys of deleted rows are retained as tombstones. This is required so a node can differentiate between receiving an update for a row it has never seen vs seeing an update for a row it has deleted.
- Depending on network topology, different garbage collection algorithms can be deployed to drop the "tombstone" records after certain conditions are met.
- If you need user defined ordering of rows in your table, see [[docs/crdts-fractional-index]].

# Alternatives

- If you want semantics that allow un-deletion you can implement this in user space. Rather than deleting the row, add an "is_deleted" column which functions as a LWW register and toggle the state between the deleted or not deleted.
- Support for RGAs (replicated growable arrays) for rich text and other use cases [is in the works](https://github.com/vlcn-io/cr-sqlite/issues/65).
- To understand how individual rows in a table are modeled (and alternatives), see [lww map](./crdts-lww-map).
