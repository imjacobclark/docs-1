---
layout: docs
title: Partial Replication
---

vlcn & cr-sqlite currently allow you to replicate an entire database from one device/user/service to another. This "whole db sync" is great for use cases like:

- Each user having their own DB that they want access to everywhere
- Applications that store each "document" in its own database file. This is analogous to word/excel/photoshop and their docx/xls/psd files -- essentially using [sqlite as an application file format](https://www.sqlite.org/appfileformat.html)
  - Collaborating with many people on the same file is trivial in this setup since each edit by each user can be replicated to every other through whole db replication.

This approach breaks down when it runs into:
1. A traditional centralized service
2. Sharing at lower levels of granularity (e.g., share a paragraph in a doc and not the entire doc)
3. Very large datasets

# Cloud Apps, Sharing Granularity, Large Datasets

Traditional cloud apps hit all three of the above problems since they usually store all of their data in a big Postgres or MySQL instance. This DB (or set of DBs) will also hold data from many different users. Any given user, however, is only interested in, only has access to, and would only be able to store a subset of the total data available in the central database(s). 

These problems are also not unique to centralized services. They pop up in peer 2 peer settings when there is an imbalance in resources between peers (a mobile device can only process a fraction of what a desktop can) or when peers have data they don't want to share with other peers.

# Subset

The above is pointing to the fact that we need a way to replicate a subset rather than an entire database.

To make this a bit more concrete, think of a collaborative presentation editor that saves all presentations in a shared database rather than using one file per presentation.

A basic slide deck (presentation) has:

1. A creator
2. A title
3. Collaborators (those that can see and edit the deck)
4. Slides
5. Components on the slides (text, drawings, embeds)
6. Speaker notes

Which can be modeled with the following tables:

![relations](../assets/blog/replicated-subgraph/relations.svg)

The subgraph of a given presentation being:

![subgraph](../assets/blog/replicated-subgraph/subgraph.svg)

For any given presentation, we'll want to be able to sync the state of all items in that subgraph between all collaborators.

# Representing Subsets

Luckily we know how to represent a subset (or subgraph) of a database. A subset of a database can be represented by one or more queries that return the rows belonging to that subset.

Based on that insight, `vlcn` will be making some updates to `crsql_changes`.

`crsql_changes` currently gives you all the changes in the database since a given `db_version`. The update to `crsql_changes` will allow you to get the changesets for any subset by passing the primary keys of rows you want changes for to `crsql_changes`.

Example:

```
SELECT * FROM crsql_changes
  WHERE table = table_name
  AND primary_key IN (...your subset queries or primary keys...)
  AND db_version > ?;
```

It is up to you to:
1. Track the db_version that you last synced for the given subset to use for future calls to `crsql_point_changes`. You could of course always pass `0` for the version and receive all changes for the subset every time (not ideal).
2. Know when to re-query your subset for changes

To understand how to do that, keep an eye out for our guide on building sync protocols atop `cr-sqlite`.

While `crsql_changes` can technically do this now, it is currently only optimized for whole db replication rather than pulling changes to an aribtrary set of rows. Partial db replication optimizations will be included in the next minor release.

# Future

A more wholistic solution (e.g., one that can understand when changes were made to query dependencies for you) may eventually be desired in time but this building block should get you started.