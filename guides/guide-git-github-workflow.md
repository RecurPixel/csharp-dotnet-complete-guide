# Git & GitHub Workflow Quick Reference

---

## What is Git?

**Git** = Distributed version control system for tracking code changes

**Key Features:**
- ✅ **Distributed** - Every developer has full history
- ✅ **Branching** - Easy and fast branch creation
- ✅ **Fast** - Most operations are local
- ✅ **Data Integrity** - SHA-1 checksums
- ✅ **Non-linear Development** - Multiple parallel branches

**Git vs GitHub:**
- **Git** = Version control system (software)
- **GitHub** = Web-based hosting service for Git repositories (platform)

---

## Git Installation & Configuration

### Installation

```bash
# Check if Git is installed
git --version

# Windows: Download from git-scm.com
# macOS: brew install git
# Linux: sudo apt-get install git
```

### Initial Configuration

```bash
# Set your identity (required)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim

# Set default branch name
git config --global init.defaultBranch main

# View all config
git config --list

# View specific config
git config user.name
git config user.email
```

---

## Git Basics

### Creating a Repository

```bash
# Initialize new repository
git init

# Clone existing repository
git clone https://github.com/username/repository.git

# Clone to specific folder
git clone https://github.com/username/repository.git my-folder

# Clone specific branch
git clone -b develop https://github.com/username/repository.git
```

### The Three States

```
Working Directory  →  Staging Area  →  Repository
    (modified)         (staged)         (committed)
```

```bash
# Check status
git status

# Short status
git status -s
```

### Adding Files (Staging)

```bash
# Add specific file
git add filename.txt

# Add all files in current directory
git add .

# Add all files in repository
git add -A

# Add files by pattern
git add *.cs

# Add files interactively
git add -i

# Add parts of files (patch mode)
git add -p filename.txt
```

### Committing Changes

```bash
# Commit staged changes
git commit -m "Add customer registration feature"

# Commit with detailed message
git commit
# Opens editor for multi-line message

# Stage and commit in one command
git commit -am "Fix bug in login validation"

# Amend last commit (change message or add files)
git add forgotten-file.txt
git commit --amend -m "New commit message"

# Amend without changing message
git commit --amend --no-edit
```

### Viewing History

```bash
# View commit history
git log

# One line per commit
git log --oneline

# Graph view
git log --graph --oneline --all

# Last N commits
git log -3

# Commits by author
git log --author="John Doe"

# Commits in date range
git log --since="2024-01-01" --until="2024-12-31"

# Show changes in each commit
git log -p

# Show statistics
git log --stat

# Pretty format
git log --pretty=format:"%h - %an, %ar : %s"
# %h = abbreviated hash
# %an = author name
# %ar = author date, relative
# %s = subject
```

### Viewing Changes

```bash
# Show unstaged changes
git diff

# Show staged changes
git diff --staged

# Show changes in specific file
git diff filename.txt

# Show changes between commits
git diff commit1 commit2

# Show changes between branches
git diff main develop
```

---

## Branching & Merging

### Creating Branches

```bash
# Create new branch
git branch feature/login

# Create and switch to new branch
git checkout -b feature/login
# Or (Git 2.23+)
git switch -c feature/login

# Create branch from specific commit
git branch feature/login abc123

# List all branches
git branch

# List all branches (including remote)
git branch -a

# List remote branches
git branch -r
```

### Switching Branches

```bash
# Switch to existing branch
git checkout main
# Or (Git 2.23+)
git switch main

# Switch to previous branch
git checkout -
git switch -
```

### Merging Branches

```bash
# Merge feature branch into current branch
git checkout main
git merge feature/login

# Merge with custom message
git merge feature/login -m "Merge login feature"

# Abort merge if conflicts
git merge --abort

# Continue merge after resolving conflicts
git add resolved-file.txt
git commit
```

### Merge Strategies

```bash
# Fast-forward merge (default if possible)
git merge feature/login

# No fast-forward (always create merge commit)
git merge --no-ff feature/login

# Squash merge (combine all commits into one)
git merge --squash feature/login
git commit -m "Add login feature"
```

### Deleting Branches

```bash
# Delete merged branch
git branch -d feature/login

# Force delete unmerged branch
git branch -D feature/login

# Delete remote branch
git push origin --delete feature/login
```

---

## Resolving Merge Conflicts

### When Conflicts Occur

```bash
# After git merge, if conflicts:
git status
# Shows conflicted files

# Conflict markers in file:
<<<<<<< HEAD
Current branch changes
=======
Incoming branch changes
>>>>>>> feature/login
```

### Resolving Conflicts

```bash
# Option 1: Manually edit file
# Remove conflict markers and keep desired changes

# Option 2: Use merge tool
git mergetool

# Option 3: Accept one version completely
git checkout --ours conflicted-file.txt   # Keep current branch
git checkout --theirs conflicted-file.txt # Keep incoming branch

# After resolving:
git add conflicted-file.txt
git commit
```

---

## Working with Remote Repositories

### Adding Remotes

```bash
# Add remote
git remote add origin https://github.com/username/repo.git

# View remotes
git remote -v

# Rename remote
git remote rename origin upstream

# Remove remote
git remote remove origin

# Change remote URL
git remote set-url origin https://github.com/username/new-repo.git
```

### Fetching & Pulling

```bash
# Fetch changes from remote (doesn't merge)
git fetch origin

# Fetch from all remotes
git fetch --all

# Pull changes (fetch + merge)
git pull origin main

# Pull with rebase instead of merge
git pull --rebase origin main

# Pull from upstream (for forks)
git pull upstream main
```

### Pushing Changes

```bash
# Push to remote
git push origin main

# Push and set upstream
git push -u origin main
# Or
git push --set-upstream origin main

# Push all branches
git push origin --all

# Push tags
git push origin --tags

# Force push (dangerous!)
git push --force origin main
# Safer force push
git push --force-with-lease origin main

# Delete remote branch
git push origin --delete feature/old-feature
```

---

## Undoing Changes

### Unstage Files

```bash
# Unstage file (keep changes)
git reset filename.txt

# Unstage all files
git reset
```

### Discard Changes

```bash
# Discard changes in working directory
git checkout -- filename.txt
# Or (Git 2.23+)
git restore filename.txt

# Discard all changes
git checkout -- .
git restore .

# Discard staged changes
git restore --staged filename.txt
```

### Reverting Commits

```bash
# Revert specific commit (creates new commit)
git revert abc123

# Revert without committing
git revert abc123 --no-commit

# Revert multiple commits
git revert abc123..def456
```

### Resetting Commits

```bash
# Soft reset (keep changes staged)
git reset --soft HEAD~1

# Mixed reset (keep changes unstaged - default)
git reset HEAD~1
git reset --mixed HEAD~1

# Hard reset (discard all changes - DANGEROUS!)
git reset --hard HEAD~1

# Reset to specific commit
git reset --hard abc123

# Reset to remote state
git reset --hard origin/main
```

### Reset vs Revert

```bash
# Use REVERT for public branches (shared with others)
# Creates new commit, safe for collaboration
git revert abc123

# Use RESET for local branches (not pushed yet)
# Rewrites history, dangerous if pushed
git reset --hard HEAD~1
```

---

## Stashing Changes

### Basic Stashing

```bash
# Stash current changes
git stash

# Stash with message
git stash save "Work in progress on login feature"

# List stashes
git stash list

# Apply last stash (keep stash)
git stash apply

# Apply specific stash
git stash apply stash@{1}

# Apply and remove last stash
git stash pop

# Remove specific stash
git stash drop stash@{1}

# Remove all stashes
git stash clear
```

### Advanced Stashing

```bash
# Stash including untracked files
git stash -u

# Stash including ignored files
git stash -a

# Stash specific files
git stash push -m "Stash login changes" -- login.cs

# Create branch from stash
git stash branch feature/login-fix
```

---

## Tagging

### Creating Tags

```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag specific commit
git tag -a v1.0.0 abc123 -m "Release version 1.0.0"

# List tags
git tag

# List tags with pattern
git tag -l "v1.*"

# Show tag details
git show v1.0.0
```

### Pushing Tags

```bash
# Push specific tag
git push origin v1.0.0

# Push all tags
git push origin --tags

# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0
```

---

## Git Workflows

### Feature Branch Workflow

```bash
# 1. Create feature branch
git checkout -b feature/user-profile

# 2. Make changes
git add .
git commit -m "Add user profile page"

# 3. Push to remote
git push -u origin feature/user-profile

# 4. Create Pull Request on GitHub

# 5. After PR approval, merge to main
git checkout main
git merge feature/user-profile

# 6. Delete feature branch
git branch -d feature/user-profile
git push origin --delete feature/user-profile
```

### GitFlow Workflow

**Branches:**
- `main` - Production code
- `develop` - Development branch
- `feature/*` - New features
- `release/*` - Release preparation
- `hotfix/*` - Production fixes

```bash
# Start new feature
git checkout develop
git checkout -b feature/shopping-cart

# Develop feature
git add .
git commit -m "Add shopping cart functionality"

# Finish feature
git checkout develop
git merge --no-ff feature/shopping-cart
git branch -d feature/shopping-cart
git push origin develop

# Start release
git checkout develop
git checkout -b release/1.2.0
# Make release-specific changes (version numbers, etc.)
git commit -am "Bump version to 1.2.0"

# Finish release
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin main --tags

git checkout develop
git merge --no-ff release/1.2.0
git push origin develop

git branch -d release/1.2.0

# Hotfix
git checkout main
git checkout -b hotfix/critical-bug

# Fix bug
git commit -am "Fix critical security bug"

# Finish hotfix
git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v1.2.1 -m "Hotfix version 1.2.1"
git push origin main --tags

git checkout develop
git merge --no-ff hotfix/critical-bug
git push origin develop

git branch -d hotfix/critical-bug
```

### GitHub Flow (Simplified)

```bash
# 1. Create branch from main
git checkout main
git pull origin main
git checkout -b feature/new-feature

# 2. Make changes and commit
git add .
git commit -m "Add new feature"

# 3. Push regularly
git push -u origin feature/new-feature

# 4. Open Pull Request when ready

# 5. After approval and tests pass, merge to main

# 6. Deploy main branch automatically
```

---

## Rebasing

### Basic Rebase

```bash
# Rebase current branch onto main
git checkout feature/login
git rebase main

# If conflicts, resolve then:
git add resolved-file.txt
git rebase --continue

# Abort rebase
git rebase --abort

# Skip conflicting commit
git rebase --skip
```

### Interactive Rebase

```bash
# Rebase last 3 commits
git rebase -i HEAD~3

# Interactive rebase options:
# pick = use commit
# reword = use commit, but edit message
# edit = use commit, but stop for amending
# squash = use commit, but meld into previous commit
# fixup = like squash, but discard commit message
# drop = remove commit

# Example: Squash last 3 commits into one
git rebase -i HEAD~3
# Change 'pick' to 'squash' for last 2 commits
# Save and edit combined commit message
```

### Rebase vs Merge

```bash
# Merge: Creates merge commit, preserves history
git merge feature/login
# History: A-B-C-D-E-F (with merge commit F)

# Rebase: Linear history, no merge commit
git rebase main
# History: A-B-C-D'-E' (commits D and E rebased)

# Golden Rule: Never rebase public branches!
# ✅ Rebase: feature branches (not pushed or only you use)
# ❌ Rebase: main, develop, or shared branches
```

---

## GitHub Features

### Pull Requests (PRs)

**Creating a PR:**
1. Push branch to GitHub
2. Navigate to repository on GitHub
3. Click "New Pull Request"
4. Select base and compare branches
5. Add title and description
6. Request reviewers
7. Create pull request

**PR Best Practices:**
```markdown
# PR Title
Add user authentication feature

# PR Description
## Changes
- Implemented JWT authentication
- Added login/register endpoints
- Created user repository

## Testing
- Unit tests for AuthService
- Integration tests for auth endpoints
- Manual testing with Postman

## Screenshots
[Add screenshots if UI changes]

## Related Issues
Closes #123
```

**PR Commands:**
```bash
# Fetch PR to local branch
git fetch origin pull/123/head:pr-123
git checkout pr-123

# Update PR with new commits
git checkout feature/login
git add .
git commit -m "Address review comments"
git push origin feature/login
```

### Code Review

**Reviewer Checklist:**
- ✅ Code follows project conventions
- ✅ Tests are included and pass
- ✅ No security vulnerabilities
- ✅ Performance considerations
- ✅ Documentation updated
- ✅ No debugging code left

**Review Comments:**
```markdown
# Requesting changes
Could you add validation for the email field?

# Suggesting improvement
Consider using a constant for this magic number

# Approving
LGTM! (Looks Good To Me)
```

### GitHub Actions (CI/CD)

```yaml
# .github/workflows/dotnet.yml
name: .NET Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 8.0.x
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build
      run: dotnet build --no-restore
    
    - name: Test
      run: dotnet test --no-build --verbosity normal
```

### GitHub Issues

**Creating Issues:**
```markdown
# Bug Report
**Description:** Login fails with valid credentials

**Steps to Reproduce:**
1. Navigate to login page
2. Enter valid email and password
3. Click "Login"

**Expected:** User logged in successfully
**Actual:** Error message "Invalid credentials"

**Environment:**
- Browser: Chrome 120
- OS: Windows 11
- Version: 1.2.0

**Screenshots:** [Attach if available]
```

```markdown
# Feature Request
**Is your feature request related to a problem?**
Yes, users can't reset their passwords.

**Describe the solution you'd like**
Add a "Forgot Password" link that sends a reset email.

**Describe alternatives you've considered**
- SMS-based reset
- Security questions

**Additional context**
Should use existing email service.
```

---

## .gitignore File

### Common .gitignore for .NET

```gitignore
# Build results
[Dd]ebug/
[Dd]ebugPublic/
[Rr]elease/
[Rr]eleases/
x64/
x86/
[Ww][Ii][Nn]32/
[Aa][Rr][Mm]/
[Aa][Rr][Mm]64/
bld/
[Bb]in/
[Oo]bj/
[Ll]og/
[Ll]ogs/

# Visual Studio
.vs/
.vscode/

# User-specific files
*.rsuser
*.suo
*.user
*.userosscache
*.sln.docstates

# Build Results
*_i.c
*_p.c
*_h.h
*.ilk
*.meta
*.obj
*.iobj
*.pch
*.pdb
*.ipdb
*.pgc
*.pgd
*.rsp
*.sbr
*.tlb
*.tli
*.tlh
*.tmp
*.tmp_proj
*_wpftmp.csproj
*.log
*.vspscc
*.vssscc
.builds
*.pidb
*.svclog
*.scc

# NuGet Packages
*.nupkg
*.snupkg
**/packages/*
!**/packages/build/

# Database
*.mdf
*.ldf
*.ndf

# Environment variables
.env
.env.local

# IDE
.idea/
*.swp
*.swo

# OS generated files
.DS_Store
Thumbs.db
```

---

## Advanced Git Commands

### Cherry-pick

```bash
# Apply specific commit to current branch
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456

# Cherry-pick without committing
git cherry-pick abc123 --no-commit
```

### Bisect (Find Bug)

```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark known good commit
git bisect good abc123

# Git will checkout commits for testing
# After each test:
git bisect good  # If commit is good
git bisect bad   # If commit is bad

# End bisect
git bisect reset
```

### Reflog (Recover Lost Commits)

```bash
# View reflog
git reflog

# Recover lost commit
git reflog
# Find commit hash
git checkout abc123
git branch recovered-branch

# Or reset to it
git reset --hard abc123
```

### Cleaning

```bash
# Show what would be deleted
git clean -n

# Remove untracked files
git clean -f

# Remove untracked files and directories
git clean -fd

# Remove ignored files too
git clean -fdx
```

### Blame (Find Who Changed Line)

```bash
# Show who last modified each line
git blame filename.txt

# Blame specific line range
git blame -L 10,20 filename.txt

# Blame with commit details
git blame -c filename.txt
```

---

## Git Aliases

### Setting Up Aliases

```bash
# Create aliases in git config
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status

# Useful aliases
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --oneline --graph --all'
git config --global alias.amend 'commit --amend --no-edit'

# Usage
git co main          # git checkout main
git visual           # git log --oneline --graph --all
```

### Common Aliases

```bash
# Add to ~/.gitconfig
[alias]
    # Shortcuts
    co = checkout
    br = branch
    ci = commit
    st = status
    
    # Useful commands
    unstage = reset HEAD --
    last = log -1 HEAD
    
    # Pretty logs
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    
    # Undo last commit
    undo = reset --soft HEAD~1
    
    # Amend without editing message
    amend = commit --amend --no-edit
    
    # Show contributors
    contributors = shortlog -sn
```

---

## Git Best Practices

### Commit Messages

**Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting, missing semicolons, etc.
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples:**
```bash
# Good commit messages
git commit -m "feat(auth): add JWT authentication"
git commit -m "fix(login): resolve validation error on empty email"
git commit -m "docs(readme): update installation instructions"
git commit -m "refactor(user-service): extract validation logic"

# Bad commit messages
git commit -m "fixes"
git commit -m "updated stuff"
git commit -m "wip"
git commit -m "asdfasdf"
```

### Branch Naming

```bash
# Good branch names
feature/user-authentication
feature/add-payment-gateway
bugfix/fix-login-validation
hotfix/critical-security-issue
release/v1.2.0

# Patterns
feature/<description>
bugfix/<description>
hotfix/<description>
release/<version>
chore/<description>

# With ticket numbers
feature/JIRA-123-user-profile
bugfix/JIRA-456-fix-cart
```

### Commit Frequency

```bash
# ✅ Good: Logical, atomic commits
git commit -m "feat(auth): add user registration endpoint"
git commit -m "feat(auth): add login endpoint"
git commit -m "feat(auth): add password hashing"
git commit -m "test(auth): add authentication tests"

# ❌ Bad: Too many tiny commits
git commit -m "add file"
git commit -m "fix typo"
git commit -m "fix another typo"
git commit -m "actually fix typo"

# ❌ Bad: One giant commit
git commit -m "implement entire authentication system"
```

### Before Pushing

```bash
# Always check before pushing
git status
git log --oneline -5
git diff origin/main

# Run tests
dotnet test

# Pull latest changes
git pull --rebase origin main

# Then push
git push origin feature/my-feature
```

---

## Troubleshooting Common Issues

### Merge Conflicts

```bash
# When conflict occurs
git status  # See conflicted files

# Resolve manually or:
git mergetool

# After resolving
git add .
git commit
```

### Accidentally Committed to Wrong Branch

```bash
# Move last commit to new branch
git branch feature/correct-branch
git reset --hard HEAD~1
git checkout feature/correct-branch
```

### Undo Pushed Commit

```bash
# DON'T use reset if pushed (rewrites history)
# Use revert instead
git revert abc123
git push origin main
```

### Recover Deleted Branch

```bash
# Find commit hash
git reflog

# Recreate branch
git checkout -b recovered-branch abc123
```

### Large Files

```bash
# If accidentally committed large files
# Install Git LFS
git lfs install

# Track large files
git lfs track "*.psd"
git lfs track "*.zip"

# Or remove from history (complex, be careful)
git filter-branch --tree-filter 'rm -rf large-file.zip' HEAD
```

---

## Collaboration Best Practices

### Pull Request Checklist

**Before Creating PR:**
- [ ] Code builds successfully
- [ ] All tests pass
- [ ] Code follows project style guide
- [ ] No debugging code (console.log, etc.)
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] Branch is up-to-date with base branch

**PR Description Should Include:**
- [ ] What changed and why
- [ ] How to test
- [ ] Screenshots (if UI change)
- [ ] Related issues
- [ ] Breaking changes (if any)

### Code Review Guidelines

**As Author:**
- Keep PRs small and focused
- Respond to comments promptly
- Don't take feedback personally
- Explain reasoning when disagreeing

**As Reviewer:**
- Review promptly (within 24 hours)
- Be constructive and kind
- Focus on code, not person
- Explain why, not just what
- Approve when requirements met

### Merge Strategy

```bash
# Squash Merge (single commit)
# Good for: Feature branches with many small commits
git merge --squash feature/login

# Regular Merge (preserve history)
# Good for: Long-lived branches, releases
git merge --no-ff develop

# Rebase and Merge (linear history)
# Good for: Clean, linear history
git rebase main
git merge feature/login
```

---

## Git Cheat Sheet

### Daily Commands

```bash
# Status and differences
git status
git diff
git diff --staged

# Add and commit
git add .
git commit -m "message"
git commit -am "message"

# Push and pull
git pull origin main
git push origin feature-branch

# Branches
git branch
git checkout -b new-branch
git switch new-branch
git branch -d old-branch

# Stash
git stash
git stash pop
```

### Undoing Things

```bash
# Unstage
git reset filename.txt

# Discard changes
git checkout -- filename.txt
git restore filename.txt

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Revert commit (safe for pushed commits)
git revert abc123
```

### Remote Operations

```bash
# View remotes
git remote -v

# Fetch
git fetch origin

# Pull
git pull origin main
git pull --rebase origin main

# Push
git push origin main
git push -u origin feature-branch

# Delete remote branch
git push origin --delete old-branch
```

---

## Quick Reference: When to Use What

### Merge vs Rebase

| Scenario | Use Merge | Use Rebase |
|----------|-----------|------------|
| Feature branch → main | ✅ | ✅ |
| Main → feature branch | ❌ | ✅ |
| Public/shared branch | ✅ | ❌ |
| Private branch | ✅ | ✅ |
| Want clean history | ❌ | ✅ |
| Want full history | ✅ | ❌ |

### Reset vs Revert

| Scenario | Use Reset | Use Revert |
|----------|-----------|------------|
| Local commits only | ✅ | ✅ |
| Pushed commits | ❌ | ✅ |
| Want to preserve history | ❌ | ✅ |
| Want to rewrite history | ✅ | ❌ |

---

**Guide Complete!** This comprehensive Git & GitHub guide covers installation, basic commands, branching strategies, workflows, collaboration, best practices, and troubleshooting. Master Git to collaborate effectively with your team! 🔀
