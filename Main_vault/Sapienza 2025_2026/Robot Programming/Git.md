**Tags:** #Git #VersionControl #VCS #RobotProgramming #Linux
![[rp_01b_git.pdf]]

---

## 1. Introduction to Git

### Origins & Goals
* **Creator:** Created by Linus Torvalds (creator of Linux) in 2005 to support Linux kernel development.
* **Key Goals:**
    * **Speed:** Optimized for performance.
    * **Non-linear Development:** Excellent support for thousands of parallel branches.
    * **Distributed:** Fully distributed architecture; every client has a complete history.
    * **Scalability:** Capable of handling massive projects (like the Linux kernel).

### Centralized vs. Distributed VCS

**Centralized VCS (e.g., SVN, CVS):**
* Relies on a single central server that holds the "official" version history.
* Users check out files to their local machine.
* **Risk:** If the server goes down, you cannot version changes. If the server disk fails without backup, the entire history is lost.

**Distributed VCS (Git):**
* Clients do not just check out the latest snapshot; they **clone** the entire repository.
* Every clone is a full backup of the server.
* Operations (commit, log, diff) are performed **locally** and offline.
* Network interaction is only needed when sharing changes (push/pull).



---

## 2. Core Concepts

### Snapshots vs. Deltas
* **Traditional Systems:** Store changes as a list of file-based changes (deltas) over time.
* **Git:** Thinks of data like a **stream of snapshots**.
    * Every time you commit, Git takes a picture of what all your files look like at that moment.
    * To be efficient, if a file hasn't changed, Git doesn't store the file again; it just stores a reference to the previous identical file.

### The Three States
Git projects consist of three main sections:
1.  **Working Directory:** The actual files on your disk that you are editing.
2.  **Staging Area (Index):** A file that stores information about what will go into your next commit.
3.  **Git Directory (Repository):** Where Git stores the metadata and object database (the history).

**The Workflow:**
1.  Modify files in the **Working Directory**.
2.  Stage the files, adding snapshots of them to the **Staging Area**.
3.  Commit, which takes the files as they are in the Staging Area and stores that snapshot permanently to the **Git Directory**.



### Data Integrity (SHA-1)
* Git uses a **[[../Cybersecurity/Low Level/Cryptographic Hash Function|SHA-1 hash]]** (a 40-character hexadecimal string) to identify everything.
* **Integrity:** It is impossible to change the contents of any file or directory without Git knowing about it.
* Commits are referred to by this hash (e.g., `24b9da6...`) rather than sequential numbers (v1, v2).

---

## 3. Basic Commands

### First-Time Setup
Configuring your identity (stored in `~/.gitconfig`):
```bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
git config --global core.editor nano
```

### Creating Repositories
**Local Initialization:**
```bash
mkdir myproject
cd myproject
git init  # Creates the .git subdirectory
```

**Cloning (Downloading):**
```bash
git clone <url> [optional_directory_name]
```

### Recording Changes
**Checking Status:**
```bash
git status      # Detailed view
git status -s   # Short view (e.g., 'M' for modified, '??' for untracked)
```

**Staging Files:**
```bash
git add filename.c  # Stages a specific file
git add .           # Stages all changes in the directory
```

**Viewing Changes:**
```bash
git diff            # Shows unstaged changes (Working Dir vs Staging)
git diff --staged   # Shows staged changes (Staging vs Repo)
```

**Committing:**
```bash
git commit -m "Fix buffer overflow bug"
```

**Undoing:**
```bash
# Unstage a file (keep changes in working dir)
git reset HEAD filename

# Discard changes (revert file to last commit state) - DANGEROUS
git checkout -- filename
```

---

## 4. Branching and Merging

Branching is Git's "killer feature". It allows you to diverge from the main line of development and continue work without messing with that main line.

### Branch Commands
```bash
# Create a branch
git branch testing

# Switch to a branch
git checkout testing

# Create and switch in one step
git checkout -b testing

# List branches (* denotes current branch)
git branch

# Delete a branch
git branch -d testing
```

### Merging
To merge changes from `testing` into `master`:
```bash
git checkout master
git merge testing
```

### Conflict Resolution
If the same part of a file was changed differently in the two branches being merged, Git creates a **Merge Conflict**.
* Git pauses the merge and modifies the file to show both versions.
* You see markers like:
```text
<<<<<<< HEAD
code in master
=======
code in testing
>>>>>>> testing
```
* **Resolution:** You must manually edit the file to remove the markers and choose the correct code. Then run `git add` and `git commit` to finalize the merge.

---

## 5. Remote Repositories

### Hosting Services
While you can use Git locally, most projects use a remote host:
* **GitHub**
* **GitLab**
* **Bitbucket**

### Remote Commands
```bash
# List remotes
git remote -v

# Add a remote
git remote add origin https://github.com/user/repo.git

# Send changes to remote
git push origin master

# Retrieve changes from remote
git pull origin master
```

### Self-Hosted Git (Home Server)
You don't strictly need GitHub. You can set up a private server using just SSH.

**On the Server:**
```bash
mkdir my_repo.git
cd my_repo.git
git init --bare   # Creates a repo without a working directory
```

**On the Client:**
```bash
# Link local repo to your private server
git remote add origin user@your_home_ip:path/to/my_repo.git
git push origin master
```

---

## 6. Resources
* **Pro Git Book:** [http://git-scm.com/book](http://git-scm.com/book) (The definitive guide).
* **Git Reference:** [http://gitref.org](http://gitref.org)
* **Visual Guide:** [Git for Computer Scientists](http://eagain.net/articles/git-for-computer-scientists/)