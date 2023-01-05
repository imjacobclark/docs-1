---
layout: docs
title: crsql_alter_commit
---

For an end to end overview of migrations, see [migrations](./migrations.html).

`crsql_alter_commit` must be called after alteration against a crr begun with `crsql_alter_begin`.

# Usage:

```sql
SELECT crsql_alter_commit('table_name');
```

# Example:
```sql
SELECT crsql_alter_begin('foo');
ALTER TABLE foo ADD COLUMN c TEXT;
SELECT crsql_alter_commit('foo');
```