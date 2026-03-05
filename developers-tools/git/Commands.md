# Git Notes

> The commands you'll actually use. Every command has a comment. Scenarios are step by step. Deep commands at the bottom.

---

## How Git Works

Three zones. Everything in Git is moving changes between them.

```
Working Directory  →  Staging Area  →  Repository
  (edit files)        (git add)        (git commit)
```

- **Working Directory** — your files on disk. Edit freely here.
- **Staging Area** — you decide what goes into the next commit. Not everything you changed has to be included.
- **Repository** — permanent snapshots stored in `.git/`. This is your history.
- **Remote** — the shared copy on GitHub/GitLab that your whole team syncs to.

---

## Core Commands

### `git status`

The most important command. Shows what's changed, what's staged, what's untracked. Run it constantly — before and after everything.

```bash
git status                 # see what's going on in your working directory right now
```

---

### `git add`

Moves changes into the staging area. You're deciding *what* goes into the next commit — not committing yet.

```bash
git add auth.js            # stage one specific file
git add src/               # stage everything inside the src folder
git add .                  # stage all changed files in the current directory
```

> The staging area exists so you can be intentional. You might have changed 5 files but only want to commit 2 of them — `git add` lets you do that.

---

### `git commit`

Saves everything staged as a permanent snapshot. Each commit gets a unique hash and is part of your project's history forever.

```bash
git commit -m "Add password reset email flow"    # commit with a message
```

**What makes a good commit message?**

Write it as a command — *"If applied, this commit will..."*

```
✅  Add rate limiting to the login endpoint
✅  Fix crash when the cart is empty
✅  Remove deprecated Stripe v1 integration

❌  fix
❌  WIP
❌  misc changes
❌  asdfjkl
```

Good messages make history readable, code reviews faster, and debugging possible.

---

### `git log`

Shows commit history. The default output is too noisy — the one-line version is almost always what you want.

```bash
git log --oneline              # one line per commit: hash + message
git log --oneline --graph      # same, but draws the branch tree visually
```

> Set up an alias once so you don't have to type flags every time:
> `git config --global alias.lg "log --oneline --graph"`
> Then just run `git lg`.

---

### `git diff`

Shows exactly what changed, line by line. Use this before staging to review your own work.

```bash
git diff                       # show changes not yet staged
git diff --staged              # show changes that are staged (what will be in the next commit)
```

> Good habit before every commit: `git diff` to review → `git add` → `git diff --staged` to confirm → `git commit`.

---

### `git branch`

Lists, creates, or deletes branches. A branch is just a pointer to a commit — creating one is instant.

```bash
git branch                     # list all local branches
git branch feature/dark-mode   # create a new branch (you stay on the current one)
git branch -d feature/done     # delete a branch (only works if it's already been merged)
```

---

### `git switch`

Moves you to a different branch. Updates all your files to match that branch's state.

```bash
git switch main                # go to the main branch
git switch feature/dark-mode   # go to an existing branch
```

> To create a branch and switch to it, do it in two steps — it's clearer:
> ```bash
> git branch feature/dark-mode   # create the branch
> git switch feature/dark-mode   # then switch to it
> ```

---

### `git fetch`

Downloads everything new from the remote — new branches, new commits. Does **not** change any of your local files or branches. Completely safe to run anytime.

```bash
git fetch origin               # download latest from remote without changing anything local
```

Think of it as: *"let me see what changed remotely before I decide what to do with it."*

---

### `git pull`

Downloads remote changes and merges them into your current branch. The everyday command for "get the latest."

```bash
git pull                       # fetch + merge in one step — use this to update your current branch
```

> `git fetch` exists for when you want to download remote changes but inspect them before merging. For day-to-day updating of your branch, `git pull` is fine.

---

### `git push`

Uploads your local commits to the remote so others can see them.

```bash
git push origin feature/dark-mode    # push a branch to remote for the first time
git push origin main                 # push the main branch
```

> The first time you push a new branch, you need to specify the remote and branch name. After that, just `git push` works.

---

### `git stash`

Temporarily shelves your uncommitted changes. Your working directory goes clean. You can come back to the work later.

```bash
git stash save "wip: user settings form"    # save current changes with a description
git stash list                               # see all saved stashes
git stash pop                                # bring back the most recent stash and remove it from the list
```

> Always give your stash a description. Without one, you'll have no idea what `stash@{2}` is a week later.

---

### `git restore`

Discards changes to a file or removes it from staging. The focused, safe way to undo local changes.

```bash
git restore auth.js             # throw away changes to a file (keeps the file, loses the edits — no undo)
git restore --staged auth.js    # unstage a file (moves it back to working dir, your changes are kept)
```

---

### `git reset`

Moves your branch pointer back to an earlier commit. The three modes control what happens to the changes being "undone."

```bash
git reset --soft HEAD~1         # undo the last commit, keep changes staged (ready to re-commit)
git reset --mixed HEAD~1        # undo the last commit, keep changes in working dir (unstaged)
git reset --hard HEAD~1         # undo the last commit and delete the changes permanently
```

`HEAD~1` = one commit before current. `HEAD~3` = three commits back.

> Use `--soft` or `--mixed` when you committed too early or with the wrong message.
> Use `--hard` only when you're absolutely sure you don't need those changes.

---

### `git revert`

Creates a new commit that is the exact opposite of a previous one. The original commit stays in history — nothing is rewritten. This is the correct tool when a commit is already pushed and shared.

```bash
git log --oneline               # find the hash of the commit you want to undo
git revert <commit-hash>        # create a new "undo" commit on top
git push origin main            # push the revert like any normal commit
```

**`reset` vs `revert` — when to use which:**

| Situation | Tool |
|---|---|
| Commit not pushed yet | `git reset` — rewrite and clean up |
| Commit already pushed | `git revert` — safe, doesn't break teammates |

---

### `git merge`

Brings another branch's commits into your current branch.

```bash
git switch main                 # go to the branch you want to merge INTO
git merge feature/dark-mode     # bring in the feature branch
git push origin main            # push the result
```

**If there's a conflict**, Git pauses and marks the file:

```
<<<<<<< HEAD
your version of the line
=======
the incoming version of the line
>>>>>>> feature/dark-mode
```

Edit the file to what it should be, remove the markers, then:

```bash
git add <the-resolved-file>     # mark it as resolved
git merge --continue            # finish the merge
```

If you want to give up and go back to before the merge started:

```bash
git merge --abort               # cancels the merge, restores everything to before
```

---

### `git rebase`

Moves your commits so they start from the tip of another branch. Instead of a merge commit, your history looks like you branched off the latest code all along.

```bash
git switch feature/my-feature   # go to your feature branch
git fetch origin                 # get the latest remote state
git rebase origin/main           # replay your commits on top of latest main
```

If a conflict comes up mid-rebase:

```bash
git add <resolved-file>          # after fixing the conflict
git rebase --continue            # continue replaying the remaining commits
```

If it gets messy and you want to start over:

```bash
git rebase --abort               # cancels everything, puts you back where you started
```

> Never rebase commits that are already on a shared remote branch. It rewrites history and will break things for anyone who pulled those commits.

---

### `git cherry-pick`

Takes one specific commit from any branch and applies it to your current branch. Useful when you need just one fix without merging a whole branch.

```bash
git log feature/xyz --oneline   # find the hash of the commit you want
git switch main                  # go to where you want to apply it
git cherry-pick <commit-hash>    # apply that one commit here
```

---

## Scenarios

### Start a new feature

```bash
git switch main                          # start from main
git pull                                 # get the latest changes from remote

git branch feature/user-settings         # create the feature branch
git switch feature/user-settings         # move to it

# ... write your code ...

git status                               # check what changed
git add .                                # stage your changes
git commit -m "Add user settings page"  # commit with a clear message
git push origin feature/user-settings   # push your branch to remote
```

---

### Merge a feature branch into main

```bash
git switch main                                    # go to main
git pull                                           # get latest changes from remote

git merge feature/user-settings                    # merge the feature in
git push origin main                               # push to remote

git branch -d feature/user-settings                # delete the local branch
git push origin --delete feature/user-settings     # delete the remote branch
```

---

### Undo a commit that hasn't been pushed yet

```bash
git log --oneline                  # confirm which commit you want to undo

git reset --soft HEAD~1            # undo it, keep changes staged so you can re-commit differently
# OR
git reset --mixed HEAD~1           # undo it, put changes back in working dir so you can review
```

---

### Undo a commit that was already pushed

```bash
git log --oneline                  # find the hash of the commit to undo
git revert <commit-hash>           # creates a new undo-commit (safe, doesn't rewrite history)
git push origin main               # push the revert
```

---

### Undo a merge that was already pushed

A merge commit has two parents. You need to tell Git which parent is the "mainline" — that's `-m 1`, meaning parent 1 (the branch you merged into).

```bash
git log --oneline --graph          # find the merge commit hash
git revert -m 1 <merge-hash>       # create a new commit that undoes the merge
git push origin main               # push it
```

---

### Update your feature branch with latest changes from main

Your feature branch is getting stale while main moves forward.

```bash
git switch feature/my-feature      # go to your branch
git fetch origin                   # download latest remote state (needed before rebase)
git rebase origin/main             # replay your commits on top of latest main
```

If there's a conflict:

```bash
# fix the conflict in the file, then:
git add <resolved-file>            # mark it resolved
git rebase --continue              # keep going

# if it gets too messy, bail out completely:
git rebase --abort                 # puts everything back to how it was before
```

---

### You accidentally committed to main

You made commits on main but they should be on a feature branch.

```bash
git branch feature/my-work         # create a branch pointing at your current commits (this saves them)
git reset --hard origin/main       # roll main back to match remote (your commits move to the branch)
git switch feature/my-work         # go to the branch where your commits now live
git push origin feature/my-work    # push it
```

---

### You're mid-feature when an urgent bug comes in

```bash
git stash save "wip: checkout refactor"   # shelf your current work

git switch main                            # go to main
git pull                                   # get latest

git branch hotfix/null-crash               # create hotfix branch
git switch hotfix/null-crash               # move to it

# ... fix the bug ...

git add .
git commit -m "Fix null crash when cart is empty"
git push origin hotfix/null-crash          # push and open a PR

# after the hotfix is merged, come back:
git switch feature/checkout-refactor       # go back to your feature branch
git stash pop                              # restore your shelved work
```

---

### Cherry-pick a fix from another branch

You fixed a bug on a feature branch. Main needs that fix now without merging the whole feature.

```bash
git log feature/xyz --oneline      # find the hash of the fix commit
git switch main                    # go to main
git cherry-pick <commit-hash>      # apply just that one commit
git push origin main               # push it
```

---

### Squash messy commits before opening a PR

You have commits like `wip`, `fix typo`, `ok this works`, `lol` and want one clean commit.

```bash
git log --oneline                  # count how many commits to squash, e.g. last 5
git rebase -i HEAD~5               # open interactive rebase for the last 5 commits
```

In the editor that opens, leave the top line as `pick` and change all others to `squash`. Save. Git will ask you to write one final combined commit message.

---

### Recover something you accidentally deleted

`git reflog` is a log of every place your HEAD has been — even after destructive commands.

```bash
git reflog                             # find the hash from just before the mistake
git branch recovery <hash>             # create a branch pointing there
git switch recovery                    # go to it and verify your work is there
```

---

## What Not To Do

**Don't `git push --force` on a shared branch.**
It overwrites the remote with your local copy, silently deleting any commits others pushed. If you need to force push after a rebase, use `git push --force-with-lease origin <branch>` instead — it refuses if someone else has pushed since your last fetch.

**Don't rebase commits already on a shared remote branch.**
Rebase rewrites history. If teammates have pulled those commits and you push rebased versions, their histories diverge and they'll hit painful conflicts. Rebase only your local, unshared commits.

**Don't commit secrets or credentials.**
Once pushed, a secret lives in history forever — even if the next commit deletes the file. Always add `.env` to `.gitignore` before the first commit. If it happens: rotate the credential immediately, then scrub the history with `git filter-repo`.

**Don't commit directly to main.**
Branch → write → PR → merge. Even on solo projects, it's worth the habit.

**Don't `git reset --hard` without checking first.**
It permanently deletes your working directory changes. Run `git status` and `git diff` first to confirm you won't lose anything. If you already ran it and regret it, try `git reflog` immediately.

**Don't make huge vague commits.**
"misc fixes" touching 15 files is impossible to review, revert, or debug. Commit one logical unit of change at a time.

---

## Deeper Commands (Quick Reference)

Commands you'll need occasionally. One-line explanation each.

| Command | What it does |
|---|---|
| `git show <hash>` | Show the full diff and message of one specific commit |
| `git blame <file>` | Show who last changed each line of a file and in which commit |
| `git log --oneline -- <file>` | See the full history of one specific file |
| `git reflog` | Every place HEAD has been — your recovery tool after destructive commands |
| `git remote -v` | List all remotes and their URLs |
| `git remote add upstream <url>` | Add the original repo as a remote (used in fork workflows) |
| `git fetch --prune` | Remove local references to remote branches that have been deleted |
| `git clean -n` | Dry run — preview what files `git clean` would delete before actually doing it |
| `git clean -fd` | Delete all untracked files and folders from the working directory |
| `git restore --source=<hash> <file>` | Restore one file to how it looked at a specific past commit |
| `git rm --cached <file>` | Stop tracking a file without deleting it from disk (good for fixing .gitignore mistakes) |
| `git branch --merged main` | List all branches already merged into main — safe to delete these |
| `git diff main...feature` | Show all changes in a feature branch since it diverged from main |
| `git log main..feature --oneline` | Show commits in feature that haven't been merged into main yet |
| `git tag -a v1.0.0 -m "Release 1.0.0"` | Tag a specific commit as a release version |
| `git push origin --tags` | Push all tags to remote (tags don't push automatically) |
| `git bisect start` | Start a binary search through history to find which commit introduced a bug |
| `git cherry-pick <h1>..<h2>` | Apply a range of commits from another branch onto the current one |
| `git commit --amend --no-edit` | Add staged changes to the last commit without changing its message |
| `git switch --detach <hash>` | Temporarily look at an old commit without being on any branch |
