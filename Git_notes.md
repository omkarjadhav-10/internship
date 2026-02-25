# Git and GitHub Fundamentals and Operations

## 1. Fundamental definitions and concepts

Understanding the distinction between tools and services is the first step in mastering version control.

- Git vs. GitHub: Git is the local software used to track file changes and manage history. GitHub is an online service used to host Git repositories. While GitHub is currently dominant, alternatives include GitLab, Bitbucket, Azure Repos, and Gitea.
- Version Control System (VCS): Functions like checkpoints in a video game. It allows developers to revert to previous states of the software if new code introduces errors.
- Repository (Repo): A specialized directory tracked by Git. It contains a hidden `.git` folder that acts as a database for the project's history.
- The HEAD: A pointer that indicates the current branch or commit the developer is currently working on.

---

## 2. The standard workflow

The daily operation of Git follows a logical progression from local changes to repository storage.

### The lifecycle of a change

1. Working directory: Where files are actively edited.
2. Staging area: A middle ground where changes are prepared. Commands like `git add <filename>` move files here.
3. Repository: The permanent record. Commands like `git commit -m "message"` move staged changes into the local database.
4. Remote: The online version. `git push` sends local commits to services like GitHub.

### Core commands

| Command | Purpose |
| --- | --- |
| `git init` | Initializes a new Git repository in the current folder. |
| `git status` | Displays the state of the working directory and staging area. |
| `git add .` | Adds all modified files to the staging area. |
| `git commit -m` | Saves the staged snapshot with a descriptive message. |
| `git log` | Shows the commit history. Use `--oneline` for a concise view. |

### Professional commit standards

- Atomic commits: Commit often and in small, logical units of work (e.g., adding a footer) rather than batching thousands of unrelated changes.
- Imperative mood: Use the present tense to "give orders" to the code base (e.g., "Add marketing section" instead of "Added marketing section").

---

## 3. Branching, merging, and conflict resolution

Branching allows for "alternate timelines," enabling developers to work on features or bug fixes without destabilizing the main code base.

- Branch management:
  - `git branch <name>` creates a new branch.
  - `git switch -c <name>` or `git checkout -b <name>` creates and switches to a new branch simultaneously.
- Merging: The process of integrating changes from one branch into another.
  - Fast-forward merge: Occurs when the main branch hasn't moved since the feature branch was created; Git simply moves the pointer forward.
  - Non-fast-forward merge: Requires a merge commit when both branches have diverged with unique changes.
- Merge conflicts: Occur when Git cannot automatically determine which changes to keep.
  - Manual resolution: Developers must open the conflicted file, identify the markers (e.g., `<<<< HEAD` vs `>>>> branch-name`), and choose which code to retain before completing the commit.

---

## 4. Internal architecture: the three musketeers

Git manages data through an object-based system. Every piece of information is stored as an object identified by a unique SHA (Secure Hash Algorithm) ID.

1. Blobs (Binary Large Objects): These represent the actual content of the files.
2. Tree objects: These act like directories, mapping names to blobs or other trees. They store file modes and names.
3. Commit objects: These contain references to a tree object (the snapshot), the parent commit(s), and metadata like author, committer, and the commit message.

### Command categories

- Porcelain commands: High-level, user-friendly commands used for daily tasks (`git add`, `git commit`).
- Plumbing commands: Low-level commands that manipulate the internal object database directly (`git hash-object`, `git update-index`, `git write-tree`). These are rarely used by developers but underpin the software's functionality.

---

## 5. Advanced utilities and maintenance

Git provides several tools for managing temporary work and keeping the repository clean.

- Git stash: Used to temporarily "shelve" uncommitted changes to switch branches without committing unfinished work.
  - `git stash` saves changes.
  - `git stash pop` applies the most recent stash and removes it from the list.
- Git ignore (`.gitignore`): A file listing patterns for files Git should not track, such as environment variables (`.env`), API secrets, or large dependency folders (`node_modules`).
- Git tags: Act as "sticky notes" for specific commits, often used to mark release versions (e.g., `v1.0`).
- Git reflog: A record of every movement the HEAD has made. It is a critical "safety net" for recovering lost commits or undoing accidental resets.

---

## 6. Remote collaboration and GitHub integration

Transitioning from local to remote work requires specific security and configuration steps.

### Secure communication (SSH)

GitHub has deprecated password-based authentication for command-line operations. Developers must use SSH keys:

1. Generate an SSH key pair on the local machine (`ssh-keygen`).
2. Add the public key to the GitHub profile settings.
3. This establishes a verified system handshake, removing the need for repeated logins.

### Synchronizing repositories

- Remote origin: A URL alias (usually named `origin`) pointing to the GitHub repository. Set using `git remote add origin <URL>`.
- Upstream tracking: Using `git push -u origin main` links the local branch to the remote branch, allowing for simplified `git push` and `git pull` operations in the future.
- Cloning: Downloading an existing repository from GitHub to a local machine using `git clone`.

---

## 7. Open source contribution ethics

Open source is the practice of contributing to public code bases, but it requires a disciplined approach to be effective.

### The professional workflow

1. Talk first: Before writing code, communicate with maintainers via Discord or issue trackers.
2. Forking: Create a personal copy of the repository on GitHub.
3. Value add: Focus on fixing legitimate bugs or adding features. Avoid spamming projects with minor README or grammar fixes, which can frustrate maintainers.
4. Pull requests (PRs): Submit your changes back to the original repository with a detailed description of the contribution.

### Best practices for PRs

- Provide a clear summary of changes.
- Explain how the change adds value or fixes a problem.
- Maintain high code quality standards consistent with the existing project.
