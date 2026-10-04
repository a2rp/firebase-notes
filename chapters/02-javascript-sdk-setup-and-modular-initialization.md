# 02. JavaScript SDK setup and modular initialization

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Firebase projects, apps, and product map](./01-firebase-projects-apps-and-product-map.md) | [Notes index](../README.md) | [Next: Authentication and account lifecycle](./03-authentication-and-account-lifecycle.md) |

## Install the JavaScript SDK

For a web application that uses npm and a module bundler, install the Firebase package from the root of the application:

~~~bash
npm install firebase
~~~

The package manager records Firebase as a dependency and updates the lock file. Commit both package.json and the lock file so other developers install the same dependency tree.

Use the modular JavaScript API for new work. It imports individual functions from service packages, which works with bundlers to remove unused code from production builds.

~~~js
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
~~~

The examples in this collection use modular imports. Older namespaced code often uses a global firebase object and chained methods such as firebase.firestore(). Avoid mixing both styles in the same feature.

## Register the web app and copy its configuration

In the Firebase console, open the correct project, register a web app, and copy the Firebase configuration object shown for that app. Use the configuration from the matching development or production project.

Do not guess resource values from a project ID. Copy the provided values because service names and configuration fields can change. A basic configuration has this shape:

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

The configuration object connects the client app to Firebase resources. It is expected to be included in a web app and is visible to users in the built JavaScript. It is not an authorization rule and it does not make database access private.

Never put service-account JSON, Admin SDK credentials, private keys, or other server secrets in a browser bundle or client environment file. Security Rules and server-side IAM policies must control access.

## Initialize Firebase once in a shared module

Create one module that initializes the app and exports only the services the application uses. For example:

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

Other modules can import the same initialized services:

~~~js
import { auth, db } from "./lib/firebase.js";

console.log(auth.app.name);
console.log(db.app.name);
~~~

This keeps service setup in one place and avoids scattering configuration through feature files. The getApps check also helps during local development when a tool reloads a module without restarting the whole page.

When a feature needs another service, initialize it from the same app. Examples include getStorage(app), getDatabase(app), getFunctions(app, "YOUR_FUNCTIONS_REGION"), and getMessaging(app). Some services need extra browser or project configuration before they can be used.

## Keep each environment pointed at its own project

A client build should connect to the Firebase project for its environment. A development app must not quietly send test data to production.

A Vite project can use local environment files:

~~~text
VITE_FIREBASE_API_KEY=your-development-web-key
VITE_FIREBASE_AUTH_DOMAIN=your-development-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-development-project
VITE_FIREBASE_APP_ID=your-development-app-id
~~~

Then build a config object from Vite's client environment:

~~~js
const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID
};
~~~

This variable syntax is specific to Vite. Other tools use their own conventions. Any value inserted into a browser build can be inspected by users, even when its source came from an environment file. Keep Admin SDK credentials on a trusted server only.

Keep real local environment files out of source control. A checked-in example file can document variable names, but it should contain placeholders rather than working credentials.

## Know where initialization belongs

In a client-rendered web app, initialize Firebase from a shared module imported by the client entry point and service features.

In a server-rendered app, server code and browser code run in different environments. Do not assume a browser Firebase app instance can be reused for server requests. Follow the framework's current Firebase guidance, particularly when user identity or request-specific data must persist across rendering.

For tests and local development, connect the SDK to the Emulator Suite before running code that could access a cloud project. Chapter 12 covers emulator setup.

## Choose the right Firestore package

The standard Cloud Firestore package is firebase/firestore. It supports the full client feature set, including real-time listeners. Firebase also provides firebase/firestore/lite for applications that only need a smaller, request-based subset.

Choose one based on the feature. A screen that listens for live changes needs the standard package. A small read-only view that requests snapshots can consider Lite. Do not import both for the same database feature unless you understand why each is needed.

## Initialization troubleshooting

| Symptom | First check |
| --- | --- |
| Firebase reports missing or invalid options | Confirm every required value was copied from the correct registered web app |
| Data appears in the wrong environment | Compare the built project's projectId with the intended environment |
| App is initialized more than once | Search for other initializeApp calls and centralize initialization |
| A service is undefined or unavailable | Confirm its package import and initialization function are correct |
| Build reports import resolution errors | Check that Firebase is installed in this package and the import path is valid |
| A secret appears in built JavaScript | Remove it, rotate the exposed credential, and move the operation to a trusted server |

## Practice

1. Install the Firebase JavaScript SDK in a small web project and inspect the package and lock file changes.
2. Register a development web app and copy its configuration into the shared initialization module.
3. Import Auth and Firestore from feature files without calling initializeApp again.
4. Add a second Firebase project for staging and explain how the build selects the correct project.
5. Inspect a development build and explain why its Firebase configuration is visible.
6. Identify which credentials must stay on a server and never enter a client bundle.
7. Compare a screen that needs live Firestore updates with a screen that only needs one-time reads.
8. Confirm which initialization code must differ between a browser app and server-rendered request code.

## Main references

- [Add Firebase to a JavaScript project](https://firebase.google.com/docs/web/setup)
- [Use module bundlers with Firebase](https://firebase.google.com/docs/web/module-bundling)
- [Firebase for web overview](https://firebase.google.com/docs/web/learn-more)
- [Understand Firebase projects and app configuration](https://firebase.google.com/docs/projects/learn-more)
- [Manage Firebase API keys](https://firebase.google.com/docs/projects/api-keys)
