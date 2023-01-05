---
layout: docs
title: Create CRR
---

CRRs are created from existing tables. As of this writing, the existing tables should be empty before creating a CRR from them. Future versions will support backfilling clock information in the cases where rows already exist in a table.

The process of creating a CRR always beings with creating a table --

```
CREATE TABLE my_table (id text primary key, name text, x float, y float);
```

Followed by running `as_crr` on it:

```
SELECT crsql_as_crr('my_table');
```

The [crsql_as_crr](./crsql_as_crr) command will ensure your table meets the requirements to become a crr and tell you if it does not.

Related:
- [crr table requirements](./bits-table-requirements)
- [crsql_as_crr](./crsql_as_crr)
- [[docs/concept-crr]]