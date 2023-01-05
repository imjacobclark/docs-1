---
layout: docs
title: crsql_as_crr
---

`crsql_as_crr` converts an existing table to a conflict free replicated relation. Once a table is tracked as a CRR, it can be replicated and merged with other instances of that CRR on other databases.

For merging, see [crsql_changes](./crsql_changes.html).

# Usage:

```sql
SELECT crsql_as_crr('table_name');
```

# Example:

```sql
BEGIN;
CREATE TABLE foo (a primary key, b);
SELECT crsql_as_crr('foo');
COMMIT;
```