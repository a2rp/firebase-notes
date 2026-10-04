# 09. Cloud Functions and event-driven work

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Cloud Storage for files](./08-cloud-storage-for-files.md) | [Notes index](../README.md) | [Next: Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md) |

## What Cloud Functions do

Cloud Functions for Firebase runs trusted JavaScript on managed server infrastructure in response to events or requests. A function can keep privileged work out of a browser, react to database changes, send a notification, or expose a callable operation.

Use a function when the work needs server credentials, must not trust client input, or should happen after a service event. Keep ordinary interface behavior in the client. A function adds another deployed service to monitor, test, and pay for.

Common function types include:

- Callable functions invoked through the Firebase client SDK.
- HTTP request functions called through an HTTP endpoint.
- Background functions triggered by Firestore, Realtime Database, Storage, Authentication, and other supported events.
- Scheduled functions for recurring server work.

## Set up a JavaScript functions project

Use the Firebase CLI from a project directory and select JavaScript when initializing Functions.

~~~bash
firebase init functions
~~~

The generated functions directory has its own package.json and entry file. Install dependencies in that directory and use a supported Node.js runtime listed by the current Firebase documentation.

Do not add Admin SDK credentials to the browser. In a managed Firebase function, initialize the Admin SDK with the function's runtime identity:

~~~js
const { initializeApp } = require("firebase-admin/app");
const { getFirestore } = require("firebase-admin/firestore");

initializeApp();

const db = getFirestore();
~~~

The Admin SDK has privileged access and bypasses Firestore Security Rules. Every function that uses it must check the caller's identity and permission before accessing data.

## Create a callable function

A callable function uses the Firebase client protocol. The SDK can include the signed-in user's authentication information and App Check token when available. The function still needs to validate input and authorize the requested action.

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

The server derives the owner path from request.auth.uid instead of accepting an arbitrary user ID from the browser. It validates input even when the client already performs validation.

Call it from the web app with the modular client SDK:

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

Use the region where the function is deployed. The client and server must agree on the function name and region.

## React to a Firestore event

An event-triggered function runs after a matching database event. It is useful for work that should not depend on the user's browser staying open.

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

A background function should return its promise when doing asynchronous work. Otherwise the platform may stop execution before that work completes.

Avoid writing to the same path that triggers the function unless the function is designed to stop after its first change. For example, a trigger on every update that writes another update to the same document can call itself repeatedly.

## Design for retries and duplicate events

Background events can be retried, and event order is not guaranteed. A function should be safe if the same event is delivered more than once. This property is called idempotency.

For important operations, store a processed event ID or use a deterministic output path so a retry does not create duplicate records. Do not assume a trigger is a single exactly-once callback.

Keep a function focused on one operation, set sensible timeouts and resource limits, and log enough context to investigate failures without logging passwords, access tokens, or full private records.

## Store secrets outside source code

Do not hardcode API keys that grant privileged access, passwords, or service credentials in source files. Use Firebase Functions secrets for values the function needs.

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

Set secrets with the Firebase CLI secret management command and grant a secret to only the functions that require it. Never print the secret to logs or return it to the browser.

## Test functions locally

The Local Emulator Suite can run Functions and supported Firebase products locally. Connect the client SDK to the emulators, use fictional data, and test successful and failing requests before deploying.

~~~bash
firebase emulators:start --only functions,firestore,auth
~~~

Callable function tests should cover signed-out calls, malformed data, valid requests, and attempts to access another user's records. Event-trigger tests should cover missing fields, retries, duplicate events, and failures from downstream services.

Use logs to diagnose deployed behavior. Add structured log fields such as an operation ID or document ID, while avoiding private content and secrets.

## Deploy carefully

Functions deployment can require a billing-enabled project and may create costs. Check the current Firebase plan requirements, region, invocation volume, memory, and other billing details before deploying to a project.

~~~bash
firebase deploy --only functions
~~~

Deploy a limited function set when appropriate, verify the active Firebase project first, and review logs after deployment. Use separate development and production projects so a test function cannot unexpectedly process production data.

## Hands-on exercise: protected note creation

1. Initialize a JavaScript Functions codebase in a development Firebase project.
2. Add a callable function that requires an authenticated user.
3. Validate the title and body on the server.
4. Derive the user document path from the authenticated UID.
5. Call the function from the modular web SDK.
6. Run Auth, Firestore, and Functions emulators with fictional test data.
7. Verify that signed-out and malformed requests fail.
8. Send the same test request twice and decide how duplicate work should be handled.
9. Add a structured log entry without logging the note body.
10. Check the current plan and billing details before deploying.

## Common mistakes

- Putting privileged Admin SDK credentials in browser code.
- Trusting a user ID sent in request data instead of using the verified callable identity.
- Assuming Firestore Security Rules protect Admin SDK writes.
- Skipping server-side validation because the browser validates the form.
- Performing work without returning or awaiting its promise.
- Assuming a background event runs exactly once or in order.
- Writing to the trigger path in a way that causes an infinite loop.
- Logging secrets or complete private records.
- Deploying without checking project selection, region, runtime, and billing.

## Practice questions

1. What work is a good fit for a Cloud Function?
2. How does a callable function differ from a plain HTTP endpoint?
3. Why should a function derive the owner ID from request.auth?
4. Which security rules apply to Admin SDK requests?
5. Why should event-triggered functions be idempotent?
6. What can cause a function to trigger itself repeatedly?
7. How should a function receive a private API key?
8. What should be checked before deploying a function to a project?

## Main references

- [Get started with Cloud Functions for Firebase](https://firebase.google.com/docs/functions/get-started)
- [Call functions from your app](https://firebase.google.com/docs/functions/callable)
- [Cloud Firestore triggers](https://firebase.google.com/docs/functions/firestore-events)
- [Write functions](https://firebase.google.com/docs/functions)
- [Configure and manage secrets](https://firebase.google.com/docs/functions/config-env)
- [Local Emulator Suite](https://firebase.google.com/docs/emulator-suite)
