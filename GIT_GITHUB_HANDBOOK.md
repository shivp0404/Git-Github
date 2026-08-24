# Git & GitHub — Complete Handbook (YouTube Tutorial Script)

> How to use this doc: each section is a chunk you can narrate on camera. Every command block is meant to be typed live in the VS Code terminal. "🎬 Demo" = do this on screen. "⚠️ Mistake" = a beginner error to call out. "💡 Tip" = the one-liner viewers should remember.

---

## 0. Before You Record — One-Time Setup

🎬 Demo in VS Code terminal:

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "you@email.com"
git config --global init.defaultBranch main
git config --list
```

⚠️ **Mistake:** Forgetting `--global` and setting name/email per-repo only, then wondering why commits show the wrong author on a new project.

💡 **Tip:** `user.name`/`user.email` just label commits — they don't need to match your GitHub login, but they should match your GitHub account's verified email if you want commits to show your avatar.

### Authentication (do this once, off-camera or edited in)
GitHub removed password auth for git operations. Two options:
- **SSH key** (recommended, no repeated logins):
  ```bash
  ssh-keygen -t ed25519 -C "you@email.com"
  cat ~/.ssh/id_ed25519.pub   # copy this into GitHub → Settings → SSH Keys
  ssh -T git@github.com       # test
  ```
- **Personal Access Token (PAT)**: GitHub → Settings → Developer settings → Tokens. Use the token as your password when prompted, or store it with `git config --global credential.helper store`.

---

## 1. What Is Git & GitHub

| | Git | GitHub |
|---|---|---|
| What | Distributed **version control system** (a tool) | Cloud platform that **hosts** Git repositories |
| Runs | On your machine, works offline | On a server, needs internet |
| Job | Tracks every change to your code, locally | Stores repos remotely, adds collaboration (PRs, issues, Actions) |

**Why developers use Git:**
- Track every change in the project (full history, not just "latest version").
- Work on new features without disturbing main code (branches).
- Collaborate with teammates without overwriting each other.
- Revert to any previous version if something breaks.
- Work fully offline — no server dependency.

**Version Control vs Source Control (old-style)**

| Version Control (Git) | Old Source Control (CVS/SVN) |
|---|---|
| Tracks every change (full history) | Only the latest version is available |
| Can revert to any previous version | Cannot go back |
| Works offline | Requires constant server connection |

**Centralized vs Distributed**
- **Centralized** (e.g. SVN): one central server holds the code. Server down → nobody can work.
- **Distributed** (e.g. Git): every developer has the *entire* repository locally. Work continues offline; push/pull to sync later.

🎬 Demo: draw/point at this flow on screen —
```
Developer  →  Git (local machine)  →  GitHub (remote repo)  →  Team
```

---

## 2. Git Architecture — The 4 Areas

Git moves your code through four stages. Understanding this is the #1 thing that makes the rest of Git click.

```
Working Directory  --(git add)-->  Staging Area  --(git commit)-->  Local Repository  --(git push)-->  Remote Repository (GitHub)
        ↑                                                                                                        │
        └───────────────────────────────────────(git pull)─────────────────────────────────────────────────────┘
```

1. **Working Directory** — the actual folder on disk. Where you create/edit/delete files. Everything starts here (untracked or modified).
2. **Staging Area (Index)** — a holding area. `git add` tells Git *which* changes you want in the next commit. Lets you commit only part of your changes.
3. **Local Repository** — the `.git` folder. `git commit` saves a permanent snapshot here with a unique SHA hash.
4. **Remote Repository (GitHub)** — the cloud copy. `git push` uploads your commits; `git pull` downloads others' commits.

💡 **Key points to say out loud on camera:**
- A commit is a **snapshot** of your project at a point in time, not a "diff" (Git stores snapshots, not deltas).
- Every commit has a unique SHA hash — you can always go back to any of them.
- Git works fully offline. You can commit without internet; you only need it for push/pull.

🎬 Demo:
```bash
git init demo-repo
cd demo-repo
echo "hello" > index.html
git status          # untracked
git add index.html
git status           # staged
git commit -m "Initial commit"
git status           # clean, in local repo
```

⚠️ **Mistake:** Running `git commit` without `git add` first and getting "nothing to commit" — beginners forget staging is a separate step.

---

## 3. Essential Git Commands (the ones you'll use 90% of the time)

| Command | Syntax | What it does |
|---|---|---|
| `git clone` | `git clone <url>` | Downloads a remote repo to your machine |
| `git init` | `git init` | Turns the current folder into a Git repo |
| `git status` | `git status` | Shows what's changed / staged / untracked |
| `git add` | `git add <file>` / `git add .` | Stages a file (or everything) |
| `git commit` | `git commit -m "message"` | Saves staged changes to local repo |
| `git push` | `git push origin <branch>` | Uploads commits to remote |
| `git pull` | `git pull origin <branch>` | Fetch + merge from remote |
| `git fetch` | `git fetch origin` | Downloads remote changes but does **not** merge |
| `git log` | `git log` / `git log --oneline --graph --all` | Commit history |
| `git diff` | `git diff` | Unstaged changes vs last commit |
| `git diff --staged` | `git diff --staged` | Staged changes vs last commit |
| `git restore` | `git restore <file>` / `git restore --staged <file>` | Discard working-dir changes / unstage a file |
| `git rm` | `git rm <file>` | Removes file from working dir + stages the removal |
| `git mv` | `git mv old.txt new.txt` | Renames/moves a file (staged automatically) |
| `git show` | `git show <commit-id>` | Full details of one commit |

**Helpful shortcuts to demo:**
```bash
git config --global alias.st status
git branch -a                     # all local branches
git remote -v                     # show remote URLs
git log --oneline --graph --all   # visual history
```

⚠️ **Mistake:** `git add .` blindly adding files you didn't mean to commit (secrets, `node_modules`, build output) — this is why `.gitignore` (Section 9) matters *before* your first commit.

💡 **Tip:** `git fetch` is safe to run anytime — it never changes your working files. `git pull` = `fetch` + `merge`, so it *can* change your files.

---

## 4. Branching & Merging

**What is a branch?** An independent line of development. Lets you build a feature, fix a bug, or experiment without touching the stable `main` code.

**Common branch naming conventions:**
- `main` / `master` — production-ready code
- `feature/*` — new features (e.g. `feature/login`)
- `release/*` — preparing a release
- `hotfix/*` — urgent production bug fixes
- `develop` — integration branch where features come together before release

```
                main
                 │
   ┌─────────────┼─────────────┐
feature/login  feature/payment  hotfix/bug-123
```
All branches are created from `main` and merged back once work is done.

**Commands:**
```bash
git branch                  # list local branches
git branch <name>           # create a branch
git checkout <name>         # switch to it        (old syntax)
git switch <name>           # switch to it        (new syntax, prefer this)
git switch -c <name>        # create + switch in one step
git branch -d <name>        # delete a merged branch
git branch -D <name>        # force-delete an unmerged branch
```

**Merging:**
```bash
git checkout main           # go to target branch
git merge feature/login     # bring feature/login into main
```

**Merge conflicts** happen when Git can't auto-decide which change to keep (both branches edited the same lines).
1. Git marks the conflicting file with `<<<<<<<`, `=======`, `>>>>>>>` markers.
2. Open the file, manually pick/edit the correct content, delete the markers.
3. `git add <file>` → `git commit`.

🎬 Demo: create a real conflict live —
```bash
git switch -c feature/demo
echo "feature line" >> index.html
git commit -am "feature change"
git switch main
echo "main line" >> index.html
git commit -am "main change"
git merge feature/demo      # conflict!
```
Open `index.html`, resolve, then:
```bash
git add index.html
git commit
```

**Best practices:**
- Create a new branch for every feature/bug fix.
- Keep branches small and focused; write meaningful commit messages.
- Never commit directly to `main`.
- Delete branches after merging.
- Review via Pull Requests before merging.

💡 **Golden Rule:** Commit to your branch, merge to `main`.

---

## 5. Merge vs Rebase vs Cherry-pick

Both **merge** and **rebase** combine changes from one branch into another — differently.

**Merge** — combines branches by creating a new merge commit. Preserves complete history. Safe on shared branches.
```
Before:        A → B → C          (main)
                     \
                      D → E        (feature)

After merge:   A → B → C → F      (F = merge commit)
                     \       ↗
                      D → E
```
```bash
git checkout main
git merge feature      # merges feature into main
```

**Rebase** — moves/"replays" your branch's commits on top of another branch. Creates a linear history (no merge commit). **Rewrites history — use with caution.**
```
Before:        A → B → C          (main)
                     \
                      D → E        (feature)

After rebase:  A → B → C → D' → E'   (commits replayed on top, new IDs)
```
```bash
git checkout feature
git rebase main        # reapplies feature commits on top of main
```

| Aspect | Merge | Rebase |
|---|---|---|
| History | Preserves history (extra merge commit) | Creates linear/clean history |
| Commit IDs | Unchanged | Changed (new SHAs) |
| Safety | Safe on shared branches | **Not safe** on shared/pushed branches |
| Use case | Team/shared branches | Cleaning up your own feature branch before a PR |

⚠️ **Important caution:**
- Rebase changes history by creating new commits with new hashes.
- If you rebase a branch others have already pulled, you'll cause conflicts for them.
- **Rule of thumb: rebase your private/local work, merge shared/public work. Never rebase what you've already pushed** (unless you know exactly why and force-push with `--force-with-lease`, and the team agrees).

**Cherry-pick** — apply one specific commit from one branch onto another.
```bash
git checkout main
git cherry-pick <commit-hash-of-Q>   # applies just that one commit onto main
```
Use case: you need *one* bugfix commit from a feature branch without merging the whole branch.

---

## 6. GitHub Workflow & Collaboration

**Key terms:**
- **Repository (Repo)** — project folder + full history, hosted on GitHub.
- **Fork** — your own copy of *someone else's* repository.
- **Clone** — a copy of a repository onto your local machine.
- **Pull Request (PR)** — a request to merge your changes into another branch.
- **Issue** — tracks tasks, bugs, or feature requests.

**Typical open-source / team contribution workflow:**
```
1. Fork → 2. Clone → 3. Branch → 4. Commit → 5. Push → 6. PR → 7. Review → 8. Merge
```
1. **Fork** — create your own GitHub copy of the repo.
2. **Clone** — `git clone <your-fork-url>` to your machine.
3. **Branch** — `git switch -c feature/xyz`.
4. **Make changes** — edit/add/fix code.
5. **Commit** — `git commit -m "meaningful message"`.
6. **Push** — `git push origin feature/xyz` to *your* fork.
7. **Pull Request** — open a PR from your branch → the original repo's `main`.
8. **Code Review** — teammates comment, request changes.
9. **Merge** — once approved, PR is merged into `main`.
10. **Delete branch** — clean up after merge.

*(If you're a direct collaborator on a repo — not an outside contributor — you skip the fork step and just clone + branch directly.)*

**Why Pull Requests?**
- Discuss and review changes before they land.
- Maintain code quality, catch bugs early.
- Track all proposed changes in one place.
- The safe, standard way to contribute to shared code.

**Best practices:**
- Create small, focused PRs (easier to review).
- Write clear commit messages and PR descriptions.
- Keep your branch updated with `main` before opening/merging a PR (`git pull` or rebase).
- Review before merging; delete branches after merge.

🎬 Demo: create a real PR on GitHub live — push a branch, open github.com, click "Compare & pull request," fill description, merge.

---

## 7. Undo Operations — How to Fix Mistakes

This is the section beginners need most — mistakes *will* happen; here's how to recover.

| Command | Undoes | Scope | Safe on shared/pushed repo? |
|---|---|---|---|
| `git reset` | Commits, by moving HEAD | Commits | ⚠️ No, if used improperly |
| `git revert` | Commits, via a new commit | Commits | ✅ Yes (safe) |
| `git restore` | Changes in working dir/staging | Files | ✅ Yes |
| `git stash` | Temporarily saves changes without committing | Working changes | ✅ Yes |

### `git reset` — moves HEAD to a previous commit
```bash
git reset --soft <commit-hash>    # keeps changes staged
git reset --mixed <commit-hash>   # keeps changes in working dir (default)
git reset --hard <commit-hash>    # DISCARDS all changes — destructive
```

| Feature | Soft Reset | Hard Reset |
|---|---|---|
| Moves HEAD | Yes | Yes |
| Keeps changes staged | Yes | No |
| Keeps changes in working dir | Yes | No |

⚠️ **Mistake:** Running `git reset --hard` on a branch that's already pushed/shared — it silently deletes work with no easy recovery for teammates who already based work on it. Use `git revert` on shared history instead.

### `git revert` — undo via a *new* commit (safe, preserves history)
```bash
git revert <commit-hash>
```
Creates a new commit that undoes the changes of a previous one — nothing is deleted, so it's safe on shared branches.

### `git restore` — undo in working directory/staging (not commits)
```bash
git restore <file>            # discard uncommitted changes in working dir
git restore --staged <file>   # unstage a file (keep the edits)
```

### `git stash` — temporarily shelve changes without committing
```bash
git stash              # save + clean working dir
git stash apply        # reapply (keeps it in the stash list)
git stash pop          # reapply + remove from stash list
git stash list          # see all stashes
git stash drop           # delete a stash
```
**Stash workflow:** working on something, need to switch branches urgently → `git stash` → do other work → come back → `git stash pop` → continue.

**Best practices recap:**
- Commit frequently with meaningful messages.
- Pull latest changes before starting new work.
- Keep branches small and focused.
- Never commit directly to `main`.
- Review code via Pull Requests.
- Never commit sensitive info (passwords, API keys) — use `.gitignore` and secrets managers.

---

## 8. `.gitignore`, Commit Messages, and Sensitive Data

**Common `.gitignore` entries:**
```
.class
.log
.env
node_modules/
target/
.jar
.war
.idea/
.vscode/
build/
```
💡 **Tip:** Add `.gitignore` *before* your first commit. Once a file is tracked, adding it to `.gitignore` later won't stop Git from tracking it — you'd need `git rm --cached <file>` first.

⚠️ **Mistake:** Committing `.env`/API keys, then adding it to `.gitignore` afterward — the secret is still in history forever unless you rewrite history (see `git filter-repo` / BFG, an advanced/rare fix). Best fix: **rotate the leaked key immediately.**

**Commit message format (Conventional Commits style):**
```
<type>(<scope>): <subject>

[optional body]
[optional footer]
```
Example:
```
feat(auth): add login API
```
Common types:
- `feat` — new feature
- `fix` — bug fix
- `docs` — documentation
- `chore` — maintenance / tooling

---

## 9. Tags & Reflog (the "important but often skipped" commands)

**`git tag`** — a fixed reference to one specific commit (used for releases, e.g. `v1.0.0`). Unlike a branch (which moves as you commit), a tag never moves.
```bash
git tag v1.0.0                          # lightweight tag on current commit
git tag -a v1.0.0 -m "Release 1.0.0"    # annotated tag (recommended, has message/author)
git push origin v1.0.0                  # tags don't push automatically
git push origin --tags                  # push all tags
git tag -d v1.0.0                       # delete a local tag
```

**`git reflog`** — your safety net. Logs every place HEAD has pointed, even after a `reset --hard` or a "lost" commit.
```bash
git reflog
git reset --hard <hash-from-reflog>   # recover a commit you thought was gone
```
💡 **Tip for the video:** demo losing a commit with `reset --hard` then recovering it with `reflog` — great "wow" moment for beginners.

---

## 10. Branch vs Tag vs Fork vs Clone (quick disambiguation)

| Concept | Difference |
|---|---|
| Git vs GitHub | Git = distributed version control system. GitHub = cloud platform to host Git repos. |
| Fetch vs Pull | Fetch downloads changes from remote. Pull = Fetch + Merge. |
| Merge vs Rebase | Merge preserves history via a merge commit. Rebase creates linear history by rewriting commits. |
| Reset vs Revert | Reset moves HEAD (can discard changes). Revert creates a new commit that undoes changes. |
| Fork vs Clone | Fork copies to GitHub server. Clone copies to your local machine. |
| Commit vs Push | Commit saves to local repo. Push uploads to remote repo. |
| Branch vs Tag | Branch is a movable pointer to a series of commits. Tag is a fixed reference to one commit. |
| Local vs Remote Repo | Local repo is on your machine. Remote repo lives on a server (e.g. GitHub). |

---

## 11. CI/CD & GitHub Actions — Architecture and Hands-On Practice

This is the "beyond the basics" piece worth learning next — automating tests/builds/deploys on every push.

### 11.1 The core idea
**CI (Continuous Integration):** every time code is pushed, automatically build + run tests, so bugs are caught immediately instead of at release time.
**CD (Continuous Delivery/Deployment):** automatically package and (optionally) deploy code that passes CI.

### 11.2 GitHub Actions architecture
```
Push/PR to GitHub
        │
        ▼
 GitHub Actions is triggered
        │
        ▼
   Workflow (.yml file in .github/workflows/)
        │
        ▼
   Runs on a Job → executed on a Runner (a fresh VM: ubuntu/windows/macos)
        │
        ▼
   Job = sequence of Steps (checkout code, install deps, run tests, build, deploy)
        │
        ▼
   ✅ Pass → merge allowed / deploy runs      ❌ Fail → PR blocked, you get notified
```
Key vocabulary:
- **Workflow** — the whole automated process, defined in a YAML file.
- **Event/Trigger** — what starts it: `push`, `pull_request`, `schedule`, `workflow_dispatch` (manual button).
- **Job** — a set of steps that run on one runner. Multiple jobs run in parallel by default.
- **Step** — a single command or a reusable "Action" (e.g. `actions/checkout@v4`).
- **Runner** — the virtual machine that executes the job (GitHub-hosted or self-hosted).
- **Secrets** — encrypted values (API keys, tokens) stored in repo Settings → Secrets, referenced as `${{ secrets.NAME }}` — never hardcoded.

### 11.3 Practice side-by-side (do this live)
1. In your repo: create the folder `.github/workflows/` and a file `ci.yml`.
2. Minimal example — runs tests on every push:
```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```
3. Commit + push this file:
```bash
git add .github/workflows/ci.yml
git commit -m "chore(ci): add GitHub Actions workflow"
git push origin main
```
4. Go to GitHub → **Actions** tab → watch the workflow run live, see each step's logs.
5. Break a test on purpose, push, show the ❌ red X and how it blocks/warns on the PR — then fix it and show ✅ green.

⚠️ **Mistake:** putting a real API key directly in the `.yml` file instead of using **Secrets** — it's committed to history and visible to anyone with repo access.

💡 **Tip:** Start every CI/CD explanation with "on every push, a fresh clean VM spins up, runs your steps, then gets destroyed" — it demystifies the whole thing.

**Next steps to mention (don't need to demo, just name-drop for the "advanced" viewers):** deploying automatically on merge to `main` (CD), matrix builds (test across multiple Node/Python versions at once), caching dependencies to speed up runs, environments/approvals for production deploys.

---

## 12. Common Beginner Mistakes — Master List (recap for the outro)

1. `git add .` without a `.gitignore` → commits secrets/junk. **Fix:** set up `.gitignore` first.
2. Committing directly to `main` instead of a feature branch. **Fix:** always branch first.
3. `git reset --hard` on a shared branch, wiping teammates' base. **Fix:** use `git revert` on shared history.
4. Rebasing a branch that's already pushed/shared. **Fix:** merge on shared branches, rebase only your own unpushed work.
5. Vague commit messages ("fix", "update", "asdf"). **Fix:** use `type(scope): subject` format.
6. Forgetting `git pull` before starting work → avoidable merge conflicts. **Fix:** pull latest `main` before branching.
7. Hardcoding secrets/API keys in code or CI YAML. **Fix:** `.gitignore` + GitHub Secrets, rotate if leaked.
8. Not deleting merged branches → cluttered branch list. **Fix:** delete after merge (`git branch -d`).
9. Force-pushing without `--force-with-lease` → can silently overwrite others' work. **Fix:** prefer `--force-with-lease`, and only on your own branch.
10. Confusing `fetch` (safe, no changes) with `pull` (can change your files). **Fix:** `fetch` first to inspect, then decide to merge.

---

## 13. Top 20 Rapid-Fire Interview / Recap Questions

1. What is Git?
2. Why is Git a distributed version control system?
3. What is the difference between Git and GitHub?
4. What is the purpose of `.gitignore`?
5. What is a repository?
6. What is the staging area?
7. What is a commit?
8. What does HEAD refer to?
9. What is `origin`?
10. What is the difference between fetch and pull?
11. What is the difference between merge and rebase?
12. What is cherry-pick?
13. What is stash and why is it used?
14. What is detached HEAD state?
15. What is a fast-forward merge?
16. How do you resolve merge conflicts?
17. What is a pull request?
18. What is a squash commit?
19. What is tagging in Git?
20. Explain a typical Git + GitHub Actions workflow end to end.

---

## 14. One-Page Cheat Sheet (pin this on screen at the end of the video)

```
SETUP        git init | git clone <url> | git config --global user.name/email
STATUS       git status | git diff | git diff --staged | git log --oneline --graph --all
STAGE/COMMIT git add <file>/. | git commit -m "msg" | git restore --staged <file>
BRANCH       git branch | git switch -c <name> | git branch -d <name>
MERGE        git merge <branch>            (safe, shared branches)
REBASE       git rebase <branch>           (private branches only, never after push)
CHERRY-PICK  git cherry-pick <hash>
SYNC         git fetch origin | git pull origin <branch> | git push origin <branch>
UNDO         git restore <file>            (working dir)
             git reset --soft/--mixed/--hard <hash>   (moves HEAD — hard is destructive)
             git revert <hash>             (safe, new undo-commit)
             git stash / stash pop         (shelve work temporarily)
RECOVER      git reflog                    (find "lost" commits)
TAGS         git tag -a v1.0.0 -m "msg" | git push origin --tags
CI/CD        .github/workflows/*.yml → triggers job on runner → steps → pass/fail gate on PR
```

---

## 15. Suggested Video Flow (recording order)

1. Intro: Git vs GitHub, why version control matters (Section 1).
2. The 4-stage architecture — draw it, then prove it live with `status` after each step (Section 2).
3. Essential commands — type each one, show output (Section 3).
4. Branching + a real merge conflict you create and resolve on camera (Section 4).
5. Merge vs rebase side-by-side, then cherry-pick (Section 5).
6. Full GitHub workflow: fork/clone → branch → PR → review → merge on github.com (Section 6).
7. Undo section — the "oops" recovery toolkit: reset/revert/restore/stash + reflog save (Sections 7, 9).
8. `.gitignore` + committing a fake secret, then explaining rotation (Section 8).
9. GitHub Actions demo: add workflow file, push, watch it run, break it, fix it (Section 11).
10. Recap with the cheat sheet + common mistakes list as the outro (Sections 12, 14).
