---
layout: docs
title: Background & Concepts
---

`vlcn` allows applications to maintain their state locally and, if desired, merge that state with other peers or a central service at some future point in time. This merging process is guaranteed to converge all nodes to the same state and not run into conflicts.

This enables:
1. Development of offline & [Local-First](https://www.inkandswitch.com/local-first/) applications
2. Apps that are responsive even in the face of network partitions
3. Reduced cloud and infrastructure costs by leveraging compute and storage of client devices

`vlcn` also provides some primitives to simplify application development by allowing developers to subscribe to the database and react on changes. The same concept as the proposal made [here](https://riffle.systems/essays/prelude/).

# Concepts

The core technologies that make this possible are:

1. SQLite
2. CRDTs & CRRs
3. Live Queries

## SQLite
[SQLite](https://sqlite.org) is the world's most used embedded database. Your phone, tv, car, desktop, laptop, wifi route, etc. all probably have one or many instances of `SQLite` embedded into them. `SQLite` is extremely tiny and fast and embeds directly into your application, meaning queries take microseconds to fulfill. `SQLite`, however, has little support for merging and applying changes from other `SQLite` databases. The support it does have requires the developer to manually resolve conflicts.

## CRDTs & CRRs
That brings us to [CRDTs](https://crdt.tech/) or "conflict free replicated data types." Any number of people can make concurrent writes to a `CRDT`. As messages are exchanged between nodes in the system, all instances of the `CRDT` are guaranteed to eventually converge to the same  state.

`CRDTs` sound a bit magic but they can be incredibly dumb (a set that can only grow is a `CRDT`), or much more complex (a data structure for capturing edits to rich text). The other thing to consider is whether or not the academic notion of "convergence" matches the expectations of your product use case. For the vast majority of use cases, `vlcn` has found the answer to be yes. Many use cases can also get away with simple `CRDT` algorithms such as [LWW maps](https://bartoszsypytkowski.com/crdt-map/#crdtmapwithlastwritewinsupdates), [LWW registers](https://bartoszsypytkowski.com/operation-based-crdts-registers-and-sets/#lastwritewinsregister), and [RW-sets](https://crdt.tech/glossary#:~:text=Remove%2Dwins%20set%20(RWSet)%3A).

At the time of this writing, `vlcn`:

1. Models tables as remove win set (RW-Sets)
2. Models rows as last write wins maps (LWW Maps)
3. Models columns as last write wins registers (LWW Registers)

In other words:
1. If multiple users concurrently add many rows (all rows have differing primary keys), all rows will be retained.
   - If a user deletes a row that another user updated, the delete wins
2. If multiple users concurrently edit different columns of the same row (same primary key), the final state of the row will be a combination of everyone's values
3. If multiple users edit the same column of the same row, the final state of the cell will be the write from the user who is deemed to have written last.

Add-wins (rather than remove-wins) semantics can easily be added by the developer via including a column that acts as an `isDeleted` flag. `vlcn` doesn't support this by default given the current climate around privacy and "deleted means deleted."

CRDTs that are on the roadmap but not yet built:

- Counters
- Sequence/Array & Rich Text CRDTs
- Multi-value registers
- *[Fractional Indexing](../blog/fractional-indexing)

One thing to point out is that `CRDTs` are eventually consistent. I.e., `vlcn` is adding partition tolerant and eventually consistent data structures to a normally strongly consistent relational model. This introduces some new angles to consider when designing a schema that includes `CRDTs` which is covered in [[docs/bits-table-requirements]].

The composition of the set of CRDTs required to fulfill the table abstraction in a RDBMS is called a `CRR` or `Conflict Free Replicated Relation`.

_*while "fractional indexing" doesn't sound like a CRDT it needs some special care to be usable in a multi-writer system._

## Live Queries

Live queries allow your application to:

1. Express the data it needs
2. Be notified whenever that data changes

This is especially helpful in situations where your database could be merging in updates from other peers or a server in the background. E.g., when doing any sort of live editing, collaboration or device sync.
