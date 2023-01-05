---
layout: docs
title: crsql_siteid
---

Each databases has a unique "site id" to identify itself to other databases. To get the site id of the current database, use `crsql_siteid`.

`site_id` is a 16 byte uuid. At the time of this writing it is a v4 uuid but is being moved to a [v7 uuid](https://www.ietf.org/archive/id/draft-peabody-dispatch-new-uuid-format-01.html#name-uuidv7-layout-and-bit-order) for better index performance for those use cases that need to index many sites (e.g., IOT).

In either case, `site_id` should be treated as an opaque value that only serves to identify a database.

# Usage

```
SELECT crsql_siteid();
```

For a hex encoded result:

```
SELECT quote(crsql_siteid());
```
