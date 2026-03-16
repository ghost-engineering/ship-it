# ship-it

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that pulls, syncs, builds, and pushes your project in one command.

Stop juggling `git pull`, env syncs, build checks, and `git push` separately. Just type `/ship-it` and it handles everything — including merge conflicts.

## What it does

1. **Commits** any uncommitted changes with a descriptive message
2. **Pulls** latest from your remote and resolves merge conflicts
3. **Syncs** environment variables from Vercel or Netlify
4. **Builds** your project to catch errors before they hit production
5. **Pushes** to your remote — deploying automatically if you have CI/CD set up

## How to install

### Prerequisites

You need [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed. If you don't have it yet:

```bash
npm install -g @anthropic-ai/claude-code
```

Then verify it's working:

```bash
claude --version
```

### Install the skill

Run this command in your terminal:

```bash
claude install-skill github:ghost-engineering/ship-it
```

That's it. The skill is now available in every project you open with Claude Code.

### Verify the install

Open any git project with Claude Code:

```bash
cd your-project
claude
```

Then type `/ship-it` — you should see it in the command list.

## How to use

### Basic usage

Open your project in Claude Code and type:

```
/ship-it
```

The skill will walk through each step automatically and give you a summary at the end.

### What you'll see

```
Committed: Add new feature component and update styles
Pulled: 3 new commits from origin/main, no conflicts
Env: Synced from Vercel (2 variables updated)
Build: Pass (14 routes)
Pushed: abc1234..def5678 to origin/main
```

### If something goes wrong

The skill stops immediately if it hits a problem it can't solve on its own:

- **Merge conflicts too complex**: It'll show you the conflicted files and ask what to do
- **Vercel/Netlify not logged in**: It'll tell you to run `npx vercel login` or `npx netlify login`
- **Build fails**: It'll show you the errors and try to fix them once. If it can't, it stops and shows you the output

Your code is never in a broken state — it won't push unless the build passes.

## Platform support

The skill auto-detects your setup:

| Platform | Detection | Env sync command |
|----------|-----------|-----------------|
| Vercel | `vercel.json` or `.vercel/` | `npx vercel env pull .env.local` |
| Netlify | `netlify.toml` or `.netlify/` | `npx netlify env:import .env` |
| Plain git | (default) | Skipped |

### Setting up Vercel env sync

If you deploy with Vercel, make sure you're logged in:

```bash
npx vercel login
```

This opens a browser for authentication. You only need to do this once per machine.

### Setting up Netlify env sync

If you deploy with Netlify:

```bash
npx netlify login
```

## Project-level install (for teams)

If you want the entire team to have `/ship-it` available when they work on a specific repo, you can add it directly to the project instead:

1. Create the commands directory in your repo:

```bash
mkdir -p .claude/commands
```

2. Copy the command file:

```bash
cp ~/.claude/skills/ship-it/commands/ship-it.md .claude/commands/ship-it.md
```

3. Commit and push:

```bash
git add .claude/commands/ship-it.md
git commit -m "Add /ship-it command for team"
git push
```

Now anyone on the team who uses Claude Code will have `/ship-it` available when they open the repo — no personal install needed.

## FAQ

**Does this work with branches other than `main`?**
Yes. It detects your current branch automatically and pushes to whatever branch you're on.

**Will it force-push?**
Never. It only does normal pushes. If the push is rejected, it pulls again and retries.

**Does it work without Vercel or Netlify?**
Yes. It skips the env sync step and handles everything else.

**What if I don't have a build script?**
It skips the build step if there's no `build` script in your `package.json`.

**Can it mess up my code?**
It won't push unless the build passes. If merge conflicts are complex, it stops and asks you. It never drops changes silently.

## License

MIT - [Ghost Engineering](https://reachbpt.com?from=ship-it)
