# Make a guided walkthrough video of this app

This is the standard, highest-quality flow: you drive the real app, capture
ordered screenshots, write the narration script, and push both. Keynoter shows
the user the screens + script to review and approve, then renders a narrated
walkthrough where each screenshot is on screen while its line is spoken.

Do the **Setup — login** in `SKILL.md` first (ensure the CLI, `keynoter login`,
the one human browser step), then follow the steps below.

## What to do

1. **Get the app running and reachable.** It's usually already running in dev, or
   the repo's README tells you how (`bun dev`, `npm run dev`, a `docker compose
   up`, a deployed URL…). Confirm the URL before capturing.
2. **Capture ordered screenshots of the key screens** (the real user journey:
   landing → the core feature → the payoff). **1–8 screens.** Save them to local
   PNG/JPG files (e.g. `shots/01-dashboard.png`, `shots/02-editor.png`). Prefer a
   clean, logged-in, populated state — no error toasts or empty placeholders.
   Use whatever browser tooling you have (a headless screenshot script, a browser
   MCP, Playwright); a 1280×800 or larger viewport reads well at 16:9.
3. **Write the script — ONE spoken sentence of narration per screen.** This is
   the star of the video and what the user reviews. Make each line concrete and
   benefit-led ("Start on the dashboard — every project at a glance"), in the
   order a new user would move through the app. Keep each to ~1 breath.
4. **Assemble the walkthrough brief** as a single JSON file (shape below). Each
   screen's `image` is the LOCAL path to the file you saved (relative to the
   brief file); the CLI uploads the bytes for you.
5. **Push it:** `keynoter create --brief brief.json`. The CLI reads the images,
   uploads everything in one request, and prints the Studio URL.
6. **Relay the Studio URL** and tell the user to open it, **review/edit the
   script**, and approve — Keynoter renders the MP4. Don't poll or wait.

## Brief shape (write exactly this JSON)

```json
{
  "source": { "sourceKind": "repo", "sourceUrl": "https://github.com/<owner>/<repo>" },
  "language": "en",
  "targetSeconds": 45,
  "projectTitle": "<product name>",
  "tagline": "<one short line>",
  "goal": "Show how <product> works",
  "audience": "<who, from the user — or null>",
  "tone": "<calm | punchy | technical | … — or null>",
  "identity": { "palette": ["#hex", "#hex"], "fonts": ["Inter"], "logoUrl": null },
  "walkthrough": {
    "screens": [
      { "image": "shots/01-dashboard.png", "narration": "Start on the dashboard — every project at a glance.", "label": "Dashboard" },
      { "image": "shots/02-editor.png",    "narration": "Open the editor to draft a new note in seconds.",    "label": "Editor" },
      { "image": "shots/03-share.png",     "narration": "Share it with one click and you're done.",           "label": "Share" }
    ]
  }
}
```

Rules for the brief:
- `source.sourceKind` is `"repo"` (or `"website"`); `source.sourceUrl` must be a
  valid URL. Both required.
- `walkthrough.screens`: **1–8** entries, in play order. Each needs a local
  `image` path and a one-sentence `narration`; `label` is an optional short
  caption shown on the screen.
- `projectTitle`, `tagline`, `goal` are required strings. `audience`/`tone` may be
  `null`, but prefer filling them from the user's answers — they steer the script.
- `targetSeconds` is an integer (default `45`); the time is split across screens.
- `identity` is optional; include a palette/fonts only if you actually found them.
- The screenshots are uploaded; do NOT paste base64 or URLs into the brief.

## If you can't capture screenshots

If the app genuinely can't be run or screenshotted, fall back to the motion
teaser flow → read `references/create-video.md`. (Keynoter's studio can also ask
the user to upload screenshots later, but a brief WITH screenshots gives the best
result.)

## After pushing

Tell the user: "Open <Studio URL> to review the screens and the script, tweak any
narration line, then approve — Keynoter will render the MP4." The studio is where
they take over.
