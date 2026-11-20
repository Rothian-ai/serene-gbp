# Routine

One weekly routine does both halves of the job:

1. **Publish** any approved post that is not on the profile yet (one per run).
2. **Prepare** the next serenebay.ae insight: write a caption (a hook, a short
   summary from its title and description, and a call to action), fetch the
   insight's own image, and open a draft pull request for review.

Merging the pull request is the approval. It also puts the image on `main`,
which is where Google fetches it from, so a post can only go out after a person
has merged it.

**Weekly, not daily**, while Serene's profile is new: Google watches new
profiles more closely, and a steady, modest rhythm of real content is the
safest. Move to a shorter interval later if you want.

**This repository has to be public**: Google takes a post's image only as a
public URL, and the images are served from `main` through
`raw.githubusercontent.com`. Secrets live in the routine environment, never in
the repository.

## Environment

Create a new environment, `Serene GBP`. Do not reuse Rothian's: its
credentials post to Rothian's profile.

**Variables** (plain text):

```
GBP_ACCOUNT_ID=<from gbp_auth.py, signed in as Serene>
GBP_LOCATION_ID=<from gbp_auth.py, Serene's verified profile>
GBP_SITE_NAME=Serene
GBP_SITEMAP_URL=https://www.serenebay.ae/sitemap.xml
GBP_PATH_PREFIX=/insights/
GBP_LINK_PREFIX=https://www.serenebay.ae/
GBP_IMAGE_BASE_URL=https://raw.githubusercontent.com/Rothian-ai/serene-gbp/main/posts/images/
GBP_POST_SINCE=2000-01-01
GBP_DRY_RUN=true
```

- `GBP_POST_SINCE=2000-01-01` takes every insight on the site, from the first
  one onwards, oldest first, one per run.
- `GBP_DRY_RUN=true` until the first dry run has been checked, then `false`.
  While it is `true`, nothing is posted to Google, but pull requests are still
  prepared, so you can review the first posts.

**API credentials** (secrets, never in the variables box):

```
GBP_CLIENT_ID=<same as Rothian's>
GBP_CLIENT_SECRET=<same as Rothian's>
GBP_REFRESH_TOKEN=<from gbp_auth.py, signed in as Serene>
POWERAUTOMATE_WEBHOOK_URL=<same as Rothian's>
```

No OpenAI key: Serene's images are not generated.

**Network policy:** allow `oauth2.googleapis.com`, `mybusiness.googleapis.com`,
`www.serenebay.ae`, the host that serves the insights' images if it is not
`www.serenebay.ae`, `raw.githubusercontent.com`, the Teams webhook's host, and
PyPI if not already allowed.

**Setup script:**

```bash
#!/bin/bash
# No set -e: a failed install is retried by the routine after checkout.
set -uo pipefail
python3 -m pip install --quiet --disable-pip-version-check requests "Pillow>=11" \
  && echo "ok: requests, Pillow" \
  || echo "WARN: pip install failed; the routine should retry it"
```

Use `python3 -m pip`, not `pip`: on these machines `pip` can belong to a
different Python than `python3`.

## The routine

- **Name:** Serene: Business Profile posts
- **Repository:** `Rothian-ai/serene-gbp`
- **Trigger:** weekly, for example Tuesday 10:07 local time
- **Environment:** `Serene GBP`

Instructions:

```
Share Serene's insights on its Google Business Profile: publish what has been
approved, then prepare the next insight for approval. A Business Profile post
is public immediately, so gbp_post.py is the only thing that posts: never call
the Business Profile API yourself, never merge a pull request, and never post
anything that has not been merged into main.

1. Run: python3 -m pip install -r requirements.txt
   Then: git checkout main && git pull

PUBLISH
2. If the variable GBP_DRY_RUN is "true", run
       python3 gbp_post.py publish --dry-run
   and keep its full output for the notification at the end.
   Otherwise run
       python3 gbp_post.py publish
   It posts at most one approved post that is due and not already on the
   profile, and sends its own Teams message. "nothing new to post" is normal.
   If it exits non-zero, it has already reported the error to Teams: note the
   error and continue to PREPARE.

PREPARE
3. Run: python3 gbp_post.py prepare
   If it prints "nothing new to prepare", skip to the end.
   Otherwise it prints PREPARED <file> and the branch name to use.
4. Read .claude/skills/gbp-insight-post/SKILL.md and follow it: read the
   insight, replace the [HOOK] and [SUMMARY] placeholders in the post's text
   with a hook and a short summary drawn from the insight's title and
   description, keep the call to action as the last paragraph, run
   `python3 gbp_post.py image <file>` to fetch the insight's own image, and
   look at the image before going on.
5. Run `python3 gbp_post.py check <file>`; it must pass.
6. Create the branch it named (post/<slug>), commit the post file and its image
   with `git -c commit.gpgsign=false commit`, and push. Open a DRAFT pull
   request against main with the GitHub MCP tools: title "GBP: <insight
   title>", body with the post text, the insight's own title and description,
   the insight URL and the image's source address.
7. Run `python3 gbp_post.py notify-pr <PR URL> <file>`.

If a PREPARE step fails, do not open a half-finished pull request: post
{"event":"run_failed","reason":"<what failed>"} to POWERAUTOMATE_WEBHOOK_URL.

At the end, send a notification only if something needs a person: a pull
request to review, a publish error, or the dry-run output while GBP_DRY_RUN is
"true". On a day with nothing published and nothing prepared, stay silent.

An error mentioning invalid_grant means gbp_auth.py must be run again, signed
in as Serene's Google account, and the new GBP_REFRESH_TOKEN saved in the
Serene GBP environment.
```
