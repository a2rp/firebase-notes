# 06. Firestore Security Rules and authorization

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Firestore reads, queries, indexes, and pagination](./05-firestore-reads-queries-indexes-and-pagination.md) | [Notes index](../README.md) | [Next: Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md) |

## What Security Rules do

Cloud Firestore Security Rules decide whether a mobile or web client may read or write a document. The rules run for every request made through the Firebase client SDK. They can check the signed-in user, the document path, existing data, proposed data, and whether field values meet expected conditions.

Authentication answers who is making a request. Authorization answers what that user may do. Firebase Authentication supplies the identity, and Security Rules enforce access to Firestore data.

Rules are not filters. Firestore does not remove documents the user cannot access from a query. The query must prove that all possible results satisfy the rule. If even one possible result could be unauthorized, Firestore rejects the complete query.

## Start with a closed database

A production database should not allow public access by default. A closed ruleset denies all client requests until specific paths receive rules.

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

A broad rule that allows every read and write exposes data to anyone who can reach the database. Do not use open rules as a shortcut while building an app. For local exploration, use a separate project or the Emulator Suite.

## Protect user-owned documents

Assume each task document has an ownerId field containing the Firebase Authentication UID of its owner. The rules below let a signed-in owner read and delete the task, create only a task for themselves, and update task fields without changing ownership.

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

For a create, request.resource.data is the proposed document. For an update, resource.data is the existing document and request.resource.data is the resulting document. A rule must check the resulting state, not assume that the client changes only the field it intended to change.

This rule checks ownership and a few field constraints. A real app may need additional checks for allowed keys, timestamps, optional fields, role membership, or state transitions. Keep the rule aligned with the actual data model.

## Match nested paths carefully

For data stored at users/{userId}/tasks/{taskId}, the path itself can identify the owner:

~~~text
match /users/{userId}/tasks/{taskId} {
  allow read, write: if request.auth != null
    && request.auth.uid == userId;
}
~~~

This path-based check avoids repeating ownerId on each task, but it shapes how the app queries and how administrators or shared collaborators access records. A match applies only to paths it covers. A rule for a parent document does not automatically secure every subcollection below that document.

Overlapping allow rules are combined permissively: a request is allowed if any matching allow expression evaluates to true. Review all matching rules when a request unexpectedly succeeds.

## Query with the rule in mind

If list access is allowed only for documents whose ownerId equals the caller's UID, scope the query to that owner:

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

The signed-in user object is convenient for building the query, but it is not the security boundary. A malicious client can change its own code. The rules must independently compare the requested data with request.auth.uid.

## Validate field changes

Rules can compare old and new values. This lets an application keep ownership fixed and validate a restricted set of fields.

~~~text
allow update: if request.auth != null
  && resource.data.ownerId == request.auth.uid
  && request.resource.data.ownerId == resource.data.ownerId
  && request.resource.data.diff(resource.data).affectedKeys()
       .hasOnly(['title', 'status', 'updatedAt']);
~~~

The affectedKeys check rejects changes to fields outside the listed set. It does not replace type and value validation. Validate every field that could affect access, billing, trust, or application behavior.

## Client SDK and server access differ

Firestore Security Rules apply to Firebase mobile and web client libraries. Server client libraries, including trusted Admin SDK code, use Google Cloud IAM instead and bypass Firestore Security Rules.

This means a server function must verify authorization itself before using privileged server credentials. Never put service account credentials in a web app. A server endpoint that trusts a client-provided user ID without checking the caller can expose every user's data.

## Test allowed and denied cases

Test both sides of each rule. For a task owned by user A, a useful matrix is:

| Request | Expected result |
| --- | --- |
| User A reads the task | Allowed |
| User B reads the task | Denied |
| Signed-out client reads the task | Denied |
| User A creates a task with their UID | Allowed |
| User A creates a task with user B's UID | Denied |
| User A changes the task ownerId | Denied |
| User A writes a title over the configured limit | Denied |
| User A deletes their task | Allowed |

Use the rules simulator for a quick check and the Local Emulator Suite for repeatable automated tests. Keep test data fictional and separate from production.

When deploying with the Firebase CLI, the local rules file can replace the rules currently published in the console. Keep the checked-in file as the source of truth and review the exact project and rules file before deployment.

## Hands-on exercise: private task data

1. Create a development project or start the Firestore emulator.
2. Add the closed ruleset and confirm that unauthenticated client reads fail.
3. Add the owner rule for tasks and create one task as user A.
4. Confirm user A can read the task, while user B cannot.
5. Attempt to create a task using user B's UID while signed in as user A.
6. Attempt to update ownerId and confirm the write is denied.
7. Test an invalid title and an invalid status.
8. Query tasks without an ownerId constraint, then add the correct constraint and compare results.
9. Check the parent and subcollection paths separately.
10. Save the passing and failing cases so they can be rerun after rule changes.

## Common mistakes

- Leaving test rules open after initial setup.
- Checking a user ID only in client-side code.
- Allowing a query to return records owned by other users.
- Assuming a parent path rule automatically covers child collections.
- Checking only the changed fields instead of the resulting document.
- Forgetting that overlapping allow rules can grant access through a broader match.
- Assuming Security Rules protect server SDK requests.
- Testing only successful access and never checking that unauthorized requests fail.

## Practice questions

1. What is the difference between authentication and authorization?
2. What are resource.data and request.resource.data?
3. Why are Firestore Security Rules not query filters?
4. How can a task rule verify that its owner is the signed-in user?
5. Why should an update rule prevent ownerId from changing?
6. Does a rule on a user document automatically cover that user's subcollection?
7. Which access-control system applies to server client libraries?
8. Name four allowed and denied cases you would test for a user-owned task collection.

## Main references

- [Get started with Cloud Firestore Security Rules](https://firebase.google.com/docs/firestore/security/get-started)
- [Writing conditions for Cloud Firestore Security Rules](https://firebase.google.com/docs/firestore/security/rules-conditions)
- [Securely query data](https://firebase.google.com/docs/firestore/security/rules-query)
- [Build unit tests for Firebase Security Rules](https://firebase.google.com/docs/rules/unit-tests)
- [Deploy Cloud Firestore Security Rules](https://firebase.google.com/docs/firestore/security/get-started#deploying_rules)
