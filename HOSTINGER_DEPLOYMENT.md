# Hostinger deployment setup

This repository currently looks like a Node.js backend project, not a Next.js
project. The current `package.json` points to `app.js`, but it does not define a
`start` or `build` script yet.

Before using `.github/workflows/deploy-nextjs-hostinger.yml`, make sure the
workflow matches the app you are deploying. The current workflow expects:

- `npm ci` to install dependencies from a committed `package-lock.json`
- `npm run build --if-present` to build the app if a build script exists
- `npm start` to run the app through PM2 on Hostinger

## Required GitHub secrets

Add these secrets in GitHub repository settings:

`Settings -> Secrets and variables -> Actions -> New repository secret`

- `HOSTINGER_HOST`
- `HOSTINGER_USERNAME`
- `HOSTINGER_PASSWORD`
- `HOSTINGER_PATH`

Optional:

- `HOSTINGER_PORT` defaults to `22` in the workflow

## Hostinger target path

Set `HOSTINGER_PATH` to the server directory where the app should live, for
example:

```text
/home/your-user/domains/your-domain.com/public_html
```

For a backend app, confirm that this path is appropriate for your Hostinger plan.
Some Hostinger plans only serve static files from `public_html`; Node.js apps
usually require a plan that supports Node.js/PM2 or a VPS.

## Required project setup

Add a real start script before deploying. For example, if `app.js` starts your
server:

```json
{
  "scripts": {
    "start": "node app.js"
  }
}
```

If you keep using `npm ci` in the workflow, commit `package-lock.json` as well.
Without a lockfile, the GitHub Actions install step will fail.

## PM2 app name

The workflow currently uses this PM2 process name:

```text
nextjs-app
```

For this backend project, consider renaming it in the workflow to something like:

```text
exit-pass-backend
```

## Deployment flow

The workflow runs on every push to `main` and can also be started manually from
GitHub Actions using `workflow_dispatch`.

At deployment time it:

1. Checks out the repository.
2. Sets up Node.js 20.
3. Installs dependencies.
4. Runs the optional build script.
5. Copies the repository to Hostinger.
6. Installs production dependencies on the server.
7. Restarts or starts the app with PM2.
