# 15. Environments, configuration, and CI/CD

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Analytics and Performance Monitoring](./14-analytics-and-performance-monitoring.md) | [Notes index](../README.md) | [Next: Billing, operations, debugging, and production checklist](./16-billing-operations-debugging-and-production-checklist.md) |

## Keep environments isolated

Use a separate Firebase project for each release environment. Development, test, staging, and production projects have separate Auth users, databases, Storage buckets, Functions, Hosting sites, and Analytics data.

This separation helps keep test writes away from real users and makes environment permissions clearer. Development data should be fictional. Staging data should be realistic but should not contain actual user records.

| Environment | Purpose | Typical data |
| --- | --- | --- |
| Development | Build features and reproduce bugs | Local or fictional records |
| Test and QA | Run automated and manual checks | Resettable test fixtures |
| Staging | Check a release with production-like configuration | Anonymized or fictional records |
| Production | Serve users | Live user data |

A development build and production build should not share the same Firebase resources just because both are registered as apps in one project. Apps in one Firebase project share backend resources.

## Name projects and CLI aliases clearly

Keep project IDs and human-readable display names easy to distinguish. Tag the production project as production in the Firebase console.

A .firebaserc file can map local aliases to project IDs:

~~~json
{
  "projects": {
    "default": "my-app-dev",
    "staging": "my-app-staging",
    "production": "my-app-prod"
  }
}
~~~

Check the active project before deploys and data migrations:

~~~bash
firebase use
firebase use staging
firebase deploy --project staging --only hosting
~~~

For production, pass the production target explicitly and confirm the selected project in the command output. Avoid relying on a developer's previous CLI selection.

## Configure browser values by environment

A web app can read Firebase web configuration from build environment variables. Vite exposes variables beginning with VITE_ to the browser bundle, so they must contain only client-safe values.

~~~js
const requiredValues = [
  "VITE_FIREBASE_API_KEY",
  "VITE_FIREBASE_AUTH_DOMAIN",
  "VITE_FIREBASE_PROJECT_ID",
  "VITE_FIREBASE_APP_ID",
];

for (const name of requiredValues) {
  if (!import.meta.env[name]) {
    throw new Error("Missing required Firebase setting: " + name);
  }
}

export const firebaseConfig = {
  apiKey: import.meta.env.VITE_FIREBASE_API_KEY,
  authDomain: import.meta.env.VITE_FIREBASE_AUTH_DOMAIN,
  projectId: import.meta.env.VITE_FIREBASE_PROJECT_ID,
  appId: import.meta.env.VITE_FIREBASE_APP_ID,
};
~~~

The Firebase web configuration identifies the app and project. It is expected to be visible in the client. It does not grant authorization by itself. Do not put service account keys, private provider secrets, or Admin SDK credentials in VITE_ variables.

Use separate local environment files or CI environment settings for each build target. Ignore real local files and commit an example file with placeholders only.

~~~text
.env.local
.env.development
.env.production
.env.example
~~~

The exact file precedence depends on the build tool. Confirm the selected mode and output configuration before generating a production build.

## Keep service configuration with the project

Check in Firebase configuration that should be reviewed and deployed with the app, such as Hosting rewrites, Firestore indexes, and database or Storage rules. Keep environment-specific project IDs in aliases or deployment configuration rather than copying production IDs into development files.

A rules deployment can replace the rules currently published in the console. Review the target, rules file, and change set before deploying. Test the rules against the matching emulator first.

Do not copy development data into production as a deployment shortcut. Migrations between environments need their own reviewed and tested process.

## Add continuous integration checks

A pull request workflow should install from the lock file, run the checks, and build the app before any deployment.

~~~yaml
name: verify
on:
  pull_request:
jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint --if-present
      - run: npm test --if-present
      - run: npm run build
~~~

Choose a Node.js version supported by the project and current Firebase tools. Keep that choice in step with local development and deployment.

The checks should not need production credentials. Use the Local Emulator Suite for tests that exercise Auth, rules, databases, Storage, or Functions. A pull request build can catch missing environment values with harmless placeholders or test-specific settings.

## Deploy previews and production from separate steps

Firebase Hosting's GitHub integration can create a preview deployment for pull requests and deploy the production site when a configured branch is updated. Review the generated workflow and the exact Firebase project before enabling it.

Give the deployment identity only the access required for the target environment. Store any required credentials in the CI secret store, never in the repository or pull request output. Do not expose deployment secrets to workflows triggered by untrusted external contributions.

Use a separate approval or protected environment for production deployment when the repository workflow requires one. The production step should run only after checks pass and should target the production Firebase project explicitly.

~~~bash
firebase deploy --project staging --only hosting
firebase deploy --project production --only hosting
~~~

Keep preview deployment, production deployment, and database or rules deployment distinct. A change to Hosting should not unintentionally deploy Functions or overwrite rules.

## A practical release pipeline

A small release pipeline can follow this order:

1. A pull request runs install, lint, tests, emulator checks, and a production build.
2. A preview deployment is created for review.
3. The reviewer checks the preview routes and feature behavior.
4. A release is merged to the approved branch.
5. The production workflow builds from the reviewed commit.
6. The workflow deploys only the intended Firebase products to the production project.
7. The team checks service health, logs, and user-visible behavior after release.

Tag release commits or record the deployed commit so that a Hosting release can be traced back to its source.

## Hands-on exercise: separate dev and production

1. Create development and production Firebase projects.
2. Register the matching web app in each project.
3. Add project aliases to .firebaserc.
4. Configure local Vite settings for the development project.
5. Confirm the production build receives only production client settings.
6. Check that local environment files are ignored and .env.example contains placeholders.
7. Add pull request checks that run without production credentials.
8. Connect the Local Emulator Suite for database and rule tests.
9. Configure a Hosting preview workflow and inspect its target project.
10. Add a production deploy step that runs only after review and targets the production alias.

## Common mistakes

- Using one Firebase project for development and production.
- Assuming two apps in the same project have separate databases or Storage buckets.
- Putting service account credentials in browser environment variables.
- Assuming a VITE_ variable is secret.
- Deploying to whichever Firebase project was selected in a previous terminal session.
- Running production credentials in workflows from untrusted pull requests.
- Deploying database rules without checking whether the local file replaces console rules.
- Testing with real user data in development or staging.
- Building with missing or mismatched project configuration.
- Deploying all Firebase products when only Hosting should change.

## Practice questions

1. Why should development and production use separate Firebase projects?
2. Which resources do apps in the same Firebase project share?
3. What information belongs in a VITE_ variable?
4. Why is a web Firebase configuration not an authorization control?
5. What should a pull request workflow verify before deployment?
6. Why should production credentials be unavailable to untrusted pull requests?
7. Why should Hosting deployment be separate from rules and Functions deployment?
8. What information helps connect a deployed release back to its source commit?

## Main references

- [Overview of Firebase environments](https://firebase.google.com/docs/projects/dev-workflows/overview-environments)
- [General best practices for Firebase projects](https://firebase.google.com/docs/projects/dev-workflows/general-best-practices)
- [Firebase security guidelines](https://firebase.google.com/docs/projects/dev-workflows/general-security-guidelines)
- [Deploy to live and preview channels with GitHub](https://firebase.google.com/docs/hosting/github-integration)
- [Configure Firebase Hosting](https://firebase.google.com/docs/hosting/full-config)
- [Firebase CLI](https://firebase.google.com/docs/cli)
