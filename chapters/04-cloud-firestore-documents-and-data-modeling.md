# 04. Cloud Firestore documents and data modeling

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Authentication and account lifecycle](./03-authentication-and-account-lifecycle.md) | [Notes index](../README.md) | [Next: Firestore reads, queries, indexes, and pagination](./05-firestore-reads-queries-indexes-and-pagination.md) |

## What Cloud Firestore stores

Cloud Firestore is a NoSQL document database. Data is organized as collections of documents. A document stores named fields and values, and a field can hold a string, number, boolean, timestamp, array, map, or a reference to another document.

A simple task might look like this:

~~~text
tasks
└── task_7f3a
    ├── title: "Review Firestore data modeling"
    ├── ownerId: "user_42"
    ├── status: "open"
    ├── priority: 2
    └── createdAt: timestamp
~~~

Collections contain documents. A collection cannot hold fields directly, and a document cannot contain another collection as a field. A document can have a subcollection beneath it. Firestore creates collection and document paths when data is written, so there is no separate command to create an empty collection.

Firestore is schemaless. Documents in one collection can technically have different fields, but keeping a consistent shape makes application code, validation, and queries easier to reason about. Decide which fields are required and what type each field should have.

## Paths and references

A collection path has an odd number of segments, while a document path has an even number.

~~~text
users
users/user_42
users/user_42/tasks
users/user_42/tasks/task_7f3a
~~~

In the modular JavaScript SDK, collection and document references describe locations. Creating a reference does not itself read or write data.

~~~js
import { collection, doc } from "firebase/firestore";
import { db } from "./lib/firebase.js";

const tasksCollection = collection(db, "tasks");
const taskDocument = doc(db, "tasks", "task_7f3a");

const userTasksCollection = collection(db, "users", "user_42", "tasks");
const userTaskDocument = doc(
  db,
  "users",
  "user_42",
  "tasks",
  "task_7f3a",
);
~~~

Use a stable document ID when the document maps naturally to an existing identifier, such as an authenticated user's UID. Use an automatically generated ID when the item is a new independent record and the client should not choose its identity.

## Choosing a data shape

A useful starting point is to list the screens and operations the app needs. For each screen, ask which records it must load, how it identifies them, and which fields it sorts or filters by.

For example, a task application might use top-level tasks:

~~~text
tasks/{taskId}
  ownerId
  title
  status
  createdAt
~~~

This shape makes a task addressable by one ID and can support queries across a user's tasks using ownerId. Chapter 5 covers those reads and query constraints.

A user-owned hierarchy can instead put tasks below each user:

~~~text
users/{userId}/tasks/{taskId}
  title
  status
  createdAt
~~~

This gives each user's tasks a clear path and can make per-user rules easier to express. Cross-user queries and collection-group queries have different setup and access implications, so choose the hierarchy based on real reads and rules.

### Embed small values or use a subcollection

Keep a small, bounded group of values inside one document when the values are normally read and changed together. For example, a user profile can have a display name and a few preferences as fields.

Use a subcollection when the child records can grow, need independent updates, or are commonly read separately. Do not put an unbounded activity history into a single array. A growing array makes each update touch the parent document and eventually becomes awkward to load or maintain.

Deleting a parent document does not automatically delete its subcollections. If a user document contains a tasks subcollection, deleting the user document leaves those task documents behind unless the application or a trusted server cleanup process deletes them too.

### Duplicate data when it serves a read

Firestore does not provide relational joins. Sometimes a list needs a small snapshot of related information, such as the author's display name beside a post. Copying that display name into the post can avoid an extra read for every row.

Duplication is a tradeoff. Decide which copy is authoritative and how updates propagate. Avoid duplicating sensitive fields, and do not assume a copied value is always current.

## Create and replace documents

Import the functions your file uses from firebase/firestore. The example assumes db is the initialized Firestore instance from chapter 2.

~~~js
import {
  addDoc,
  collection,
  doc,
  serverTimestamp,
  setDoc,
} from "firebase/firestore";
import { db } from "./lib/firebase.js";

export async function createTask(ownerId, title) {
  const tasks = collection(db, "tasks");

  const taskRef = await addDoc(tasks, {
    ownerId,
    title: title.trim(),
    status: "open",
    createdAt: serverTimestamp(),
  });

  return taskRef.id;
}

export async function saveUserProfile(user) {
  const profileRef = doc(db, "users", user.uid);

  await setDoc(profileRef, {
    displayName: user.displayName ?? "",
    email: user.email ?? "",
    updatedAt: serverTimestamp(),
  });

  return profileRef;
}
~~~

addDoc creates a document with an automatically generated ID and writes its fields. setDoc writes to the specific path you choose. By default, setDoc replaces the document's fields. Passing merge: true preserves fields that are not included in the update:

~~~js
await setDoc(
  doc(db, "users", user.uid),
  { displayName: "Ada Lovelace" },
  { merge: true },
);
~~~

Use serverTimestamp when the recorded time should come from the server rather than relying on a device clock. A server timestamp is resolved as part of the write.

## Read, update, and delete one document

A document reference can be read with getDoc. Check exists() before using data, because a valid path does not guarantee a document exists.

~~~js
import {
  deleteDoc,
  doc,
  getDoc,
  serverTimestamp,
  updateDoc,
} from "firebase/firestore";
import { db } from "./lib/firebase.js";

export async function getTask(taskId) {
  const taskRef = doc(db, "tasks", taskId);
  const snapshot = await getDoc(taskRef);

  if (!snapshot.exists()) {
    return null;
  }

  return { id: snapshot.id, ...snapshot.data() };
}

export async function renameTask(taskId, title) {
  const taskRef = doc(db, "tasks", taskId);

  await updateDoc(taskRef, {
    title: title.trim(),
    updatedAt: serverTimestamp(),
  });
}

export async function removeTask(taskId) {
  await deleteDoc(doc(db, "tasks", taskId));
}
~~~

updateDoc changes named fields and fails if the document does not exist. setDoc with merge is useful when the code should create the document if missing. deleteDoc removes the document at that path, but does not remove nested subcollections automatically.

A field path can update a nested map value without replacing the whole map. Use the field-path helper when keys contain dots or when the path needs to be unambiguous.

~~~js
import { doc, updateDoc } from "firebase/firestore";

await updateDoc(doc(db, "users", userId), {
  "preferences.theme": "dark",
});
~~~

## Store data with predictable types

Use timestamps for moments in time, numbers for numeric values, booleans for true or false, and arrays or maps only when their size and shape are controlled. A timestamp is easier to sort consistently than a date string with mixed formats.

~~~js
import { serverTimestamp, setDoc } from "firebase/firestore";

await setDoc(doc(db, "tasks", taskId), {
  title: "Practice data modeling",
  status: "open",
  labels: ["firebase", "database"],
  settings: {
    remindersEnabled: false,
  },
  createdAt: serverTimestamp(),
});
~~~

Keep fields used by common filters and ordering available in a consistent form. For example, if some tasks use status: "open" and others use completed: false, screens need extra logic and queries may not match the intended records.

Do not store secrets in client-readable documents. A field hidden by the interface is still available to a client that can read the document. Security Rules and server-side authorization are covered in chapter 6.

## Hands-on exercise: a personal task record

Use a development Firebase project or the Emulator Suite. Do not use production user data.

1. Add a task with an ownerId, trimmed title, status, and server timestamp.
2. Save the generated document ID and read the document by that ID.
3. Update its title and confirm the status remains unchanged.
4. Create a user document with a stable UID as its document ID.
5. Add two small preference fields using setDoc with merge enabled.
6. Create one task in a user's subcollection and inspect its full path.
7. Delete a parent document in a disposable test path and verify whether its subcollection still exists.
8. Write down which screen or query each proposed field supports.

A strong model gives every record a clear identity, keeps repeated fields consistent, and makes expected reads practical without placing an unbounded amount of data in one document.

## Common mistakes

- Treating a collection like a table that must be manually created first. Collections appear when documents are written.
- Assuming every document in a collection can have a different shape without extra cost. Consistency makes queries and UI code simpler.
- Using addDoc when a stable ID is required, such as the signed-in user's UID.
- Using updateDoc for a document that may not exist. It rejects instead of creating it.
- Replacing a whole document with setDoc when only a few fields should change. Use merge when appropriate.
- Storing unbounded child records in one large array.
- Expecting deletion of a parent document to cascade into subcollections.
- Treating client-side validation as a security boundary. Security Rules must enforce the allowed reads and writes.

## Practice questions

1. What is the difference between a collection and a document?
2. Which Firestore value should represent a moment in time?
3. When would a stable document ID be useful?
4. What is the difference between addDoc, setDoc, and updateDoc?
5. Why should documents in the same collection usually share a consistent field shape?
6. When is a subcollection a better fit than an array field?
7. What happens to subcollection documents when the parent document is deleted?
8. When can duplicating a small field make a screen simpler, and what update decision must accompany that duplication?

## Main references

- [Cloud Firestore data model](https://firebase.google.com/docs/firestore/data-model)
- [Add data to Cloud Firestore](https://firebase.google.com/docs/firestore/manage-data/add-data)
- [Get started with Cloud Firestore](https://firebase.google.com/docs/firestore/quickstart)
- [JavaScript SDK reference](https://firebase.google.com/docs/reference/js/firestore_)
