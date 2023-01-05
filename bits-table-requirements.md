---
layout: docs
title: Table Requirements
---

In order to be a [crr](./concept-crr) a table must:

- Have one or more primary key columns
- Have no enforced foreign key constraints. Unenforced foreign keys are fine.
- Have no other unique indices besides the primary key(s)
- Columns are either nullable or have a default value.

Tables that are not crrs have no requirements.

Related:
- [[docs/concept-crr]]
- [[docs/bits-create-crr]]