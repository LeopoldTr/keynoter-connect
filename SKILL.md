---
name: keynoter-connect
description: Use when the user wants to turn THIS repo into a short launch/announcement video with Keynoter — read the repo, assemble a brief, and push it to keynoter from the terminal. Can also install the @keynoter PR bot (a self-contained GitHub Actions workflow that renders a demo when someone comments @keynoter on a PR).
---

# Keynoter — connect this project

You turn the repository you are working in into a short, motion-designed launch
video. You do the thinking locally (read the repo, ask the user 2–3 questions,
write the brief); Keynoter renders it. There is no back-and-forth with the
Keynoter server beyond a single push.

Always do the **Setup — login** below first, then read ONLY the one reference
that matches what the user asked.

## What you can do

Pick the right reference based on what the user asked, and read only that one:

- **Make a launch/announcement video of this repo now** (the default) — read the
  repo, build a brief, push it → read `references/create-video.md`.
- **Install the @keynoter PR bot** (a workflow that renders a demo when someone
  comments `@keynoter` on a PR) — when the user asks to set up / install the bot
  ("create pr workflow", "install the keynoter PR bot") → read
  `references/install-pr-bot.md`.

## Setup — login (both flows need this; do it yourself, don't make the user do it)

You handle the tooling. The user only ever does ONE manual thing: approve the
login in their browser (a device-flow security step you cannot do for them).

1. **Ensure the CLI is available.** Run `keynoter --version`. If the command is
   not found, install it: `npm i -g keynoter` (or, if you can't install
   globally, use `bunx keynoter …` in place of `keynoter …` everywhere below).
2. **Ensure you're logged in.** If `~/.keynoter/config.json` has no token (or a
   later `keynoter create` returns "not authorized"), run `keynoter login`. It
   prints a verification URL + a short user-code, then blocks while it polls.
   **Surface that URL and code to the user** and ask them to open it, sign in,
   and approve the device — that is the one human step. The command returns once
   they approve and the token is written to `~/.keynoter/config.json`; then
   continue. Subsequent runs are fully automatic (the token is cached) — only
   re-run `keynoter login` if the token is missing or rejected.
