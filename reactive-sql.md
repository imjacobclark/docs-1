---
layout: docs
title: Reactive SQL
---

To get started with reacting to changes to your database, take a look at the `TodoMVC` example and its `hooks`. https://github.com/vlcn-io/cr-sqlite/blob/main/js/examples/todomvc/src/hooks.ts

# Theory & Roadmap

The ideas behind Reactive SQL are best captured by this paper: [Building data-centric apps with a reactive relational database](https://riffle.systems/essays/prelude/)

Standard SQL can be used but a derivative syntax to make this easier and being [pioneered by vlcn and others](https://twitter.com/schickling/status/1599076832107630594).

There are a number of approaches people have taken to reactivity in the context of SQL.

1. `update_hook` based like [RxGRDB](https://github.com/RxSwiftCommunity/RxGRDB)
2. Table based like Riffle
3. Incremental view maintenance like [Noria](https://github.com/mit-pdos/noria) & [ReadySet](https://readyset.io/)
4. Differential dataflow like [Materialize](https://materialize.com/blog/life-in-differential-dataflow/)

In the spirit of getting something shipped fast and iterating, `vlcn.io` currently uses a mix of (1) & (2). That being said, we plan to develop a completely reactive system that only processes the pieces of data that have changed.

`vlcn` will have a few advantages over other approaches to reactivity.
The reason is given that `SQLite` is embedded directly into the app using and we:
1. Know all data returned by the db and currently in use by the app
2. Know all queries run by the app
3. See all writes to the DB before they're written to the DB

Look out for more on this in Q1 2023.