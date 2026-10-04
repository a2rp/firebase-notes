# 14. Analytics and Performance Monitoring

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Firebase Cloud Messaging for web](./13-firebase-cloud-messaging-for-web.md) | [Notes index](../README.md) | [Next: Environments, configuration, and CI/CD](./15-environments-configuration-and-ci-cd.md) |

## Measure useful product behavior

Google Analytics for Firebase collects events that describe app usage. Some events are collected automatically. Add custom events for product questions that cannot be answered from the default events.

Before adding an event, write down the question it should answer. Choose a stable event name and a small set of useful parameters. Avoid collecting data simply because it is available.

Examples of useful events include:

- A user completed account creation.
- A user saved a task.
- A user selected a content category.
- A checkout step failed.

Event names are case-sensitive. Keep names consistent, use a documented naming style, and do not create a new event for every minor interface detail.

## Initialize Analytics only when supported

Analytics support depends on the browser environment. Check support before creating the Analytics instance.

~~~js
import { getAnalytics, isSupported } from "firebase/analytics";
import { app } from "./lib/firebase.js";

export async function createAnalyticsIfSupported() {
  if (!(await isSupported())) {
    return null;
  }

  return getAnalytics(app);
}
~~~

Analytics must be enabled for the Firebase project and web app. Check the project configuration and the browser console if no events appear.

## Log a small custom event

Use logEvent to record a specific user action with a small number of non-sensitive parameters.

~~~js
import { logEvent } from "firebase/analytics";

export function recordTaskSaved(analytics, taskCategory) {
  if (!analytics) {
    return;
  }

  logEvent(analytics, "task_saved", {
    task_category: taskCategory,
    method: "manual",
  });
}
~~~

Use recommended event names and parameters when they fit the behavior. Keep custom parameter values bounded and predictable so they can be analyzed as groups rather than producing thousands of distinct values.

Do not include a person's name, email address, phone number, message body, or other identifying details in event names or parameters. A database ID can also identify a person when it can be joined with other data.

## Respect collection choices

Analytics collection may need to follow the app's consent and privacy design. When the user has not enabled collection, disable it. When the user changes that choice, apply it to the Analytics instance.

~~~js
import { setAnalyticsCollectionEnabled } from "firebase/analytics";

export function applyAnalyticsChoice(analytics, enabled) {
  if (!analytics) {
    return;
  }

  setAnalyticsCollectionEnabled(analytics, enabled);
}
~~~

Choose the initial collection state before sending custom events. Explain the relevant choice in the product interface, store the user's preference according to the privacy design, and honor changes. Requirements depend on the app, location, and data collected, so check the privacy rules that apply to the product.

## Read Analytics reports

Analytics reports summarize collected events and user behavior in the Firebase console. Newly logged data can take time to appear in standard reports. Use the development event view for immediate validation when available, and wait for processing before diagnosing a missing long-term report.

Use reports to answer the question the event was designed for. Compare trends over time, segment by meaningful values, and avoid conclusions from very small or biased samples.

## What Performance Monitoring measures

Firebase Performance Monitoring helps inspect page load behavior and network request performance. The web SDK automatically collects supported page load metrics and HTTP/S request traces. The web Performance Monitoring SDK is currently marked as beta in the official documentation, so review its current support and limitations before relying on it for production decisions.

Initialize it from the shared Firebase app when the browser supports it:

~~~js
import { getPerformance, isSupported } from "firebase/performance";
import { app } from "./lib/firebase.js";

export async function createPerformanceMonitoringIfSupported() {
  if (!(await isSupported())) {
    return null;
  }

  return getPerformance(app);
}
~~~

The browser batches performance data before sending it, so the console does not update instantly. Keep the test page open long enough for events to be transmitted, then review the Performance dashboard.

## Add a custom trace

A custom trace measures the duration of one meaningful operation, such as loading a task list. Use a stable trace name and stop the trace even when the operation fails.

~~~js
import { trace } from "firebase/performance";

export async function loadTasksWithTrace(performance, loadTasks) {
  if (!performance) {
    return loadTasks();
  }

  const taskTrace = trace(performance, "load_task_list");
  taskTrace.start();

  try {
    const tasks = await loadTasks();
    taskTrace.putMetric("task_count", tasks.length);
    taskTrace.putAttribute("result", "success");
    return tasks;
  } catch (error) {
    taskTrace.putAttribute("result", "error");
    throw error;
  } finally {
    taskTrace.stop();
  }
}
~~~

A trace records duration by default. Add only a few attributes and numeric metrics that help explain performance differences. Do not include personally identifying information in trace attributes or metrics.

Instrument operations that users can feel, such as initial page load, a slow data request, or a large image workflow. Avoid starting traces repeatedly at very high frequency.

## Read network traces and Core Web Vitals

Review page load traces, HTTP/S request duration, response size, and error rate together. A slow screen might be caused by JavaScript work, a large image, a slow API, or a poor network rather than the Firebase database alone.

Core Web Vitals provide user-centered measures of loading, interactivity, and visual stability. Use them with real user context and performance dashboards to find the slow pages and devices that need attention.

Do not put user identifiers, full URLs containing private data, or sensitive request details into custom attributes. Keep trace names and attributes stable so reports can be compared across releases.

## Hands-on exercise: measure a task list

1. Enable Analytics and Performance Monitoring for a development web app.
2. Check browser support before creating either service.
3. Add a task_saved event with a bounded task category.
4. Confirm that no email, task text, or user ID is included.
5. Add a load_task_list trace around the data-loading operation.
6. Record the number of returned tasks as a numeric metric.
7. Stop the trace on both success and failure.
8. Disable collection when the user has not enabled analytics.
9. Inspect the development events and Performance dashboard after data has been transmitted.
10. Compare results before and after one measured performance change.

## Common mistakes

- Logging events without knowing what question they answer.
- Changing an event name or parameter meaning between releases.
- Sending personal data in event parameters or trace attributes.
- Enabling collection before applying the app's consent choice.
- Expecting reports to update immediately.
- Assuming Analytics support exists in every browser context.
- Adding traces around every tiny operation and producing noisy measurements.
- Logging a trace without stopping it after an error.
- Treating correlation in a report as proof that one change caused an outcome.
- Depending on a beta web Performance Monitoring SDK without checking current limits.

## Practice questions

1. What question should a custom Analytics event answer?
2. Why should event names and parameter values stay consistent?
3. Which kinds of values should not be sent as event parameters?
4. How can a web app check whether Analytics is supported?
5. What does setAnalyticsCollectionEnabled control?
6. What is the default metric for a custom Performance trace?
7. Why should a trace stop in a finally block?
8. Why can an Analytics report take time to show a new event?

## Main references

- [Get started with Google Analytics for Firebase](https://firebase.google.com/docs/analytics/get-started?platform=web)
- [Log events in web apps](https://firebase.google.com/docs/analytics/web/events)
- [Firebase Analytics JavaScript reference](https://firebase.google.com/docs/reference/js/analytics)
- [Get started with Performance Monitoring for web](https://firebase.google.com/docs/perf-mon/get-started-web)
- [Add custom monitoring for specific app code](https://firebase.google.com/docs/perf-mon/custom-code-traces)
- [Firebase privacy and data collection](https://firebase.google.com/support/privacy)
