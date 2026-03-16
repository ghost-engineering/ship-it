---
name: ship-it
description: Pull latest from remote, resolve conflicts, sync environment variables, verify the build, and push — all in one command. Supports Vercel, Netlify, and plain git workflows.
license: MIT
metadata:
  author: ghost-engineering
  version: "1.0.0"
---

# Ship It

A single command to sync, verify, and deploy your project. Run `/ship-it` and it handles the full workflow:

1. Commit uncommitted changes
2. Pull latest from remote and resolve conflicts
3. Sync environment variables (Vercel, Netlify, or skip)
4. Run the build to catch errors before pushing
5. Push to remote

## Platform Detection

The skill auto-detects your deployment platform:

- **Vercel**: Looks for `vercel.json` or `.vercel/` directory, runs `npx vercel env pull .env.local`
- **Netlify**: Looks for `netlify.toml` or `.netlify/` directory, runs `npx netlify env:import .env`
- **None**: Skips env sync gracefully

## Build Command Detection

The skill reads `package.json` to determine the build command:

- Uses the `build` script from `package.json` if present (`npm run build`)
- Falls back to common framework CLIs if no build script exists
- Skips build step if no build command can be determined

## Conflict Resolution

When merge conflicts occur:

- Simple conflicts (whitespace, import ordering, non-overlapping changes) are resolved automatically
- Complex conflicts that require human judgment will STOP the workflow and ask for guidance
- The skill never silently drops changes from either side
