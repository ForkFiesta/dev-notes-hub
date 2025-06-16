# Git - Version Control System

> **Quick Summary**: Git is a distributed version control system for tracking changes in source code. Master branching, merging, and collaboration workflows.

## 🚀 Getting Started

### Initial Setup
```bash
# Configure Git globally
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Check configuration
git config --list
git config user.name
git config user.email

# Initialize a new repository
git init

# Clone an existing repository
git clone https://github.com/username/repository.git
git clone https://github.com/username/repository.git my-folder-name
```

### Basic Workflow
```bash
# Check repository status
git status

# Add files to staging area
git add filename.txt          # Add specific file
git add .                     # Add all files in current directory
git add *.js                  # Add all JavaScript files
git add -A                    # Add all files (including deleted)

# Commit changes
git commit -m "Add new feature"
git commit -am "Add and commit in one step" # For tracked files only

# View commit history
git log
git log --oneline            # Compact view
git log --graph              # Visual graph
git log --author="John"      # Filter by author
git log --since="2 weeks ago" # Filter by date
```

## 🌿 Branching

### Branch Management
```bash
# List branches
git branch                   # Local branches
git branch -r               # Remote branches
git branch -a               # All branches

# Create new branch
git branch feature-name
git checkout -b feature-name # Create and switch
git switch -c feature-name   # Modern syntax

# Switch branches
git checkout main
git switch main             # Modern syntax

# Delete branches
git branch -d feature-name  # Safe delete (merged only)
git branch -D feature-name  # Force delete
git push origin --delete feature-name # Delete remote branch
```

### Branch Workflows
```bash
# Feature branch workflow
git checkout main
git pull origin main
git checkout -b feature/user-authentication
# Make changes and commits
git push -u origin feature/user-authentication

# Merge feature branch
git checkout main
git merge feature/user-authentication
git branch -d feature/user-authentication

# Rebase workflow (cleaner history)
git checkout feature-branch
git rebase main             # Rebase onto main
git checkout main
git merge feature-branch    # Fast-forward merge
```

## 🔄 Remote Repositories

### Remote Management
```bash
# Add remote repository
git remote add origin https://github.com/username/repo.git

# List remotes
git remote -v

# Change remote URL
git remote set-url origin https://github.com/username/new-repo.git

# Remove remote
git remote remove origin
```

### Push and Pull
```bash
# Push changes
git push origin main
git push -u origin feature-branch # Set upstream
git push --all                    # Push all branches
git push --tags                   # Push tags

# Pull changes
git pull origin main
git pull --rebase origin main    # Pull with rebase

# Fetch changes (without merging)
git fetch origin
git fetch --all                  # Fetch all remotes
```

## 🔧 Advanced Operations

### Stashing
```bash
# Stash current changes
git stash
git stash push -m "Work in progress on feature X"

# List stashes
git stash list

# Apply stash
git stash apply              # Apply latest stash
git stash apply stash@{2}    # Apply specific stash
git stash pop               # Apply and remove latest stash

# Drop stash
git stash drop stash@{1}
git stash clear             # Remove all stashes
```

### Undoing Changes
```bash
# Undo working directory changes
git checkout -- filename.txt    # Restore specific file
git checkout .                  # Restore all files

# Unstage files
git reset HEAD filename.txt     # Unstage specific file
git reset HEAD                  # Unstage all files

# Undo commits
git reset --soft HEAD~1         # Undo last commit, keep changes staged
git reset --mixed HEAD~1        # Undo last commit, unstage changes
git reset --hard HEAD~1         # Undo last commit, discard changes

# Revert commits (safe for shared history)
git revert HEAD                 # Revert last commit
git revert abc123               # Revert specific commit
```

### Rewriting History
```bash
# Interactive rebase
git rebase -i HEAD~3            # Edit last 3 commits
# Options: pick, reword, edit, squash, fixup, drop

# Amend last commit
git commit --amend -m "New commit message"
git commit --amend --no-edit    # Add changes to last commit

# Cherry-pick commits
git cherry-pick abc123          # Apply specific commit
git cherry-pick abc123..def456  # Apply range of commits
```

## 🔍 Inspecting Changes

### Viewing Differences
```bash
# View changes
git diff                        # Working directory vs staging
git diff --staged              # Staging vs last commit
git diff HEAD                  # Working directory vs last commit
git diff branch1 branch2       # Compare branches

# View specific file changes
git diff filename.txt
git diff HEAD~1 filename.txt   # Compare with previous commit

# View commit details
git show abc123                # Show specific commit
git show HEAD~2                # Show commit 2 steps back
```

### Searching
```bash
# Search in files
git grep "search term"
git grep -n "search term"      # Show line numbers
git grep -i "search term"      # Case insensitive

# Search in commit messages
git log --grep="bug fix"
git log -S "function_name"     # Search for code changes

# Find when a line was changed
git blame filename.txt
git blame -L 10,20 filename.txt # Specific lines
```

## 🏷️ Tags

### Tag Management
```bash
# Create tags
git tag v1.0.0                 # Lightweight tag
git tag -a v1.0.0 -m "Version 1.0.0" # Annotated tag

# List tags
git tag
git tag -l "v1.*"              # Filter tags

# Push tags
git push origin v1.0.0         # Push specific tag
git push origin --tags         # Push all tags

# Delete tags
git tag -d v1.0.0              # Delete local tag
git push origin --delete v1.0.0 # Delete remote tag
```

## 🔀 Merging and Conflicts

### Merge Strategies
```bash
# Regular merge
git merge feature-branch

# No fast-forward merge (always create merge commit)
git merge --no-ff feature-branch

# Squash merge (combine all commits into one)
git merge --squash feature-branch
git commit -m "Add feature X"
```

### Conflict Resolution
```bash
# When conflicts occur
git status                     # See conflicted files

# Edit conflicted files, then:
git add conflicted-file.txt    # Mark as resolved
git commit                     # Complete the merge

# Abort merge
git merge --abort

# Use merge tools
git mergetool
git config --global merge.tool vimdiff # Set default tool
```

## 🔄 GitHub Workflows

### Pull Requests
```bash
# Create feature branch
git checkout -b feature/new-feature
git push -u origin feature/new-feature

# After creating PR on GitHub, sync with main
git checkout main
git pull origin main
git checkout feature/new-feature
git rebase main               # Or merge main into feature

# After PR is merged
git checkout main
git pull origin main
git branch -d feature/new-feature
```

### GitHub CLI
```bash
# Install GitHub CLI
# macOS: brew install gh
# Windows: winget install GitHub.cli

# Authenticate
gh auth login

# Create repository
gh repo create my-project --public
gh repo create my-project --private

# Clone repository
gh repo clone username/repository

# Create pull request
gh pr create --title "Add new feature" --body "Description"
gh pr list                    # List PRs
gh pr view 123               # View specific PR
gh pr merge 123              # Merge PR
```

## 🛠️ Configuration

### Git Configuration
```bash
# Global configuration
git config --global core.editor "code --wait"
git config --global init.defaultBranch main
git config --global pull.rebase false

# Aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'

# Use aliases
git st                       # Same as git status
git co main                  # Same as git checkout main
```

### .gitignore
```bash
# Create .gitignore file
touch .gitignore

# Common .gitignore patterns
node_modules/
*.log
.env
.DS_Store
dist/
build/
*.tmp
*.swp

# Language-specific ignores
# Node.js
node_modules/
npm-debug.log*
.npm

# Python
__pycache__/
*.py[cod]
.venv/

# IDE
.vscode/
.idea/
*.sublime-*
```

## 🚨 Troubleshooting

### Common Issues
```bash
# Undo last commit (not pushed)
git reset --soft HEAD~1

# Remove file from Git but keep locally
git rm --cached filename.txt

# Fix "detached HEAD" state
git checkout main            # Switch to a branch

# Recover deleted branch
git reflog                   # Find commit hash
git checkout -b recovered-branch abc123

# Clean untracked files
git clean -n                 # Preview what will be deleted
git clean -f                 # Delete untracked files
git clean -fd                # Delete untracked files and directories

# Reset to remote state
git fetch origin
git reset --hard origin/main

# Fix merge conflicts in binary files
git checkout --theirs binary-file.png  # Use their version
git checkout --ours binary-file.png    # Use our version
```

### Recovery Commands
```bash
# View reflog (history of HEAD changes)
git reflog

# Recover lost commits
git checkout abc123          # Go to lost commit
git checkout -b recovery-branch # Create branch from lost commit

# Find lost files
git fsck --lost-found

# Garbage collection
git gc                       # Clean up repository
git gc --aggressive          # More thorough cleanup
```

## 📋 Common Workflows

### Feature Development
```bash
# 1. Start new feature
git checkout main
git pull origin main
git checkout -b feature/user-login

# 2. Develop feature
# Make changes, add commits
git add .
git commit -m "Add login form"
git commit -m "Add validation"

# 3. Push feature branch
git push -u origin feature/user-login

# 4. Create pull request (on GitHub)
# 5. After review and approval, merge
git checkout main
git pull origin main
git branch -d feature/user-login
```

### Hotfix Workflow
```bash
# 1. Create hotfix branch from main
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# 2. Fix the issue
git add .
git commit -m "Fix critical security issue"

# 3. Push and create PR
git push -u origin hotfix/critical-bug

# 4. After merge, clean up
git checkout main
git pull origin main
git branch -d hotfix/critical-bug
```

## ✅ Quick Checklist

- [ ] Configure Git with your name and email
- [ ] Use meaningful commit messages
- [ ] Create feature branches for new work
- [ ] Keep commits small and focused
- [ ] Pull before pushing to avoid conflicts
- [ ] Use .gitignore to exclude unnecessary files
- [ ] Review changes before committing
- [ ] Use pull requests for code review
- [ ] Keep main branch stable
- [ ] Tag releases appropriately

## 🔗 Related Topics
- [[GitHub]] - Git hosting and collaboration
- [[npm-yarn|Package Management]] - Managing dependencies with Git
- [[Node.js]] - Using Git in Node.js projects

---
*Remember: Git is about collaboration and history. Use branches, write good commit messages, and always review before pushing!* 