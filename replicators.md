---
layout: docs
title: Replicatiors
---

A client-server sync implementation in `TypeScript` is in progress here:

- [client](https://github.com/vlcn-io/cr-sqlite/tree/main/js/sync-reference/client)
- [server](https://github.com/vlcn-io/cr-sqlite/tree/main/js/sync-reference/server)
- [example app](https://github.com/vlcn-io/cr-sqlite/tree/main/js/examples/todomvc)


A native sync client that can be loaded directly into `SQLite` is also in progress.

These replicators do "whole database" replication which is not ideal for many use cases. To replicate only a subset of your db, see [[docs/replicated-subgraphs]]

<video src="https://user-images.githubusercontent.com/1009003/206575697-90160393-1db7-4ff8-a576-ec3807a65477.mov" data-canonical-src="https://user-images.githubusercontent.com/1009003/206575697-90160393-1db7-4ff8-a576-ec3807a65477.mov" controls="controls" muted="muted" class="d-block rounded-bottom-2 border-top width-fit" style="max-height:640px;">

  </video>