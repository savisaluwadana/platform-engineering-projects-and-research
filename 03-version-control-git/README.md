# Version Control with Git

## Theory

### Core Concepts

#### Git Architecture: Distributed Version Control
Git uses a distributed model where every developer has a complete repository history:

- **Working Directory**: Your local file system where you edit files. This is the active workspace containing the current state of your project.
- **Staging Area (Index)**: An intermediate area where changes are prepared before committing. This allows you to craft precise, logical commits by selectively staging related changes.
- **Local Repository**: The `.git` directory containing the complete history of your project, including all commits, branches, and tags. Enables offline work and fast operations.
- **Remote Repository**: A shared repository (on GitHub, GitLab, Bitbucket) that facilitates collaboration. Common remote names include `origin` (default) and `upstream` (for forks).
- **Objects**: Git stores data as objects - blobs (file contents), trees (directory structures), commits (snapshots), and tags (named references).
- **Hash-based Integrity**: Every object is identified by a SHA-1 hash, ensuring data integrity and detecting corruption.

Understanding Git's architecture enables efficient workflows and helps troubleshoot issues when they arise.

#### Basic Operations: The Git Workflow
Master these fundamental commands to work effectively with Git:

- **init**: Initialize a new Git repository in the current directory, creating the `.git` folder.
- **clone**: Create a local copy of a remote repository with complete history.
- **add**: Stage changes for the next commit. Use `git add -p` for interactive staging of partial changes.
- **commit**: Save staged changes to the repository with a descriptive message. Each commit represents a snapshot of your project.
- **push**: Upload local commits to a remote repository, sharing your work with others.
- **pull**: Fetch and merge changes from a remote repository, combining `fetch` and `merge` operations.
- **fetch**: Download objects and refs from a remote without merging, allowing inspection before integration.
- **status**: Show the working tree status, displaying modified, staged, and untracked files.
- **log**: Display commit history with options for formatting and filtering.

These operations form the foundation of daily Git usage.

#### Branching: Parallel Development Made Easy
Branches enable isolated development of features, bug fixes, and experiments:

- **Branch Creation**: `git branch feature-name` creates a new branch; `git checkout -b feature-name` creates and switches to it.
- **Branching Strategy**: Organize work with different branch types:
  - **main/master**: Stable, production-ready code
  - **develop**: Integration branch for features
  - **feature/***: Individual feature development
  - **hotfix/***: Urgent production fixes
  - **release/***: Release preparation and final testing
- **Merging**: Integrate changes from one branch into another. Fast-forward merges move the pointer forward; three-way merges create a merge commit.
- **Rebasing**: Reapply commits on top of another branch, creating a linear history. Useful for feature branches but avoid rebasing shared commits.
- **Cherry-pick**: Apply specific commits from one branch to another, useful for selective backporting.
- **Branch Management**: Regularly delete merged branches to keep repositories clean.

Effective branching enables parallel development while maintaining code stability.

#### Workflows: Collaboration Patterns
Different team workflows suit different project needs:

- **Git Flow**: Feature-rich workflow with main, develop, feature, release, and hotfix branches. Good for scheduled releases and larger teams.
  - Structured release process
  - Clear separation of development and production code
  - Supports multiple versions in production
- **GitHub Flow**: Simplified workflow with main branch and feature branches. Deploy from feature branches after code review.
  - Continuous deployment friendly
  - Simple and easy to understand
  - Fast feedback loops
- **Trunk-Based Development**: Developers commit to a single main branch (trunk) with short-lived feature branches.
  - Encourages continuous integration
  - Reduces merge conflicts
  - Requires strong automated testing
  - Enables true continuous delivery

Choose a workflow that matches your team's size, release cadence, and deployment practices.

#### Advanced Features: Power User Tools
Beyond basics, Git offers powerful features for complex scenarios:

- **Stash**: Temporarily save uncommitted changes without committing. Useful for switching context quickly with `git stash` and `git stash pop`.
- **Reset**: Move HEAD to a different commit, optionally modifying the staging area and working directory.
  - `--soft`: Keep changes staged
  - `--mixed`: Unstage changes (default)
  - `--hard`: Discard all changes (use with caution)
- **Revert**: Create a new commit that undoes previous changes, maintaining history integrity.
- **Reflog**: Reference log showing all HEAD movements, invaluable for recovering lost commits.
- **Bisect**: Binary search through history to find the commit that introduced a bug. Automate with `git bisect run`.
- **Interactive Rebase**: Rewrite commit history by reordering, squashing, editing, or dropping commits. Use for cleaning up feature branch history before merging.
- **Worktrees**: Multiple working directories for the same repository, enabling work on different branches simultaneously.

These advanced features help maintain clean history and recover from mistakes.

#### Collaboration: Working with Others
Git shines in collaborative environments:

- **Pull Requests/Merge Requests**: Propose changes for review before merging. Include description, screenshots, and test results.
- **Code Reviews**: Systematically examine code changes for quality, bugs, and adherence to standards. Provide constructive feedback.
- **Conflict Resolution**: Occurs when changes overlap. Git marks conflicts in files, requiring manual resolution. Use tools like `git mergetool` or IDE integrations.
- **Forking Workflow**: Create personal copies of repositories for contributions. Submit changes via pull requests from your fork.
- **Protected Branches**: Enforce policies like required reviews, status checks, and restricted force pushes on important branches.
- **CODEOWNERS**: Automatically request reviews from designated owners when files change.

Effective collaboration practices maintain code quality while enabling rapid development.

#### Hooks: Automating Git Workflows
Git hooks are scripts that run automatically on specific events:

- **Client-side Hooks**: Run on developer machines
  - **pre-commit**: Run before creating a commit (linting, formatting, tests)
  - **prepare-commit-msg**: Modify commit message before editing
  - **commit-msg**: Validate commit message format
  - **post-commit**: Run after commit is created (notifications)
  - **pre-push**: Run before pushing (integration tests)
- **Server-side Hooks**: Run on Git server
  - **pre-receive**: Run before accepting pushed commits (policy enforcement)
  - **update**: Run for each branch being updated
  - **post-receive**: Run after push is accepted (deployment, notifications)

Hooks enable automation of quality checks, enforcing standards, and triggering CI/CD pipelines.

### Key Skills
1. **Manage code versions effectively**: Use commits to create logical units of change, write clear commit messages following conventions (Conventional Commits), and maintain a clean, understandable history.
2. **Collaborate with team members**: Communicate through pull requests, provide thoughtful code reviews, and use collaboration features effectively to maintain team productivity.
3. **Resolve merge conflicts**: Understand conflict markers, use merge tools effectively, and prevent conflicts through good branching practices and communication.
4. **Implement branching strategies**: Choose and implement appropriate workflows for your team, create consistent branch naming conventions, and manage branch lifecycle effectively.
5. **Automate workflows with hooks**: Set up pre-commit hooks for code quality, implement commit message validation, and trigger automated deployments on specific events.

## Projects

### Beginner Level

#### Project 1: Git Automation Scripts
**Objective**: Create helpful Git automation utilities
- Automatic commit message generator (based on file changes)
- Branch cleanup script (delete merged branches)
- Repository statistics generator
- Backup script for repositories
- Git alias configuration manager

**Learning Outcomes**: Git commands, shell scripting, automation

#### Project 2: Commit Message Validator
**Objective**: Build a pre-commit hook for commit message validation
- Enforce conventional commits format
- Check commit message length
- Prevent commits to protected branches
- Validate ticket/issue references
- Add emojis based on commit type

**Learning Outcomes**: Git hooks, commit standards, validation

#### Project 3: Repository Health Checker
**Objective**: Analyze repository health and quality
- Check for large files
- Detect sensitive data (API keys, passwords)
- Analyze commit history patterns
- Identify unused branches
- Generate health score report

**Learning Outcomes**: Repository management, security scanning, analysis

### Intermediate Level

#### Project 4: Custom Git Workflow Manager
**Objective**: Implement an automated Git workflow
- Automated feature branch creation
- Auto-merge development branches
- Release branch management
- Version tagging automation
- Changelog generation

**Learning Outcomes**: Git workflows, automation, release management

#### Project 5: Git-Based Deployment System
**Objective**: Create a Git-triggered deployment pipeline
- Monitor repository for changes
- Trigger builds on specific branches
- Automated testing before deployment
- Rollback mechanism
- Deployment notifications

**Learning Outcomes**: Git hooks, CI/CD basics, deployment automation

#### Project 6: Monorepo Management Tool
**Objective**: Build tools for managing monorepos
- Selective directory commits
- Independent versioning for packages
- Dependency graph visualization
- Affected package detection
- Parallel build orchestration

**Learning Outcomes**: Monorepo patterns, dependency management, build optimization

### Advanced Level

#### Project 7: Git-Based Configuration Management
**Objective**: Use Git as a source of truth for infrastructure
- GitOps implementation
- Automated sync to target environments
- Drift detection and remediation
- Multi-environment management
- Audit trail and compliance

**Learning Outcomes**: GitOps, infrastructure as code, compliance

#### Project 8: Code Review Analytics Platform
**Objective**: Build insights from code review data
- Pull request metrics collection
- Review time analysis
- Reviewer workload distribution
- Code quality trends
- Automated reviewer suggestions based on expertise

**Learning Outcomes**: Git API, data analysis, team metrics

#### Project 9: Advanced Merge Strategy Tool
**Objective**: Create intelligent merge conflict resolver
- Conflict pattern detection
- Semi-automated conflict resolution
- Merge strategy recommendations
- Historical conflict analysis
- Integration testing before merge

**Learning Outcomes**: Git internals, conflict resolution, automation

## Resources

### Documentation
- [Pro Git Book](https://git-scm.com/book/en/v2)
- [GitHub Docs](https://docs.github.com/)
- [GitLab Docs](https://docs.gitlab.com/)

### Interactive Learning
- [Learn Git Branching](https://learngitbranching.js.org/)
- [Git Exercises](https://gitexercises.fracz.com/)
- [Oh My Git!](https://ohmygit.org/)

### Tools
- Git
- GitHub Desktop
- GitKraken
- Sourcetree
- Lazygit

### Best Practices
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)
- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
