---
allowed-tools: Bash(git pull:*), Bash(git fetch:*), Bash(git status:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(git merge:*), Bash(git log:*), Bash(git diff:*), Bash(git stash:*), Bash(git rebase:*), Bash(git branch:*), Bash(git remote:*), Bash(git checkout:*), Bash(gh pr create:*), Bash(gh pr list:*), Bash(gh pr view:*), Bash(npx vercel env pull:*), Bash(npx netlify env*), Bash(npm run build:*), Bash(cat package.json), Bash(ls -la:*), Bash(test -f:*), Bash(test -d:*)
description: Pull latest, resolve conflicts, sync env, build, and push — with PR support for protected branches
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

Read `package.json` to confirm a `build` script exists. If there is no `build` script, skip the build step later.

Determine the branch context:

- Get the current branch name from `git branch --show-current`
- Determine the default branch: run `git remote show origin` and look for "HEAD branch"
- Do not assume `main` — it could be `master` or another branch

### 2. Check for uncommitted changes

Run `git status`. If there are uncommitted changes:
- Run `git diff HEAD` to understand what changed
- Stage all changed files with `git add .`
- Create a commit with a clear, descriptive message based on the actual diff content
- Do NOT proceed to pull until local changes are committed

If the working tree is clean, move on.

### 3. Pull latest from remote

Run `git fetch origin` then `git pull origin <current-branch> --no-rebase`.

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

If `package.json` has a `build` script, run `npm run build` to verify everything compiles cleanly.

If the build fails:
- Read the error output carefully
- Attempt to fix build errors if they are straightforward (missing imports, type errors from the merge)
- Re-run `npm run build` after fixes
- If you cannot fix the errors after one attempt, STOP and show the user the full error output

If there is no build script, skip this step.

### 6. Push or create PR

Determine the push strategy based on branch context:

**If you are on the default branch (main/master):**
- Try `git push origin <branch>`
- If the push is **rejected because the branch is protected** (e.g., "protected branch hook declined", "required status check", "required pull request reviews"):
  1. Create a feature branch: `git checkout -b ship-it/<short-description>` (use the commit message to derive a short kebab-case name)
  2. Push the feature branch: `git push -u origin ship-it/<short-description>`
  3. Create a PR: `gh pr create --title "<commit summary>" --body "Automated PR created by /ship-it"`
  4. Tell the user the PR URL and that the branch is protected
- If the push is **rejected because remote has new commits**:
  1. Pull again: `git pull origin <branch> --no-rebase`
  2. Resolve any new conflicts
  3. Re-run build
  4. Push again
  5. If it fails a second time, STOP and ask the user

**If you are on a feature branch:**
- Push the branch: `git push -u origin <branch>`
- Check if a PR already exists: `gh pr view <branch>` (ignore errors if no PR exists)
- If no PR exists, ask the user: "Want me to create a PR to merge `<branch>` into `<default-branch>`?"
  - If yes: `gh pr create --title "<summary>" --body "Automated PR created by /ship-it"` targeting the default branch
  - If no: just confirm the push succeeded

### 7. Summary

After all steps complete, provide a concise summary:

- **Committed**: What was committed (or "nothing — tree was clean")
- **Pulled**: Whether new changes came in, and if conflicts were resolved
- **Env**: Whether env was synced and which platform (or "skipped — plain git")
- **Build**: Pass, fail, or skipped (no build script)
- **Pushed**: Commit range pushed and to which branch, OR the PR URL if a PR was created

You MUST execute all steps sequentially. If any step fails and cannot be recovered, STOP immediately and explain what went wrong. Do not continue past a failure.
