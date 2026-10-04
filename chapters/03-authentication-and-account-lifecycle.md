# 03. Authentication and account lifecycle

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: JavaScript SDK setup and modular initialization](./02-javascript-sdk-setup-and-modular-initialization.md) | [Notes index](../README.md) | [Next: Cloud Firestore documents and data modeling](./04-cloud-firestore-documents-and-data-modeling.md) |

## Authentication answers who is signed in

Firebase Authentication verifies a user's identity using a configured sign-in method. It can provide a stable UID for the signed-in Firebase user and expose profile fields such as email, display name, and provider information.

Authentication does not decide which documents, files, or operations that user can access. Your database and Storage Security Rules must authorize each client request. A signed-in user is not automatically allowed to read every record.

Keep three concerns separate:

| Concern | Question | Firebase mechanism |
| --- | --- | --- |
| Authentication | Who signed in? | Firebase Authentication |
| Authorization | What can that identity read or change? | Security Rules or trusted server checks |
| Profile data | What app-specific details should the product store? | A protected database document or another approved source |

Do not store passwords in Firestore or Realtime Database. Authentication providers manage credentials.

## Enable a sign-in method

In the Firebase console, open Authentication and configure the provider your application will use. For email and password sign-in, enable that method. For Google or another federated provider, configure the provider and any required project settings.

For a web app, check the authorized domain configuration for the hosts that will sign in. Local development, staging, and production may use different domains and Firebase projects.

Only show a sign-in button for providers that are enabled for the current project. Provider configuration and redirect behavior are part of the authentication setup, not just a client import.

## Create and sign in with an email account

The modular Auth SDK exposes promise-based functions. Validate form input for a helpful user experience, then handle service errors as part of the flow.

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

A new user is signed in after the account is created. The returned user object contains the Firebase UID. Send verification when the product needs a verified address, and make the application check the refreshed verification state before enabling sensitive actions.

For an existing account:

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

Catch rejected promises at the UI boundary. Show a clear message that helps the user recover, such as checking credentials or trying a reset. Avoid displaying raw internal error details. Do not expose whether a particular email address has an account if that would reveal private account information.

## Observe authentication state

A page can first load before the SDK has restored its saved sign-in state. The currentUser property may be null during initialization, so use an observer to decide which UI to show.

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

The observer fires when the state becomes available and when a user signs in or out. In a component-based UI, call the returned unsubscribe function when the component or listener owner is removed. Avoid adding duplicate observers every time a page renders.

Use the UID as the stable owner key for that Firebase account. Do not use email as a database document key because email addresses can change and may be shared or reassigned under some systems.

## Choose how long the browser remembers a session

Web Authentication uses local persistence by default when the browser supports it. The user remains signed in across browser restarts until sign-out or browser storage is cleared.

You can select a different persistence before sign-in:

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

The main web choices are:

| Persistence | Behavior |
| --- | --- |
| browserLocalPersistence | Retains the sign-in after closing and reopening the browser |
| browserSessionPersistence | Retains the sign-in for the current browser tab or window session |
| inMemoryPersistence | Keeps the state in memory and clears it when the page is refreshed |

Choose persistence according to the product and device. It changes how long a session is remembered, not what the user is authorized to access. Local persistence can synchronize sign-in state across tabs; session and in-memory states have different tab behavior.

## Sign out cleanly

Call signOut to clear the Firebase Auth session, then let the observer update the UI:

~~~js
import { getAuth, signOut } from "firebase/auth";

const auth = getAuth();

async function signOutCurrentUser() {
  await signOut(auth);
}
~~~

Also clear any application state that belongs only to that user, such as cached private records. Do not assume hiding a user menu is equivalent to signing out.

## Add password recovery

A password reset flow asks for an email address and sends a recovery message through the configured Firebase project:

~~~js
import { getAuth, sendPasswordResetEmail } from "firebase/auth";

const auth = getAuth();

async function requestPasswordReset(email) {
  await sendPasswordResetEmail(auth, email);
}
~~~

Use Firebase's configured templates and authorized redirect settings. Display a neutral confirmation where appropriate so the form does not reveal whether the address has an account.

Security-sensitive account changes can require recent authentication. If an operation asks for reauthentication, prompt the user to sign in again through the configured provider, then retry the operation.

## Keep profile data and permissions safe

The Authentication user profile is not a full product profile. If the app stores a display name, preferences, or other domain data in Firestore, protect the document using the user's UID and Security Rules.

Treat user-controlled display names, photo URLs, and other profile strings as untrusted input. Escape or safely render them in the interface. A profile field does not grant elevated permissions. Administrative roles must be assigned through trusted server code and enforced by Rules or backend authorization.

For a federated provider, handle account linking and duplicate-provider cases deliberately. A person signing in with two providers may need an account-linking flow rather than a second unrelated profile.

## Handle failures as distinct cases

| Failure | Helpful response |
| --- | --- |
| Invalid email or password format | Explain the required input format before submission |
| Sign-in rejected | Give a neutral sign-in message and offer password recovery |
| Email is not verified | Offer to resend verification and refresh the user state after they return |
| Provider is disabled | Enable the provider in the correct Firebase project or hide its button |
| Domain is not authorized | Add the intended development or production domain in project settings |
| Too many requests | Slow repeated attempts and follow the current provider limits |
| Recent sign-in required | Reauthenticate through the user's existing provider |

Do not build a parallel password store to work around Auth errors. Check the project, provider configuration, current user state, and exact error code.

## Practice

1. Enable email and password sign-in in a development project and create a test account.
2. Sign out, sign in again, and explain how the Auth observer updates the page.
3. Set session persistence before a new sign-in and compare it with the default.
4. Send an email verification message and check the refreshed verification state.
5. Request password recovery and describe a neutral success message.
6. Sign out and clear the app's private in-memory state.
7. Explain why a UID is a better owner key than an email address.
8. Write a Firestore rule that checks the authenticated UID before allowing access to a user's own profile document.

## Main references

- [Get started with Firebase Authentication for web](https://firebase.google.com/docs/auth/web/start)
- [Manage users in Firebase Authentication](https://firebase.google.com/docs/auth/web/manage-users)
- [Authentication state persistence](https://firebase.google.com/docs/auth/web/auth-state-persistence)
- [Firebase Authentication security](https://firebase.google.com/docs/auth)
