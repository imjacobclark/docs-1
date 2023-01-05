---
layout: docs
title: Migrations
---

When modifying the schema of a `crr`:
1. Be sure you start that modification with a call to `crsql_alter_begin` 
2. Complete that modification with a call to `crsql_alter_commit`

calls to [`crsql_alter_begin`](./crsql_alter_begin.html) and [`crsql_alter_commit`](./crsql_alter_commit.html) may be nested as well as occur in an outer transaction.

If the alter fails, all levels of transactions and savepoints will be rolled back.

# Usage:
```sql
SELECT crsql_alter_begin('table_name');
...
SELECT crsql_alter_commit('table_name');
```

# Example:
```sql
SELECT crsql_alter_begin('foo');
ALTER TABLE foo ADD COLUMN c TEXT;
SELECT crsql_alter_commit('foo');
SELECT crsql_alter_begin('bar');
ALTER TABLE bar ADD COLUMN baz TEXT;
SELECT crsql_alter_commit('bar');
```

Currently supported schema alterations:

1. add columns
2. drop columns

Support for column renames, table renames, column type modification is a work in progress.
