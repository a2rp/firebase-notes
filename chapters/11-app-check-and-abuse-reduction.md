# 11. App Check and abuse reduction

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Firebase Hosting and web delivery](./10-firebase-hosting-and-web-delivery.md) | [Notes index](../README.md) | [Next: Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md) |

## What App Check does

Firebase App Check helps a supported Firebase service verify that a request comes from a registered app using a configured attestation provider. It can reduce some unauthorized use of backend resources.

App Check and Authentication solve different problems:

- Authentication identifies a signed-in user.
- Security Rules decide which records that user may access.
- App Check helps the service assess whether a request comes from an expected app environment.

Use these controls together. App Check does not replace user authorization, input validation, rate limits, or server-side permission checks.

## Register a web app and provider

For a web app, Firebase currently documents reCAPTCHA Enterprise as the recommended provider for new integrations. Create a score-based web key, register the app in the Firebase console, and configure the allowed domains. Keep development and production domains and keys organized by environment.

Never put a reCAPTCHA secret key in client code. The web app uses the site key, which is designed to be public. The server-side secret remains in the provider and Firebase configuration.

## Initialize App Check before other services

Initialize App Check early in the application startup, before the app begins using protected Firebase services.

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

The site key is not a secret, but the Vite environment value is still included in the client build. Set the matching provider key in the console and allow the domains where the app actually runs.

Automatic token refresh is disabled unless enabled in the initialization options. Refresh frequency and token lifetime affect security, latency, quota, and provider cost. Follow the current provider guidance instead of choosing an unusually short lifetime without a reason.

## Monitor before enabling enforcement

After the updated app starts sending App Check tokens, review request metrics for every protected product. Some legitimate clients may not yet send valid tokens, especially older app versions or unregistered environments.

Enable enforcement for one product at a time after the metrics show that legitimate traffic is covered. Requests without valid tokens can be rejected after enforcement. Keep a rollout record and monitor errors after each change.

Services and enforcement behavior can differ. Check the current Firebase documentation for the exact product, SDK, and platform combination before enabling enforcement.

## Use a debug provider only in controlled development

Local development and automated test environments may not pass a production attestation provider. Firebase provides a debug provider for these cases.

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

Use the Firebase App Check debug provider as described in the official setup guide for your SDK version. The example above marks where the environment-specific provider belongs; it is not a complete debug-provider implementation. Register debug tokens only for development, keep them out of committed files and screenshots, and never ship a debug provider in a production build.

A debug token bypasses the normal attestation flow for the registered test client. Treat it as sensitive access material. Rotate or remove it if exposed.

## Protect custom backends

When a custom backend needs to verify that a request carries a valid App Check token, follow Firebase's server verification guidance. Verification of an App Check token does not identify the signed-in user or authorize access to a particular record. Check Firebase Authentication and the application's own permission model separately.

For callable Functions, the Firebase SDK can attach App Check tokens when the app is configured. Confirm the supported enforcement and token behavior in the current Functions documentation.

## Diagnose rejected requests

When requests fail after enforcement, check:

- Whether App Check initialized before the first protected service request.
- Whether the correct site key and registered app are used.
- Whether the current hostname is listed for the provider key.
- Whether the request metrics include older or unsupported clients.
- Whether local tests are using an approved debug setup.
- Whether a project, product, or app registration was changed recently.
- Whether the failure is actually an Authentication or Security Rules denial.

Do not respond to failures by globally opening database or Storage rules. Diagnose the product's request metrics and preserve existing authorization checks.

## Hands-on exercise: staged rollout

1. Register a development web app with the current recommended provider.
2. Add App Check initialization before Auth, Firestore, and Storage initialization.
3. Confirm the development hostname and site key match the console configuration.
4. Run the app and inspect App Check request metrics without enabling enforcement.
5. Identify which clients have missing or invalid tokens.
6. Configure the documented debug provider for local development only.
7. Enable enforcement for one development product after checking its metrics.
8. Test normal, signed-out, and unauthorized user requests.
9. Confirm that Security Rules still prevent one user from reading another user's data.
10. Record how to disable enforcement for a product if a release blocks legitimate traffic.

## Common mistakes

- Treating App Check as a replacement for Authentication or Security Rules.
- Initializing App Check after the first request to a protected service.
- Putting a provider secret key in a web app.
- Enabling enforcement before checking request metrics.
- Forgetting older clients and test environments during rollout.
- Committing a debug token or shipping the debug provider in production.
- Assuming a valid App Check token grants permission to private user data.
- Disabling data rules when an App Check request fails.
- Ignoring token lifetime effects on latency, quota, or cost.

## Practice questions

1. What does App Check help a Firebase service assess?
2. How does App Check differ from Authentication?
3. Why should App Check initialize before other Firebase services?
4. Which key is safe to use in the web client, and which key must remain private?
5. What should be reviewed before enabling enforcement?
6. Why should a debug token be kept out of production?
7. Does App Check authorize a user to read another user's document?
8. What should be checked when requests fail after enforcement?

## Main references

- [App Check with reCAPTCHA Enterprise for web apps](https://firebase.google.com/docs/app-check/web/recaptcha-enterprise-provider)
- [Monitor App Check request metrics](https://firebase.google.com/docs/app-check/monitor-metrics)
- [Enable App Check enforcement](https://firebase.google.com/docs/app-check/enable-enforcement)
- [Use the App Check debug provider for web apps](https://firebase.google.com/docs/app-check/web/debug-provider)
- [App Check overview](https://firebase.google.com/docs/app-check)
