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

# The Hoops

## COOP and COEP Headers

Given the use of `Atomics`, the first hoop that must be jumped through is to enable COOP and COEP headers. This will let SQLite persist to the filesystem when run in a web-worker.

```
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
```

If you only use `SQLite` from within that worker then the interface to query `SQLite` is unchanged.

If you need to access `SQLite` from the main UI thread, you'll need to pass messages from the UI to the worker and back.

> You can read more about `SQLite` persistence in the browser at the SQLite website: https://sqlite.org/wasm/doc/trunk/persistence.md

## Message Passing

Given that once you enable persistence for SQLite (beyond simple kvvfs persistence) it'll be running in a worker. To get data out of that worker and to the UI thread you'll need to do some message passing.

To make this easy, the `vlcn` project provides a [Comlink](https://github.com/GoogleChromeLabs/comlink)-based interface for this. Lets see how that works.

# Comlink API

TODO: this API needs major cleaning up and to be made compatible with the `xplat-api` interface.

<div class="runnable-code">
  <div>
// if you're using TS you can import type defs:
// import { API } from "@vlcn.io/crsqlite-wasm/dist/comlinkable";
return new Promise(async (resolve, reject) => {
  const Comlink = await import("https://esm.sh/comlink@4.3.1");
  // spawn a comlink-compatible worker for SQLite
  // https://esm.sh/@vlcn.io/crsqlite-wasm@0.5.7/dist/comlinked.js
  const worker = new Worker('/assets/interactive/crsqlite-wasm/dist/comlinked.js', { type: 'module' });
  self.sqlite = Comlink.wrap(worker);
  sqlite.onReady(
    {
      wasmUrl: "https://esm.sh/@vlcn.io/crsqlite-wasm@0.5.7/dist/sqlite3.wasm",
      // https://esm.sh/@vlcn.io/crsqlite-wasm@0.5.7/dist/sqlite3-opfs-async-proxy.js
      proxyUrl: "/assets/interactive/crsqlite-wasm/dist/sqlite3-opfs-async-proxy.js",
    },
    Comlink.proxy(onReady),
    Comlink.proxy(onError)
  );
  async function onReady() {
    self.db = await sqlite.open("comlinked-persist");
    window.onbeforeunload = () => {
      return sqlite.close(db);
    };
    resolve(db);
  }
  function onError(e) {
    reject(e);
  }
});
  </div>
</div>

Now lets write some data to be sure it all worked.

<div class="runnable-code">
  <div>
return (async () => {
  await sqlite.execMany(db, [
    "CREATE TABLE IF NOT EXISTS foo (a, b);",
    "INSERT INTO foo VALUES (1, 2), (3, 4);",
  ]);

  return await sqlite.execO(db, "SELECT * FROM foo");
})();
  </div>
</div>

Full code / standalone example: https://github.com/vlcn-io/cr-sqlite/blob/main/js/examples/basic-browser-setup/src/comlinked-official.ts