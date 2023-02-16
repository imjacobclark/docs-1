---
layout: docs
title: Browser Persistence
---

<style type="text/css">
@import url("../assets/interactive/runnable-code.css");
@import url("../assets/interactive/json-viewer.css");
</style>

<script type="module" src="../assets/docs/guides/common.js"></script>

> Note: this guide is only relevant to browser environments. Persistence in all other environments has no special considerations.

Most of the guides you've seen so far have used an in-memory database. `SQLite` of course offers persistence and the API for a persisted DB does not differ at all from that of an in-memory DB.

In a web browser, however, there are a few extra hoops you must jump through to enable persistence and some more hoops to expose a persisted DB to the UI thread.

# Background

## Browser I/O Model

One of the main design principles of web browsers is that the UI thread must never block. This means that the UI thread can not ever call a function that performs I/O and wait for it snychronously on the UI thread.

In other words, you cannot do

```js
const fileData = fileHandle.getFile();
// do something with fileData
```

on the UI thread. You must instead do:

```js
fileHandle.getFile().then((fileData) => {
  // do something with fileData
});
```

or -

```js
const fileData = await fileHandle.getFile();
// do something with fileData
```

which is just syntactic sugar for the promise example. `await` releases control of the UI thread rather than blocking it. The code following `await` is called back into once the promise is resolved.

## SQLite I/O Model

While all I/O functions in JavaScript are asynchronous (there are caveats -- such as in worker threads which we will not get into), all I/O in SQLite is synchronous.

This presents some problems when porting `SQLite` to a browser environment. `SQLite` solves this by using the [Atomics APIs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics) and [createSyncAccessHandle](https://developer.mozilla.org/en-US/docs/Web/API/FileSystemFileHandle/createSyncAccessHandle).

While this successfully creates an interface for synchronous I/O, that synchronous interface is only allowed to be used from within a web worker.

This leaves us a few options.

# Persistence Options

A rather large fork in the road exists here for those targeting the browser. There exists:

1. The official SQLite WASM port
2. An unofficial WASM port called [wa-sqlite](https://github.com/rhashimoto/wa-sqlite)

Ideally we could use only one port but, once persistence enters the picture, there are rather large tradeoffs between the two builds.

## The Official Port

**The pros:**

1. Maintained and supported by the SQLite team
2. Aggressively focused on performance
3. [Permissive license (public domain)](https://www.sqlite.org/copyright.html)

**The cons:**

The official SQLite build can only use `OPFS` or `session/localStorage` for persistence. Session & local storage aren't great options given they're capped at 5MB, really leaving us only with `OPFS`.

The cons all come down to current `OPFS` limitations:

1. Safari - Persistence via OPFS does not work in Safari. This will be rectified when https://github.com/WebKit/WebKit/pull/4349 makes it into a Safari release.
2. Firefox - Persistence via OPFS [does not work in `Firefox`](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API#api.filesystemhandle). OPFS is currently being implemented by the Firefox team. 
3. `OPFS` has coarse grained locking and doesn't behave well when many tabs interact with the same persisted database. This could be solved by using `WebLocks` the VFS implementation. Something yet to be explored.
4. Requires you to set COOP and COEP headers on your site.

```
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
```

## The wa-sqlite Port

**The pros:**

The pros of the unofficial port come down to its ability to use `indexeddb` for persistence rather than `OPFS`. `indexeddb` has been around a long time and is well supported by all major browsers.

1. Works in all browsers (chrome, safari, firefox, edge)
2. Can run SQLite + Persistence in the UI thread
3. Handles concurrent access to the same db by many tabs well
4. Can be used in a `shared worker` to share the same db instance across multiple tabs

**The cons:**

1. May not be as fast as the official port although an apples to apples comparison has yet to be done. [benchmarks](https://rhashimoto.github.io/wa-sqlite/demo/benchmarks.html). With persistence enabled, it hits 25k inserts in 0.25 seconds on an m1 laptop.

Given the portability of `wa-sqlite`, that port will be used for the rest of this guide.

# Persistence with wa-crsqlite

Import the required packages and init the wasm:

<div class="runnable-code">
  <div>
return import('https://esm.sh/@vlcn.io/wa-crsqlite@0.6.3').then(async (initWasm) => {
  return self.sqlite = await initWasm.default(() => "https://esm.sh/@vlcn.io/wa-crsqlite@0.6.3/dist/wa-sqlite-async.wasm");
});
  </div>
</div>

Now create a database. To make it persisted all you need to do is pass in a filename. Omitting the filename runs the database in memory.

<div class="runnable-code">
  <div>
return (async () => {
  self.db = await sqlite.open("guide-persistence.db");
  window.onbeforeunload = () => {
    return db.close();
  };
  await db.exec(
  "CREATE TABLE IF NOT EXISTS todo (id primary key, text, completed)"
  );
  await db.exec("SELECT crsql_as_crr('todo')");
  return 'created db & applied schema';
})();
  </div>
</div>

And finally, write then read some data to check it out:

<div class="runnable-code">
  <div>
const items = ['Milk', 'Bread', 'Peas', 'Carrots', 'Lentils', 'Cake', 'Squirrels'];
return (async () => {
  await db.exec(
    `INSERT INTO todo (id, text, completed) VALUES (
      '${nanoid()}',
      'Buy ${items[(Math.random() * items.length) | 0]}',
      0
    )`);
  return db.execO("SELECT * FROM todo");
})();
  </div>
</div>

You can also see a more detailed setup guide on ObservableHQ: https://observablehq.com/@tantaman/cr-sqlite-basic-setup