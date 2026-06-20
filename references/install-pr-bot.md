# Install the @keynoter PR bot

This flow installs a self-contained GitHub Actions workflow into THIS repo. Do
the **Setup — login** in `SKILL.md` first (ensure the CLI, `keynoter login`, the
one human browser step), then follow the steps below.

You install a GitHub Actions workflow into THIS repo that renders a Keynoter
demo of a PR's diff whenever someone comments `@keynoter` on that PR, then posts
the share link back on the PR (and optionally to Discord). You set everything up
yourself — the user only approves the browser login.

## 1. Ask the user (one at a time, locally)

1. **Trigger** — how should the bot fire?
   - `comment` (default): someone comments `@keynoter` on a PR.
   - `pr`: every PR opened / updated (opened + synchronize).
   - `label`: a `keynoter` label is added to a PR.
2. **Discord** — also post the result to a Discord channel? If yes, get the
   webhook URL.
3. **Defaults** — language (default `en`) and targetSeconds (default `60`).
   Keep this light; accept the defaults if they don't care.

## 2. Set up auth + secrets yourself

1. **CLI + login** — same as the **Setup — login** section in `SKILL.md`: ensure
   `keynoter` is available, then `keynoter login` and surface the URL + code so
   the user approves the device in their browser.
2. **Mint a prod api-key** for CI (it prints only the key to stdout):
   ```bash
   keynoter token create --name <repo>-pr-bot
   ```
   Capture stdout. **Never echo or print the key.**
3. **Set it as a repo secret** (pass it via stdin/`--body`, never inline in a
   logged command):
   ```bash
   gh secret set KEYNOTER_TOKEN --body "<key>"
   ```
   If Discord was chosen:
   ```bash
   gh secret set DISCORD_WEBHOOK_URL --body "<webhook-url>"
   ```

## 3. Write `.github/workflows/keynoter.yml`

Write the template below into the user's repo, filled from the answers. It is
**self-contained** — it depends on no external/published action (the keynoter
repo is private, there is nothing to `uses:`). It does everything with `gh`
(preinstalled + authed via `GH_TOKEN`), `curl`, and `jq`.

Fill the placeholders:

- **`on:` block** — pick by the chosen trigger:
  - `comment` → `issue_comment: { types: [created] }`
  - `pr` → `pull_request: { types: [opened, synchronize] }`
  - `label` → `pull_request: { types: [labeled] }`
- **`if:` guard** — only for `comment`/`label` (drop it for `pr`):
  - `comment`: `github.event.issue.pull_request && contains(github.event.comment.body, '@keynoter') && (github.event.comment.author_association == 'OWNER' || github.event.comment.author_association == 'MEMBER' || github.event.comment.author_association == 'COLLABORATOR')`
  - `label`: `github.event.label.name == 'keynoter'`
- **`PR_NUMBER`** — `comment` → `${{ github.event.issue.number }}`; `pr`/`label`
  → `${{ github.event.pull_request.number }}`.
- **`KEYNOTER_LANG` / `TARGET_SECONDS`** env vars — the chosen defaults (`en` / `60`).
- **Discord step** — keep it only if Discord was chosen.
- The `react` step (👍 on the comment) is only meaningful for the `comment`
  trigger — drop it for `pr`/`label`.

```yaml
name: Keynoter PR Demo
on:
  issue_comment:
    types: [created]
permissions:
  pull-requests: write
  contents: read
jobs:
  keynoter:
    if: github.event.issue.pull_request && contains(github.event.comment.body, '@keynoter') && (github.event.comment.author_association == 'OWNER' || github.event.comment.author_association == 'MEMBER' || github.event.comment.author_association == 'COLLABORATOR')
    runs-on: ubuntu-latest
    env:
      GH_TOKEN: ${{ github.token }}
      KEYNOTER_TOKEN: ${{ secrets.KEYNOTER_TOKEN }}
      API: ${{ vars.KEYNOTER_API_URL || 'https://keynoter.app' }}
      PR_NUMBER: ${{ github.event.issue.number }}
      KEYNOTER_LANG: "en"
      TARGET_SECONDS: "60"
      MARKER: "<!-- keynoter-bot -->"
    steps:
      - name: Ack with a reaction
        continue-on-error: true
        run: |
          gh api -X POST \
            "repos/${{ github.repository }}/issues/comments/${{ github.event.comment.id }}/reactions" \
            -f content=+1

      - name: Gather PR context
        run: |
          gh pr view "$PR_NUMBER" --json title,body,files > pr.json
          jq -r '.title' pr.json > title.txt
          jq -r '.body // ""' pr.json > body.txt
          jq -r '.files[].path' pr.json > files.txt
          gh pr diff "$PR_NUMBER" | head -c 12000 > diff.txt
          STEER="$(printf '%s' "${{ github.event.comment.body }}" | sed -n 's/.*@keynoter//p' | head -1)"
          printf '%s' "${STEER# }" > steering.txt

      - name: Start the render
        run: |
          BODY="$(jq -n \
            --arg repoUrl "https://github.com/${{ github.repository }}" \
            --argjson prNumber "$PR_NUMBER" \
            --arg title "$(cat title.txt)" \
            --rawfile body body.txt \
            --rawfile diff diff.txt \
            --rawfile steering steering.txt \
            --slurpfile files <(jq -R . files.txt | jq -s .) \
            --arg language "$KEYNOTER_LANG" \
            --argjson targetSeconds "$TARGET_SECONDS" \
            '{repoUrl:$repoUrl, prNumber:$prNumber, title:$title,
              body:(if $body=="" then null else $body end),
              diff:$diff, changedFiles:$files[0],
              steering:(if $steering=="" then null else $steering end),
              language:$language, targetSeconds:$targetSeconds}')"
          RESP="$(curl -sS -X POST "$API/api/connect/pr" \
            -H "Authorization: Bearer $KEYNOTER_TOKEN" \
            -H "Content-Type: application/json" \
            -d "$BODY")"
          JOB_ID="$(printf '%s' "$RESP" | jq -r '.id // empty')"
          SHARE="$(printf '%s' "$RESP" | jq -r '.shareUrl // empty')"
          if [ -z "$JOB_ID" ]; then
            gh pr comment "$PR_NUMBER" --body "$MARKER
            ❌ Keynoter couldn't start: $(printf '%s' "$RESP" | head -c 500)"
            exit 1
          fi
          echo "JOB_ID=$JOB_ID" >> "$GITHUB_ENV"
          echo "SHARE_URL=$API$SHARE" >> "$GITHUB_ENV"
          gh pr comment "$PR_NUMBER" --body "$MARKER
          🎬 Keynoter is rendering a demo of this PR…"

      - name: Poll until done
        run: |
          STATUS="queued"; ERR=""
          for _ in $(seq 1 90); do
            sleep 10
            J="$(curl -sS "$API/api/jobs/$JOB_ID" -H "Authorization: Bearer $KEYNOTER_TOKEN")"
            STATUS="$(printf '%s' "$J" | jq -r '.status // "queued"')"
            ERR="$(printf '%s' "$J" | jq -r '.error.message // ""')"
            [ "$STATUS" = "done" ] && break
            [ "$STATUS" = "failed" ] && break
          done
          echo "STATUS=$STATUS" >> "$GITHUB_ENV"
          echo "ERR=$ERR" >> "$GITHUB_ENV"

      - name: Post result
        run: |
          if [ "$STATUS" = "done" ]; then
            gh pr comment "$PR_NUMBER" --body "$MARKER
            ✅ Keynoter demo ready: $SHARE_URL"
          elif [ "$STATUS" = "failed" ]; then
            gh pr comment "$PR_NUMBER" --body "$MARKER
            ❌ Keynoter failed: ${ERR:-unknown error}"
            exit 1
          else
            gh pr comment "$PR_NUMBER" --body "$MARKER
            ⏳ Still rendering — check the studio at $API."
          fi

      - name: Notify Discord
        if: env.STATUS == 'done'
        run: |
          curl -sS -X POST "${{ secrets.DISCORD_WEBHOOK_URL }}" \
            -H "Content-Type: application/json" \
            -d "$(jq -n --arg c "🎬 New Keynoter demo for ${{ github.repository }} #$PR_NUMBER: $SHARE_URL" '{content:$c}')"
```

## 4. After writing

Tell the user:

1. Commit + push the workflow. **A comment-triggered workflow only runs once the
   file is on the repo's DEFAULT branch** (GitHub limitation) — merge it to
   `main` first.
2. Then comment `@keynoter` on any PR (add steering text after it if you want,
   e.g. `@keynoter focus on the new dashboard`).
3. The bot reacts 👍, posts "rendering…", then replies with the share link when
   the MP4 is ready.
