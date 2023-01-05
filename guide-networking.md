---
layout: docs
title: Writing a networking layer
---

[[docs/guide-sync]] showed you how to sync changes from one database to another using the primitives of [[docs/crsql_changes]] and [[docs/crsql_tracked_peers]]. That guide, however, was interacting when both databases can be mounted in the same process. In this guide we'll discuss how to sync database state over a network.

TODO: split into:
- using an existing networking layer
- writing your own networking layer

# Topology

You can sync databases in a Peer2Peer fashion or in a client-server setup. We'll cover writing a client-server networking layer followed by peer 2 peer.

# Client-Server

Within client-server there is another choice:

1. Send updates via a push model
2. Send updates via a pull model

We'll assume we're using a push model via [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API).

## The Client

The client will be responsible for configuring two data streams:

1. The client to server stream
2. The server to client stream

These change streams start at a given point in time -- the last time the client sent changes to the server and the last time the server sent changes to the client. The client will need to track these points in time in the [[docs/crsql_tracked_peers]] table in order to configure the streams.

> One hard requirement for networking layers is that they do not record having seen a clock value from a peer until they've persisted all changes prior and up to that clock value. If a networking layer records itself as having processed change 10 when it only processed up to change 5, all changes between 5 and 10 could be lost.



# Note:
- There is not yet an off the shelf sync server that you can just grab and use
- Prototype sync services have been built and a production grade sync server is on the roadmap
  - [p2p prototype](https://github.com/vlcn-io/cr-sqlite/tree/main/js/sync-reference/p2p)
    - [usage](https://github.com/vlcn-io/cr-sqlite/tree/main/js/examples/p2p-todomvc)
  - client-server prototype
    - [usage](https://github.com/vlcn-io/cr-sqlite/tree/main/js/examples/todomvc)
    - [client](https://github.com/vlcn-io/cr-sqlite/tree/main/js/sync-reference/client)
    - [server](https://github.com/vlcn-io/cr-sqlite/tree/main/js/sync-reference/server)

