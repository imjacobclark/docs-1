---
layout: docs
title: Cross Tab Comms
---

<style type="text/css">
@import url("../assets/interactive/runnable-code.css");
@import url("../assets/interactive/json-viewer.css");
@import url("../assets/docs/guides/solving-tabs.css");
</style>

<script type="module" src="../assets/docs/guides/common.js"></script>

An amazing benefit of storing your application state in `SQLite` is that you get cross tab sync mostly for free. Let's take a look at how this works.

> Note: no CRDTs or CRRs are required for cross tab sync given all tabs are operating locally against the same database file. Given that, this guide will mostly cover run of the mill SQLite features.

Start by importing the cr-sqlite wasm bundle

<div class="runnable-code">
  <div>
return import('https://esm.sh/@vlcn.io/wa-crsqlite@0.6.3').then(async (initWasm) => {
  return self.sqlite = await initWasm.default(() => "https://esm.sh/@vlcn.io/wa-crsqlite@0.6.3/dist/wa-sqlite-async.wasm");
});
  </div>
</div>

then create a database and table to store our demo state.

<div class="runnable-code">
  <div>
return (async () => {
  self.db = await sqlite.open("guide-tabs.db");
  window.onbeforeunload = () => {
    return db.close();
  };
  await db.exec(
    "CREATE TABLE IF NOT EXISTS note (id primary key, content)"
  );
  return 'created db & applied schema';
})();
  </div>
</div>

Now lets stick a row in there.

<div class="runnable-code">
  <div>
return db.exec(`INSERT OR IGNORE INTO note VALUES (1, 'Type something');`);
  </div>
</div>

Next, we'll need a UI to display and update our note. A `textarea` has been added to this document further down so let's attach event listeners to it. In the listener we'll write to the database when the textarea changes.

For a sophisticated editor you'd want to save deltas and not the entire document contents on each edit.

<div class="runnable-code">
  <div>
self.notepad = document.getElementById("notepad");
notepad.oninput = () => {
  return db.exec(`UPDATE note SET content = '${notepad.value}' WHERE id = 1`);
};
  </div>
</div>

Finally, subscribe to database changes. One additional trick we throw in there is to forward db change events to a broadcast channel so other tabs can be made aware when the underlying database has changed.

<div class="runnable-code">
  <div>
const bc = new BroadcastChannel("tabs-demo");
const onDbUpdate = (
  updateType,
  db,
  tableName,
  rowid,
  fromBc
) => {
  if (!fromBc) {
    bc.postMessage({updateType, db, tableName, rowid});
    return;
  }
  if (tableName === "note" && rowid === 1n) {
    updateNote();
  }
};
bc.onmessage = (msg) => {
  const data = msg.data;
  onDbUpdate(data.updateType, data.db, data.tableName, data.rowid, true);
}
const updateNote = async () => {
  const note = await db.execO(
    "SELECT content FROM note WHERE id = 1"
  );
  notepad.value = note[0].content;
};
db.onUpdate(onDbUpdate);
updateNote();
  </div>
</div>

To see that everything works, open this page in multiple tabs or windows and start typing away into the notepad below. You should see the changes propagate to all tabs.

<textarea id="notepad"></textarea>

# Doing Better

This showed you the base primitives (`db.onUpdate` and `BroadcastChannel`) that you can use to keep all tabs in sync. Integration with React, Svelte and other UI frameworks make this vastly simpler.

For example, in React you can do:

```
const allTodos: Todo[] = useQuery<Todo>(
  ctx,
  ["todo"],
  "SELECT * FROM todo ORDER BY id DESC"
).data;
```

to subscribe a component to a query. This method is used in the [todomvc example](https://github.com/vlcn-io/cr-sqlite/tree/main/js/examples/todomvc).
