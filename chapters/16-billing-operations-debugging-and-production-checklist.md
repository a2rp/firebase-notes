# 16. Billing, operations, debugging, and production checklist

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Environments, configuration, and CI/CD](./15-environments-configuration-and-ci-cd.md) | [Notes index](../README.md) | [Next: All code samples](./98-all-code-samples.md) |

## Understand Firebase billing

Firebase plans apply to the whole Firebase project and its registered apps. Some products are available without a billing account, while other features require a billing-enabled project or have quotas that depend on the plan.

Check the current pricing page for each product before enabling it. Usage amounts, quotas, and included features can change. Review reads, writes, storage, downloads, function invocations, network egress, and any connected Google Cloud services.

A budget alert is a notification, not a spending cap. If a service supports spend caps, review and configure them separately. A billing-enabled project can incur charges when usage exceeds included quotas.

## Measure usage before launch

Use Firebase and Google Cloud usage dashboards to understand which product creates usage. Set a normal baseline, then watch for unusual changes after a release.

| Area | What to review |
| --- | --- |
| Cloud Firestore | Document reads and writes, storage, index growth |
| Realtime Database | Downloaded data, connections, storage |
| Cloud Storage | Stored bytes, uploads, downloads, egress |
| Cloud Functions | Invocations, execution time, memory, retries |
| Hosting | Storage, transfer, request patterns |
| Analytics and messaging | Collection settings, provider usage, delivery outcomes |

A request pattern that is harmless with a few test records can be costly at larger scale. Limit collection reads, paginate lists, avoid repeated listeners, use appropriately sized images, and review Functions retries.

## Log useful operational context

Use structured logs with operation names and non-sensitive identifiers. Logs should help answer what happened, where it happened, and whether it succeeded.

~~~js
const { logger } = require("firebase-functions");

async function processOrder(orderId, ownerId) {
  logger.info("Order processing started", {
    orderId,
    ownerId,
  });

  try {
    const result = await validateAndProcessOrder(orderId);

    logger.info("Order processing completed", {
      orderId,
      status: result.status,
    });

    return result;
  } catch (error) {
    logger.error("Order processing failed", {
      orderId,
      errorCode: error.code ?? "unknown",
    });

    throw error;
  }
}
~~~

Do not log passwords, access tokens, complete documents, payment details, private messages, or other sensitive values. Restrict access to logs and choose retention according to the operational and privacy requirements of the app.

## Debug a production issue systematically

Start with the user's visible symptom, the affected feature, the release version, and the time window. Compare browser errors, Firebase request errors, Function logs, service health, and usage changes.

A practical sequence is:

1. Confirm which environment and Firebase project is affected.
2. Check whether the issue affects all users or a specific browser, account, or route.
3. Check recent releases, rules deployments, configuration changes, and provider status.
4. Inspect request errors and function logs for a shared cause.
5. Reproduce the issue in a development project or emulator.
6. Apply the smallest safe mitigation and verify it.
7. Record the cause, impact, and follow-up action.

An App Check failure, Authentication failure, permission denial, missing Firestore index, network error, and billing limit can look similar in the interface. Preserve the original error details in restricted logs and report a useful message to the user without exposing private internals.

## Plan backups and recovery

A backup is useful only if the team can restore it. Choose a recovery approach for every product that stores important data, document its retention and access controls, and test recovery with a separate environment.

For Cloud Firestore, review the current backup, point-in-time recovery, and export options for the selected edition and database configuration. For other services, check their current product-specific recovery guidance. Do not assume that a source-control copy of rules or application code contains user data.

A recovery plan should record:

- Which data and configuration are backed up.
- How frequently backups or exports are created.
- Who can read and restore them.
- Where they are stored and how long they are retained.
- How to restore into an isolated project.
- How the team verifies restored records and application behavior.

Do a restore exercise before an incident. A successful backup job alone does not prove that the data can be recovered.

## Use least privilege and manage credentials

Give each developer and service identity only the access it needs. Use separate identities for local development, CI, and production where practical. Review project IAM access when team membership or responsibilities change.

If a provider key or service credential is exposed, revoke or rotate it, inspect recent usage, and check whether another credential or secret was also exposed. Do not rely on deleting a secret from the latest commit because it may remain in repository history and build logs.

Keep public web configuration separate from server secrets. Web configuration belongs in the client build; service account credentials, private API keys, and privileged tokens do not.

## Handle releases and rollback

Before a release, record the commit, target project, Firebase products being deployed, and operator or workflow identity. Deploy only the intended resources.

After release, check the Hosting URL, authentication, key user journeys, database access, file upload, Function logs, error rates, and billing or usage dashboards. Keep a known-good Hosting release and a data recovery plan available.

Rolling back code does not automatically reverse database writes, migrations, or external side effects. Plan data changes so the old application version can safely run during rollback, or document a separate recovery action.

## Production readiness checklist

Before launch, verify each of the following:

- Development, staging, and production use isolated Firebase projects.
- Production is clearly identified in the Firebase console and CLI aliases.
- Authentication providers and account recovery work as expected.
- Firestore, Realtime Database, and Storage rules deny unapproved access.
- Rules and queries have been checked together with emulator tests.
- Functions validate requests, verify authorization, and handle retries.
- App Check metrics were reviewed before enforcement.
- Analytics collection follows the app's privacy design.
- User-facing pages handle network failure, empty data, and denied access.
- Indexes and deployment configuration are checked into source control.
- Backups and data recovery steps have been tested.
- Budget alerts, usage dashboards, and supported spend controls are configured.
- CI builds and tests without production secrets in pull request workflows.
- The production deployment target and rollback plan are documented.
- Logs do not contain passwords, tokens, or private user content.

## Hands-on exercise: prepare an operations page

1. Identify every Firebase product used by the app.
2. Record the Firebase project ID and billing plan for each environment.
3. Find the usage dashboard or quota page for each product.
4. Add a structured log to one Function without logging private fields.
5. Set a budget alert and identify any applicable spend cap separately.
6. Write down the backup or export method for every important data store.
7. Restore a backup into an isolated project and verify sample records.
8. Record the last known good Hosting release and its source commit.
9. Simulate a denied database request and follow the debugging sequence.
10. Complete the production checklist with the team before launch.

## Common mistakes

- Assuming a budget alert prevents charges.
- Treating plan quotas or prices as permanent values.
- Using unbounded reads or listeners without measuring their cost.
- Logging entire documents or private values to speed up debugging.
- Believing that a backup is valid without testing a restore.
- Assuming code rollback also undoes data migrations.
- Leaving broad IAM access after a temporary task.
- Relying on a public client configuration value as a secret.
- Deploying to the wrong project or deploying more products than intended.
- Treating the emulator as a production backup or load test.

## Practice questions

1. Does a Firebase plan apply separately to each registered app?
2. What is the difference between a budget alert and a spend cap?
3. Which usage measures should be reviewed for Firestore, Storage, and Functions?
4. What information makes a structured log useful?
5. Which values should never be written to operational logs?
6. Why should a restore be tested in an isolated project?
7. Does rolling back Hosting reverse database writes?
8. What should be checked before and after a production release?

## Main references

- [Firebase pricing plans](https://firebase.google.com/docs/projects/billing/firebase-pricing-plans)
- [Avoid surprise bills](https://firebase.google.com/docs/projects/billing/avoid-surprise-bills)
- [Firebase launch checklist](https://firebase.google.com/support/guides/launch-checklist)
- [Back up and restore Cloud Firestore data](https://firebase.google.com/docs/firestore/backups)
- [Firebase project security guidelines](https://firebase.google.com/docs/projects/dev-workflows/general-security-guidelines)
- [Firebase Hosting release management](https://firebase.google.com/docs/hosting/manage-hosting-resources)
