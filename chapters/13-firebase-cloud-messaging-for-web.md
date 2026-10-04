# 13. Firebase Cloud Messaging for web

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Local Emulator Suite and testing](./12-local-emulator-suite-and-testing.md) | [Notes index](../README.md) | [Next: Analytics and Performance Monitoring](./14-analytics-and-performance-monitoring.md) |

## What Firebase Cloud Messaging does

Firebase Cloud Messaging (FCM) delivers messages to supported web browsers through the browser Push API and a service worker. A web app can receive a message while open, and the browser can display a notification while the app is in the background.

Web push requires a secure HTTPS origin. Browsers also require permission from the user before displaying notifications. Check current browser support and product requirements before building the feature.

Use push messages for timely, useful updates. Do not put passwords, private records, or other sensitive content in a notification payload because a notification may appear on a shared or locked screen.

## Configure the web app

Enable Cloud Messaging for the Firebase project, register the web app, and configure the Web Push public key in the Firebase console. A public VAPID key can be included in the web client. Keep private server credentials outside the browser.

FCM also requires a messaging service worker at the app's origin. The default file name is firebase-messaging-sw.js. It must be served from the correct scope and be available over HTTPS.

## Ask permission after the user chooses

Do not ask for notification permission immediately on page load. Explain the benefit and request it after the user turns on notifications or presses an enable button.

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

A denied permission generally requires the user to change browser settings. Do not repeatedly prompt after denial. Provide a clear settings hint only when the user tries to enable the feature again.

## Register the app instance

The current web setup uses Firebase Installation IDs (FIDs) to identify an app installation for message delivery. Register only after permission is granted. Save the FID through an authenticated server operation so it is associated with the correct user.

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

Call register in response to the user's enable action. The onRegistered callback can run again if the installation ID changes or registration is repeated. Update the server record when a new ID is reported. Do not log or publicly expose installation IDs.

The server must verify the signed-in user before storing an FID. Treat an FID as a routing identifier, not as proof of the user's identity or permission to read application data.

## Handle messages while the page is open

Use onMessage in the page to receive foreground messages. Update the app interface or show an in-app notice based on the message.

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

Keep the payload small and validate any data before using it to navigate or change the interface. The message is a signal to refresh or display information, not an authorization decision.

## Handle background messages in the service worker

When the app is in the background, the browser and service worker handle message delivery. For data-only messages, a service worker can create a notification and respond to a click.

A simple service worker can use the Firebase compatibility packages through importScripts. Replace the SDK version placeholder with the exact Firebase JavaScript SDK version used by the web app.

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

The config object identifies the Firebase web app and is public by design. The service worker handles only Messaging. If you use the modular SDK in a service worker, bundle the worker so its module imports are supported by your build.

If the server sends a notification payload, the browser may display it automatically while the app is in the background. Avoid displaying a second notification for the same message.

## Send from trusted server code

Send messages from trusted server code using the Firebase Admin SDK or FCM HTTP v1 API. Never place Admin SDK credentials or server authorization in the browser.

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

This sample uses a data message so the service worker can decide how to display it. Validate that the authenticated user is allowed to receive the message before selecting the FID. Handle invalid or no-longer-registered installations and remove stale identifiers from your server records.

For a visible system notification, include a notification payload or display one from the service worker according to the desired foreground and background behavior. Test the exact payload with each supported browser.

## Manage consent and registration lifecycle

Store the notification preference and app installation registration on the server only after the user opts in. On sign-out, remove the user's association with the installation if that matches the product's privacy design. A shared browser installation can have more than one account over time.

When the user opts out, stop sending to the installation and remove its association. Recheck permission and registration when the user enables notifications again. Keep an unsubscribe or disable action accessible.

## Test web messaging

Test with a real supported browser over HTTPS or a local secure development origin. Verify permission, service worker scope, active registration, FID registration, foreground delivery, background delivery, notification click behavior, and opt-out behavior.

Use a development Firebase project and a test installation. Do not send messages to real users while validating payloads. Check browser developer tools and service worker status when a message is not received.

## Hands-on exercise: opt-in updates

1. Configure the public Web Push key and register the messaging service worker.
2. Add an enable notifications button with a short explanation.
3. Request permission only after the user presses the button.
4. Register the app instance and receive its FID.
5. Send the FID to an authenticated server operation and associate it with the current user.
6. Display a foreground message inside the app.
7. Send a data-only test message and display it from the service worker.
8. Open a same-origin path when the user clicks the notification.
9. Sign out, disable notifications, and remove the user's FID association.
10. Confirm that no server secret or private message content is present in the browser bundle.

## Common mistakes

- Requesting permission before the user understands why notifications are useful.
- Assuming every browser supports the Messaging SDK.
- Serving the app or worker over an insecure origin.
- Forgetting the service worker or placing it under the wrong scope.
- Storing an FID without verifying which signed-in user submitted it.
- Sending messages directly from a browser with server credentials.
- Using message data as authorization to access a record.
- Displaying duplicate background notifications.
- Forgetting to remove stale installation records or honor opt-out.
- Putting private content in notification text.

## Practice questions

1. What does FCM use in a web app to receive background messages?
2. Why must a web push app use HTTPS?
3. When should an app ask for notification permission?
4. What does an FID identify?
5. Why must the server associate an FID with an authenticated user?
6. How does onMessage differ from a background service worker handler?
7. Why should a notification click destination be checked?
8. What should happen when a user opts out or signs out?

## Main references

- [Get started with FCM in web apps](https://firebase.google.com/docs/cloud-messaging/web/get-started)
- [Receive messages in web apps](https://firebase.google.com/docs/cloud-messaging/web/receive-messages)
- [Send a message with the Firebase Admin SDK](https://firebase.google.com/docs/cloud-messaging/send/admin-sdk)
- [Manage FCM registration tokens and installation IDs](https://firebase.google.com/docs/cloud-messaging/manage-tokens)
- [Service workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
