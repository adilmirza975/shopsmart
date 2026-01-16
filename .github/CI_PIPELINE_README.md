# CI/CD Pipeline Documentation

This document describes the automated CI/CD pipeline for the ShopSmart application.

## Pipeline Overview

The CI/CD pipeline consists of three main workflows:

### 1. **CI Pipeline** (`ci.yml`)

Runs on every push and pull request to `main` and `develop` branches.

**Jobs:**

- **Backend Tests & Lint**: Tests backend with Node.js 18.x and 20.x
  - Installs dependencies
  - Runs linter
  - Executes tests with coverage
  - Uploads coverage to Codecov
- **Frontend Tests & Lint**: Tests frontend with Node.js 18.x and 20.x

  - Installs dependencies
  - Runs ESLint
  - Executes tests with coverage
  - Builds the application
  - Uploads coverage to Codecov

- **Build & Package**: Runs after tests pass

  - Builds both frontend and backend artifacts
  - Verifies build outputs

- **Security Audit**: Checks dependencies for vulnerabilities

  - Audits backend dependencies
  - Audits frontend dependencies
  - Uses moderate audit level (non-blocking)

- **Deployment Ready**: Notifies when code is ready for production
  - Only runs on successful builds to `main` branch

### 2. **Deployment Pipeline** (`deploy.yml`)

Handles deployment to production platforms.

**Jobs:**

- **Deploy Backend to Render**: Deploys backend to Render platform
  - Requires `RENDER_API_KEY` and `RENDER_BACKEND_SERVICE_ID` secrets
- **Deploy Frontend to Vercel**: Deploys frontend to Vercel

  - Requires `VERCEL_TOKEN`, `VERCEL_PROJECT_ID`, and `VERCEL_ORG_ID` secrets

- **Notify Deployment Status**: Reports deployment results

### 3. **Code Quality** (`quality.yml`)

Runs continuous quality checks.

**Jobs:**

- **Code Quality Analysis**: Checks code standards
  - Searches for debug statements
  - Analyzes file sizes
- **Dependency Analysis**: Reviews project dependencies

  - Lists backend dependencies
  - Lists frontend dependencies

- **Test Report Summary**: Provides overall status summary

## Setup Instructions

### Prerequisites

- GitHub account with this repository
- Node.js 18.x or 20.x
- npm with package-lock.json files

### Step 1: Configure Secrets

Go to **Settings → Secrets and variables → Actions** in your GitHub repository and add:

#### For Render Deployment:

```
RENDER_API_KEY - Your Render API key
RENDER_BACKEND_SERVICE_ID - Your Render backend service ID
```

#### For Vercel Deployment:

```
VERCEL_TOKEN - Your Vercel authentication token
VERCEL_PROJECT_ID - Your Vercel project ID
VERCEL_ORG_ID - Your Vercel organization ID
```

### Step 2: Configure npm Scripts

Ensure your `package.json` files have the required scripts:

**Backend (`server/package.json`):**

```json
{
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "jest",
    "lint": "eslint . --ext js"
  }
}
```

**Frontend (`client/package.json`):**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint . --ext js,jsx",
    "test": "vitest"
  }
}
```

### Step 3: Add Test Configuration Files

**Backend - Jest Configuration (`server/jest.config.js`):**

```javascript
module.exports = {
  testEnvironment: "node",
  collectCoverageFrom: ["src/**/*.js"],
  coveragePathIgnorePatterns: ["/node_modules/"],
};
```

**Frontend - Vitest Configuration (`client/vitest.config.js`):**

```javascript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: "jsdom",
    coverage: {
      provider: "v8",
    },
  },
});
```

### Step 4: Create ESLint Configuration

**Backend (`server/.eslintrc.json`):**

```json
{
  "env": {
    "node": true,
    "es2021": true,
    "jest": true
  },
  "extends": "eslint:recommended",
  "parserOptions": {
    "ecmaVersion": "latest"
  },
  "rules": {
    "no-unused-vars": "warn"
  }
}
```

**Frontend (`client/.eslintrc.cjs`):**

```javascript
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
  ],
  ignorePatterns: ["dist", ".eslintrc.cjs"],
  parserOptions: { ecmaVersion: "latest", sourceType: "module" },
  settings: { react: { version: "18.2" } },
  rules: {
    "react-refresh/only-export-components": "warn",
  },
};
```

## Workflow Triggers

| Workflow        | Trigger                | Branches      |
| --------------- | ---------------------- | ------------- |
| CI Pipeline     | `push`, `pull_request` | main, develop |
| Deploy Pipeline | `push` to main         | main          |
| Code Quality    | `push`, `pull_request` | main, develop |

## Environment Variables

### Backend (Server)

The following environment variables should be set in your Render deployment:

- `PORT=10000` - Server port
- `NODE_ENV=production` - Environment mode
- `DATABASE_URL` - Database connection string (if using database)

### Frontend (Client)

The following environment variables should be set in your Vercel deployment:

- `VITE_API_URL` - Backend API URL (set via Render environment)

## Monitoring & Troubleshooting

### View Workflow Runs

1. Go to your repository
2. Click **Actions** tab
3. Select the workflow you want to monitor
4. Click on a specific run to see detailed logs

### Common Issues

**Build Fails Due to Missing Dependencies**

- Ensure `package-lock.json` exists for both client and server
- Run `npm ci` instead of `npm install` in CI environments

**Tests Fail**

- Check test files are properly configured
- Ensure all test dependencies are installed
- Review test output logs in GitHub Actions

**Linting Errors**

- Fix issues in source code
- Or update ESLint configuration rules
- Run locally: `npm run lint`

**Deployment Fails**

- Verify secrets are correctly configured
- Check Render/Vercel service IDs are correct
- Review deployment service logs on Render/Vercel dashboard

## Best Practices

1. **Always write tests**: Maintain >80% code coverage
2. **Keep dependencies updated**: Regularly review audit reports
3. **Use branches**: Create PR to `develop` for review before merging to `main`
4. **Review logs**: Check workflow runs for warnings and errors
5. **Monitor deployments**: Track application health after deployment

## Customization

### To add SonarCloud integration:

Add this step to `ci.yml`:

```yaml
- name: SonarCloud Scan
  uses: SonarSource/sonarcloud-github-action@master
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    SONARCLOUD_TOKEN: ${{ secrets.SONARCLOUD_TOKEN }}
```

### To add Slack notifications:

Add this step to `deploy.yml`:

```yaml
- name: Notify Slack
  uses: slackapi/slack-github-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
```

## Support

For issues or questions about the CI/CD pipeline, refer to:

- [GitHub Actions Documentation](https://docs.github.com/actions)
- [Render Documentation](https://render.com/docs)
- [Vercel Documentation](https://vercel.com/docs)
