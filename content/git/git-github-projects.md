---
title: "Git & GitHub Projects — Beginner to Advanced (3 Real Projects)"
date: 2024-03-21
draft: false
description: "3 hands-on Git and GitHub projects: beginner personal portfolio repo, intermediate team collaboration workflow, and advanced GitOps automation with GitHub Actions."
categories: ["git"]
tags: ["git", "github", "version-control", "projects", "devops", "automation"]
showToc: true
TocOpen: true
---

## Why These Projects? 🎯

Git is used in every single DevOps job. These 3 projects teach you real Git workflows — not just commands, but how teams actually work with Git in production.

```
PROJECT ROADMAP:
──────────────────────────────────────────────────────────────
Project 1 (Beginner)     → Personal Portfolio with Git Workflow
Project 2 (Intermediate) → Team Collaboration & Branch Strategy
Project 3 (Advanced)     → Automated GitOps Release Pipeline
──────────────────────────────────────────────────────────────
```

---

## Project 1: Personal Portfolio with Git 🟢 Beginner

### What You'll Build

A personal portfolio website managed with Git — with proper commits, branches, tags, and pushed to GitHub Pages for free hosting.

```
WHAT YOU'LL PRACTICE:
──────────────────────────────────────────────────────────────
✅ Initialize repo and first commit
✅ Branching for features
✅ Writing good commit messages
✅ Tagging releases (v1.0, v2.0)
✅ Pushing to GitHub
✅ GitHub Pages deployment
✅ .gitignore setup
──────────────────────────────────────────────────────────────
```

### Skills You'll Learn

- Git init, add, commit, push
- Branching and merging
- Tagging versions
- GitHub Pages hosting
- Writing proper commit messages

### Step 1: Initialize the Project

```bash
# Create project folder
mkdir my-portfolio
cd my-portfolio

# Initialize Git
git init
git checkout -b main

# Configure Git (if not done)
git config user.name "Pramendra Rajput"
git config user.email "pramendraatwork@gmail.com"

# Check status
git status
```

### Step 2: Create the Portfolio

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pramendra Rajput | DevOps Engineer</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <header>
        <nav>
            <h1>Pramendra Rajput</h1>
            <ul>
                <li><a href="#about">About</a></li>
                <li><a href="#skills">Skills</a></li>
                <li><a href="#projects">Projects</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>

    <section id="about">
        <h2>👋 Hey, I'm Pramendra</h2>
        <p>DevOps Engineer passionate about automation, cloud, and building reliable systems.</p>
    </section>

    <section id="skills">
        <h2>🛠️ Skills</h2>
        <div class="skills-grid">
            <span class="skill">Linux</span>
            <span class="skill">Docker</span>
            <span class="skill">Kubernetes</span>
            <span class="skill">AWS</span>
            <span class="skill">Terraform</span>
            <span class="skill">Jenkins</span>
            <span class="skill">Git</span>
            <span class="skill">CI/CD</span>
        </div>
    </section>

    <section id="projects">
        <h2>🚀 Projects</h2>
        <div class="project-card">
            <h3>DevOps Blog</h3>
            <p>Personal blog covering DevOps topics built with Hugo and GitHub Pages.</p>
            <a href="https://pramendraatwork.github.io/my-devops-blogs">View Project</a>
        </div>
    </section>

    <section id="contact">
        <h2>📬 Contact</h2>
        <p>Email: <a href="mailto:pramendraatwork@gmail.com">pramendraatwork@gmail.com</a></p>
        <p>GitHub: <a href="https://github.com/pramendraatwork">github.com/pramendraatwork</a></p>
        <p>LinkedIn: <a href="https://linkedin.com/in/pramendra-rajpoot-0518803b0">LinkedIn Profile</a></p>
    </section>
</body>
</html>
```

```css
/* style.css */
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
    font-family: 'Segoe UI', sans-serif;
    background: #0d1117;
    color: #c9d1d9;
    line-height: 1.6;
}

header {
    background: #161b22;
    padding: 1rem 2rem;
    border-bottom: 1px solid #30363d;
}

nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

nav h1 { color: #58a6ff; }

nav ul {
    display: flex;
    gap: 2rem;
    list-style: none;
}

nav a { color: #c9d1d9; text-decoration: none; }
nav a:hover { color: #58a6ff; }

section {
    max-width: 900px;
    margin: 4rem auto;
    padding: 0 2rem;
}

h2 {
    color: #58a6ff;
    margin-bottom: 1.5rem;
    font-size: 1.8rem;
}

.skills-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
}

.skill {
    background: #21262d;
    border: 1px solid #30363d;
    padding: 0.5rem 1rem;
    border-radius: 6px;
    color: #58a6ff;
}

.project-card {
    background: #161b22;
    border: 1px solid #30363d;
    border-radius: 8px;
    padding: 1.5rem;
    margin-bottom: 1rem;
}

.project-card h3 { color: #58a6ff; margin-bottom: 0.5rem; }
.project-card a { color: #3fb950; }
```

### Step 3: .gitignore

```bash
cat > .gitignore << 'EOF'
# OS files
.DS_Store
Thumbs.db

# Editor files
.vscode/
.idea/
*.swp

# Build output
dist/
node_modules/

# Environment
.env
EOF
```

### Step 4: First Commit

```bash
# See what's untracked
git status

# Stage all files
git add .

# First commit — use conventional format!
git commit -m "feat: initial portfolio setup with HTML and CSS"

# View commit log
git log --oneline
```

### Step 5: Feature Branches

```bash
# Create branch for a new feature
git switch -c feature/add-projects-section

# Make changes — add more projects to index.html
# ... edit index.html ...

git add index.html
git commit -m "feat: add DevOps projects section"

# Merge back to main
git switch main
git merge feature/add-projects-section

# Delete feature branch (clean up)
git branch -d feature/add-projects-section

# View history
git log --oneline --graph
```

### Step 6: Tag a Release

```bash
# Tag version 1.0
git tag -a v1.0.0 -m "Release v1.0.0 - Initial portfolio"

# View tags
git tag

# See tag details
git show v1.0.0
```

### Step 7: Push to GitHub & Enable Pages

```bash
# Create repo on GitHub: github.com/new
# Name: my-portfolio (Public)

# Connect and push
git remote add origin https://github.com/pramendraatwork/my-portfolio.git
git push -u origin main
git push origin --tags    # push tags too

# Enable GitHub Pages:
# Settings → Pages → Source: Deploy from branch → main → / (root)
# URL: https://pramendraatwork.github.io/my-portfolio
```

### Step 8: Practice These Commands Daily

```bash
# View full history with graph
git log --oneline --graph --all

# See what changed in a commit
git show abc1234

# Compare branches
git diff main..feature/new-feature

# Check who changed what
git blame index.html

# Find when a bug was introduced
git bisect start
git bisect bad HEAD
git bisect good v1.0.0

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Save work temporarily
git stash
git stash pop
```

### What You Learned

- ✅ Git init, add, commit, push workflow
- ✅ Feature branches
- ✅ Release tagging
- ✅ GitHub Pages hosting
- ✅ Writing proper commit messages
- ✅ .gitignore setup

---

## Project 2: Team Collaboration Workflow 🟡 Intermediate

### What You'll Build

Simulate a real team Git workflow — multiple developers, feature branches, Pull Requests, code reviews, conflict resolution, and protected main branch. This is exactly how companies work.

```
TEAM WORKFLOW:
──────────────────────────────────────────────────────────────
main          ●────────────────────────────●──────▶  (protected)
               \                          /
develop         ●──●──●──●──●──●──●──●──●
                    \         \
feature/login        ●──●──●   \
                                ●──●──● feature/dashboard
──────────────────────────────────────────────────────────────
```

### Project: Collaborative Todo App

#### Repository Setup

```bash
# Create the project
mkdir team-todo-app
cd team-todo-app
git init
git checkout -b main

# Create initial app
cat > README.md << 'EOF'
# Team Todo App
A collaborative todo application.

## Branch Strategy
- main: production-ready code (protected)
- develop: integration branch
- feature/*: new features
- hotfix/*: emergency fixes
- release/*: release preparation

## Commit Convention
feat:     new feature
fix:      bug fix
docs:     documentation
refactor: code change
test:     adding tests
EOF

cat > app.js << 'EOF'
// Simple Todo App
const todos = [];

function addTodo(task) {
    const todo = {
        id: Date.now(),
        task,
        done: false,
        createdAt: new Date().toISOString()
    };
    todos.push(todo);
    console.log(`Added: ${task}`);
    return todo;
}

function completeTodo(id) {
    const todo = todos.find(t => t.id === id);
    if (todo) {
        todo.done = true;
        console.log(`Completed: ${todo.task}`);
    }
}

function listTodos() {
    return todos.filter(t => !t.done);
}

module.exports = { addTodo, completeTodo, listTodos };
EOF

git add .
git commit -m "feat: initial todo app setup"

# Create develop branch
git checkout -b develop
git push -u origin main
git push -u origin develop
```

#### Developer 1: Feature/Login

```bash
# Developer 1 starts working on login feature
git checkout develop
git pull origin develop
git checkout -b feature/user-login

# Create login module
cat > login.js << 'EOF'
// User Login Module
const users = [
    { id: 1, username: 'pramendra', password: 'pass123', role: 'admin' },
    { id: 2, username: 'john', password: 'pass456', role: 'user' }
];

function login(username, password) {
    const user = users.find(
        u => u.username === username && u.password === password
    );

    if (user) {
        console.log(`✅ Login successful: ${username} (${user.role})`);
        return { success: true, user };
    }

    console.log(`❌ Login failed for: ${username}`);
    return { success: false };
}

function logout(username) {
    console.log(`👋 ${username} logged out`);
    return { success: true };
}

module.exports = { login, logout };
EOF

git add login.js
git commit -m "feat(auth): add user login and logout module"

# Add tests
cat > login.test.js << 'EOF'
const { login, logout } = require('./login');

// Test login
const result1 = login('pramendra', 'pass123');
console.assert(result1.success === true, 'Login should succeed');

const result2 = login('pramendra', 'wrongpassword');
console.assert(result2.success === false, 'Wrong password should fail');

console.log('✅ All login tests passed!');
EOF

git add login.test.js
git commit -m "test(auth): add login unit tests"

# Push feature branch
git push -u origin feature/user-login
```

#### Opening a Pull Request

```bash
# After pushing, go to GitHub and open PR:
# base: develop ← compare: feature/user-login
# Title: feat(auth): Add user authentication
# Description:
# ## What this PR does
# - Adds login/logout functionality
# - Includes unit tests
# - Validates username and password
#
# ## How to test
# node login.test.js
#
# ## Checklist
# - [x] Tests added
# - [x] No sensitive data committed
# - [x] Follows commit convention
```

#### Developer 2: Feature/Dashboard (Parallel Work)

```bash
# Developer 2 works simultaneously
git checkout develop
git pull origin develop
git checkout -b feature/dashboard

cat > dashboard.js << 'EOF'
// Dashboard Module
const { listTodos } = require('./app');

function showDashboard(username) {
    const todos = listTodos();
    console.log(`\n📊 Dashboard for: ${username}`);
    console.log('═══════════════════════════');
    console.log(`Total pending: ${todos.length}`);

    if (todos.length === 0) {
        console.log('🎉 All done! No pending tasks.');
    } else {
        todos.forEach((todo, i) => {
            console.log(`${i+1}. ${todo.task}`);
        });
    }
    console.log('═══════════════════════════\n');
}

function getStats() {
    return {
        total: listTodos().length,
        timestamp: new Date().toISOString()
    };
}

module.exports = { showDashboard, getStats };
EOF

git add dashboard.js
git commit -m "feat(dashboard): add user dashboard with stats"
git push -u origin feature/dashboard
```

#### Handling Merge Conflicts

```bash
# Both developers modified README.md — conflict!

# After PR merge of feature/user-login, Developer 2 needs to rebase
git checkout feature/dashboard
git fetch origin
git rebase origin/develop

# Git shows conflict in README.md
# Open the file and see:
# <<<<<<< HEAD (your changes)
# ## Features
# - Dashboard view
# =======
# ## Features
# - User authentication
# >>>>>>> origin/develop (their changes)

# Fix the conflict — keep BOTH features:
cat > README.md << 'EOF'
# Team Todo App

## Features
- User authentication (login/logout)
- Dashboard view with stats
- Todo management
EOF

# Stage the resolved file
git add README.md

# Continue rebase
git rebase --continue

# Push (force needed after rebase)
git push --force-with-lease origin feature/dashboard
```

#### Release Process

```bash
# After all features merged to develop
# Create release branch
git checkout develop
git pull origin develop
git checkout -b release/v1.0.0

# Update version
cat > package.json << 'EOF'
{
  "name": "team-todo-app",
  "version": "1.0.0",
  "description": "Collaborative todo application",
  "main": "app.js"
}
EOF

git add package.json
git commit -m "chore: bump version to 1.0.0"

# Merge to main
git checkout main
git merge --no-ff release/v1.0.0 -m "release: v1.0.0"

# Tag the release
git tag -a v1.0.0 -m "Release v1.0.0
- User authentication
- Dashboard view
- Todo management"

# Merge back to develop
git checkout develop
git merge --no-ff release/v1.0.0 -m "chore: merge release v1.0.0 back to develop"

# Push everything
git push origin main develop
git push origin --tags

# Delete release branch
git branch -d release/v1.0.0
git push origin --delete release/v1.0.0
```

#### Hotfix — Emergency Bug Fix

```bash
# Critical bug found in production!
git checkout main
git checkout -b hotfix/fix-login-crash

# Fix the bug
sed -i 's/u.password === password/u.password === String(password)/' login.js

git add login.js
git commit -m "fix(auth): handle non-string password input"

# Merge to main
git checkout main
git merge --no-ff hotfix/fix-login-crash -m "fix: login crash with non-string password"
git tag -a v1.0.1 -m "Hotfix v1.0.1 - Fix login crash"

# Also merge to develop!
git checkout develop
git merge --no-ff hotfix/fix-login-crash
git push origin main develop
git push origin --tags

# Clean up
git branch -d hotfix/fix-login-crash
```

### GitHub Branch Protection Setup

```
In GitHub: Settings → Branches → Add rule

Branch name pattern: main

Rules to enable:
✅ Require a pull request before merging
✅ Require approvals: 1
✅ Require status checks to pass before merging
✅ Require branches to be up to date before merging
✅ Do not allow bypassing the above settings
```

### What You Learned

- ✅ GitFlow branching strategy
- ✅ Pull Request workflow
- ✅ Code review process
- ✅ Merge conflict resolution
- ✅ Release tagging
- ✅ Hotfix process
- ✅ Branch protection rules

---

## Project 3: Automated GitOps Release Pipeline 🔴 Advanced

### What You'll Build

A fully automated release pipeline using Git — auto versioning, changelog generation, GitHub releases, and deployment triggered purely by Git tags. Pure GitOps.

```
GITOPS FLOW:
──────────────────────────────────────────────────────────────
Developer merges PR to main
         │
         ▼
GitHub Actions: run tests
         │
         ▼
Auto-bump version (v1.0.0 → v1.1.0)
         │
         ▼
Auto-generate CHANGELOG.md from commits
         │
         ▼
Create GitHub Release with release notes
         │
         ▼
Build & push Docker image with version tag
         │
         ▼
Deploy to production automatically ✅
──────────────────────────────────────────────────────────────
```

### Project Structure

```
gitops-pipeline/
├── .github/
│   └── workflows/
│       ├── ci.yml           ← runs on every PR
│       ├── release.yml      ← runs on merge to main
│       └── deploy.yml       ← runs on new tag
├── scripts/
│   ├── bump-version.sh      ← auto version bumping
│   ├── generate-changelog.sh ← auto changelog
│   └── validate-commit.sh   ← commit message linter
├── src/
│   └── app.js
├── CHANGELOG.md
├── VERSION
└── package.json
```

### Version Bump Script

```bash
#!/bin/bash
# scripts/bump-version.sh
# Auto bumps version based on commit messages
# feat: → minor bump (1.0.0 → 1.1.0)
# fix:  → patch bump (1.0.0 → 1.0.1)
# BREAKING CHANGE: → major bump (1.0.0 → 2.0.0)

set -e

CURRENT_VERSION=$(cat VERSION)
MAJOR=$(echo $CURRENT_VERSION | cut -d. -f1)
MINOR=$(echo $CURRENT_VERSION | cut -d. -f2)
PATCH=$(echo $CURRENT_VERSION | cut -d. -f3)

# Get commits since last tag
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
if [ -z "$LAST_TAG" ]; then
    COMMITS=$(git log --oneline)
else
    COMMITS=$(git log ${LAST_TAG}..HEAD --oneline)
fi

echo "Commits since $LAST_TAG:"
echo "$COMMITS"

# Determine bump type
if echo "$COMMITS" | grep -q "BREAKING CHANGE\|!:"; then
    BUMP="major"
    NEW_VERSION="$((MAJOR + 1)).0.0"
elif echo "$COMMITS" | grep -qE "^[a-f0-9]+ feat"; then
    BUMP="minor"
    NEW_VERSION="${MAJOR}.$((MINOR + 1)).0"
elif echo "$COMMITS" | grep -qE "^[a-f0-9]+ fix"; then
    BUMP="patch"
    NEW_VERSION="${MAJOR}.${MINOR}.$((PATCH + 1))"
else
    BUMP="none"
    NEW_VERSION=$CURRENT_VERSION
    echo "No version bump needed"
    exit 0
fi

echo "Bump type: $BUMP"
echo "Version: $CURRENT_VERSION → $NEW_VERSION"

# Update VERSION file
echo "$NEW_VERSION" > VERSION

# Update package.json
if [ -f "package.json" ]; then
    sed -i "s/\"version\": \".*\"/\"version\": \"$NEW_VERSION\"/" package.json
fi

echo "NEW_VERSION=$NEW_VERSION" >> $GITHUB_ENV 2>/dev/null || true
echo "::set-output name=version::$NEW_VERSION" 2>/dev/null || true
echo "$NEW_VERSION"
```

### Changelog Generator

```bash
#!/bin/bash
# scripts/generate-changelog.sh
# Generates CHANGELOG.md from git commits

set -e

VERSION=${1:-$(cat VERSION)}
DATE=$(date +%Y-%m-%d)
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")

echo "Generating changelog for v$VERSION..."

# Get commits since last tag
if [ -z "$LAST_TAG" ]; then
    COMMITS=$(git log --oneline --pretty=format:"%s|%h|%an")
else
    COMMITS=$(git log ${LAST_TAG}..HEAD --pretty=format:"%s|%h|%an")
fi

# Categorize commits
FEATURES=""
FIXES=""
DOCS=""
OTHERS=""

while IFS='|' read -r msg hash author; do
    link="[\`$hash\`](https://github.com/$GITHUB_REPOSITORY/commit/$hash)"
    entry="- $msg $link by **$author**"

    if echo "$msg" | grep -qE "^feat"; then
        FEATURES="${FEATURES}\n${entry}"
    elif echo "$msg" | grep -qE "^fix"; then
        FIXES="${FIXES}\n${entry}"
    elif echo "$msg" | grep -qE "^docs"; then
        DOCS="${DOCS}\n${entry}"
    else
        OTHERS="${OTHERS}\n${entry}"
    fi
done <<< "$COMMITS"

# Build changelog entry
ENTRY="## [v${VERSION}] - ${DATE}\n"

if [ -n "$FEATURES" ]; then
    ENTRY="${ENTRY}\n### ✨ Features\n${FEATURES}\n"
fi

if [ -n "$FIXES" ]; then
    ENTRY="${ENTRY}\n### 🐛 Bug Fixes\n${FIXES}\n"
fi

if [ -n "$DOCS" ]; then
    ENTRY="${ENTRY}\n### 📚 Documentation\n${DOCS}\n"
fi

if [ -n "$OTHERS" ]; then
    ENTRY="${ENTRY}\n### 🔧 Other Changes\n${OTHERS}\n"
fi

# Prepend to CHANGELOG.md
if [ -f CHANGELOG.md ]; then
    echo -e "# Changelog\n\n${ENTRY}\n$(tail -n +2 CHANGELOG.md)" > CHANGELOG.md
else
    echo -e "# Changelog\n\n${ENTRY}" > CHANGELOG.md
fi

echo "✅ Changelog updated!"
cat CHANGELOG.md | head -30
```

### Commit Message Validator

```bash
#!/bin/bash
# scripts/validate-commit.sh
# Validates commit message follows conventional commits

COMMIT_MSG="$1"

# Pattern: type(scope): description
PATTERN="^(feat|fix|docs|style|refactor|test|chore|ci|perf|revert)(\(.+\))?: .{1,100}$"

if echo "$COMMIT_MSG" | grep -qE "$PATTERN"; then
    echo "✅ Valid commit message: $COMMIT_MSG"
    exit 0
else
    echo "❌ Invalid commit message: $COMMIT_MSG"
    echo ""
    echo "Expected format: type(scope): description"
    echo ""
    echo "Types: feat, fix, docs, style, refactor, test, chore, ci, perf, revert"
    echo ""
    echo "Examples:"
    echo "  feat(auth): add JWT authentication"
    echo "  fix(api): handle null response"
    echo "  docs: update README"
    echo "  chore: upgrade dependencies"
    exit 1
fi
```

### CI Workflow (Every PR)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [develop]

jobs:
  validate:
    name: Validate Commits
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Validate commit messages
        run: |
          # Validate last commit message
          COMMIT_MSG=$(git log -1 --pretty=%s)
          bash scripts/validate-commit.sh "$COMMIT_MSG"

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm test

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
```

### Release Workflow (Merge to Main)

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  packages: write

jobs:
  release:
    name: Auto Release
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Configure Git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Bump version
        id: version
        run: |
          chmod +x scripts/bump-version.sh
          NEW_VERSION=$(bash scripts/bump-version.sh)
          echo "version=$NEW_VERSION" >> $GITHUB_OUTPUT

      - name: Generate changelog
        if: steps.version.outputs.version != ''
        run: |
          chmod +x scripts/generate-changelog.sh
          bash scripts/generate-changelog.sh ${{ steps.version.outputs.version }}

      - name: Commit version bump
        if: steps.version.outputs.version != ''
        run: |
          git add VERSION CHANGELOG.md package.json
          git commit -m "chore(release): v${{ steps.version.outputs.version }} [skip ci]"
          git tag -a "v${{ steps.version.outputs.version }}" \
            -m "Release v${{ steps.version.outputs.version }}"
          git push origin main --tags

      - name: Create GitHub Release
        if: steps.version.outputs.version != ''
        uses: softprops/action-gh-release@v1
        with:
          tag_name: v${{ steps.version.outputs.version }}
          name: Release v${{ steps.version.outputs.version }}
          body_path: CHANGELOG.md
          draft: false
          prerelease: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        if: steps.version.outputs.version != ''
        run: |
          echo ${{ secrets.DOCKERHUB_TOKEN }} | \
            docker login -u ${{ secrets.DOCKERHUB_USERNAME }} --password-stdin

          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/my-app:${{ steps.version.outputs.version }} .
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/my-app:latest .
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/my-app:${{ steps.version.outputs.version }}
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/my-app:latest
```

### Deploy Workflow (On New Tag)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  release:
    types: [published]

jobs:
  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Get release version
        run: echo "VERSION=${GITHUB_REF#refs/tags/}" >> $GITHUB_ENV

      - name: Deploy to server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: deploy
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            echo "🚀 Deploying ${{ env.VERSION }}..."

            docker pull ${{ secrets.DOCKERHUB_USERNAME }}/my-app:${{ env.VERSION }}

            docker stop my-app || true
            docker rm my-app || true

            docker run -d \
              --name my-app \
              --restart unless-stopped \
              -p 80:3000 \
              -e VERSION=${{ env.VERSION }} \
              ${{ secrets.DOCKERHUB_USERNAME }}/my-app:${{ env.VERSION }}

            echo "✅ Deployed ${{ env.VERSION }} successfully!"

      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "🚀 Deployed *${{ env.VERSION }}* to production!\nRelease: ${{ github.event.release.html_url }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### How to Use This Project

```bash
# Setup
mkdir gitops-pipeline && cd gitops-pipeline
git init && git checkout -b main

# Create all scripts
mkdir -p .github/workflows scripts src

# Add all files from above
# Make scripts executable
chmod +x scripts/*.sh

# Initialize VERSION
echo "0.0.0" > VERSION

# First commit
git add .
git commit -m "chore: initial project setup"

# Push to GitHub
git remote add origin https://github.com/pramendraatwork/gitops-pipeline.git
git push -u origin main

# Now make a feature commit
git checkout -b feature/new-api
# make changes...
git commit -m "feat(api): add user endpoint"
git push origin feature/new-api

# Open PR → merge to main
# Watch GitHub Actions automatically:
# 1. Run tests
# 2. Bump version (0.0.0 → 0.1.0)
# 3. Generate CHANGELOG
# 4. Create GitHub Release
# 5. Build Docker image
# 6. Deploy to production!
```

### What You Learned

- ✅ GitOps — Git as source of truth
- ✅ Automated versioning (semver)
- ✅ Auto changelog generation
- ✅ GitHub Releases
- ✅ Full CI/CD triggered by Git
- ✅ Commit message linting
- ✅ Zero-touch deployments

---

## Summary 📋

| Project | Level | What You Built | Key Skills |
|---|---|---|---|
| Personal Portfolio | 🟢 Beginner | Portfolio with Git workflow | branches, tags, pages |
| Team Collaboration | 🟡 Intermediate | GitFlow team workflow | PRs, conflicts, releases |
| GitOps Pipeline | 🔴 Advanced | Fully automated release system | auto-versioning, GitOps |

> 💪 **Challenge**: Complete all 3 projects and put them on GitHub. These alone show a recruiter you understand real Git workflows — not just `git push`!