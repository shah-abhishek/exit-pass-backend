# Hostinger deployment setup

This repository is a Node.js backend project. The current `package.json` points
to `app.js` and uses `npm start` to run the server.

Before using `.github/workflows/deploy-nextjs-hostinger.yml`, make sure the
workflow matches the app you are deploying. The workflow expects:

- `npm ci` to install dependencies from a committed `package-lock.json`
- `npm run build --if-present` to build the app if a build script exists
- `npm start` to run the app through PM2 on Hostinger
- `HOSTINGER_PASSWORD` to contain the SSH/SFTP password, not your GitHub password

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

The project should include a start script like this:

```json
{
  "scripts": {
    "start": "node app.js"
  }
}
```

Because the workflow uses `npm ci`, commit `package-lock.json` as well. Without
a lockfile, the GitHub Actions install step will fail.

## PM2 app name

The workflow currently uses this PM2 process name:

```text
exit-pass-backend
```

## Deployment flow

The workflow runs on every push to `main` and can also be started manually from
GitHub Actions using `workflow_dispatch`.

At deployment time it:

1. Checks out the repository.
2. Sets up Node.js 22.
3. Installs dependencies.
4. Runs the optional build script.
5. Copies the repository to Hostinger.
6. Installs production dependencies on the server.
7. Restarts or starts the app with PM2.
