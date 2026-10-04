# 05. Firestore reads, queries, indexes, and pagination

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md) | [Notes index](../README.md) | [Next: Firestore Security Rules and authorization](./06-firestore-security-rules-and-authorization.md) |

## Start with the read you need

Firestore supports one-time reads and live listeners. Use a one-time read when a screen needs a snapshot of current data. Use a listener when the interface should react to later changes, and unsubscribe when that screen or component is no longer active.

A document read returns a DocumentSnapshot. A query read returns a QuerySnapshot containing the matching documents. Check whether a document exists before reading its data, and map query documents into plain objects when the UI needs an ID alongside each record.

~~~js
import {
  collection,
  doc,
  getDoc,
  getDocs,
} from "firebase/firestore";
import { db } from "./lib/firebase.js";

export async function readTask(taskId) {
  const snapshot = await getDoc(doc(db, "tasks", taskId));

  if (!snapshot.exists()) {
    return null;
  }

  return { id: snapshot.id, ...snapshot.data() };
}

export async function readAllTasks() {
  const snapshot = await getDocs(collection(db, "tasks"));

  return snapshot.docs.map((taskDocument) => ({
    id: taskDocument.id,
    ...taskDocument.data(),
  }));
}
~~~

Reading an entire collection is usually not a good default for a growing dataset. Build a query that returns only the records the screen needs, and limit the result.

## Build queries with constraints

The modular SDK composes a query from a collection reference and constraints. A query does not change stored data.

~~~js
import {
  collection,
  getDocs,
  limit,
  orderBy,
  query,
  where,
} from "firebase/firestore";
import { db } from "./lib/firebase.js";

export async function readOpenTasks(ownerId) {
  const tasksQuery = query(
    collection(db, "tasks"),
    where("ownerId", "==", ownerId),
    where("status", "==", "open"),
    orderBy("createdAt", "desc"),
    limit(20),
  );

  const snapshot = await getDocs(tasksQuery);

  return snapshot.docs.map((taskDocument) => ({
    id: taskDocument.id,
    ...taskDocument.data(),
  }));
}
~~~

Common constraints include:

- where for equality, range, and supported array membership checks.
- orderBy for a predictable sort order.
- limit to bound the number of returned documents.
- startAt, startAfter, endAt, and endBefore to define a range or cursor.

A basic prefix search can use a range over a consistently normalized field, but Firestore does not provide general full-text search. For typo tolerance, relevance ranking, stemming, and larger search experiences, use a search service designed for those requirements.

## Understand ordering and missing fields

orderBy sorts matching documents by a field. Documents that do not contain the ordered field are not included in that query. Keep queried fields present and consistently typed across the documents that should appear.

~~~js
import {
  collection,
  getDocs,
  limit,
  orderBy,
  query,
} from "firebase/firestore";

const newestTasksQuery = query(
  collection(db, "tasks"),
  orderBy("createdAt", "desc"),
  limit(10),
);

const newestTasks = await getDocs(newestTasksQuery);
~~~

A stable order matters for pagination. If many records have the same timestamp, the document snapshot cursor still identifies the position in the result. If constructing a cursor from field values, include enough ordered fields to distinguish records.

## Indexes

Firestore uses indexes to serve queries. It creates many single-field indexes automatically. A query combining filters and ordering may need a composite index. When a query needs an index, the error message includes a link to create the required index in the Firebase console.

Treat the index as part of the query design. Review the fields and sort directions before creating it, and keep the index definition with the project when using a checked-in index configuration.

~~~text
Collection: tasks
Fields:
  ownerId      Ascending
  status       Ascending
  createdAt    Descending
~~~

Indexes have storage and write costs. Avoid adding indexes for fields that the application never filters or sorts on. Exempt large fields from indexing when they are not queried, using the current Firebase guidance for the data shape.

## Cursor pagination

Do not fetch every matching record and slice the array in the browser. Ask Firestore for a bounded page, then use the final document snapshot as the next cursor.

~~~js
import {
  collection,
  getDocs,
  limit,
  orderBy,
  query,
  startAfter,
  where,
} from "firebase/firestore";
import { db } from "./lib/firebase.js";

const pageSize = 20;

export async function readTaskPage(ownerId, lastDocument = null) {
  const constraints = [
    where("ownerId", "==", ownerId),
    orderBy("createdAt", "desc"),
    limit(pageSize),
  ];

  if (lastDocument) {
    constraints.splice(constraints.length - 1, 0, startAfter(lastDocument));
  }

  const pageQuery = query(collection(db, "tasks"), ...constraints);
  const snapshot = await getDocs(pageQuery);

  return {
    items: snapshot.docs.map((taskDocument) => ({
      id: taskDocument.id,
      ...taskDocument.data(),
    })),
    nextCursor: snapshot.docs.at(-1) ?? null,
    hasMore: snapshot.size === pageSize,
  };
}
~~~

Keep the returned nextCursor for the next request. Pass it back as lastDocument to load the following page. Reset the cursor when the filter or sort order changes. A full final page does not prove another page exists, so an app that needs an exact answer can request one more document or handle an empty next page.

Cursors are preferable to offsets for page navigation. An offset skips documents after they are read internally, which can add cost and latency as the offset grows.

## Live query listeners

A listener emits an initial snapshot and later snapshots as matching data changes. The registration function returns an unsubscribe function.

~~~js
import {
  collection,
  onSnapshot,
  query,
  where,
} from "firebase/firestore";
import { db } from "./lib/firebase.js";

export function watchTasks(ownerId, onTasks, onError) {
  const tasksQuery = query(
    collection(db, "tasks"),
    where("ownerId", "==", ownerId),
  );

  return onSnapshot(
    tasksQuery,
    (snapshot) => {
      const tasks = snapshot.docs.map((taskDocument) => ({
        id: taskDocument.id,
        ...taskDocument.data(),
      }));

      onTasks(tasks);
    },
    onError,
  );
}

// In a component cleanup:
const stopWatching = watchTasks(userId, setTasks, setError);
// Later, when the view is removed:
stopWatching();
~~~

Use listeners only where the user benefits from live updates. A listener can produce repeated reads as documents change, so consider the screen lifecycle, expected traffic, and current billing model.

## Queries and Security Rules

Firestore Security Rules are not filters. A query must be written so that every possible document in its result satisfies the rule. Firestore rejects a query that could return even one document the caller is not allowed to read.

For example, if access is allowed only when a task's ownerId equals the signed-in user's UID, include an ownerId equality constraint in the query. Hiding another user's records in the interface does not secure them. Chapter 6 covers rules and query authorization in detail.

## Hands-on exercise: paginated task list

Use a development project or the Emulator Suite with sample records.

1. Create at least 25 task documents with ownerId, status, and createdAt fields.
2. Read only one document by its known ID and handle a missing document.
3. Query open tasks for one owner and sort newest first.
4. Add a page limit of 5 and return the final snapshot as nextCursor.
5. Load a second page using startAfter and confirm that no document appears in both pages.
6. Change the status filter and reset the cursor before loading again.
7. Add a live listener to a test screen and unsubscribe when the screen is removed.
8. If Firestore reports a missing index, inspect the suggested fields and add the index needed by this query.
9. Compare the query constraints with the Security Rules you plan to write in chapter 6.

## Common mistakes

- Loading a full collection when the interface only needs a small page.
- Forgetting to check exists() for a document read.
- Ordering by a field that some intended documents do not contain.
- Reusing an old cursor after the filter or sort order changes.
- Registering a listener without calling its unsubscribe function.
- Treating Security Rules as post-query filters.
- Adding more filters and order constraints without checking whether a composite index is required.
- Expecting a basic range query to provide full-text search.

## Practice questions

1. What is the difference between getDoc and getDocs?
2. What does a QuerySnapshot contain?
3. Why should a growing collection query include a limit?
4. What does orderBy do when a document lacks the ordered field?
5. When might Firestore request a composite index?
6. How does a document snapshot work as a cursor?
7. Why should a live listener be unsubscribed?
8. Why must the query itself satisfy the Firestore Security Rules?

## Main references

- [Get data with Cloud Firestore](https://firebase.google.com/docs/firestore/query-data/get-data)
- [Order and limit data](https://firebase.google.com/docs/firestore/query-data/order-limit-data)
- [Paginate data with query cursors](https://firebase.google.com/docs/firestore/query-data/query-cursors)
- [Manage Cloud Firestore indexes](https://firebase.google.com/docs/firestore/query-data/indexing)
- [Securely query data](https://firebase.google.com/docs/firestore/security/rules-query)
