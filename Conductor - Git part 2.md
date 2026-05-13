# BOOST Git part 2 conductor

Session: Version control for reproducible research, part 2  
Length: 60 minutes  
Example repository: https://github.com/vncntprvst/2P-proc

## Session goal

Participants should leave with a concrete mental model of a daily Git workflow:

1. Start from the shared `main` branch.
2. Create a branch for one focused change.
3. Commit that change locally.
4. Merge it back into `main`.
5. Push it to GitHub.
6. Pull it on another machine.
7. Tag an important project state.

Keep the session focused on a single clean path. Mention pull requests and conflicts only as extensions unless the room asks.

## Preparation before the session

Do this before participants arrive:

```bash
git --version
gh auth status
```

Have a clean local clone ready:

```bash
cd ~/Code
git clone https://github.com/vncntprvst/2P-proc.git 2P-proc-boost-demo
cd 2P-proc-boost-demo
git status
git log --oneline --decorate --graph --all -n 10
```

Also prepare a "remote machine" clone. This can be a real laptop, a cluster login node, or a second local folder that acts as the remote machine:

```bash
cd ~/Code
git clone https://github.com/vncntprvst/2P-proc.git 2P-proc-boost-remote
cd 2P-proc-boost-remote
git status
```

Recommended demo feature: add a small usage note, checklist, or example config documentation rather than editing scientific pipeline logic live. This keeps the Git lesson about Git, not debugging Python dependencies.

Suggested branch name:

```bash
feature/boost-config-checklist
```

Suggested file to create:

```text
docs/config-checklist.md
```

If you want the demo to be fully reversible, push a demo branch and tag, then delete them after the session.

## Time plan

| Time | Segment | Goal | Commands / actions |
|---:|---|---|---|
| 0-5 min | Reframe from part 1 | Remind them: Git is local history; GitHub is a hosting/collaboration site. | Show `git status`, `git log --oneline --graph --decorate --all -n 10`. |
| 5-12 min | Remotes and starting point | Show that the local repo tracks GitHub and starts from `main`. | `git remote -v`, `git branch -vv`, `git pull --ff-only`. |
| 12-22 min | Create a branch | Branches isolate work until it is ready. | `git switch -c feature/boost-config-checklist`, create/edit `docs/config-checklist.md`. |
| 22-32 min | Commit the feature | A commit is a named snapshot with intent. | `git status`, `git diff`, `git add docs/config-checklist.md`, `git commit -m "Add config checklist documentation"`. |
| 32-42 min | Merge to main | Bring the finished feature back to the stable branch. | `git switch main`, `git merge feature/boost-config-checklist`, `git log --oneline --graph --decorate --all -n 12`. |
| 42-48 min | Push to GitHub | Publish local history to the shared remote. | `git push origin main`. Show GitHub page refreshing. |
| 48-54 min | Pull on remote machine | A second machine gets the same history from GitHub. | In second clone: `git pull --ff-only`, `git log --oneline --decorate -n 5`, open the new file. |
| 54-58 min | Add a tag | Tags name milestones: release, submitted manuscript, analysis freeze. | `git tag -a boost-demo-2026-05 -m "BOOST Git part 2 demo state"`, `git push origin boost-demo-2026-05`. |
| 58-60 min | Wrap | Give the reusable workflow and when to ask for help. | Show summary commands and point to practice guide. |

If time is running short, skip the live tag push and show the tag commands only. Do not skip the branch, commit, merge, push, and pull loop.

## Live demo script

### 1. Start clean

```bash
cd ~/Code/2P-proc-boost-demo
git status
git branch -vv
git remote -v
git pull --ff-only
```

Say:

> I am making sure my local `main` branch agrees with GitHub before starting work.

### 2. Create a branch

```bash
git switch -c feature/boost-config-checklist
git status
```

Say:

> This branch is my workbench. I can make a change without disturbing `main`.

### 3. Add a small feature

Create `docs/config-checklist.md`:

```markdown
# Configuration checklist

Before running the 2P-proc pipeline, check:

- The config JSON points to the correct raw data folder.
- Export paths point to a writable output location.
- Imaging frame rate and image dimensions match the acquisition.
- Motion correction and extraction settings match the intended backend.
- Large raw data and generated outputs are not committed to Git.
```

Optional if there is already a `docs/` folder missing:

```bash
mkdir -p docs
```

### 4. Inspect, stage, commit

```bash
git status
git diff
git add docs/config-checklist.md
git status
git commit -m "Add config checklist documentation"
```

Say:

> `git diff` is the habit that prevents accidental commits. `git add` chooses what goes into the snapshot. `git commit` records it.

### 5. Merge back into `main`

```bash
git switch main
git status
git merge feature/boost-config-checklist
git log --oneline --graph --decorate --all -n 12
```

Say:

> The feature branch let me work separately; the merge moves the completed change back to the branch others normally use.

### 6. Push to GitHub

```bash
git push origin main
```

Open GitHub and show the new commit and file.

If authentication gets in the way, use:

```bash
gh auth login
```

or switch to GitHub Desktop for the push only.

### 7. Pull on the remote machine

In the second clone:

```bash
cd ~/Code/2P-proc-boost-remote
git status
git pull --ff-only
git log --oneline --decorate -n 5
```

Open `docs/config-checklist.md`.

Say:

> The remote machine did not receive files directly from my laptop. It received commits from GitHub.

### 8. Add a tag

Back in the first clone:

```bash
git tag -a boost-demo-2026-05 -m "BOOST Git part 2 demo state"
git tag
git push origin boost-demo-2026-05
```

Say:

> A branch moves as work continues. A tag is a stable name for one exact commit.

## Optional cleanup after the session

Only run this if the demo branch and tag should not stay on GitHub:

```bash
git push origin --delete boost-demo-2026-05
git tag -d boost-demo-2026-05
git branch -d feature/boost-config-checklist
```

If you pushed a real commit to `main`, either keep it if useful or revert it with a new commit:

```bash
git revert HEAD
git push origin main
```

Do not rewrite public history during the session.

## Backup plan

If GitHub authentication fails:

1. Continue locally through branch, commit, and merge.
2. Explain that push/pull require authentication.
3. Show `git remote -v`, `gh auth login`, and the GitHub Desktop alternative.
4. Use the second local clone as the "remote machine" only after you can push.

If the room gets stuck installing Git:

1. Demonstrate from your machine.
2. Ask participants to follow conceptually.
3. Point them to the practice guide for the commands they can repeat later.

## Key phrases to repeat

- Git is the local history; GitHub is a place to host and share that history.
- Branches are cheap workspaces.
- Commit small, coherent changes.
- Pull before starting; push when the work should be shared.
- Tags name important exact states.
- Generated outputs and large raw data usually do not belong in Git.

