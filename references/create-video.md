# Make a launch/announcement video of this repo now

This is the standard flow: read the repo, build a brief, push it. Do the
**Setup — login** in `SKILL.md` first (ensure the CLI, `keynoter login`, the one
human browser step), then follow the steps below.

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
