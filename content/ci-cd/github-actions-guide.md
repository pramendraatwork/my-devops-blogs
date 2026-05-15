---
title: "CI/CD with GitHub Actions — Complete Guide"
date: 2024-02-05
draft: false
description: "Build real CI/CD pipelines with GitHub Actions. Workflow syntax, jobs, secrets, and a production pipeline."
categories: ["ci-cd"]
tags: ["github-actions", "ci-cd", "pipeline", "yaml"]
showToc: true
---

## What is CI/CD?

- **CI** — auto build and test every code change
- **CD** — auto deploy tested code to an environment

## Workflow anatomy

```yaml
name: My Pipeline
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci && npm test
```

## Multi-job pipeline

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: myrepo/app:latest

  deploy:
    needs: build
    environment: production
    runs-on: ubuntu-latest
    steps:
      - run: echo "deploy here"
```

## Secrets

```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

## Things that got me

- Jobs run in PARALLEL by default — use needs to sequence
- Secrets not available on PRs from forks
- Use fetch-depth: 0 if build needs full git history
