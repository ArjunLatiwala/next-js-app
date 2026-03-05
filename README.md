# next-js-app — CI/CD Learning Project

This repository is a fork of [harishneel1/next-js-app](https://github.com/harishneel1/next-js-app), used as a hands-on project to learn and implement CI/CD pipelines using GitHub Actions, Docker, and branch protection rules.

---

## Original Repository

| | |
|---|---|
| **Forked from** | [harishneel1/next-js-app](https://github.com/harishneel1/next-js-app) |
| **Framework** | Next.js |
| **Language** | TypeScript |

---

## What I Implemented

### 1. CI Pipeline — GitHub Actions (`integration.yml`)

A continuous integration workflow that automatically runs on every push to `main` and every pull request targeting `main`.

The pipeline runs two parallel jobs across **3 Node.js versions (18.x, 20.x, 22.x)**:

- **build** — installs dependencies and runs `npm run build` to verify the app compiles
- **unit-tests** — installs dependencies and runs `npm run test` to verify all test cases pass

This ensures that broken code can never be merged into `main` without passing all checks first.

```yaml
# .github/workflows/integration.yml
name: Integration
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x, 22.x]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"
      - run: npm i
      - run: npm run build
  unit-tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x, 22.x]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"
      - run: npm i
      - run: npm run test
```

---

### 2. CD Pipeline — Docker Build & Push (`deploy.yml`)

A continuous deployment workflow that triggers on every push to `main`. It builds a Docker image of the Next.js app and pushes it to Docker Hub automatically.

```yaml
# .github/workflows/deploy.yml
name: Deployment
on:
  push:
    branches: ["main"]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_PASSWORD }}
      - run: docker build -t arjunlatiwala/next-js-app:latest .
      - run: docker push arjunlatiwala/next-js-app:latest
```

Docker Hub credentials are stored securely as **GitHub Repository Secrets** (`DOCKERHUB_USERNAME` and `DOCKERHUB_PASSWORD`) and never exposed in the codebase.

---

### 3. Branch Protection Rules

Branch protection rules are configured on `main` to enforce code quality:

- All changes to `main` must go through a **Pull Request** — direct pushes are blocked
- All CI checks (build + unit-tests across all Node versions) must **pass** before a PR can be merged
- This mirrors real-world team workflows where no broken code can enter the main branch

---

## Project Structure

```
next-js-app/
├── .github/
│   └── workflows/
│       ├── integration.yml   ← CI pipeline
│       └── deploy.yml        ← CD pipeline
├── __tests__/                ← Jest test files
├── app/                      ← Next.js app directory
├── Dockerfile                ← Docker configuration
├── jest.config.js
└── package.json
```

---

## How to Run Locally

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Run tests
npm run test

# Build for production
npm run build
```

---

## Docker

```bash
# Pull the latest image
docker pull arjunlatiwala/next-js-app:latest

# Run the container
docker run -p 3000:3000 arjunlatiwala/next-js-app:latest
```

---

## Key Learnings

- How GitHub Actions workflows are triggered by push and pull request events
- How matrix strategy runs jobs across multiple Node.js versions in parallel
- How to securely store and use secrets in GitHub Actions
- How branch protection rules enforce a PR-based workflow
- How to build and push Docker images automatically from a CI/CD pipeline
- The difference between CI (testing/building) and CD (deploying)
- How workflow files must exist on the base branch (`main`) to be triggered by PRs
