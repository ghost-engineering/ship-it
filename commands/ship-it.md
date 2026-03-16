---
allowed-tools: Bash(git pull:*), Bash(git fetch:*), Bash(git status:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(git merge:*), Bash(git log:*), Bash(git diff:*), Bash(git stash:*), Bash(git rebase:*), Bash(git branch:*), Bash(git remote:*), Bash(npx vercel env pull:*), Bash(npx netlify env*), Bash(npm run build:*), Bash(cat package.json), Bash(ls -la:*), Bash(test -f:*), Bash(test -d:*)
description: Pull latest, resolve conflicts, sync env, build, and push to deploy
---

## Context

- Current branch: !`git branch --show-current`
- Current git status: !`git status`
- Remote tracking: !`git remote -v`

## Your task

Ship the current state of the project. Follow these steps in order. Execute each step sequentially — do not skip steps.

### 1. Detect environment

Determine the deployment platform and build setup:

- Check for `vercel.json` or `.vercel/` directory → **Vercel**
- Check for `netlify.toml` or `.netlify/` directory → **Netlify**
- Otherwise → **Plain git** (skip env sync later)

Read `package.json` to confirm a `build` script exists.

Determine the main branch:
- Check `git remote show origin` or use the current branch
- Do not assume `main` — it could be `master` or another branch

### 2. Check for uncommitted changes

Run `git status`. If there are uncommitted changes:
- Run `git diff HEAD` to understand what changed
- Stage all changed files with `git add .`
- Create a commit with a clear, descriptive message based on the actual diff content
- Do NOT proceed to pull until local changes are committed

If the working tree is clean, move on.

### 3. Pull latest from remote

Run `git fetch origin` then `git pull origin <branch> --no-rebase`.

If there are merge conflicts:
- List all conflicted files
- For each conflict, read the file and resolve it intelligently:
  - Prefer keeping both sides when changes don't overlap
  - For true conflicts (same lines changed), prefer the remote version but preserve local additions
- Stage resolved files with `git add <file>`
- Complete the merge with `git commit`
- If conflicts are too complex to resolve automatically, STOP and ask the user for guidance. Do not guess.

If already up to date, move on.

### 4. Sync environment variables

Based on the platform detected in Step 1:

**Vercel:**
```
npx vercel env pull .env.local
```

**Netlify:**
```
npx netlify env:import .env
```

**Plain git:** Skip this step.

If the command fails due to authentication:
- Tell the user which login command to run (`npx vercel login` or `npx netlify login`)
- STOP and wait — do not skip this step or continue without env sync

### 5. Build check

Run `npm run build` to verify everything compiles cleanly.

If the build fails:
- Read the error output carefully
- Attempt to fix build errors if they are straightforward (missing imports, type errors from the merge)
- Re-run `npm run build` after fixes
- If you cannot fix the errors after one attempt, STOP and show the user the full error output

### 6. Push to remote

Run `git push origin <branch>`.

If the push is rejected (remote has new commits since our pull):
- Run `git pull origin <branch> --no-rebase` again
- Resolve any new conflicts
- Re-run build
- Push again

### 7. Summary

After all steps complete, provide a concise summary:

- **Committed**: What was committed (or "nothing — tree was clean")
- **Pulled**: Whether new changes came in, and if conflicts were resolved
- **Env**: Whether env was synced and which platform
- **Build**: Pass or fail
- **Pushed**: Commit range pushed (e.g., `abc123..def456`)

You MUST execute all steps sequentially. If any step fails and cannot be recovered, STOP immediately and explain what went wrong. Do not continue past a failure.
