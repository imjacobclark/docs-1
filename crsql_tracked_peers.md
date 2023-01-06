---
layout: docs
title: crsql_tracked_peers
---

`crsql_tracked_peers` is a table for tracking what updates have been received from peers or sent to peers.

This table is useful for authors of custom network / syncing layers. While `crsqlite` will automatically insert & update rows for peers when applying changes via `INSERT INTO crsql_changes ...`, users of `crsqlite` may also insert or update rows of this table for their own purposes.

> Coming soon: If you do not want `crsqlite` to do any automatic tracking of peers, provide -1 to the hidden column `tag` of `crsql_changes` when performing an insert. You may also set `tag` to any other nonzero custom value

# Table Structure

```sql
CREATE TABLE crsql_tracked_peers(
  [site_id] BLOB NOT NULL,
  [version] INTEGER NOT NULL,
  [tag] INTEGER,
  [event] INTEGER,
  PRIMARY KEY ([site_id], [tag], [event])
) STRICT;
```

1. **site_id** - the unique id of the peer
2. **version** - falls into two categories:
  1. For receive events: The clock value we last _received_ from that peer
  2. For send events: The clock value we last _sent_ that peer. I.e., For sends, this is _our_ db's clock not the peer's.
3. **tag** - used to differentiate sync sets. Tag is set to `0` for whole database syncs. Inserts into `crsql_changes` where tag is set to `-1` will not create tracking events. If you're only syncing subsets of the databse then this tag represents the subset you're syncing, such as an id of a "root node" of some sub-graph. Coming soon -- see [[docs/replicated-subgraphs]]
4. **event** - 0 for receive events, 1 for send events. Values below 1000 are reserved for `crsqlite`

# Using `crsql_tracked_peer`

`crsqlite` automatically inserts & updates this table when processing inserts into `crsql_changes`. On transaction commit, `crsqlite`:

1. Inserts rows for `site_id`, `tag`, `event` tuples that have not been seen before the current transaction, setting their `clock` to the maximum value seen by `crsql_changes` in the current transaction.
2. Updates rows for `site_id`, `tag`, `event` tuples that were seen in prior transactions, setting their `clock` to the max of the existing value and the max seen in the current transaction.

If the user does not provide a `tag` value during insert to `crsql_changes` it is set to `0`.

`crsqlite` only updates `tracked_peers` on receive events and thus always sets event to `0`. Custom sync providers can use this table to track send events by setting the event to `1` on insert.

See the [sync guide](./guide-sync) to understand how to leverage this table when creating a custom sync provider.

## Tag Values

- `0` - WHOLE_DATABASE
- `-1` - reserved
- All other values are open for use by clients.

## Event Values

- `0` - RECEIVE
- `1` - SEND
- [2-1000] - reserved
- All other values are open for use by clients. Prefer `tag` over `event` if you need to track many version for a given peer.