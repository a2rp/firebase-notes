# 07. Realtime Database and live synchronization

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Firestore Security Rules and authorization](./06-firestore-security-rules-and-authorization.md) | [Notes index](../README.md) | [Next: Cloud Storage for files](./08-cloud-storage-for-files.md) |

## What Realtime Database stores

Firebase Realtime Database stores JSON data as one large tree. Each value lives at a path, and child paths form a hierarchy. The JavaScript SDK uses references to read and write paths.

~~~text
/
├── users
│   └── user_42
│       ├── displayName: "Ada"
│       └── online: true
└── rooms
    └── room_7
        └── messages
            └── message_91
                ├── authorId: "user_42"
                └── text: "Hello"
~~~

Realtime Database and Firestore both support realtime updates, but their data models differ. Realtime Database organizes one JSON tree and is useful for low-latency synchronization patterns. Firestore organizes documents and collections and provides document queries. Pick a product based on the access patterns, data shape, and product requirements rather than assuming they are interchangeable.

## Initialize and reference a path

Add the Realtime Database service to the shared Firebase app setup from chapter 2. If you use a regional database instance, configure its URL from the Firebase console.

~~~js
import { getDatabase } from "firebase/database";
import { app } from "./lib/firebase.js";

export const database = getDatabase(
  app,
  "https://YOUR_DATABASE_URL",
);
~~~

A reference identifies a path. It does not fetch the value until a read operation or listener is attached.

~~~js
import { ref } from "firebase/database";
import { database } from "./lib/realtime-database.js";

const usersRef = ref(database, "users");
const currentUserRef = ref(database, "users/user_42");
const messagesRef = ref(database, "rooms/room_7/messages");
~~~

Use the exact database URL shown for your instance. Avoid writing unrelated application data at the root because root operations can transfer and overwrite much more data than a feature needs.

## Write values and partial updates

set replaces all data at the referenced path and all child paths below it. Use it when the application intends to define the complete value at that path.

~~~js
import { ref, set } from "firebase/database";
import { database } from "./lib/realtime-database.js";

export async function saveProfile(userId, profile) {
  await set(ref(database, "users/" + userId), {
    displayName: profile.displayName,
    photoURL: profile.photoURL ?? null,
  });
}
~~~

update changes only the named child values. It can also update multiple paths in one operation.

~~~js
import { ref, update } from "firebase/database";
import { database } from "./lib/realtime-database.js";

export async function updateProfile(userId, displayName) {
  await update(ref(database), {
    ["users/" + userId + "/displayName"]: displayName,
    ["users/" + userId + "/updatedAt"]: Date.now(),
  });
}
~~~

The multi-path update is useful when the same fact is intentionally stored in more than one location. Plan how duplicate values stay consistent, and protect every path with rules. Do not let a client update arbitrary paths simply because one update call contains several paths.

Use push to generate a unique child key for list-like records. The generated key can be created before the value is written.

~~~js
import { push, ref, set } from "firebase/database";
import { database } from "./lib/realtime-database.js";

export async function addMessage(roomId, userId, text) {
  const messagesRef = ref(database, "rooms/" + roomId + "/messages");
  const messageRef = push(messagesRef);

  await set(messageRef, {
    authorId: userId,
    text: text.trim(),
    createdAt: Date.now(),
  });

  return messageRef.key;
}
~~~

Remove data with remove at the narrowest required path. Treat deletion as an intentional operation because removing a parent removes its descendants.

## Read once or listen for changes

Use get when a value is needed once. Check exists before using val, because an empty path returns no data.

~~~js
import { get, ref } from "firebase/database";
import { database } from "./lib/realtime-database.js";

export async function getProfile(userId) {
  const snapshot = await get(ref(database, "users/" + userId));

  if (!snapshot.exists()) {
    return null;
  }

  return snapshot.val();
}
~~~

Use onValue when the interface needs to react to changes at a path. The initial callback contains the current value, and later changes trigger another callback. Attach the listener as low in the tree as the feature allows.

~~~js
import { onValue, ref } from "firebase/database";
import { database } from "./lib/realtime-database.js";

export function watchMessages(roomId, onMessages, onError) {
  const messagesRef = ref(database, "rooms/" + roomId + "/messages");

  return onValue(
    messagesRef,
    (snapshot) => {
      const messages = [];

      snapshot.forEach((messageSnapshot) => {
        messages.push({
          id: messageSnapshot.key,
          ...messageSnapshot.val(),
        });
      });

      onMessages(messages);
    },
    onError,
  );
}

// Stop listening when the view is removed:
const stopWatching = watchMessages(roomId, setMessages, setError);
stopWatching();
~~~

A listener at the database root can download a very large tree whenever any descendant changes. Listen at the smallest path that gives the screen the data it needs.

## Query and order a bounded list

Realtime Database queries need an ordering method and can use range or limit constraints. For example, order messages by creation time and read only the newest 50.

~~~js
import {
  limitToLast,
  onValue,
  orderByChild,
  query,
  ref,
} from "firebase/database";
import { database } from "./lib/realtime-database.js";

const recentMessagesQuery = query(
  ref(database, "rooms/room_7/messages"),
  orderByChild("createdAt"),
  limitToLast(50),
);

const stopWatchingRecentMessages = onValue(
  recentMessagesQuery,
  (snapshot) => {
    const messages = [];

    snapshot.forEach((messageSnapshot) => {
      messages.push({
        id: messageSnapshot.key,
        ...messageSnapshot.val(),
      });
    });

    renderMessages(messages);
  },
);
~~~

Realtime Database returns the limited result in query order. If the interface needs newest first, reverse a copied array rather than mutating shared state unexpectedly. Add an index in Realtime Database Rules for fields used by orderByChild when required.

## Use a transaction for concurrent changes

A read followed by a write can lose updates when two clients change the same value at the same time. Use a transaction when the next value depends on the current value.

~~~js
import { ref, runTransaction } from "firebase/database";
import { database } from "./lib/realtime-database.js";

export async function addOneToCounter(counterId) {
  const counterRef = ref(database, "counters/" + counterId);

  const result = await runTransaction(counterRef, (currentValue) => {
    return (currentValue ?? 0) + 1;
  });

  return result.snapshot.val();
}
~~~

The transaction callback may run more than once while the SDK resolves concurrent writes. Keep it free of side effects such as sending an email or showing a notification. Perform those actions only after the transaction result is known.

## Presence with connection state and onDisconnect

The special .info/connected path reports whether this client is connected to the Realtime Database server. onDisconnect registers an operation that the server performs if the client connection closes. Register the disconnect action before writing the online state.

~~~js
import {
  onDisconnect,
  onValue,
  ref,
  remove,
  serverTimestamp,
  set,
} from "firebase/database";
import { database } from "./lib/realtime-database.js";

export function watchPresence(userId) {
  const connectionRef = ref(database, ".info/connected");
  const statusRef = ref(database, "status/" + userId);

  return onValue(connectionRef, async (snapshot) => {
    if (snapshot.val() !== true) {
      return;
    }

    try {
      await onDisconnect(statusRef).remove();
      await set(statusRef, {
        state: "online",
        lastChanged: serverTimestamp(),
      });
    } catch (error) {
      console.error("Could not update presence", error);
    }
  });
}
~~~

Presence is a useful signal, not a permanent fact. Devices can lose power or network access, and clients can reconnect. Treat status as time-sensitive and design the UI around transient state.

## Protect paths with Realtime Database Rules

Realtime Database Rules define read and write access at paths in the JSON tree. A rule can use the authenticated user's UID to protect each user's record.

~~~json
{
  "rules": {
    "users": {
      "$userId": {
        ".read": "auth != null && auth.uid === $userId",
        ".write": "auth != null && auth.uid === $userId"
      }
    }
  }
}
~~~

Rules can also validate values and declare indexes for query fields. For example, if clients order records by createdAt, declare an index at the collection path:

~~~json
{
  "rules": {
    "rooms": {
      "$roomId": {
        "messages": {
          ".indexOn": ["createdAt"]
        }
      }
    }
  }
}
~~~

Do not publish rules that grant broad unauthenticated access to private data. Test both authorized and unauthorized paths with the Realtime Database emulator. Authentication and data validation must be enforced by rules, not only by the interface.

## Hands-on exercise: live room messages

1. Create a development Realtime Database instance and copy its URL into local configuration.
2. Write one profile at users/{userId} and read it once.
3. Add three messages beneath a room using push-generated keys.
4. Attach a listener only to that room's messages path.
5. Add a message from a second test client and observe the list update.
6. Stop the listener when the view is removed.
7. Query the newest messages by createdAt and add the required index.
8. Increment a shared counter with runTransaction from two clients.
9. Test the user path with the matching UID, another UID, and a signed-out client.
10. Use the emulator before changing any deployed rules.

## Common mistakes

- Treating the JSON tree like a relational table without planning path-based reads.
- Replacing a parent value with set when the intention was to update one child.
- Listening at the root and downloading unrelated data.
- Using a read followed by a write for a counter that many clients can update.
- Running side effects inside a transaction callback.
- Assuming a successful client write proves the rules are secure.
- Forgetting to unsubscribe from a listener.
- Treating presence as a durable statement that a user is active.
- Forgetting query indexes for fields used to order data.

## Practice questions

1. How is Realtime Database data organized?
2. What happens to child paths when set replaces a parent value?
3. When should update be used instead of set?
4. What does onValue do after its initial callback?
5. Why should a listener be attached low in the data tree?
6. When is runTransaction safer than reading and then writing?
7. What does onDisconnect allow a client to register?
8. How do Realtime Database Rules use an authenticated user's UID?

## Main references

- [Read and write data on the web](https://firebase.google.com/docs/database/web/read-and-write)
- [Structure your database](https://firebase.google.com/docs/database/web/structure-data)
- [Realtime Database offline capabilities](https://firebase.google.com/docs/database/web/offline-capabilities)
- [Realtime Database Security Rules](https://firebase.google.com/docs/database/security)
- [Firebase Local Emulator Suite](https://firebase.google.com/docs/emulator-suite)
