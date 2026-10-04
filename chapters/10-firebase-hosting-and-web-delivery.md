# 10. Firebase Hosting and web delivery

[Back to notes index](../README.md)

| Previous | Notes index | Next |
| --- | --- | --- |
| [Previous: Cloud Functions and event-driven work](./09-cloud-functions-and-event-driven-work.md) | [Notes index](../README.md) | [Next: App Check and abuse reduction](./11-app-check-and-abuse-reduction.md) |

## What Firebase Hosting serves

Firebase Hosting serves static web assets such as HTML, CSS, JavaScript, and images over HTTPS through a global content delivery network. It also supports redirects, rewrites, response headers, custom domains, and preview channels.

For a single-page application, the browser app usually serves many routes from one index.html file. Hosting needs a rewrite so a direct request to a client-side route returns that file.

Firebase Hosting and Firebase App Hosting serve different deployment needs. Hosting is a fit for static assets and supported integrations. Framework applications that need server rendering may fit App Hosting or another server platform better.

## Connect a local project

Install or update the Firebase CLI by following the official CLI setup, sign in, and initialize Hosting from the app directory.

~~~bash
firebase login
firebase init hosting
~~~

Choose the Firebase project and the directory produced by the app's build command. For a Vite application, that directory is commonly dist. For a plain HTML site, it might be public.

Build the app before deploying, and confirm the output directory contains the expected index.html and assets.

~~~bash
npm run build
~~~

The Firebase CLI creates firebase.json and .firebaserc. The first file describes deployment behavior; the second maps local aliases to Firebase project IDs. Keep the correct project alias with the app and check it before every deployment.

~~~bash
firebase use
firebase use development
~~~

Use separate aliases for development and production rather than repeatedly editing a project ID inside deployment commands.

## Configure a single-page app

A small Hosting configuration for a static single-page app can look like this:

~~~json
{
  "hosting": {
    "public": "dist",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
~~~

The rewrite sends otherwise unmatched paths to the app entry point. The client router then decides which page to render. Do not add this rewrite to a multi-page site unless every unknown path should load the same app shell.

Use redirects when a URL has permanently moved. Use rewrites when the browser should keep the requested URL while Hosting serves another file or a backend response.

## Add response headers

Response headers can set browser caching and security behavior. A common pattern is to avoid long-lived caching for the app shell while allowing long caching for content-hashed build assets.

~~~json
{
  "hosting": {
    "headers": [
      {
        "source": "/index.html",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "no-cache"
          }
        ]
      },
      {
        "source": "**/*.@(js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "public,max-age=31536000,immutable"
          }
        ]
      }
    ]
  }
}
~~~

Only use the immutable asset rule when the build output gives changed content a new filename. If an asset keeps the same filename after its content changes, a browser may continue using a cached copy. Review security headers and caching against the app's actual asset names and release process.

## Test locally and share a preview

Run the local Hosting emulator to inspect the built site before uploading it.

~~~bash
npm run build
firebase emulators:start --only hosting
~~~

A Hosting preview channel creates a temporary URL for review without replacing the live site.

~~~bash
firebase hosting:channel:deploy review-notes
~~~

Preview channels can expire. Use them to check page routes, mobile layouts, form behavior, console errors, and asset loading with realistic test data. A preview is a deployed site, so do not include credentials or private content in its build.

## Deploy and roll back

Deploy only Hosting when that is the intended change:

~~~bash
firebase deploy --only hosting
~~~

The Hosting release is associated with the selected Firebase project and site. Verify the target before running the command, then open the resulting URL and check the deployed version.

Firebase Hosting keeps release history. If a release introduces a problem, use the Firebase console or the current CLI rollback workflow to restore a prior Hosting release. Confirm the target site and selected release before rolling back.

Connect a custom domain through the Firebase console and follow its DNS verification steps. Wait for the domain and HTTPS certificate setup to complete, then test both the custom domain and Firebase-provisioned domain.

## Hands-on exercise: deploy a single-page app

1. Build the app and confirm that dist/index.html and its assets exist.
2. Initialize Hosting and select a development Firebase project.
3. Set public to the actual build output folder.
4. Add a rewrite only if the app uses client-side routes.
5. Run the Hosting emulator and open a nested route directly.
6. Deploy a preview channel and check it on desktop and mobile.
7. Inspect cache headers for index.html and fingerprinted assets.
8. Deploy to the development site and verify the URL and project.
9. Review the release history and find the rollback control without applying it.
10. Record the production target and the checks required before release.

## Common mistakes

- Setting public to the source directory instead of the build output.
- Deploying before generating the latest production build.
- Forgetting an SPA rewrite, which makes direct nested URLs return not found.
- Adding an SPA rewrite to a multi-page site that should return a real 404 page.
- Applying immutable caching to assets whose filenames do not change with content.
- Using a production project for local experiments.
- Running a deploy command without checking the active Firebase project.
- Assuming a preview deployment contains the same environment configuration as production.
- Treating a successful upload as proof that every route and asset works.

## Practice questions

1. What type of files does Firebase Hosting primarily deliver?
2. Why does a client-side router usually need a Hosting rewrite?
3. When should a redirect be used instead of a rewrite?
4. Why can immutable caching be useful for content-hashed assets?
5. What can a Hosting preview channel be used to review?
6. Which Firebase project receives a Hosting deployment?
7. What should be checked before rolling back a release?
8. When might a server-rendered app need a different Firebase hosting product?

## Main references

- [Get started with Firebase Hosting](https://firebase.google.com/docs/hosting/quickstart)
- [Configure Hosting behavior](https://firebase.google.com/docs/hosting/full-config)
- [Test locally, share changes, and deploy](https://firebase.google.com/docs/hosting/test-preview-deploy)
- [Firebase CLI reference](https://firebase.google.com/docs/cli)
- [Connect a custom domain](https://firebase.google.com/docs/hosting/custom-domain)
