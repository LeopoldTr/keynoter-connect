# keynoter-connect

A coding-agent **skill** (Claude Code · Codex · Cursor · Mistral) that turns the repo you're working in into a short, motion-designed launch video with [Keynoter](https://keynoter.app).

Your agent does the work locally — it installs the `keynoter` CLI, signs you in, reads the repo, asks you 2–3 questions, writes a brief, and runs `keynoter create`. You approve the one-time sign-in in your browser, then refine and render in the Keynoter Studio.

## Install

```bash
npx skills add LeopoldTr/keynoter-connect
```

Then, in your agent, just ask:

```
Make a launch video for this repo with keynoter
```

## What happens

1. The agent installs the `keynoter` CLI (or runs it with `bunx`).
2. It signs you in once (device flow — you approve in the browser). That's the only manual step.
3. It reads the repo (what to announce + the design), asks 2–3 questions, assembles a brief, and runs `keynoter create`.
4. `keynoter create` prints a Studio link — open it to refine the audience, format, and look, then approve. Keynoter renders the MP4.

Learn more at [keynoter.app](https://keynoter.app).
