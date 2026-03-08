# Git Practice Notes — Abdul Basit Bhat

---
Local > Stage > Commit > Push > Merge

## 1. Initial Setup

```bash
git config --global user.name "Abdul Basit Bhat"
git config --global user.email "basit316316@gmail.com"
git config --list          # verify config
```

---

## 2. Create a Repository

```bash
git init                   # initialize local repo
```
> Also create a repository on GitHub manually via the website.

---

## 3. Basic Add → Commit Workflow

```bash
# Check what has changed
git status

# Stage all files, or a specific file
git add .
git add <filename>

# Commit with a message
git commit -m "commit message"
```

---

## 4. Undoing a Commit with `git revert`

`git revert` creates a **new commit** that undoes a previous one. History is preserved.

```bash
git log                            # find the commit hash to revert
git revert <commit-hash>           # a text editor opens — add a message, then save & close
```

> Press `q` to exit the `git log` screen.

---

## 5. Staging & Unstaging Files

```bash
# Remove a file from staging area (keep changes in working directory)
git restore --staged practiceFiles/file1.txt

# Discard changes in the working directory entirely
git restore practiceFiles/file1.txt
```

---

## 6. Connecting to GitHub & Pushing

```bash
git remote add origin https://github.com/abdulbasitbhat/git-practice.git

# Push the master branch and set upstream tracking
git push -u origin master
```

> `origin` is just an alias for the remote URL.

---

## 7. Working with Branches

```bash
git branch feature/file2           # create new branch
git checkout feature/file2         # switch to it

# Create files, then:
git add .
git commit -m "File2"

# Push the new branch to remote
git push -u origin feature/file2
git push                           # subsequent pushes (upstream already set)
```

---

## 8. Reverting a Single Bad Push (after push)

```bash
git log                            # get the hash of the bad commit
git revert 355f4566b7e96c82acf8b0a3020eff13232ec0cd
# Editor opens → add a message → save & close
git push
```

---

## 9. Reverting a Range of Commits

Use this when multiple sequential commits are bad.

```bash
# Syntax: revert from oldest to newest (^ means "include this commit")
git revert <oldest-hash>^..<newest-hash>

# Example:
git revert d2a396e57e677a83300bd8596cc4f3d7d7ef7d3c^..284e09be7aac4c7a27889b79c14b4d0807db2500
```

### Handling Merge Conflicts During Revert

If a commit outside the range (e.g. "try 5") conflicts with the revert:

```
<<<<<<< HEAD
Change 5
Change 6
Change 7
=======
Change 5
>>>>>>> parent of 284e09b (Multi commit bad push revert try 4)
```

**Resolution:** Edit the file to keep the correct version, then:

```bash
git status
git add .
git commit -m "merge conflict resolve"
git push
```

---

## 10. Merging a Feature Branch into Master

```bash
git checkout master
git pull origin master             # fetch latest remote master
git merge feature/file2
git push origin master
```

> If master had no new commits ahead of `feature/file2`, Git performs a **fast-forward merge** — it simply moves the master pointer forward. No merge commit is created.

---

## 11. Undoing a Merge with `git reflog` + `git reset`

```bash
git log                            # confirm the unwanted merge
git reflog show master             # view master's history including the merge
git reset --hard HEAD@{n}          # replace n with the state just BEFORE the merge line
```

### Force-Pushing the Reset (overriding remote)

Because the remote is now ahead of your local branch, a normal push will be rejected:

```bash
git push                           # ❌ Error: remote is ahead
git push --force-with-lease origin master   # ✅ Safe force push
```

> `--force-with-lease` is safer than `--force`: it only pushes if no one else has pushed new commits to the remote in the meantime.

---

## Quick Reference Cheat Sheet

| Goal | Command |
|------|---------|
| Check status | `git status` |
| Stage all | `git add .` |
| Stage one file | `git add <file>` |
| Unstage (keep changes) | `git restore --staged <file>` |
| Discard changes | `git restore <file>` |
| Commit | `git commit -m "message"` |
| View history | `git log` |
| Revert one commit | `git revert <hash>` |
| Revert a range | `git revert <old>^..<new>` |
| Create branch | `git branch <name>` |
| Switch branch | `git checkout <name>` |
| Push new branch | `git push -u origin <branch>` |
| Merge branch | `git merge <branch>` |
| Undo merge (local) | `git reset --hard HEAD@{n}` |
| Force push safely | `git push --force-with-lease origin <branch>` |
