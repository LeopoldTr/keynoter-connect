---
name: keynoter-connect
description: Use when the user wants to turn THIS repo into a short launch/announcement video with Keynoter — read the repo, assemble a brief, and push it to keynoter from the terminal.
---

# Keynoter — connect this project

You turn the repository you are working in into a short, motion-designed launch
video. You do the thinking locally (read the repo, ask the user 2–3 questions,
write the brief); Keynoter renders it. There is no back-and-forth with the
Keynoter server beyond a single push.

## Setup — do this yourself, don't make the user do it

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

## What to do

1. **Read the repo for WHAT to announce.** Skim the README, recent commits /
   CHANGELOG, and the main entry points. Identify the single most compelling
   thing to announce (a new feature, a launch, a capability) — not a feature
   dump.
2. **Read the repo for the DESIGN (DA).** Look for brand colors, fonts, a logo,
   and the product's tone (playful, enterprise, technical). Collect a small
   palette (hex) and font names if present.
3. **Ask the user 2–3 questions, locally** — do not guess these:
   - Who is the audience? (e.g. developers, B2B decision-makers, end users)
   - What is the one thing to put front and center?
   - Tone / format preference? (punchy vs. calm; default format `motion`)
4. **Assemble the brief** as a single JSON file matching the shape below.
5. **Push it:** `keynoter create --brief brief.json`. The CLI prints the Studio
   URL — relay it to the user and tell them to open it to refine and approve the
   video.

## Brief shape (write exactly this JSON)

```json
{
  "source": { "sourceKind": "repo", "sourceUrl": "https://github.com/<owner>/<repo>" },
  "language": "en",
  "targetSeconds": 30,
  "analyzed": {
    "goal": "Announce <the one thing>",
    "audience": "<who, from the user's answer>",
    "tone": "<punchy | calm | technical | …>",
    "format": "motion",
    "projectTitle": "<product name>",
    "tagline": "<one short line>",
    "segments": [
      { "brief": "hook on the logo / problem", "narration": "<one spoken sentence>" },
      { "brief": "show the key capability",    "narration": "<one spoken sentence>" },
      { "brief": "call to action",             "narration": "<one spoken sentence>" }
    ]
  },
  "identity": { "palette": ["#hex", "#hex"], "fonts": ["Inter"], "logoUrl": null }
}
```

Rules for the brief:
- `source.sourceKind` is `"repo"` (or `"website"`); `source.sourceUrl` must be a
  valid URL. Both are required.
- `format` is `"motion"` for the standard narrated branded animation, or
  `"kinetic"` for a music-driven montage. These are the only two values.
- `targetSeconds` is an integer (default `60`, max `600`); `30` is a good short
  default.
- `segments`: 1–8 entries. Each `narration` is ONE spoken sentence (it becomes
  the voice-over + on-screen caption). 3 segments is a good default.
- `analyzed.audience` / `analyzed.tone` may be `null` if the user truly has no
  preference, but prefer filling them from the user's answers — they steer the
  script. `goal`, `projectTitle`, and `tagline` are always required strings.
- `identity` is optional; include a palette/fonts/logo only if you actually
  found them in the repo.

## After pushing

Tell the user: "Open <Studio URL> to refine the target, format and look, then
approve — Keynoter will render the MP4." Do not poll or wait; the studio is
where they take over.
