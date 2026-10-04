# 12. Local Emulator Suite and testing

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: App Check and abuse reduction](./11-app-check-and-abuse-reduction.md) | [Notes index](../README.md) | [Next: Firebase Cloud Messaging for web](./13-firebase-cloud-messaging-for-web.md) |

## Why use the Local Emulator Suite

The Firebase Local Emulator Suite runs local versions of supported Firebase products so an app can be developed and tested without writing to production data. The emulators can work together. For example, a Firestore write can trigger a local Cloud Function.

The suite supports local development, rule testing, integration tests, and continuous integration. It is built to help test Firebase behavior, not to act as a production service or as a performance benchmark.

## Install and configure the emulators

Install the Firebase CLI, sign in if needed, then initialize the emulators from the project directory.

~~~bash
firebase init emulators
~~~

Select only the emulators the project uses. The CLI writes configuration into firebase.json. A representative configuration may look like this:

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

Use the actual configuration generated for the installed CLI version. The emulator UI is commonly available at localhost:4000 when enabled. Ports can be changed if another local service is already using them.

Start only the products needed for a test session:

~~~bash
firebase emulators:start --only auth,firestore,functions
~~~

Use one project ID consistently across the CLI, client configuration, and emulator suite. Avoid mixing live services and emulators in one test run.

## Connect the web SDK to local services

The app must explicitly connect each service SDK to its emulator. Add the connections after initializing the service instances and before making requests.

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

A service should be connected only once during app startup. Repeated connections can cause errors or confusing test behavior. Use separate entry configuration for emulators if the app's initialization structure makes the order hard to control.

For Storage and Realtime Database, use their corresponding connectStorageEmulator and connectDatabaseEmulator functions with the configured local host and ports. Keep those services out of the sample until the app initializes them.

## Keep emulator access local

Use a local environment setting to opt into the emulators:

~~~text
VITE_USE_FIREBASE_EMULATORS=true
~~~

Use false or leave the setting unset in other builds. Do not set the emulator flag for a production build. Keep the connection code behind both a development check and an explicit environment setting.

When using a shared app module, be deliberate about initialization order. Auth persistence, App Check setup, service initialization, and emulator connections can affect when the first request occurs. Complete emulator configuration before the app reads or writes data.

## Test Security Rules

The Firestore, Realtime Database, and Storage emulators evaluate their Security Rules. Test each rule with both a permitted identity and a denied identity.

A useful rules test suite includes:

- Signed-out requests.
- A user reading and writing their own record.
- A different user requesting the same record.
- Invalid field types, sizes, or status values.
- A query missing the required ownership constraint.
- Storage uploads with an allowed and disallowed file type.
- Deletion and update requests.
- A rule change that should deny access.

Automated rule tests can load a ruleset, create test identities, make operations against the emulator, and check whether each operation succeeds or fails. Keep fixture data fictional and clear it between tests.

## Run integration tests

Start the required emulators, then run the app or tests against the local endpoints.

~~~bash
firebase emulators:exec --only auth,firestore,functions "npm test"
~~~

emulators:exec starts the selected services for the command and shuts them down afterward. Confirm that the test command actually uses the local endpoints; otherwise the app can still send requests to a live Firebase project.

The Emulator Suite UI can help inspect documents, users, function logs, and local requests. It does not prove that a production deployment is configured correctly.

## Import and export local data

The CLI can load saved emulator data and export emulator state. Use a dedicated test-data directory and do not store private production data in it.

~~~bash
firebase emulators:start --import=./emulator-data --export-on-exit=./emulator-data
~~~

Add local state to .gitignore unless the repository intentionally stores small, fictional fixtures that have been reviewed. Never export real user data into a public notes repository.

## App Check and local testing

Production App Check providers may reject localhost or continuous integration environments. Use the documented debug provider only for local development or a controlled test runner. Store any registered debug token in a local ignored file or CI secret store.

A debug token grants a test client access without normal attestation. Do not share it, commit it, or include it in a production build.

## Hands-on exercise: test an owner rule

1. Configure Auth and Firestore emulators in the project.
2. Start them with one local project ID.
3. Connect the web SDK before any Auth or Firestore request.
4. Create two local test users.
5. Create a document owned by the first user.
6. Confirm the owner can read and update it.
7. Confirm the second user and a signed-out client are denied.
8. Run the same test after changing the rules and verify that the expected result changes.
9. Inspect the request in the Emulator Suite UI.
10. Run the test with emulators:exec and confirm the suite shuts down afterward.

## Common mistakes

- Assuming that starting an emulator automatically redirects the web SDK.
- Connecting a service after the app has already made its first request.
- Connecting one service more than once during startup.
- Using different project IDs in the CLI and client app.
- Accidentally mixing local and production services.
- Treating emulator behavior as a production load or security test.
- Using real user information in emulator fixtures.
- Committing exported emulator data that contains private records.
- Leaving a debug token enabled in production.

## Practice questions

1. What is the purpose of the Local Emulator Suite?
2. Does starting an emulator automatically connect the Firebase web SDK to it?
3. Why should the app use one project ID consistently?
4. Which emulators can evaluate Firebase Security Rules?
5. What does emulators:exec do?
6. Why should an emulator suite not be treated as a production service?
7. How should local test data be handled?
8. What is the risk of a committed App Check debug token?

## Main references

- [Introduction to Firebase Local Emulator Suite](https://firebase.google.com/docs/emulator-suite)
- [Install, configure, and integrate the Emulator Suite](https://firebase.google.com/docs/emulator-suite/install_and_configure)
- [Connect your app to the Firestore emulator](https://firebase.google.com/docs/emulator-suite/connect_firestore)
- [Connect your app to the Authentication emulator](https://firebase.google.com/docs/emulator-suite/connect_auth)
- [Test Firebase Security Rules](https://firebase.google.com/docs/rules/unit-tests)
