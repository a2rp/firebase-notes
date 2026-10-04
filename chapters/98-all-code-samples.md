# 98. All code samples

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Billing, operations, debugging, and production checklist](./16-billing-operations-debugging-and-production-checklist.md) | [Notes index](../README.md) | [Next: Complete questions and answers](./99-complete-q-and-a.md) |

This appendix gathers every fenced code sample from the 16 core chapters. The source link above each sample points to its full explanation and practice context.

## 01. Firebase projects, apps, and product map

### Understand the project hierarchy

Source: [01. Firebase projects, apps, and product map](./01-firebase-projects-apps-and-product-map.md)

~~~text
Google Cloud project with Firebase enabled
|
+-- Firebase project settings and shared resources
|   +-- Authentication
|   +-- Cloud Firestore or Realtime Database
|   +-- Cloud Storage
|   +-- Cloud Functions and Hosting
|
+-- Registered Firebase apps
    +-- Web app
    +-- Android app
    +-- iOS app
~~~

### Separate development environments

Source: [01. Firebase projects, apps, and product map](./01-firebase-projects-apps-and-product-map.md)

~~~text
project-example-dev      local development and test data
project-example-staging  release-candidate checks
project-example-prod     real users and production data
~~~

## 02. JavaScript SDK setup and modular initialization

### Install the JavaScript SDK

Source: [02. JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md)

~~~bash
npm install firebase
~~~

### Install the JavaScript SDK

Source: [02. JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md)

~~~js
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
~~~

### Register the web app and copy its configuration

Source: [02. JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md)

~~~js
const firebaseConfig = {
  apiKey: "YOUR_WEB_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "COPY_THE_BUCKET_FROM_FIREBASE",
  messagingSenderId: "COPY_FROM_FIREBASE",
  appId: "COPY_FROM_FIREBASE"
};
~~~

### Initialize Firebase once in a shared module

Source: [02. JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md)

~~~js
// src/lib/firebase.js
import { getApp, getApps, initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";

const firebaseConfig = {
  apiKey: "YOUR_WEB_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "COPY_THE_BUCKET_FROM_FIREBASE",
  messagingSenderId: "COPY_FROM_FIREBASE",
  appId: "COPY_FROM_FIREBASE"
};

const app = getApps().length ? getApp() : initializeApp(firebaseConfig);

const auth = getAuth(app);
const db = getFirestore(app);

export { app, auth, db };
~~~

### Initialize Firebase once in a shared module

Source: [02. JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md)

~~~js
import { auth, db } from "./lib/firebase.js";

console.log(auth.app.name);
console.log(db.app.name);
~~~

### Keep each environment pointed at its own project

Source: [02. JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md)

~~~text
VITE_FIREBASE_API_KEY=your-development-web-key
VITE_FIREBASE_AUTH_DOMAIN=your-development-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-development-project
VITE_FIREBASE_APP_ID=your-development-app-id
~~~

### Keep each environment pointed at its own project

Source: [02. JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md)

~~~js
const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID
};
~~~

## 03. Authentication and account lifecycle

### Create and sign in with an email account

Source: [03. Authentication and account lifecycle](./03-authentication-and-account-lifecycle.md)

~~~js
import {
  createUserWithEmailAndPassword,
  getAuth,
  sendEmailVerification
} from "firebase/auth";

const auth = getAuth();

async function createAccount(email, password) {
  const credential = await createUserWithEmailAndPassword(
    auth,
    email,
    password
  );

  await sendEmailVerification(credential.user);
  return credential.user;
}
~~~

### Create and sign in with an email account

Source: [03. Authentication and account lifecycle](./03-authentication-and-account-lifecycle.md)

~~~js
import { getAuth, signInWithEmailAndPassword } from "firebase/auth";

const auth = getAuth();

async function signIn(email, password) {
  const credential = await signInWithEmailAndPassword(
    auth,
    email,
    password
  );

  return credential.user;
}
~~~

### Observe authentication state

Source: [03. Authentication and account lifecycle](./03-authentication-and-account-lifecycle.md)

~~~js
import { getAuth, onAuthStateChanged } from "firebase/auth";

const auth = getAuth();

const unsubscribe = onAuthStateChanged(auth, (user) => {
  if (user) {
    console.log("Signed in UID:", user.uid);
  } else {
    console.log("Signed out");
  }
});
~~~

### Choose how long the browser remembers a session

Source: [03. Authentication and account lifecycle](./03-authentication-and-account-lifecycle.md)

~~~js
import {
  browserSessionPersistence,
  getAuth,
  setPersistence,
  signInWithEmailAndPassword
} from "firebase/auth";

const auth = getAuth();

async function signInForThisTab(email, password) {
  await setPersistence(auth, browserSessionPersistence);
  return signInWithEmailAndPassword(auth, email, password);
}
~~~

### Sign out cleanly

Source: [03. Authentication and account lifecycle](./03-authentication-and-account-lifecycle.md)

~~~js
import { getAuth, signOut } from "firebase/auth";

const auth = getAuth();

async function signOutCurrentUser() {
  await signOut(auth);
}
~~~

### Add password recovery

Source: [03. Authentication and account lifecycle](./03-authentication-and-account-lifecycle.md)

~~~js
import { getAuth, sendPasswordResetEmail } from "firebase/auth";

const auth = getAuth();

async function requestPasswordReset(email) {
  await sendPasswordResetEmail(auth, email);
}
~~~

## 04. Cloud Firestore documents and data modeling

### What Cloud Firestore stores

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

~~~text
tasks
└── task_7f3a
    ├── title: "Review Firestore data modeling"
    ├── ownerId: "user_42"
    ├── status: "open"
    ├── priority: 2
    └── createdAt: timestamp
~~~

### Paths and references

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

~~~text
users
users/user_42
users/user_42/tasks
users/user_42/tasks/task_7f3a
~~~

### Paths and references

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

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

### Choosing a data shape

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

~~~text
tasks/{taskId}
  ownerId
  title
  status
  createdAt
~~~

### Choosing a data shape

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

~~~text
users/{userId}/tasks/{taskId}
  title
  status
  createdAt
~~~

### Create and replace documents

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

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

### Create and replace documents

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

~~~js
await setDoc(
  doc(db, "users", user.uid),
  { displayName: "Ada Lovelace" },
  { merge: true },
);
~~~

### Read, update, and delete one document

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

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

### Read, update, and delete one document

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

~~~js
import { doc, updateDoc } from "firebase/firestore";

await updateDoc(doc(db, "users", userId), {
  "preferences.theme": "dark",
});
~~~

### Store data with predictable types

Source: [04. Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md)

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

## 05. Firestore reads, queries, indexes, and pagination

### Start with the read you need

Source: [05. Firestore reads, queries, indexes, and pagination](./05-firestore-reads-queries-indexes-and-pagination.md)

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

### Build queries with constraints

Source: [05. Firestore reads, queries, indexes, and pagination](./05-firestore-reads-queries-indexes-and-pagination.md)

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

### Understand ordering and missing fields

Source: [05. Firestore reads, queries, indexes, and pagination](./05-firestore-reads-queries-indexes-and-pagination.md)

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

### Indexes

Source: [05. Firestore reads, queries, indexes, and pagination](./05-firestore-reads-queries-indexes-and-pagination.md)

~~~text
Collection: tasks
Fields:
  ownerId      Ascending
  status       Ascending
  createdAt    Descending
~~~

### Cursor pagination

Source: [05. Firestore reads, queries, indexes, and pagination](./05-firestore-reads-queries-indexes-and-pagination.md)

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

### Live query listeners

Source: [05. Firestore reads, queries, indexes, and pagination](./05-firestore-reads-queries-indexes-and-pagination.md)

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

## 06. Firestore Security Rules and authorization

### Start with a closed database

Source: [06. Firestore Security Rules and authorization](./06-firestore-security-rules-and-authorization.md)

~~~text
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
~~~

### Protect user-owned documents

Source: [06. Firestore Security Rules and authorization](./06-firestore-security-rules-and-authorization.md)

~~~text
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    function signedIn() {
      return request.auth != null;
    }

    function ownsTask() {
      return signedIn()
        && resource.data.ownerId == request.auth.uid;
    }

    match /tasks/{taskId} {
      allow get, list: if ownsTask();

      allow create: if signedIn()
        && request.resource.data.ownerId == request.auth.uid
        && request.resource.data.title is string
        && request.resource.data.title.size() > 0
        && request.resource.data.title.size() <= 200
        && request.resource.data.status in ['open', 'done'];

      allow update: if ownsTask()
        && request.resource.data.ownerId == resource.data.ownerId
        && request.resource.data.title is string
        && request.resource.data.title.size() > 0
        && request.resource.data.title.size() <= 200
        && request.resource.data.status in ['open', 'done'];

      allow delete: if ownsTask();
    }
  }
}
~~~

### Match nested paths carefully

Source: [06. Firestore Security Rules and authorization](./06-firestore-security-rules-and-authorization.md)

~~~text
match /users/{userId}/tasks/{taskId} {
  allow read, write: if request.auth != null
    && request.auth.uid == userId;
}
~~~

### Query with the rule in mind

Source: [06. Firestore Security Rules and authorization](./06-firestore-security-rules-and-authorization.md)

~~~js
import {
  collection,
  getDocs,
  query,
  where,
} from "firebase/firestore";
import { db } from "./lib/firebase.js";

export async function listMyTasks(user) {
  const tasksQuery = query(
    collection(db, "tasks"),
    where("ownerId", "==", user.uid),
  );

  const snapshot = await getDocs(tasksQuery);

  return snapshot.docs.map((taskDocument) => ({
    id: taskDocument.id,
    ...taskDocument.data(),
  }));
}
~~~

### Validate field changes

Source: [06. Firestore Security Rules and authorization](./06-firestore-security-rules-and-authorization.md)

~~~text
allow update: if request.auth != null
  && resource.data.ownerId == request.auth.uid
  && request.resource.data.ownerId == resource.data.ownerId
  && request.resource.data.diff(resource.data).affectedKeys()
       .hasOnly(['title', 'status', 'updatedAt']);
~~~

## 07. Realtime Database and live synchronization

### What Realtime Database stores

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Initialize and reference a path

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

~~~js
import { getDatabase } from "firebase/database";
import { app } from "./lib/firebase.js";

export const database = getDatabase(
  app,
  "https://YOUR_DATABASE_URL",
);
~~~

### Initialize and reference a path

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

~~~js
import { ref } from "firebase/database";
import { database } from "./lib/realtime-database.js";

const usersRef = ref(database, "users");
const currentUserRef = ref(database, "users/user_42");
const messagesRef = ref(database, "rooms/room_7/messages");
~~~

### Write values and partial updates

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Write values and partial updates

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Write values and partial updates

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Read once or listen for changes

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Read once or listen for changes

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Query and order a bounded list

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Use a transaction for concurrent changes

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Presence with connection state and onDisconnect

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Protect paths with Realtime Database Rules

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

### Protect paths with Realtime Database Rules

Source: [07. Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md)

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

## 08. Cloud Storage for files

### What Cloud Storage is for

Source: [08. Cloud Storage for files](./08-cloud-storage-for-files.md)

~~~text
Cloud Storage:
users/user_42/profile/avatar_8a21.webp

Cloud Firestore:
users/user_42
  displayName: "Ada"
  avatarPath: "users/user_42/profile/avatar_8a21.webp"
~~~

### Initialize the Storage service

Source: [08. Cloud Storage for files](./08-cloud-storage-for-files.md)

~~~js
import { getStorage } from "firebase/storage";
import { app } from "./lib/firebase.js";

export const storage = getStorage(app);
~~~

### Select and validate a file

Source: [08. Cloud Storage for files](./08-cloud-storage-for-files.md)

~~~js
export function validateImageFile(file) {
  const allowedTypes = new Set([
    "image/jpeg",
    "image/png",
    "image/webp",
  ]);
  const maximumBytes = 5 * 1024 * 1024;

  if (!allowedTypes.has(file.type)) {
    throw new Error("Choose a JPEG, PNG, or WebP image.");
  }

  if (file.size > maximumBytes) {
    throw new Error("Choose an image smaller than 5 MB.");
  }

  return file;
}
~~~

### Upload with progress and cancellation

Source: [08. Cloud Storage for files](./08-cloud-storage-for-files.md)

~~~js
import {
  ref,
  uploadBytesResumable,
} from "firebase/storage";
import { storage } from "./lib/storage.js";

export function uploadProfileImage(userId, file, onProgress) {
  const safeFile = validateImageFile(file);
  const objectPath = "users/" + userId + "/profile/" + crypto.randomUUID();
  const storageRef = ref(storage, objectPath);

  const uploadTask = uploadBytesResumable(storageRef, safeFile, {
    contentType: safeFile.type,
    customMetadata: {
      ownerId: userId,
    },
  });

  const unsubscribe = uploadTask.on(
    "state_changed",
    (snapshot) => {
      const progress =
        snapshot.totalBytes === 0
          ? 0
          : snapshot.bytesTransferred / snapshot.totalBytes;

      onProgress(progress);
    },
  );

  const finished = uploadTask.then((snapshot) => ({
    path: snapshot.ref.fullPath,
    contentType: snapshot.metadata.contentType,
    size: snapshot.metadata.size,
  }));

  return {
    finished,
    cancel: () => uploadTask.cancel(),
    stopProgressUpdates: unsubscribe,
  };
}
~~~

### Get a download URL

Source: [08. Cloud Storage for files](./08-cloud-storage-for-files.md)

~~~js
import { getDownloadURL, ref } from "firebase/storage";
import { storage } from "./lib/storage.js";

export async function getProfileImageUrl(objectPath) {
  return getDownloadURL(ref(storage, objectPath));
}
~~~

### Get a download URL

Source: [08. Cloud Storage for files](./08-cloud-storage-for-files.md)

~~~js
const previewUrl = URL.createObjectURL(file);

// When the preview is removed:
URL.revokeObjectURL(previewUrl);
~~~

### Delete and list objects

Source: [08. Cloud Storage for files](./08-cloud-storage-for-files.md)

~~~js
import { deleteObject, ref } from "firebase/storage";
import { storage } from "./lib/storage.js";

export async function deleteStoredImage(objectPath) {
  await deleteObject(ref(storage, objectPath));
}
~~~

### Protect user files with Storage Rules

Source: [08. Cloud Storage for files](./08-cloud-storage-for-files.md)

~~~text
rules_version = '2';

service firebase.storage {
  match /b/{bucket}/o {
    match /users/{userId}/profile/{fileName} {
      allow read: if request.auth != null
        && request.auth.uid == userId;

      allow create, update: if request.auth != null
        && request.auth.uid == userId
        && request.resource.size < 5 * 1024 * 1024
        && request.resource.contentType.matches('image/.*');

      allow delete: if request.auth != null
        && request.auth.uid == userId;
    }
  }
}
~~~

## 09. Cloud Functions and event-driven work

### Set up a JavaScript functions project

Source: [09. Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md)

~~~bash
firebase init functions
~~~

### Set up a JavaScript functions project

Source: [09. Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md)

~~~js
const { initializeApp } = require("firebase-admin/app");
const { getFirestore } = require("firebase-admin/firestore");

initializeApp();

const db = getFirestore();
~~~

### Create a callable function

Source: [09. Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md)

~~~js
const { onCall, HttpsError } = require("firebase-functions/v2/https");
const { initializeApp } = require("firebase-admin/app");
const { getFirestore, FieldValue } = require("firebase-admin/firestore");

initializeApp();
const db = getFirestore();

exports.createPrivateNote = onCall(async (request) => {
  const userId = request.auth?.uid;
  const title = request.data?.title;
  const body = request.data?.body;

  if (!userId) {
    throw new HttpsError("unauthenticated", "Sign in before saving a note.");
  }

  if (
    typeof title !== "string"
    || title.trim().length === 0
    || title.length > 120
    || typeof body !== "string"
    || body.length > 10000
  ) {
    throw new HttpsError("invalid-argument", "The note fields are invalid.");
  }

  const noteRef = await db.collection("users")
    .doc(userId)
    .collection("notes")
    .add({
      title: title.trim(),
      body,
      createdAt: FieldValue.serverTimestamp(),
    });

  return { id: noteRef.id };
});
~~~

### Create a callable function

Source: [09. Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md)

~~~js
import { getFunctions, httpsCallable } from "firebase/functions";
import { app } from "./lib/firebase.js";

const functions = getFunctions(app, "YOUR_FUNCTIONS_REGION");
const createPrivateNote = httpsCallable(functions, "createPrivateNote");

export async function saveNote(title, body) {
  const result = await createPrivateNote({ title, body });
  return result.data.id;
}
~~~

### React to a Firestore event

Source: [09. Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md)

~~~js
const { onDocumentCreated } = require("firebase-functions/v2/firestore");
const { logger } = require("firebase-functions");
const { initializeApp } = require("firebase-admin/app");

initializeApp();

exports.logNewOrder = onDocumentCreated(
  "orders/{orderId}",
  (event) => {
    const order = event.data?.data();

    if (!order) {
      return;
    }

    logger.info("New order received", {
      orderId: event.params.orderId,
      ownerId: order.ownerId,
    });
  },
);
~~~

### Store secrets outside source code

Source: [09. Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md)

~~~js
const { defineSecret } = require("firebase-functions/params");
const { onRequest } = require("firebase-functions/v2/https");

const paymentApiKey = defineSecret("PAYMENT_API_KEY");

exports.checkPayment = onRequest(
  { secrets: [paymentApiKey] },
  async (request, response) => {
    const apiKey = paymentApiKey.value();

    if (!apiKey) {
      response.status(500).send("Payment service is not configured.");
      return;
    }

    response.json({ configured: true });
  },
);
~~~

### Test functions locally

Source: [09. Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md)

~~~bash
firebase emulators:start --only functions,firestore,auth
~~~

### Deploy carefully

Source: [09. Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md)

~~~bash
firebase deploy --only functions
~~~

## 10. Firebase Hosting and web delivery

### Connect a local project

Source: [10. Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md)

~~~bash
firebase login
firebase init hosting
~~~

### Connect a local project

Source: [10. Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md)

~~~bash
npm run build
~~~

### Connect a local project

Source: [10. Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md)

~~~bash
firebase use
firebase use development
~~~

### Configure a single-page app

Source: [10. Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md)

~~~json
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
~~~

### Add response headers

Source: [10. Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md)

~~~json
{
  "hosting": {
    "headers": [
      {
        "source": "/index.html",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "no-cache"
          }
        ]
      },
      {
        "source": "**/*.@(js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "public,max-age=31536000,immutable"
          }
        ]
      }
    ]
  }
}
~~~

### Test locally and share a preview

Source: [10. Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md)

~~~bash
npm run build
firebase emulators:start --only hosting
~~~

### Test locally and share a preview

Source: [10. Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md)

~~~bash
firebase hosting:channel:deploy review-notes
~~~

### Deploy and roll back

Source: [10. Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md)

~~~bash
firebase deploy --only hosting
~~~

## 11. App Check and abuse reduction

### Initialize App Check before other services

Source: [11. App Check and abuse reduction](./11-app-check-and-abuse-reduction.md)

~~~js
import { initializeApp } from "firebase/app";
import {
  initializeAppCheck,
  ReCaptchaEnterpriseProvider,
} from "firebase/app-check";

const app = initializeApp(firebaseConfig);

const appCheck = initializeAppCheck(app, {
  provider: new ReCaptchaEnterpriseProvider(
    import.meta.env.VITE_RECAPTCHA_ENTERPRISE_SITE_KEY,
  ),
  isTokenAutoRefreshEnabled: true,
});

// Initialize Auth, Firestore, Storage, and other services after App Check.
~~~

### Use a debug provider only in controlled development

Source: [11. App Check and abuse reduction](./11-app-check-and-abuse-reduction.md)

~~~js
import {
  initializeAppCheck,
  ReCaptchaEnterpriseProvider,
} from "firebase/app-check";

const provider =
  import.meta.env.DEV
    ? new CustomProvider({
        getToken: async () => {
          throw new Error("Configure the official App Check debug provider.");
        },
      })
    : new ReCaptchaEnterpriseProvider(
        import.meta.env.VITE_RECAPTCHA_ENTERPRISE_SITE_KEY,
      );
~~~

## 12. Local Emulator Suite and testing

### Install and configure the emulators

Source: [12. Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md)

~~~bash
firebase init emulators
~~~

### Install and configure the emulators

Source: [12. Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md)

~~~json
{
  "emulators": {
    "auth": {
      "port": 9099
    },
    "database": {
      "port": 9000
    },
    "firestore": {
      "port": 8080
    },
    "functions": {
      "port": 5001
    },
    "hosting": {
      "port": 5000
    },
    "storage": {
      "port": 9199
    },
    "ui": {
      "enabled": true,
      "port": 4000
    },
    "singleProjectMode": true
  }
}
~~~

### Install and configure the emulators

Source: [12. Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md)

~~~bash
firebase emulators:start --only auth,firestore,functions
~~~

### Connect the web SDK to local services

Source: [12. Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md)

~~~js
import { connectAuthEmulator } from "firebase/auth";
import { connectFirestoreEmulator } from "firebase/firestore";
import { connectFunctionsEmulator } from "firebase/functions";
import { auth, db, functions } from "./lib/firebase.js";

const useEmulators =
  import.meta.env.DEV
  && import.meta.env.VITE_USE_FIREBASE_EMULATORS === "true";

if (useEmulators) {
  connectAuthEmulator(auth, "http://127.0.0.1:9099");
  connectFirestoreEmulator(db, "127.0.0.1", 8080);
  connectFunctionsEmulator(functions, "127.0.0.1", 5001);
}
~~~

### Keep emulator access local

Source: [12. Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md)

~~~text
VITE_USE_FIREBASE_EMULATORS=true
~~~

### Run integration tests

Source: [12. Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md)

~~~bash
firebase emulators:exec --only auth,firestore,functions "npm test"
~~~

### Import and export local data

Source: [12. Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md)

~~~bash
firebase emulators:start --import=./emulator-data --export-on-exit=./emulator-data
~~~

## 13. Firebase Cloud Messaging for web

### Ask permission after the user chooses

Source: [13. Firebase Cloud Messaging for web](./13-firebase-cloud-messaging-for-web.md)

~~~js
export async function requestNotificationPermission() {
  if (!("Notification" in window)) {
    return "unsupported";
  }

  if (Notification.permission === "denied") {
    return "denied";
  }

  if (Notification.permission === "granted") {
    return "granted";
  }

  return Notification.requestPermission();
}
~~~

### Register the app instance

Source: [13. Firebase Cloud Messaging for web](./13-firebase-cloud-messaging-for-web.md)

~~~js
import {
  getMessaging,
  isSupported,
  onRegistered,
  register,
} from "firebase/messaging";
import { app } from "./lib/firebase.js";

export async function registerForPush(onInstallationId) {
  if (!(await isSupported())) {
    throw new Error("Messaging is not supported in this browser.");
  }

  const permission = await requestNotificationPermission();

  if (permission !== "granted") {
    return null;
  }

  const messaging = getMessaging(app);

  onRegistered(messaging, (installationId) => {
    onInstallationId(installationId);
  });

  await register(messaging, {
    vapidKey: import.meta.env.VITE_FIREBASE_VAPID_PUBLIC_KEY,
  });

  return messaging;
}
~~~

### Handle messages while the page is open

Source: [13. Firebase Cloud Messaging for web](./13-firebase-cloud-messaging-for-web.md)

~~~js
import { getMessaging, onMessage } from "firebase/messaging";
import { app } from "./lib/firebase.js";

const messaging = getMessaging(app);

export function watchForegroundMessages(showMessage) {
  return onMessage(messaging, (payload) => {
    showMessage({
      title: payload.notification?.title ?? "New update",
      body: payload.notification?.body ?? "",
      data: payload.data ?? {},
    });
  });
}
~~~

### Handle background messages in the service worker

Source: [13. Firebase Cloud Messaging for web](./13-firebase-cloud-messaging-for-web.md)

~~~js
// firebase-messaging-sw.js
importScripts(
  "https://www.gstatic.com/firebasejs/YOUR_FIREBASE_JS_SDK_VERSION/firebase-app-compat.js",
);
importScripts(
  "https://www.gstatic.com/firebasejs/YOUR_FIREBASE_JS_SDK_VERSION/firebase-messaging-compat.js",
);

firebase.initializeApp({
  apiKey: "YOUR_WEB_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_WEB_APP_ID",
});

const messaging = firebase.messaging();

messaging.onBackgroundMessage((payload) => {
  const title = payload.data?.title ?? "New update";
  const body = payload.data?.body ?? "";

  self.registration.showNotification(title, {
    body,
    data: {
      path: payload.data?.path ?? "/",
    },
  });
});

self.addEventListener("notificationclick", (event) => {
  event.notification.close();

  const requestedPath = event.notification.data?.path ?? "/";
  const destination = new URL(requestedPath, self.location.origin);

  if (destination.origin !== self.location.origin) {
    return;
  }

  event.waitUntil(self.clients.openWindow(destination.href));
});
~~~

### Send from trusted server code

Source: [13. Firebase Cloud Messaging for web](./13-firebase-cloud-messaging-for-web.md)

~~~js
const { getMessaging } = require("firebase-admin/messaging");

async function sendUpdate(fid, title, body, path) {
  return getMessaging().send({
    fid,
    data: {
      title,
      body,
      path,
    },
  });
}
~~~

## 14. Analytics and Performance Monitoring

### Initialize Analytics only when supported

Source: [14. Analytics and Performance Monitoring](./14-analytics-and-performance-monitoring.md)

~~~js
import { getAnalytics, isSupported } from "firebase/analytics";
import { app } from "./lib/firebase.js";

export async function createAnalyticsIfSupported() {
  if (!(await isSupported())) {
    return null;
  }

  return getAnalytics(app);
}
~~~

### Log a small custom event

Source: [14. Analytics and Performance Monitoring](./14-analytics-and-performance-monitoring.md)

~~~js
import { logEvent } from "firebase/analytics";

export function recordTaskSaved(analytics, taskCategory) {
  if (!analytics) {
    return;
  }

  logEvent(analytics, "task_saved", {
    task_category: taskCategory,
    method: "manual",
  });
}
~~~

### Respect collection choices

Source: [14. Analytics and Performance Monitoring](./14-analytics-and-performance-monitoring.md)

~~~js
import { setAnalyticsCollectionEnabled } from "firebase/analytics";

export function applyAnalyticsChoice(analytics, enabled) {
  if (!analytics) {
    return;
  }

  setAnalyticsCollectionEnabled(analytics, enabled);
}
~~~

### What Performance Monitoring measures

Source: [14. Analytics and Performance Monitoring](./14-analytics-and-performance-monitoring.md)

~~~js
import { getPerformance, isSupported } from "firebase/performance";
import { app } from "./lib/firebase.js";

export async function createPerformanceMonitoringIfSupported() {
  if (!(await isSupported())) {
    return null;
  }

  return getPerformance(app);
}
~~~

### Add a custom trace

Source: [14. Analytics and Performance Monitoring](./14-analytics-and-performance-monitoring.md)

~~~js
import { trace } from "firebase/performance";

export async function loadTasksWithTrace(performance, loadTasks) {
  if (!performance) {
    return loadTasks();
  }

  const taskTrace = trace(performance, "load_task_list");
  taskTrace.start();

  try {
    const tasks = await loadTasks();
    taskTrace.putMetric("task_count", tasks.length);
    taskTrace.putAttribute("result", "success");
    return tasks;
  } catch (error) {
    taskTrace.putAttribute("result", "error");
    throw error;
  } finally {
    taskTrace.stop();
  }
}
~~~

## 15. Environments, configuration, and CI/CD

### Name projects and CLI aliases clearly

Source: [15. Environments, configuration, and CI/CD](./15-environments-configuration-and-ci-cd.md)

~~~json
{
  "projects": {
    "default": "my-app-dev",
    "staging": "my-app-staging",
    "production": "my-app-prod"
  }
}
~~~

### Name projects and CLI aliases clearly

Source: [15. Environments, configuration, and CI/CD](./15-environments-configuration-and-ci-cd.md)

~~~bash
firebase use
firebase use staging
firebase deploy --project staging --only hosting
~~~

### Configure browser values by environment

Source: [15. Environments, configuration, and CI/CD](./15-environments-configuration-and-ci-cd.md)

~~~js
const requiredValues = [
  "VITE_FIREBASE_API_KEY",
  "VITE_FIREBASE_AUTH_DOMAIN",
  "VITE_FIREBASE_PROJECT_ID",
  "VITE_FIREBASE_APP_ID",
];

for (const name of requiredValues) {
  if (!import.meta.env[name]) {
    throw new Error("Missing required Firebase setting: " + name);
  }
}

export const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};
~~~

### Configure browser values by environment

Source: [15. Environments, configuration, and CI/CD](./15-environments-configuration-and-ci-cd.md)

~~~text
.env.local
.env.development
.env.production
.env.example
~~~

### Add continuous integration checks

Source: [15. Environments, configuration, and CI/CD](./15-environments-configuration-and-ci-cd.md)

~~~yaml
name: verify
on:
  pull_request:
jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint --if-present
      - run: npm test --if-present
      - run: npm run build
~~~

### Deploy previews and production from separate steps

Source: [15. Environments, configuration, and CI/CD](./15-environments-configuration-and-ci-cd.md)

~~~bash
firebase deploy --project staging --only hosting
firebase deploy --project production --only hosting
~~~

## 16. Billing, operations, debugging, and production checklist

### Log useful operational context

Source: [16. Billing, operations, debugging, and production checklist](./16-billing-operations-debugging-and-production-checklist.md)

~~~js
const { logger } = require("firebase-functions");

async function processOrder(orderId, ownerId) {
  logger.info("Order processing started", {
    orderId,
    ownerId,
  });

  try {
    const result = await validateAndProcessOrder(orderId);

    logger.info("Order processing completed", {
      orderId,
      status: result.status,
    });

    return result;
  } catch (error) {
    logger.error("Order processing failed", {
      orderId,
      errorCode: error.code ?? "unknown",
    });

    throw error;
  }
}
~~~
