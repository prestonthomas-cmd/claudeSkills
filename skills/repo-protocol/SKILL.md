---
name: repo-protocol
description: Branch protocol for every git repository. Use at the start of any session that works in a git repo, and before any commit, push, or merge. Work happens on `dev`; `main` only changes after the user approves a merge.
---

# Repo protocol: main + dev

Every repository has two long-lived branches:

- **`main`**: stable. Never commit or push to it directly.
- **`dev`**: where all work happens. Claude commits and pushes here.

## 1. At the start of a session (before changing any files)

For each git repository in the session:

1. `git fetch origin` (retry network errors up to 4 times with backoff).
2. Make sure `main` exists:
   - If the repo has no commits yet, create an initial commit on `main` and push it with `git push -u origin main`.
   - If the default branch has another name (e.g. `master`), **don't rename it**. Tell the user and treat that branch as `main` for this protocol.
3. Make sure `dev` exists:
   - If `origin/dev` exists, check it out: `git checkout -B dev origin/dev`.
   - Otherwise, create it from main and push: `git checkout -b dev main && git push -u origin dev`.
4. If `dev` is behind `main`, bring it up to date (`git merge main`) before starting work.
5. Tell the user in one line: "Working on `dev` in <repo>."

If the session was started on another branch (e.g. a `claude/...` branch), switch to `dev` anyway and say so. If pushing to `dev` is refused, stop and tell the user. Don't fall back to pushing to `main`.

## 2. While working

- Commit to `dev` in small, clear commits.
- Push with `git push -u origin dev` once a logical piece of work is done.
- Never force-push `main`. Force-push `dev` only if the user asks.

## 3. After pushing to dev: ask about main

After every push to `dev` that completes a piece of work, ask the user:

> Pushed to `dev`: <one-line summary, N commits>. **Merge `dev` into `main`?**

Show a short list of the commits that would go to main (`git log --oneline main..dev`).

- **Yes**:
  ```bash
  git checkout main
  git pull --ff-only origin main
  git merge --no-ff dev -m "Merge dev into main"
  git push origin main
  git checkout dev
  ```
  If the merge has conflicts, stop and show them to the user. Don't resolve them silently.
  If pushing to `main` is refused (e.g. branch protection), offer to open a pull request from `dev` to `main` instead.
- **No, or no answer**: leave `main` untouched and keep working on `dev`.

Only merge to `main` on an explicit yes in the current conversation. Earlier approvals don't carry over.
