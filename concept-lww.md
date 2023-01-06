---
layout: docs
title: Last Write Wins
---

What does it mean for the "last write" to win? Especially in a distributed system where physical clocks will not be perfectly synchronized or not synchronized at all?

`cr-sqlite` & `vlcn` use causal consistency to determine the last write. Each database has its own internal logical clock which it increments for every write operation it makes. When two databases exchange information, they bump their logical clock to be the max of the two databases. This ensures that given two logical clocks from two different databases we know which writes were after or concurrent with which other writes.

For the default configuration of `cr-sqlite`, [crrs](./concept-crr) use [Lamport Timestamps](https://en.wikipedia.org/wiki/Lamport_timestamp) as their logical clocks.

Further reading: 
- https://lars.hupel.info/topics/crdt/07-deletion/#last-write-wins
- https://tantaman.com/2022-10-18-lamport-sufficient-for-lww.html