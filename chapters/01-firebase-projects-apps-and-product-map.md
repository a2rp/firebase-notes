# 01. Firebase projects, apps, and product map

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md) |

## What Firebase provides

Firebase is a set of application services backed by Google Cloud. A Firebase project brings registered client apps and their shared backend resources together. You add only the services your application needs.

| Service | Main job | Example |
| --- | --- | --- |
| Firebase Authentication | Identify users and manage sign-in methods | A person signs in with email and password |
| Cloud Firestore | Store documents and query collections | Save a user's profile and project records |
| Realtime Database | Store JSON data and deliver live updates | Share a live room status |
| Cloud Storage | Store files as objects | Save profile images or attachments |
| Cloud Functions | Run trusted server-side code | Validate and react to backend events |
| Firebase Hosting | Deliver web assets and web apps | Publish a built single-page site |
| App Check | Help verify requests come from registered app instances | Reduce requests from unauthorized clients |
| Cloud Messaging | Send messages to supported client apps | Deliver a notification to a web browser |
| Analytics and Performance Monitoring | Observe product usage and app performance | Measure page events and load traces |

Services have different data models, pricing, security rules, and product requirements. Choose a service because its behavior matches the feature, not because it appears beside another service in the console.

## Understand the project hierarchy

A Firebase project is the main container for registered apps and provisioned services. A project is also a Google Cloud project with Firebase services and settings enabled.

A registered Firebase app identifies one client application within a project. You may register related platform variants, such as the web, Android, and iOS versions of the same product. Apps registered to one project share access to that project's backend resources.

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

A Firebase app registration is not a separate backend. Adding two unrelated products to one project can cause them to share authentication, data, rules, analytics, and billing. Keep apps in the same project when they are client variants of the same product and should use the same backend. Use separate projects for products that must not share resources or user data.

## Separate development environments

Keep development, test, staging, and production resources isolated. Firebase recommends a separate project for each environment in a release workflow. This reduces the chance that a test build changes production data or mixes development events into production analytics.

~~~text
project-example-dev      local development and test data
project-example-staging  release-candidate checks
project-example-prod     real users and production data
~~~

Use fake or anonymized data in pre-production. Apply Security Rules in every environment. Give production project access only to people who need it, and label the production project clearly in the console.

For one-person experiments that do not need persistent cloud resources, the Local Emulator Suite can provide a local environment. The emulator workflow is covered in chapter 12.

## Know the project identifiers

Firebase shows several values that look like identifiers:

- Project name is a human-readable label used to recognize the project in the console.
- Project ID is a unique string that appears in resource names and URLs. Choose it carefully because it cannot be changed after project creation.
- Project number is a numeric identifier used by some Google Cloud and Firebase APIs.

Do not confuse the project name with the project ID. The name can help people recognize an environment, but the ID determines resource names such as default Hosting domains. Treat IDs as visible information, not as passwords.

A web app configuration object connects the client SDK to a project and app registration. Values such as the project ID and Firebase API key identify the application configuration. They do not grant database or file access by themselves. Security Rules, Authentication, service settings, and applicable API restrictions provide the access controls.

## Choose data location deliberately

Backend services can store data in selected locations. Region choice affects request latency, legal or organizational requirements, and how services interact. Some service locations cannot be changed after the resource is created, so check the current service documentation and project requirements before provisioning a database or bucket.

Choose a location near the expected users and related backend services when possible. Do not assume every Firebase product in a project automatically uses the same region.

## Create a project and register the first web app

A practical setup sequence is:

1. Create or select a Firebase project for the intended environment.
2. Choose a clear project name and project ID.
3. Review the project's access, billing, and organization settings.
4. Register a web app with a nickname that identifies its role.
5. Select the Firebase services required by the feature.
6. Decide each service's data location before creating the resource.
7. Add the modular JavaScript SDK in chapter 2.
8. Configure Authentication and Security Rules before storing real user data.
9. Keep development and production clients pointed at their matching projects.

An app nickname is a label for your team in the console. It does not affect the app's permissions or display name to end users.

## Read the product boundaries

The browser client is an untrusted environment. A user can inspect downloaded JavaScript and send requests outside your interface. Hiding a button or a configuration value in the client does not authorize or deny a backend operation.

Use Authentication to establish a user identity, Security Rules to authorize client access to Firestore, Realtime Database, and Storage, and trusted server code for operations that must not be controlled by the browser. Server SDKs and service accounts have different privileges from browser SDKs.

## Practice

1. Draw the relationship among a Google Cloud project, a Firebase project, registered apps, and shared services.
2. Give one example of platform apps that belong in the same project and one example of products that need separate projects.
3. Create a naming scheme for development, staging, and production project IDs.
4. Find the project name, project ID, and project number in the console and explain what each identifies.
5. List the Firebase services a small web application needs and state why each is needed.
6. Explain why a Firebase web configuration object is not a replacement for Security Rules.
7. Choose a region for a test database and list the factors to check before creating the production database.
8. Identify one experiment that can use the Emulator Suite instead of a persistent cloud project.

## Main references

- [Understand Firebase projects](https://firebase.google.com/docs/projects/learn-more)
- [Firebase project setup best practices](https://firebase.google.com/docs/projects/dev-workflows/general-best-practices)
- [Firebase environment overview](https://firebase.google.com/docs/projects/dev-workflows/overview-environments)
- [Firebase security guidelines for environments](https://firebase.google.com/docs/projects/dev-workflows/general-security-guidelines)
- [Firebase launch checklist](https://firebase.google.com/support/guides/launch-checklist)
