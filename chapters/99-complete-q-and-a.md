# 99. Complete questions and answers

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: All code samples](./98-all-code-samples.md) | [Notes index](../README.md) | End of notes |

This chapter answers review questions for every core chapter. Chapters 1 to 3 use questions drawn from their hands-on practice sections. Chapters 4 to 16 use their chapter review questions.

## Chapter 1: Firebase projects, apps, and product map

[Open the chapter](./01-firebase-projects-apps-and-product-map.md)

### 1. How do a cloud project, a Firebase project, registered apps, and Firebase products relate to one another?

**Answer:** A cloud project is the Google Cloud resource container. Enabling Firebase adds Firebase management and products to that project. Registered apps identify client applications inside it, while services such as Auth and Firestore are configured for the project and can be used by its apps.

### 2. When should platform apps share a project, and when should products use separate projects?

**Answer:** Android, iOS, and web clients for one product and one environment commonly share a project so they can use the same users and data. Keep unrelated products or isolated environments in separate projects when access, data, billing, or release risk must be separated.

### 3. How can development, staging, and production project IDs be named safely?

**Answer:** Use distinct, descriptive IDs such as myapp-dev, myapp-staging, and myapp-prod. Project IDs are globally unique and effectively permanent, so decide them carefully and never rely on a display name to select a deployment target.

### 4. What does the project name, project ID, and project number identify?

**Answer:** The project name is a human-readable label that can be changed. The project ID is the stable identifier used by tools and configuration. The project number is a numeric identifier assigned to the Google Cloud project.

### 5. Which Firebase services might a small web app need, and what does each service do?

**Answer:** Authentication manages sign-in, Firestore stores structured app data, Storage stores uploaded files, and Hosting serves a web build. Add only services the product needs, then define access with the corresponding security controls.

### 6. Why does a Firebase web configuration object not replace Security Rules?

**Answer:** The web configuration tells the SDK which Firebase project and app to contact. Client configuration is visible by design. Authentication identifies a user, while Security Rules decide which data that user may read or change.

### 7. What should be considered before choosing a database region?

**Answer:** Check user latency, legal or organizational location requirements, supported services, and whether the database region can be changed later. Choose production locations deliberately because some database location choices are permanent.

### 8. When is the Emulator Suite a better place for an experiment than a cloud project?

**Answer:** Use emulators for local integration tests, rule checks, and disposable experiments. A cloud project is useful when a feature specifically depends on a hosted service or behavior the local emulator does not provide.

## Chapter 2: JavaScript SDK setup and modular initialization

[Open the chapter](./02-javascript-sdk-setup-and-modular-initialization.md)

### 1. What files normally change when the Firebase JavaScript SDK is installed?

**Answer:** The package manifest records the Firebase dependency and the lock file records the resolved package tree. Commit both so another developer or build server installs the same dependency versions.

### 2. Where should a registered web app configuration be initialized?

**Answer:** Initialize the SDK once in a shared module that imports the needed Firebase functions and exports the app and service instances. Keep environment selection in configuration rather than repeating initialization in each feature.

### 3. Why should feature modules reuse one initialized Firebase app?

**Answer:** Repeated initialization can create conflicting app instances and make Auth or data clients use different configuration. Reusing the shared instance keeps the app connected to the intended project consistently.

### 4. How should a build choose between development and staging projects?

**Answer:** Provide separate environment configuration and select it through the build or deployment environment. Verify the resulting project ID before using data or deploying, especially for production commands.

### 5. Why can Firebase web configuration be visible in a development build?

**Answer:** A browser must receive the project and app identifiers to connect to Firebase. Anything embedded in a client build can be inspected, so it must not contain server credentials or be treated as a secret.

### 6. Which credentials must stay on a server and out of a browser bundle?

**Answer:** Service account private keys, Admin SDK credentials, private API keys, and other privileged server secrets belong in a protected server or CI secret store. Never bundle them into browser JavaScript.

### 7. When should a screen use a live Firestore listener versus a one-time read?

**Answer:** Use a live listener when the screen must reflect updates as they happen and the subscription is properly cleaned up. Use a one-time read for a snapshot view that does not need continuous updates.

### 8. Why does server-rendered request code need different initialization care from browser code?

**Answer:** A browser app can keep one client app instance for its lifetime. Server-rendered code handles concurrent requests, so request-specific user state must not leak between requests and privileged credentials must stay server-side.

## Chapter 3: Authentication and account lifecycle

[Open the chapter](./03-authentication-and-account-lifecycle.md)

### 1. What should be enabled and tested before adding email and password sign-in to an app?

**Answer:** Enable the desired sign-in provider in a development Firebase project, configure its allowed domains when needed, and test account creation, sign-in, error handling, and sign-out with a non-production account.

### 2. How does the Auth observer keep the page in sync after signing in or out?

**Answer:** The observer reports the current user when authentication state changes. Render loading, signed-out, or signed-in UI from that state instead of assuming that a sign-in call is the only source of truth.

### 3. How does session persistence differ from the default persistence choice?

**Answer:** The default browser persistence can retain a session across browser restarts. Session persistence ends when the tab or window session ends, while in-memory persistence lasts only for the current page instance.

### 4. How can an app confirm that an email verification state has been refreshed?

**Answer:** Send the verification email, then reload or call the user refresh method before checking the updated verification property. Do not assume the local user object changes immediately after the email is verified.

### 5. Why should password recovery show a neutral response?

**Answer:** A neutral message avoids revealing whether an email address has an account. Show the same confirmation for eligible and ineligible addresses, then handle any follow-up through the email link.

### 6. What private app state should be cleared on sign-out?

**Answer:** Clear cached private records, user-specific query results, and local UI state when signing out. This prevents data from the previous account remaining visible to the next person using the browser.

### 7. Why is a Firebase Auth UID a better ownership key than an email address?

**Answer:** A UID is stable for the account and is not changed when the user updates an email address. Email addresses can change and are personal information, so they are a poor key for ownership checks.

### 8. How can a Firestore rule restrict a profile document to its signed-in owner?

**Answer:** Match the profile document path to the authenticated UID and require request.auth.uid to equal that UID in the rule. Also validate the fields a client may write instead of relying only on a hidden UI.

## Chapter 4: Cloud Firestore documents and data modeling

[Open the chapter](./04-cloud-firestore-documents-and-data-modeling.md)

### 1. What is the difference between a collection and a document?

**Answer:** A collection contains documents and provides a path for querying related records. A document stores fields and can contain subcollections, but it is not itself a collection.

### 2. Which Firestore value should represent a moment in time?

**Answer:** Use a Firestore Timestamp for a moment in time. It avoids inconsistent formatted strings and supports timestamp comparisons, ordering, and date conversion in the SDK.

### 3. When would a stable document ID be useful?

**Answer:** A stable ID lets the app address the same record across reads, updates, and relationships. It is useful when another system supplies a durable key or when a retry must not create a duplicate.

### 4. What is the difference between addDoc, setDoc, and updateDoc?

**Answer:** addDoc creates a document with an automatically generated ID. setDoc writes to a chosen ID and can replace or merge data. updateDoc changes selected fields and fails when the target document does not exist.

### 5. Why should documents in the same collection usually share a consistent field shape?

**Answer:** A consistent shape makes query results and UI rendering predictable, reduces missing-field cases, and supports rule validation. Optional fields should be clearly handled rather than guessed by each screen.

### 6. When is a subcollection a better fit than an array field?

**Answer:** A subcollection fits a large or independently queried set of child records. An array is convenient for a small bounded list read and changed with its parent, but it grows with the document and is not independently queried as documents.

### 7. What happens to subcollection documents when the parent document is deleted?

**Answer:** Deleting a parent document does not automatically remove documents in its subcollections. Plan explicit recursive cleanup or a backend cleanup process for nested data.

### 8. When can duplicating a small field make a screen simpler, and what update decision must accompany that duplication?

**Answer:** Duplicating a small display field can avoid an extra read or simplify a list row. Decide which copy is authoritative and update every dependent copy when that source value changes.

## Chapter 5: Firestore reads, queries, indexes, and pagination

[Open the chapter](./05-firestore-reads-queries-indexes-and-pagination.md)

### 1. What is the difference between getDoc and getDocs?

**Answer:** getDoc reads one document reference. getDocs executes a collection or query and returns the documents matching that query.

### 2. What does a QuerySnapshot contain?

**Answer:** A QuerySnapshot contains the documents returned by a query plus metadata such as whether the result is empty and the number of documents. Iterate its docs to access individual document snapshots.

### 3. Why should a growing collection query include a limit?

**Answer:** A limit caps result size, reduces reads and rendering work, and creates a clear basis for pagination. Choose a sensible page size and fetch more only when the user needs it.

### 4. What does orderBy do when a document lacks the ordered field?

**Answer:** Documents without the ordered field are excluded from results for that orderBy query. Ensure the field exists consistently or choose a query model that does not omit valid records.

### 5. When might Firestore request a composite index?

**Answer:** A composite index is needed when a query combines filters or ordering in a way that is not covered by a built-in single-field index. Firestore reports a link to create the required index.

### 6. How does a document snapshot work as a cursor?

**Answer:** A document snapshot cursor records the sort values and document position of the last result. Pass it to startAfter so the next page begins after that record without offset scanning.

### 7. Why should a live listener be unsubscribed?

**Answer:** A listener keeps receiving updates and consumes resources while active. Unsubscribe when the view unmounts, the account changes, or the result is no longer needed.

### 8. Why must the query itself satisfy the Firestore Security Rules?

**Answer:** Rules evaluate whether the complete query could return only documents the client is allowed to read. A query that might include forbidden documents is denied instead of filtered down.

## Chapter 6: Firestore Security Rules and authorization

[Open the chapter](./06-firestore-security-rules-and-authorization.md)

### 1. What is the difference between authentication and authorization?

**Answer:** Authentication establishes who is making a request. Authorization decides whether that identity may perform a particular operation on a resource.

### 2. What are resource.data and request.resource.data?

**Answer:** resource.data is the existing document data. request.resource.data is the proposed document after a create or update, so rules can compare old and new values.

### 3. Why are Firestore Security Rules not query filters?

**Answer:** Rules are not post-query filters. Firestore rejects a query if its possible result set could include a document the caller is not allowed to read.

### 4. How can a task rule verify that its owner is the signed-in user?

**Answer:** Require request.auth to exist and compare request.auth.uid with the task owner field in the document path or stored data. Apply the check to reads and writes as appropriate.

### 5. Why should an update rule prevent ownerId from changing?

**Answer:** If a client could change ownerId, it could transfer or claim another account's data. Compare the existing owner to the proposed owner and require them to remain equal during updates.

### 6. Does a rule on a user document automatically cover that user's subcollection?

**Answer:** No. Each subcollection path needs a matching rule. A rule for a user document does not automatically authorize nested collection documents.

### 7. Which access-control system applies to server client libraries?

**Answer:** Server client libraries authenticate through Google Cloud IAM and bypass Firestore Security Rules. Give server identities only the IAM permissions they need and validate authorization in server code.

### 8. Name four allowed and denied cases you would test for a user-owned task collection.

**Answer:** Test that the owner can read and update their own task, another user cannot read it, another user cannot update or delete it, and a signed-out client is denied. Also test invalid ownership changes.

## Chapter 7: Realtime Database and live synchronization

[Open the chapter](./07-realtime-database-and-live-synchronization.md)

### 1. How is Realtime Database data organized?

**Answer:** Realtime Database stores data as one JSON tree addressed by paths. Design shallow paths that support the reads and access checks the app needs.

### 2. What happens to child paths when set replaces a parent value?

**Answer:** set replaces the value at the target path, including its descendants. Writing a value at a parent can therefore remove existing child paths.

### 3. When should update be used instead of set?

**Answer:** Use update to change selected child paths while preserving siblings. Use set when replacing the whole value at a path is intended.

### 4. What does onValue do after its initial callback?

**Answer:** onValue first calls the callback with the current value, then calls it again whenever data at the subscribed path or below it changes.

### 5. Why should a listener be attached low in the data tree?

**Answer:** A listener on a high-level path downloads and processes more of the JSON tree whenever descendants change. Subscribe to the smallest path that contains the data the screen needs.

### 6. When is runTransaction safer than reading and then writing?

**Answer:** Use runTransaction when the new value depends on the current value and concurrent clients could update it. The SDK may retry the update function with a newer value.

### 7. What does onDisconnect allow a client to register?

**Answer:** onDisconnect registers a server-side action to run when the client connection closes. It is commonly used to mark a presence record offline after first arranging the online status safely.

### 8. How do Realtime Database Rules use an authenticated user's UID?

**Answer:** Rules can compare the authenticated UID with a path segment or data owner field. Require an authenticated user and matching UID for private user-owned records.

## Chapter 8: Cloud Storage for files

[Open the chapter](./08-cloud-storage-for-files.md)

### 1. What kind of data belongs in Cloud Storage?

**Answer:** Cloud Storage is suited to large binary objects such as images, video, audio, and documents. Store searchable metadata and references in Firestore rather than placing file bytes in a document.

### 2. Why might an app store an object path in Firestore?

**Answer:** A Firestore field can keep the Storage object path beside the record metadata. The app can use that stable path to locate the file and apply Storage rules independently.

### 3. What does uploadBytesResumable provide over a basic upload?

**Answer:** uploadBytesResumable reports progress and supports pausing, resuming, and observing completion or errors. A basic upload is simpler when progress handling is unnecessary.

### 4. Why is the browser's file type check not enough?

**Answer:** A browser-provided MIME type and file name can be forged. Validate file size and content as appropriate, and enforce allowed paths, sizes, and content types in Storage rules or trusted backend processing.

### 5. What does getDownloadURL return?

**Answer:** getDownloadURL returns a URL that can be used to retrieve the object. Treat it as access-bearing data and avoid exposing it where the file is intended to stay private.

### 6. Why should a private download URL be handled carefully?

**Answer:** A download URL may be copied and used outside the app, so possession can grant access depending on its token and object setup. Do not treat an obscure URL as user authorization.

### 7. Does deleting a Firestore document delete its associated file?

**Answer:** No. Deleting the Firestore record does not delete the separate Storage object. Remove both explicitly or use a trusted cleanup function and handle failures between the two operations.

### 8. Which checks should a Storage rule make for a user's image upload?

**Answer:** Require an authenticated user, restrict the object path to that user's UID, limit file size, and check an allowed content type. Remember that MIME metadata alone does not prove the actual file contents.

## Chapter 9: Cloud Functions and event-driven work

[Open the chapter](./09-cloud-functions-and-event-driven-work.md)

### 1. What work is a good fit for a Cloud Function?

**Answer:** Use a Cloud Function for trusted server-side work, integration with private APIs, scheduled jobs, or reactions to Firebase events that should not run with client privileges.

### 2. How does a callable function differ from a plain HTTP endpoint?

**Answer:** A callable function uses the Firebase client SDK protocol and can include Auth and App Check context automatically. A plain HTTP endpoint handles its own request parsing, authentication, CORS, and response format.

### 3. Why should a function derive the owner ID from request.auth?

**Answer:** The client can send a forged owner ID. Use the authenticated identity in request.auth to determine ownership, then validate any requested resource against it.

### 4. Which security rules apply to Admin SDK requests?

**Answer:** Admin SDK operations bypass Firebase Security Rules. Protect them with IAM and perform authorization checks in the function before changing data.

### 5. Why should event-triggered functions be idempotent?

**Answer:** Event delivery can be retried, and the same logical event may be processed more than once. Make the handler safe to repeat by using stable event IDs, idempotent writes, or a processed-event record.

### 6. What can cause a function to trigger itself repeatedly?

**Answer:** A function can trigger itself when its write changes a document that matches its own trigger. Check whether the data needs a change before writing and avoid recursive updates.

### 7. How should a function receive a private API key?

**Answer:** Store a private key in Secret Manager or the supported Functions secrets configuration, grant access only to the needed function, and never include it in source or client code.

### 8. What should be checked before deploying a function to a project?

**Answer:** Confirm the target project, runtime configuration, secrets, IAM, trigger paths, and required APIs. Test locally or in staging and deploy only the intended function targets.

## Chapter 10: Firebase Hosting and web delivery

[Open the chapter](./10-firebase-hosting-and-web-delivery.md)

### 1. What type of files does Firebase Hosting primarily deliver?

**Answer:** Firebase Hosting primarily delivers static web assets such as HTML, JavaScript, CSS, images, and fonts. It can also route requests to supported dynamic backends through configured rewrites.

### 2. Why does a client-side router usually need a Hosting rewrite?

**Answer:** A client-side router handles routes after the app loads, but a direct browser request to a nested URL otherwise looks like a missing file. Rewrite that request to the app entry point.

### 3. When should a redirect be used instead of a rewrite?

**Answer:** Use a redirect when the browser should navigate to a new URL, such as after a page permanently moves. Use a rewrite when the URL should remain while Hosting serves content from another destination.

### 4. Why can immutable caching be useful for content-hashed assets?

**Answer:** Content-hashed asset names change when their contents change. Those files can use long immutable cache lifetimes because a new build gets a different URL.

### 5. What can a Hosting preview channel be used to review?

**Answer:** A preview channel gives reviewers a temporary hosted build to inspect before it replaces the live site. Check the exact build, routing, and environment behavior there.

### 6. Which Firebase project receives a Hosting deployment?

**Answer:** The active Firebase project selected by the CLI receives the deployment. Verify the project alias or explicit project ID before every production deployment.

### 7. What should be checked before rolling back a release?

**Answer:** Check the current live version, the version being restored, the target project, routing, and any backend compatibility. A Hosting rollback does not undo database writes or other service changes.

### 8. When might a server-rendered app need a different Firebase hosting product?

**Answer:** An app that needs server rendering or server-side request handling may need an integrated framework deployment or a Hosting rewrite to a supported server backend instead of static Hosting alone.

## Chapter 11: App Check and abuse reduction

[Open the chapter](./11-app-check-and-abuse-reduction.md)

### 1. What does App Check help a Firebase service assess?

**Answer:** App Check helps a Firebase service assess whether a request likely comes from an authentic app or device. It reduces abuse but does not establish which user is signed in.

### 2. How does App Check differ from Authentication?

**Answer:** Authentication identifies the user. App Check attests the app instance or environment making the request; an app can need both checks for different purposes.

### 3. Why should App Check initialize before other Firebase services?

**Answer:** Initialize App Check early so Firebase service requests can include valid App Check tokens from the start. Late initialization can leave early requests unprotected or rejected after enforcement.

### 4. Which key is safe to use in the web client, and which key must remain private?

**Answer:** The site key for the chosen web provider is used by the client and is expected to be public. Provider secret keys and other server credentials must remain private on a trusted server.

### 5. What should be reviewed before enabling enforcement?

**Answer:** Review request metrics, valid app versions, provider setup, and expected traffic first. Monitor for missing or invalid attestations and fix legitimate clients before enabling enforcement.

### 6. Why should a debug token be kept out of production?

**Answer:** A debug token bypasses normal attestation for local testing. If it is exposed or enabled in a production build, unauthorized clients may be able to use it, so register it only for development and protect it.

### 7. Does App Check authorize a user to read another user's document?

**Answer:** No. App Check evaluates app authenticity, while Firestore Security Rules or server authorization decide whether a user may read another user's document.

### 8. What should be checked when requests fail after enforcement?

**Answer:** Check provider configuration, registered app IDs, initialization order, enforcement status, valid debug setup for local testing, and service request metrics. Inspect the exact service and error before relaxing production enforcement.

## Chapter 12: Local Emulator Suite and testing

[Open the chapter](./12-local-emulator-suite-and-testing.md)

### 1. What is the purpose of the Local Emulator Suite?

**Answer:** The Local Emulator Suite runs local versions of supported Firebase services so developers can test app behavior, data flows, and rules without using production resources.

### 2. Does starting an emulator automatically connect the Firebase web SDK to it?

**Answer:** No. The app must explicitly call the relevant connect-to-emulator function for each SDK service before making requests.

### 3. Why should the app use one project ID consistently?

**Answer:** A consistent project ID lets the app, emulators, and imported data refer to the same logical project. Mismatched IDs can make services appear disconnected or split test data.

### 4. Which emulators can evaluate Firebase Security Rules?

**Answer:** The Authentication, Cloud Firestore, Realtime Database, and Cloud Storage emulators can evaluate their corresponding Firebase Security Rules. Use the supported emulator for each service under test.

### 5. What does emulators:exec do?

**Answer:** emulators:exec starts the configured emulators, runs a command such as a test suite, and shuts the emulators down when that command finishes.

### 6. Why should an emulator suite not be treated as a production service?

**Answer:** Emulators are local development tools and do not provide the production availability, access control, backup, or operational guarantees of hosted services.

### 7. How should local test data be handled?

**Answer:** Use disposable, documented fixtures and reset or isolate them between tests. Do not import production personal data into a local test environment without a clear need and protection.

### 8. What is the risk of a committed App Check debug token?

**Answer:** A committed debug token can let anyone who obtains it use a debug App Check client. Keep it out of source control and production builds, and revoke it if exposed.

## Chapter 13: Firebase Cloud Messaging for web

[Open the chapter](./13-firebase-cloud-messaging-for-web.md)

### 1. What does FCM use in a web app to receive background messages?

**Answer:** A web app uses a service worker to receive and handle background FCM messages when the page is not active. The worker must be registered and configured for the app.

### 2. Why must a web push app use HTTPS?

**Answer:** Web push requires a secure context. Production sites use HTTPS, while localhost is generally treated as secure for development.

### 3. When should an app ask for notification permission?

**Answer:** Explain the value of notifications in context and ask after a user action that signals interest. Do not prompt immediately on page load without context.

### 4. What does an FID identify?

**Answer:** A Firebase Installation ID identifies a Firebase installation for a particular app installation. It is distinct from a user account and can change when the installation changes.

### 5. Why must the server associate an FID with an authenticated user?

**Answer:** An installation identifier alone does not prove which person owns it. Bind the registration to an authenticated account through a trusted backend and remove or update that binding when account ownership changes.

### 6. How does onMessage differ from a background service worker handler?

**Answer:** onMessage handles foreground messages while the page is active. The service worker handles background messages and can display a notification when the page is not active.

### 7. Why should a notification click destination be checked?

**Answer:** Validate the notification destination before navigation so an untrusted payload cannot send the user to an unsafe external URL or unintended application route.

### 8. What should happen when a user opts out or signs out?

**Answer:** Respect a clear opt-out, stop sending to the installation, and remove or update the account-to-installation association on sign-out. The user should be able to change notification preferences.

## Chapter 14: Analytics and Performance Monitoring

[Open the chapter](./14-analytics-and-performance-monitoring.md)

### 1. What question should a custom Analytics event answer?

**Answer:** A custom Analytics event should answer a product question, such as whether users complete a key action. Define its meaning before adding instrumentation.

### 2. Why should event names and parameter values stay consistent?

**Answer:** Consistent names and parameter values make reports comparable over time. Changing spellings or mixing categories fragments the same behavior into multiple report entries.

### 3. Which kinds of values should not be sent as event parameters?

**Answer:** Do not send directly identifying or sensitive personal information, credentials, message contents, or private records as event parameters. Use approved, non-identifying values only.

### 4. How can a web app check whether Analytics is supported?

**Answer:** Call the Analytics support check provided by the Web SDK before initializing or using Analytics, because some browsers or environments are unsupported.

### 5. What does setAnalyticsCollectionEnabled control?

**Answer:** setAnalyticsCollectionEnabled controls whether Analytics data collection is enabled for the app instance. Use it to honor a user choice or a product requirement.

### 6. What is the default metric for a custom Performance trace?

**Answer:** A custom Performance trace records duration by default, measured from its start until it is stopped. Add custom metrics only when they clarify the measured operation.

### 7. Why should a trace stop in a finally block?

**Answer:** Stopping in finally ensures a trace ends whether the operation succeeds or throws. Otherwise failed paths may leave incomplete timing measurements.

### 8. Why can an Analytics report take time to show a new event?

**Answer:** Reports can take time to process and populate after an event is sent. Check event spelling and collection settings, then allow reporting time before concluding that instrumentation failed.

## Chapter 15: Environments, configuration, and CI/CD

[Open the chapter](./15-environments-configuration-and-ci-cd.md)

### 1. Why should development and production use separate Firebase projects?

**Answer:** Separate projects prevent development or test data, configuration, and access from being mixed with production. They also let teams control release permissions and risk by environment.

### 2. Which resources do apps in the same Firebase project share?

**Answer:** Apps within a project can share project-level Auth users, databases, Storage, settings, quotas, and billing context. Registering another app does not create a fully isolated backend.

### 3. What information belongs in a VITE_ variable?

**Answer:** A VITE_ variable may contain public client configuration needed by browser code. Anything with that prefix can be exposed in the built assets, so private credentials do not belong there.

### 4. Why is a web Firebase configuration not an authorization control?

**Answer:** The web configuration points the SDK at a project but is visible to every client. Authorization comes from Security Rules, backend checks, and IAM controls.

### 5. What should a pull request workflow verify before deployment?

**Answer:** A pull request workflow should install locked dependencies, run linting and tests, validate rules and build output, and deploy only to an approved preview or staging target.

### 6. Why should production credentials be unavailable to untrusted pull requests?

**Answer:** Untrusted pull request code could print or misuse secrets available to its job. Do not provide production credentials to workflows triggered by code that has not been trusted and reviewed.

### 7. Why should Hosting deployment be separate from rules and Functions deployment?

**Answer:** Hosting files, Security Rules, indexes, and Functions have different review and release risks. Deploy only the intended targets so a site change does not unexpectedly alter access rules or backend behavior.

### 8. What information helps connect a deployed release back to its source commit?

**Answer:** Record the source commit hash, build identifier, target project, and deployment time. This makes a live release traceable to its reviewed source.

## Chapter 16: Billing, operations, debugging, and production checklist

[Open the chapter](./16-billing-operations-debugging-and-production-checklist.md)

### 1. Does a Firebase plan apply separately to each registered app?

**Answer:** A Firebase billing plan applies at the project level, not independently to each registered app. Apps in one project share its billing setup and relevant usage context.

### 2. What is the difference between a budget alert and a spend cap?

**Answer:** A budget alert notifies people when estimated spending crosses thresholds. It does not automatically stop usage or guarantee a maximum bill.

### 3. Which usage measures should be reviewed for Firestore, Storage, and Functions?

**Answer:** Review Firestore document reads and writes and storage, Storage bytes and operations, and Functions invocations, execution time, and outbound traffic. Compare the usage with expected product activity.

### 4. What information makes a structured log useful?

**Answer:** A useful structured log includes a timestamp, severity, operation, request or correlation ID, outcome, and non-sensitive context needed to diagnose the event.

### 5. Which values should never be written to operational logs?

**Answer:** Never log passwords, tokens, private keys, payment details, message contents, or unnecessary personally identifying data. Redact secrets and restrict access to operational logs.

### 6. Why should a restore be tested in an isolated project?

**Answer:** An isolated restore test verifies the backup can actually be recovered without overwriting live data. It also exposes missing steps and compatibility problems before an incident.

### 7. Does rolling back Hosting reverse database writes?

**Answer:** No. A Hosting rollback changes the served release, but database writes and changes to other Firebase services remain. Plan a separate safe recovery for data and backend changes.

### 8. What should be checked before and after a production release?

**Answer:** Before release, check the target project, reviewed commit, tests, rules, indexes, secrets, and rollback path. After release, verify key user flows, service errors, logs, and usage metrics.
