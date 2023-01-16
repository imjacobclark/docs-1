---
layout: docs
title: Writing a networking layer
---

[[docs/guide-sync]] showed you how to sync changes from one database to another using the primitives of [[docs/crsql_changes]] and [[docs/crsql_tracked_peers]]. That guide, however, was interacting when both databases can be mounted in the same process. In this guide we'll discuss how to sync database state over a network.

# Network Agnostic

`cr-sqlite` is technically network agnostic. The reason is that there are many different configurations you may want for your network layer. Such as:

- P2P vs client-server
- Push vs pull
- TCP vs UDP vs WebSockets vs other

All those considerations aside, `vlcn` does provide two default networking layers. The first being a client-server setup and the second being peer 2 peer. Both allow you to pick your transport.

# Client-Server Networking

Both the client and server are pushed based.

Whenever a client makes a change locally, it will send that change to the server.

Whenever the server receives a change from a client it will broadcast that change to all other connected clients.

![client-server-forward](../assets/docs/guides/client-server-forward.svg)

Thus everyone is kept in sync as writes are made. If a client has been offline for a while, it will:

1. Send all changes that it has made since it was last online to the server
2. Receive all changes that it has missed, while offline, from the server

If the server is offline clients can continue to make changes locally. When the server comes back, they will send all changes they made while the server was offline and the server will re-broadcast those changes to clients as they connect.

The client and server are currently written in `TypeScript`. Native versions are on the roadmap which will be cross-compiled and includable as libraries from any language.

- Reference sync services have been built and a production grade versions are on the roadmap
  - [p2p reference](https://github.com/vlcn-io/cr-sqlite/tree/main/js/sync-reference/p2p)
    - [usage](https://github.com/vlcn-io/cr-sqlite/tree/main/js/examples/p2p-todomvc)
  - client-server reference
    - [usage](https://github.com/vlcn-io/cr-sqlite/tree/main/js/examples/todomvc)
    - [client](https://github.com/vlcn-io/cr-sqlite/tree/main/js/sync-reference/client)
    - [server](https://github.com/vlcn-io/cr-sqlite/tree/main/js/sync-reference/server)

