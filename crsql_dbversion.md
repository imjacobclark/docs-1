---
layout: docs
title: crsql_dbversion
---

`crsql_dbversion` returns the current lamport timestamp of the database. This value is incremented on every transaction made to the database and moved up to the `MAX(this_db_version, peer_db_version)` when syncing in order to allow causal ordering of changes between databases.

# Usage

```
SELECT crsql_dbversion();
```