---
layout: docs
title: crsql_changes
---

`crsql_changes` is a virtual table that can be used to pull a changeset out of the database and/or apply a changeset to the database.

# Table Structure

```sql
CREATE TABLE crsql_changes(
  [table] TEXT NOT NULL,
  [pk] TEXT NOT NULL,
  [cid] TEXT NOT NULL,
  [val] ANY,
  [col_version] INTEGER NOT NULL,
  [db_version] INTEGER NOT NULL,
  [site_id] BLOB
) STRICT;
```

1. **table** - this is the name of the CRR that the current change applies to
2. **pk** - the primary key columns for that table, encoded in a format that can be sent over the wire and validated on receipt.
3. **cid** - the name of the column that was modified. Special values are used in the case of deletes and creates.
4. **val** - the value of the column. Encoded for safe wire transport and validation on receipt.
5. **col_version** - version / lamport clock of the column. Used for merging.
5. **db_version** - the lamport clock of the database. Used to track whether or not a site has seen changes from another database.
6. **site_id** - the site responsible for the change

Note: if you migrate the schema of a `crr` without using `crsql_alter_begin` you can end up in a case where tables and columns in a changeset do not exist. The changeset applicator will ignore these but this can lead to a divergence between databases. See the [migrations](./migrations.html) section.

# Pulling Changesets

To retrieve a changeset from the database you can query the `crsql_changes` table like any other table. The preferred way to do this, however, is as follows:

```sql
SELECT "table", "pk", "cid", "val", "col_version", "db_version", "site_id"
  FROM "crsql_changes"
  WHERE "db_version" > ?
  AND "site_id" != ?
```

This query will return all changes from the databse after a provided version and not applied by a given peer.

- The `site_id != ?` is included as an optimization that reduces total sync roundtrips by one by preventing sites from receiving their own changes back.
- Passing `site_id IS NULL` will only fetch changes that were made locally and not receive through data sync.

You may also query against other columns -- such as the table name -- if you are only interested in a subset of changes like those to a given table.

Seen the [sync guide](guide-sync) to learn more about pulling changes from this table.

# Applying Changesets

The output of

```sql
SELECT "table", "pk", "cid", "val", "col_version", "db_version", "site_id"
  FROM "crsql_changes"
  WHERE "db_version" > ?
  AND "site_id" != ?
```

can be directly fed back into a `crsql_changes` table on another database.

## Example:

```sql
INSERT INTO "crsql_changes"
  ("table", "pk", "cid", "val", "col_version", "db_version", "site_id")
  VALUES (?, ?, ?, ?, ?, ?, ?);
```

Note that you **MUST** use SQL bind parameters and **NOT** include the values of `crsql_changes` directly in a SQL string.

`crsqlite` will process the bound values and ensure they are safe before applying them against the database. If the values are interpolated into the string, undefined behavior may result.
