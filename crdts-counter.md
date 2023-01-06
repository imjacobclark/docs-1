---
layout: docs
title: Counter CRDT
---

If your application needs to model a count of something (say number of upvotes on a post) then a LWW register won't cut it. You'd need to sum the values across all nodes rather than just take the latest write.

Coming soon -- counter CRDT column types.