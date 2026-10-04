# 08. Cloud Storage for files

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Realtime Database and live synchronization](./07-realtime-database-and-live-synchronization.md) | [Notes index](../README.md) | [Next: Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md) |

## What Cloud Storage is for

Cloud Storage for Firebase stores files such as images, documents, audio, and video in a bucket. The JavaScript SDK addresses each object by a path. Store file metadata and ownership in an appropriate database when the app needs searchable records, captions, or relationships.

A common arrangement is:

~~~text
Cloud Storage:
users/user_42/profile/avatar_8a21.webp

Cloud Firestore:
users/user_42
  displayName: "Ada"
  avatarPath: "users/user_42/profile/avatar_8a21.webp"
~~~

Keep the storage path as the durable reference in application data. A download URL can be requested when the app needs to display or share the object.

## Initialize the Storage service

Initialize Storage from the existing Firebase app in the shared module. Use the bucket value provided by Firebase for the project.

~~~js
import { getStorage } from "firebase/storage";
import { app } from "./lib/firebase.js";

export const storage = getStorage(app);
~~~

Check that the project's Storage product and bucket are configured before attempting an upload. Development and production apps should point to the intended bucket for their environment.

## Select and validate a file

A file input provides a browser File object. The accept attribute helps the user choose a file, but it is not a security check. Validate size and expected type in the interface for clear feedback, and enforce allowed paths, size, and content type with Storage Security Rules.

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

Browser-provided type metadata can be inaccurate or deliberately changed. For applications that accept risky file types or need stronger content inspection, validate the uploaded content in a trusted server process before making it available.

## Upload with progress and cancellation

uploadBytesResumable supports task state and progress events. Keep a reference to the task so the user can cancel an upload if needed.

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

The random path prevents accidental collisions. It does not prove ownership; Security Rules must check that the signed-in user may write to that path. A storage task can pause, resume, or cancel. Show a useful status and handle failures such as permission denied, unavailable network, or user cancellation.

For a small upload where progress tracking is not needed, uploadBytes returns a promise after the object is written.

## Get a download URL

After upload, getDownloadURL can return a URL for the object. Save the storage path in your database, then request a URL when displaying the file.

~~~js
import { getDownloadURL, ref } from "firebase/storage";
import { storage } from "./lib/storage.js";

export async function getProfileImageUrl(objectPath) {
  return getDownloadURL(ref(storage, objectPath));
}
~~~

A download URL can function as a bearer link. Anyone who obtains a usable tokenized URL may be able to access the file. Avoid exposing private file URLs publicly, and use authenticated Storage SDK reads when access must follow the current user's permissions.

For temporary local previews before upload, create an object URL from the File and revoke it when no longer needed:

~~~js
const previewUrl = URL.createObjectURL(file);

// When the preview is removed:
URL.revokeObjectURL(previewUrl);
~~~

## Delete and list objects

Delete an object with deleteObject using its full path. Keep the path in your database so the application can identify which object to remove.

~~~js
import { deleteObject, ref } from "firebase/storage";
import { storage } from "./lib/storage.js";

export async function deleteStoredImage(objectPath) {
  await deleteObject(ref(storage, objectPath));
}
~~~

Removing a Firestore record does not automatically delete the matching Storage object. Decide how to keep the two products consistent. A trusted Cloud Function can handle cleanup when a database record is removed, or the application can perform an authorized coordinated workflow and retry failures.

Storage listing APIs can enumerate objects beneath a prefix. Use them when the product actually needs a directory-like view, and secure the listing path. For an application's own media library, storing object paths in a database can provide better metadata and pagination.

## Protect user files with Storage Rules

Storage Rules can check the path, authenticated UID, proposed content type, and proposed size.

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

Keep read, create, update, and delete decisions intentional. A rule that permits a user to write under their own folder should not also expose other users' files. Content type checks help control the declared metadata; they are not a complete file inspection system.

Test rules with a signed-in owner, a different signed-in user, an unauthenticated client, an oversized file, and a disallowed type. The Emulator Suite can test Storage operations locally.

## Hands-on exercise: private profile image

1. Set up a development Storage bucket and add the Storage service to shared initialization.
2. Add an image file input that accepts PNG, JPEG, and WebP files.
3. Validate type and size in the interface and show a clear error for a rejected file.
4. Upload under users/{userId}/profile with a random filename.
5. Display upload progress and add a cancel action.
6. Store the returned object path in the user's database record.
7. Request a download URL for the image and display it.
8. Delete the image and clear its database path.
9. Test owner, other user, signed-out user, oversized, and wrong-type cases against emulator rules.
10. Confirm that deleting the database record alone does not leave an untracked file.

## Common mistakes

- Treating the file input accept attribute as authorization.
- Trusting a filename or MIME type supplied by the browser.
- Using the same object path for every upload and overwriting a user's file accidentally.
- Saving only a download URL when the application needs to manage the object later.
- Forgetting that a download URL can be shared as a bearer link.
- Allowing all signed-in users to read or overwrite every user's files.
- Assuming a database document deletion also deletes a Storage object.
- Leaving upload listeners active after the interface no longer needs progress updates.
- Validating file size only in the client and omitting a rule-side limit.

## Practice questions

1. What kind of data belongs in Cloud Storage?
2. Why might an app store an object path in Firestore?
3. What does uploadBytesResumable provide over a basic upload?
4. Why is the browser's file type check not enough?
5. What does getDownloadURL return?
6. Why should a private download URL be handled carefully?
7. Does deleting a Firestore document delete its associated file?
8. Which checks should a Storage rule make for a user's image upload?

## Main references

- [Upload files with Cloud Storage on Web](https://firebase.google.com/docs/storage/web/upload-files)
- [Download files with Cloud Storage on Web](https://firebase.google.com/docs/storage/web/download-files)
- [Understand Firebase Security Rules for Cloud Storage](https://firebase.google.com/docs/storage/security)
- [Storage Security Rules conditions](https://firebase.google.com/docs/storage/security/core-syntax)
- [Use the Local Emulator Suite](https://firebase.google.com/docs/emulator-suite)
