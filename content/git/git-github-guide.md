---
title: "Git & GitHub — Complete Guide from Beginner to Advanced"
date: 2024-02-15
draft: false
description: "Everything about Git and GitHub: version control concepts, all commands, branching, merging, rebasing, and real DevOps workflows with diagrams."
categories: ["git"]
tags: ["git", "github", "version-control", "branching", "devops"]
showToc: true
TocOpen: true
---

## 1. What is Version Control? 🤔

Imagine you're writing a document and you make changes. Then you realize you broke something and want to go back. What do you do?

Most people do this:

```
report.docx
report_final.docx
report_final_v2.docx
report_final_v2_ACTUAL_FINAL.docx
report_final_v2_ACTUAL_FINAL_use_this_one.docx
```

😅 Sound familiar? **Version control solves this forever.**

Version Control is a system that:
- 📸 Takes **snapshots** of your code at any point
- ⏪ Lets you **go back** to any previous state
- 👥 Lets **multiple people** work on the same code without conflicts
- 🔍 Shows you **who changed what and why**
- 🌿 Lets you work on **multiple features** simultaneously

> 💡 **Think of it like this**: Version control is like having infinite Ctrl+Z for your entire project — forever.

---

## 2. What is Git? 🔧

Git is the world's most popular **distributed version control system**, created by **Linus Torvalds** (the same person who created Linux) in 2005.

### Centralized vs Distributed

```
CENTRALIZED (old way — SVN, CVS)
────────────────────────────────
     [Central Server]
          │
    ┌─────┼─────┐
    │     │     │
 Dev1   Dev2  Dev3
 (no full copy) (no full copy)

Problem: Server goes down = everyone stops working!

DISTRIBUTED (Git's way)
────────────────────────────────
     [Remote Server - GitHub]
          │
    ┌─────┼─────┐
    │     │     │
 Dev1   Dev2  Dev3
(full  (full  (full
 copy)  copy)  copy)

Every developer has the FULL history locally!
Works offline. Server down? Keep working!
```

### Key Git concepts

```
┌─────────────────────────────────────────────────────────┐
│                    GIT AREAS                            │
│                                                         │
│  Working Directory → Staging Area → Local Repo → Remote │
│       (edit)           (git add)   (git commit) (push)  │
│                                                         │
│  📝 You edit       📦 You stage    💾 You commit  ☁️ Push │
│     files here        changes        a snapshot    online│
└─────────────────────────────────────────────────────────┘
```

| Area | What it is | Command to move forward |
|---|---|---|
| **Working Directory** | Your actual files on disk | `git add` |
| **Staging Area** | Changes ready to commit | `git commit` |
| **Local Repository** | Your local commit history | `git push` |
| **Remote Repository** | GitHub/GitLab/Bitbucket | others can pull |

---

## 3. What is GitHub? 🐙

Git = the tool (local)
GitHub = the cloud hosting for Git repositories (remote)

```
GIT vs GITHUB
─────────────────────────────────────────────
GIT                      GITHUB
────                     ──────
• Local tool             • Cloud platform
• Command line           • Web interface
• Version control        • Collaboration
• Works offline          • Pull Requests
• Free, open source      • Issues & Projects
• Created by Linus       • Owned by Microsoft
• Installed on your PC   • Lives at github.com
```

GitHub also gives you:
- 🔄 **Pull Requests** — review code before merging
- 🐛 **Issues** — track bugs and features
- ⚡ **GitHub Actions** — CI/CD pipelines
- 📖 **GitHub Pages** — free website hosting (like this blog!)
- 🔒 **Branch protection** — prevent bad code from merging

---

## 4. Installing & Configuring Git ⚙️

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install git -y

# macOS
brew install git

# Verify
git --version

# ── First time setup (do this once!) ────────────────────────
git config --global user.name "Pramendra Rajput"
git config --global user.email "pramendraatwork@gmail.com"
git config --global core.editor "code --wait"    # VS Code as editor
git config --global init.defaultBranch main       # use main not master
git config --global color.ui auto                 # colored output

# View your config
git config --list
git config user.name
```

---

## 5. Core Git Commands 💻

### Starting a Repository

```bash
# Start fresh
mkdir my-project && cd my-project
git init                        # creates .git folder — this IS the repo

# OR clone an existing repo
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder   # custom folder name
git clone --depth=1 https://github.com/user/repo.git   # shallow clone (faster)
```

### The Daily Workflow

```
┌─────────────────────────────────────────────────────────┐
│              DAILY GIT WORKFLOW                         │
│                                                         │
│  1. git pull          ← get latest changes              │
│  2. (write code)      ← do your work                    │
│  3. git status        ← see what changed                │
│  4. git add .         ← stage changes                   │
│  5. git commit -m ""  ← save snapshot                   │
│  6. git push          ← share with team                 │
└─────────────────────────────────────────────────────────┘
```

```bash
# Check status — run this ALL the time
git status                      # what's changed?
git status -s                   # short format

# Stage changes
git add file.txt                # stage one file
git add .                       # stage ALL changes
git add *.js                    # stage all JS files
git add -p                      # interactively choose what to stage

# Commit
git commit -m "feat: add login page"        # commit with message
git commit -am "fix: typo in header"        # add + commit tracked files
git commit --amend -m "new message"         # fix last commit message

# Push & Pull
git push                        # push to remote
git push origin main            # push specific branch
git push -u origin main         # set upstream (first time)
git push --force-with-lease     # safer force push
git pull                        # fetch + merge
git pull --rebase               # fetch + rebase (cleaner history)
git fetch                       # download changes but don't merge
```

### Viewing History

```bash
git log                         # full commit history
git log --oneline               # one line per commit
git log --oneline --graph       # visual branch graph
git log --oneline -10           # last 10 commits
git log --author="Pramendra"    # commits by author
git log --since="2 weeks ago"   # commits in last 2 weeks
git log -p file.txt             # history of a specific file
git show abc1234                # show a specific commit
git diff                        # unstaged changes
git diff --staged               # staged changes
git diff main..feature          # diff between branches
```

### Undoing Things

```bash
# Unstage a file
git restore --staged file.txt   # unstage (keep changes)

# Discard changes in working directory
git restore file.txt            # discard changes (CAREFUL — can't undo!)
git restore .                   # discard ALL changes

# Undo commits (safe — creates new commit)
git revert HEAD                 # undo last commit
git revert abc1234              # undo specific commit

# Reset (DANGEROUS — rewrites history)
git reset --soft HEAD~1         # undo commit, keep changes staged
git reset --mixed HEAD~1        # undo commit, keep changes unstaged
git reset --hard HEAD~1         # undo commit, DELETE changes forever!

# Recover lost work
git reflog                      # see ALL git actions (lifesaver!)
git checkout abc1234            # go back to any commit temporarily
```

> ⚠️ **Rule**: Never use `git reset --hard` on shared branches. Use `git revert` instead — it's safe and keeps history.

---

## 6. Branching — The Most Powerful Feature 🌿

Branches let you work on features without touching the main code.

```
BRANCHING FLOW
──────────────────────────────────────────────────────

main    ●─────────────────────────────────●─────▶
         \                               /
feature   ●──●──●──●──●──●──●──●──●──●
              (work on feature safely)
                              (merge back when done)

──────────────────────────────────────────────────────

main    ●────────────●────────────────────●──────▶
         \          / \                  /
hotfix    ●──●──●──●   \                /
                        ●──●──●──●──●──●
                           feature-xyz
```

### Branch Commands

```bash
# View branches
git branch                      # list local branches
git branch -a                   # list all (local + remote)
git branch -v                   # list with last commit

# Create branches
git branch feature-login        # create branch
git checkout -b feature-login   # create AND switch (old way)
git switch -c feature-login     # create AND switch (new way ✅)

# Switch branches
git checkout main               # old way
git switch main                 # new way ✅

# Rename branch
git branch -m old-name new-name

# Delete branch
git branch -d feature-login     # delete (safe — won't delete unmerged)
git branch -D feature-login     # force delete
git push origin --delete feature-login   # delete remote branch
```

### Merging

```bash
# Merge feature into main
git switch main
git merge feature-login

# Types of merges:
# Fast-forward (no diverging history)
git merge feature-login

# No fast-forward (always creates merge commit)
git merge --no-ff feature-login

# Squash (combine all feature commits into one)
git merge --squash feature-login
git commit -m "feat: add login feature"
```

### Merge Conflicts — Don't Panic! 😅

```bash
# When you see this:
<<<<<<< HEAD
your changes here
=======
their changes here
>>>>>>> feature-login

# Steps to resolve:
# 1. Open the file
# 2. Choose what to keep (delete the markers)
# 3. git add the resolved file
# 4. git commit to complete the merge
```

```bash
# Useful conflict tools
git status                      # see conflicted files
git diff                        # see all conflicts
git mergetool                   # open visual merge tool
git merge --abort               # cancel the merge entirely
```

---

## 7. Rebasing — Clean History 🧹

Rebase re-applies your commits on top of another branch. Result: clean, linear history.

```
BEFORE REBASE:
main    ●──●──●──●
         \
feature   ●──●──●

AFTER REBASE (feature rebased onto main):
main    ●──●──●──●
                  \
feature            ●──●──● (commits replayed on top)

AFTER MERGE (fast-forward):
main    ●──●──●──●──●──●──●
```

```bash
# Rebase feature onto main
git switch feature-login
git rebase main

# Interactive rebase — rewrite history
git rebase -i HEAD~3            # edit last 3 commits
# Options: pick, squash, reword, drop, edit

# Rebase onto remote
git pull --rebase origin main

# Abort rebase
git rebase --abort
```

> 💡 **Golden rule**: Never rebase commits that have been pushed to a shared branch!

---

## 8. Remote Repository Commands 🌐

```bash
# View remotes
git remote -v                   # list remotes with URLs
git remote show origin          # detailed remote info

# Add/change remotes
git remote add origin https://github.com/user/repo.git
git remote set-url origin https://github.com/user/new-repo.git
git remote remove origin

# Syncing
git fetch origin                # download all remote changes
git fetch --all                 # fetch from all remotes
git pull origin main            # fetch + merge main
git push origin main            # push main
git push origin --all           # push all branches
git push origin --tags          # push all tags
```

---

## 9. Tags — Marking Releases 🏷️

```bash
# Create tags
git tag v1.0.0                              # lightweight tag
git tag -a v1.0.0 -m "Release version 1.0" # annotated tag (better)
git tag -a v1.0.0 abc1234 -m "Tag old commit"

# View tags
git tag                         # list all tags
git tag -l "v1.*"               # filter tags
git show v1.0.0                 # show tag details

# Push tags
git push origin v1.0.0          # push one tag
git push origin --tags          # push all tags

# Delete tags
git tag -d v1.0.0               # delete local
git push origin --delete v1.0.0 # delete remote
```

---

## 10. Stashing — Save Work Temporarily 🗃️

You're in the middle of work but need to switch branches urgently. Don't commit half-done work — stash it!

```bash
git stash                       # stash current changes
git stash push -m "half done login form"    # stash with name
git stash list                  # see all stashes
git stash pop                   # apply latest stash + delete it
git stash apply stash@{2}       # apply specific stash (keep it)
git stash drop stash@{0}        # delete a stash
git stash clear                 # delete ALL stashes
git stash branch feature-login  # create branch from stash
```

---

## 11. GitHub Workflow — Pull Requests 🔄

This is how real teams work:

```
GITHUB PULL REQUEST WORKFLOW
─────────────────────────────────────────────────────────

1. Fork or clone repo
         │
2. Create feature branch
   git switch -c feature/user-auth
         │
3. Make commits
   git commit -m "feat: add JWT authentication"
         │
4. Push branch to GitHub
   git push origin feature/user-auth
         │
5. Open Pull Request on GitHub
   • Write description of changes
   • Request reviewers
   • Link related issues
         │
6. Code Review
   • Team reviews the code
   • Leave comments
   • Request changes
         │
7. Fix review comments
   git commit -m "fix: address review comments"
   git push
         │
8. Approval + Merge
   • Squash and merge / Rebase and merge / Merge commit
         │
9. Delete feature branch
   git branch -d feature/user-auth
         │
10. Pull latest main
    git switch main && git pull
```

### Writing Good Commit Messages

```
FORMAT:
<type>(<scope>): <short description>

<body — what and why, not how>

<footer — breaking changes, issue references>

TYPES:
feat:     new feature
fix:      bug fix
docs:     documentation only
style:    formatting (no logic change)
refactor: code restructure (no feature/fix)
test:     adding tests
chore:    build process, dependencies
ci:       CI/CD changes

EXAMPLES:
✅ feat(auth): add JWT token refresh
✅ fix(api): handle null response from user service
✅ docs: update README with setup instructions
✅ chore: upgrade Node.js to v20

❌ fixed stuff
❌ WIP
❌ asdfgh
❌ changes
```

---

## 12. .gitignore — What NOT to Track 🚫

```bash
# Create .gitignore in project root
cat > .gitignore << 'EOF'
# Dependencies
node_modules/
vendor/

# Build output
dist/
build/
*.class
*.o

# Environment & secrets
.env
.env.local
*.pem
*.key
secrets.yaml

# OS files
.DS_Store
Thumbs.db
desktop.ini

# IDE files
.vscode/
.idea/
*.swp

# Logs
*.log
logs/

# Terraform
*.tfstate
*.tfstate.backup
.terraform/
EOF

# Check what's being ignored
git check-ignore -v filename

# Stop tracking a file that's already committed
git rm --cached .env
git commit -m "chore: remove .env from tracking"
```

---

## 13. Advanced Git Tricks 🚀

```bash
# Cherry-pick — apply a specific commit to current branch
git cherry-pick abc1234
git cherry-pick abc1234..def5678    # range of commits

# Bisect — find which commit introduced a bug
git bisect start
git bisect bad                      # current commit is bad
git bisect good v1.0.0              # this version was good
# Git checks out middle commit — test it
git bisect good                     # or git bisect bad
# Repeat until Git finds the culprit
git bisect reset                    # when done

# Worktree — multiple branches checked out simultaneously
git worktree add ../hotfix hotfix-branch
git worktree list
git worktree remove ../hotfix

# Aliases — shortcuts for long commands
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --all"
# Now use: git lg, git st, git co, git br

# Clean untracked files
git clean -n                        # dry run — see what would be deleted
git clean -fd                       # delete untracked files + dirs

# Show who changed each line
git blame file.txt
git blame -L 10,20 file.txt         # specific lines
```

---

## 14. Git Cheatsheet 📋

```
SETUP
git config --global user.name "Name"
git config --global user.email "email"

START
git init
git clone <url>

DAILY
git status
git add .
git commit -m "message"
git push
git pull

BRANCHES
git switch -c feature-name    # create + switch
git switch main               # switch
git merge feature-name        # merge
git branch -d feature-name    # delete

UNDO
git restore file.txt          # discard changes
git restore --staged file.txt # unstage
git revert HEAD               # undo last commit (safe)
git reset --hard HEAD~1       # undo + delete (dangerous!)

HISTORY
git log --oneline --graph
git diff
git show <commit>
git blame <file>

REMOTE
git remote add origin <url>
git push origin main
git pull origin main
git fetch

STASH
git stash
git stash pop
git stash list

TAGS
git tag -a v1.0.0 -m "release"
git push origin --tags
```

---

## 15. Real DevOps Git Workflow 🏗️

```
GITFLOW / TRUNK-BASED DEVELOPMENT
───────────────────────────────────────────────────────────

main (production)  ●────────────────────────────●──────▶
                    \                           /
develop             ●──●──●──●──●──●──●──●──●
                         \       \       /
feature/login             ●──●──●  \   /
                                    ●──● feature/payment

RULES:
• main     = always deployable, protected branch
• develop  = integration branch
• feature/ = one branch per feature
• hotfix/  = emergency fixes to main
• release/ = prepare release

BRANCH PROTECTION (set on GitHub):
✅ Require PR before merging
✅ Require 1+ approvals
✅ Require status checks (CI must pass)
✅ No force pushes
✅ No deletions
```

---

## What's Next? 🚀

Now that you know Git, these topics build directly on it:

- **GitHub Actions** — automate pipelines triggered by git push
- **Jenkins** — enterprise CI/CD that integrates with Git
- **GitOps** — using Git as the single source of truth for infrastructure
- **Docker** — version control for your runtime environment

> 💪 **Practice**: Create a GitHub repo today, commit something every day for 30 days. Your green contribution graph will thank you!