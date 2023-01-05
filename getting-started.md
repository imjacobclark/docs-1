---
layout: docs
title: Getting Started
---

> For more information no how `vlcn` is able to merge databases that have had concurrent writes or been offline, see [[docs/background-concepts]].

`vlcn` aims to simplify device sync & collaborative applications by

1. Pushing the complexity of eventual consistency and state merging to the database
2. Exposing a declarative way choose what CRDTs to use on what tables

Most of the heavy lifting is done through the `cr-sqlite` extension. `cr-sqlite` is a run time loadable extension that augments `SQLite` with the ability to merge databases that have had concurrent modifications or been offline for an indefinite period.

# Configuring SQLite

There are two ways to use `vlcn` & `cr-sqlite` with `SQLite`:
1. Use an existing build of `SQLite` and load the required extensions at runtime
2. Use a bundled and pre-packaged build of `SQLite`

## Existing Build

`vlcn` provides `CR-SQLite` as a [run time loadable extension](https://www.sqlite.org/loadext.html) to augment `SQLite`. When using your existing build of `SQLite`, you'll need to load the `CR-SQLite` extension on each connection made to your database. You'll also need to unload it when disconnecting.

To obtain a copy of the extension:

```bash
git clone git@github.com:vlcn-io/cr-sqlite.git
cd cr-sqlite/core
make loadable
```

This will create a shared library at dist/crsqlite.[lib extension]

[lib extension]:
- Linux: .so
- Darwin / OS X: .dylib
- Windows: .dll

You can then copy that library to where you need it and load it into sqlite via the `load_extension` command.

```
SELECT load_extension('crsqlite');
```

Be sure to [unload the extension before terminating you database connection](https://sqlite.org/forum/forumpost/c94f943821).

```
SELECT crsql_finalize();
```

## Bundle

For browser environments, bundled sqlite + cr-sqlite builds are provided
- [Official SQLite WASM build including cr-sqlite](https://github.com/vlcn-io/cr-sqlite/tree/main/js/browser/crsqlite-wasm)
- [wa-sqlite bundled to include cr-sqlite](https://github.com/vlcn-io/cr-sqlite/tree/main/js/browser/wa-crsqlite)

For node you can use
- [@vlcn.io/crsqlite-allinone](https://github.com/vlcn-io/cr-sqlite/tree/main/js/node-allinone)

# Syncing Data

See [[docs/guide-sync]] for a step by step guide on how to create conflict free replicated relations and sync between databases.
