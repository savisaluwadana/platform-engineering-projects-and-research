# Git Version Control Cheatsheet

## Quick Reference Guide for Platform Engineers

### Initial Setup
```bash
# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global core.editor "vim"
git config --global init.defaultBranch main

# View configuration
git config --list
git config user.name

# Setup SSH key for GitHub/GitLab
ssh-keygen -t ed25519 -C "your.email@example.com"
cat ~/.ssh/id_ed25519.pub  # Copy this to GitHub/GitLab
```

### Repository Basics
```bash
# Initialize repository
git init                    # Create new repo
git clone <url>            # Clone existing repo
git clone <url> <dir>      # Clone to specific directory

# Basic workflow
git status                 # Check status
git add <file>            # Stage file
git add .                 # Stage all changes
git commit -m "message"   # Commit changes
git commit -am "message"  # Stage and commit tracked files

# View history
git log                   # Show commit history
git log --oneline         # Compact view
git log --graph --all     # Visual branch history
git log -p <file>        # Show changes in file
git log --author="name"   # Filter by author
git log --since="2 weeks ago" # Recent commits
git show <commit>         # Show commit details
```

### Branching & Merging
```bash
# Branch management
git branch                # List local branches
git branch -a            # List all branches (including remote)
git branch <name>        # Create branch
git checkout <branch>    # Switch branch
git checkout -b <name>   # Create and switch branch
git switch <branch>      # Modern way to switch
git switch -c <name>     # Create and switch (modern)

# Merging
git merge <branch>       # Merge branch into current
git merge --no-ff <branch> # Force merge commit
git merge --squash <branch> # Squash commits

# Delete branches
git branch -d <branch>   # Delete merged branch
git branch -D <branch>   # Force delete branch
git push origin --delete <branch> # Delete remote branch
```

### Remote Operations
```bash
# Remote management
git remote -v            # List remotes
git remote add origin <url> # Add remote
git remote remove origin # Remove remote
git remote rename old new # Rename remote

# Fetching and pulling
git fetch                # Fetch all remotes
git fetch origin        # Fetch specific remote
git pull                # Fetch and merge
git pull --rebase       # Fetch and rebase

# Pushing
git push                # Push to default remote
git push origin main    # Push to specific branch
git push -u origin main # Set upstream and push
git push --force-with-lease # Safer force push
git push --all          # Push all branches
git push --tags         # Push all tags
```

### Undoing Changes
```bash
# Discard changes
git checkout -- <file>  # Discard working directory changes
git restore <file>      # Modern way to discard changes
git clean -fd           # Remove untracked files and directories

# Unstage changes
git reset HEAD <file>   # Unstage file
git restore --staged <file> # Modern way to unstage

# Modify commits
git commit --amend      # Modify last commit
git commit --amend --no-edit # Amend without changing message

# Reset commits
git reset --soft HEAD~1 # Undo commit, keep changes staged
git reset --mixed HEAD~1 # Undo commit and unstage (default)
git reset --hard HEAD~1 # Undo commit and discard changes
git reset --hard origin/main # Match remote branch

# Revert commits
git revert <commit>     # Create new commit undoing changes
git revert HEAD         # Revert last commit
```

### Stashing
```bash
# Save work temporarily
git stash               # Stash changes
git stash save "message" # Stash with message
git stash -u            # Include untracked files
git stash --all         # Include ignored files

# View stashes
git stash list          # List all stashes
git stash show          # Show latest stash
git stash show -p       # Show changes in stash

# Apply stashes
git stash apply         # Apply latest stash (keep in list)
git stash apply stash@{2} # Apply specific stash
git stash pop           # Apply and remove from list
git stash drop          # Remove latest stash
git stash clear         # Remove all stashes
```

### Tags
```bash
# Create tags
git tag                 # List tags
git tag v1.0.0         # Lightweight tag
git tag -a v1.0.0 -m "Version 1.0.0" # Annotated tag
git tag -a v1.0.0 <commit> # Tag specific commit

# Push tags
git push origin v1.0.0  # Push specific tag
git push --tags         # Push all tags

# Delete tags
git tag -d v1.0.0      # Delete local tag
git push origin --delete v1.0.0 # Delete remote tag
```

### Rebasing
```bash
# Basic rebase
git rebase main         # Rebase current branch onto main
git rebase --continue   # Continue after resolving conflicts
git rebase --abort      # Abort rebase
git rebase --skip       # Skip current commit

# Interactive rebase
git rebase -i HEAD~3    # Rebase last 3 commits
# Options: pick, reword, edit, squash, fixup, drop

# Rebase workflow
git fetch origin
git rebase origin/main
```

### Cherry-picking
```bash
# Apply specific commits
git cherry-pick <commit> # Apply commit to current branch
git cherry-pick <commit1> <commit2> # Multiple commits
git cherry-pick --continue # Continue after conflicts
git cherry-pick --abort    # Abort cherry-pick
```

### Diff & Comparison
```bash
# View differences
git diff                # Working directory vs staging
git diff --staged       # Staging vs last commit
git diff HEAD           # Working directory vs last commit
git diff <branch1> <branch2> # Compare branches
git diff <commit1> <commit2> # Compare commits
git diff --stat         # Summary of changes
git diff --name-only    # Only file names

# Word-level diff
git diff --word-diff    # Word-by-word comparison
```

### Search & Find
```bash
# Search in code
git grep "pattern"      # Search in tracked files
git grep -n "pattern"   # Show line numbers
git grep --count "pattern" # Count matches

# Find in history
git log -S "pattern"    # Commits that add/remove pattern
git log -G "regex"      # Commits matching regex
git log -- <file>      # History of specific file

# Blame
git blame <file>        # Show who changed each line
git blame -L 10,20 <file> # Specific line range
```

### Collaboration Workflows

#### Feature Branch Workflow
```bash
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Work on feature
git add .
git commit -m "Add feature"

# 3. Push feature
git push -u origin feature/new-feature

# 4. Create Pull Request (on GitHub/GitLab)

# 5. After approval, merge and cleanup
git checkout main
git pull origin main
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```

#### GitFlow
```bash
# Main branches: main (production), develop (integration)

# Start feature
git checkout develop
git checkout -b feature/new-feature

# Finish feature
git checkout develop
git merge --no-ff feature/new-feature
git branch -d feature/new-feature
git push origin develop

# Start release
git checkout -b release/1.0.0 develop

# Finish release
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0
git checkout develop
git merge --no-ff release/1.0.0
git branch -d release/1.0.0

# Hotfix
git checkout -b hotfix/1.0.1 main
# ... fix ...
git checkout main
git merge --no-ff hotfix/1.0.1
git tag -a v1.0.1
git checkout develop
git merge --no-ff hotfix/1.0.1
git branch -d hotfix/1.0.1
```

### Submodules
```bash
# Add submodule
git submodule add <url> <path>

# Clone with submodules
git clone --recursive <url>

# Initialize submodules (if not cloned recursively)
git submodule init
git submodule update

# Update submodules
git submodule update --remote

# Remove submodule
git submodule deinit <path>
git rm <path>
rm -rf .git/modules/<path>
```

### Hooks (Automation)
```bash
# Hooks location: .git/hooks/

# Common hooks
pre-commit          # Before commit (run tests, lint)
prepare-commit-msg  # Before commit message editor
commit-msg          # Validate commit message
post-commit         # After commit
pre-push           # Before push
post-checkout      # After checkout

# Example pre-commit hook
#!/bin/bash
npm test || exit 1  # Run tests before commit
```

### Advanced Operations

#### Squashing Commits
```bash
# Method 1: Interactive rebase
git rebase -i HEAD~3
# Change 'pick' to 'squash' for commits to combine

# Method 2: Soft reset and recommit
git reset --soft HEAD~3
git commit -m "Combined commit message"
```

#### Bisect (Find Bug Introduction)
```bash
git bisect start
git bisect bad          # Current commit is bad
git bisect good <commit> # Known good commit
# Git will checkout commits for testing
git bisect good         # After testing (if good)
git bisect bad          # After testing (if bad)
git bisect reset        # When done
```

#### Worktrees (Multiple Working Directories)
```bash
git worktree add ../feature-branch feature-branch
git worktree list
git worktree remove ../feature-branch
```

### Configuration Files

#### .gitignore
```bash
# Patterns
*.log                   # All .log files
!important.log         # Exception
/tmp                   # Root tmp directory
tmp/                   # Any tmp directory
**/*.swp              # All .swp files

# Common patterns
node_modules/
.env
*.pyc
__pycache__/
.DS_Store
dist/
build/
```

#### .gitattributes
```bash
# Line endings
* text=auto
*.sh text eol=lf
*.bat text eol=crlf

# Binary files
*.png binary
*.jpg binary

# Language specific
*.sql linguist-detectable=true
*.md linguist-documentation
```

### GitHub CLI (gh)
```bash
# Authentication
gh auth login

# Repository
gh repo create <name>
gh repo clone <repo>
gh repo view

# Pull requests
gh pr create
gh pr list
gh pr checkout <number>
gh pr merge <number>
gh pr review <number>

# Issues
gh issue create
gh issue list
gh issue view <number>

# Workflows
gh workflow list
gh workflow run <name>
gh run list
gh run view <id>
```

## Common Scenarios & Solutions

### Merge Conflict Resolution
```bash
# 1. Identify conflicts
git status

# 2. Edit conflicted files
# Look for markers: <<<<<<<, =======, >>>>>>>

# 3. Mark as resolved
git add <file>

# 4. Complete merge
git commit  # Or git merge --continue / git rebase --continue
```

### Undo Pushed Commits (Safe)
```bash
# Create revert commits
git revert HEAD~2..HEAD
git push origin main
```

### Change Last Commit Message
```bash
# Not yet pushed
git commit --amend -m "New message"

# Already pushed (use with caution)
git commit --amend -m "New message"
git push --force-with-lease
```

### Move Commits to Different Branch
```bash
# On wrong branch with uncommitted work
git stash
git checkout correct-branch
git stash pop

# Already committed
git checkout correct-branch
git cherry-pick <commit-hash>
git checkout wrong-branch
git reset --hard HEAD~1
```

## Best Practices

1. **Write meaningful commit messages**: Use imperative mood ("Add feature" not "Added feature")
2. **Commit often**: Small, atomic commits are easier to review and revert
3. **Pull before push**: Avoid conflicts by staying up to date
4. **Use branches**: Never work directly on main/master
5. **Review before committing**: Use `git diff` to check changes
6. **Don't commit secrets**: Use .gitignore and environment variables
7. **Use .gitignore properly**: Keep repository clean
8. **Tag releases**: Version your software properly
9. **Keep history clean**: Use rebase for feature branches (when safe)
10. **Backup important work**: Push regularly to remote

## Pro Tips

- Use `git config --global alias.st status` to create shortcuts
- `git add -p` for interactive staging (stage parts of files)
- `git commit -v` to see diff in commit message editor
- `git reflog` to recover "lost" commits
- `git bisect` to find bug-introducing commits
- Use SSH keys for passwordless authentication
- Learn to read and write good commit messages
- Master interactive rebase for clean history
- Use `git stash` frequently when switching contexts
- Keep a personal cheat sheet of your most-used commands
