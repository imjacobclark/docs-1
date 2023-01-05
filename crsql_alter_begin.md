---
layout: docs
title: crsql_alter_begin
---

For an end to end overview of migrations, see [migrations](./migrations.html).

`crsql_alter_begin` starts an alteration of a `crr`. If a transaction has not already begun, `crsql_alter_begin` starts one. Any call to `crsql_alter_begin` must be followed by a call to [`crsql_alter_commit`](./crsql_alter_commit.html) after the schema alteration is complete.

# Usage:

```sql
SELECT crsql_alter_begin('table_name');
```

# Example:
```sql
SELECT crsql_alter_begin('foo');
ALTER TABLE foo ADD COLUMN c TEXT;
SELECT crsql_alter_commit('foo');
```