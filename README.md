# serene-gbp

Shares each insight from serenebay.ae/insights on Serene's Google Business
Profile, with the insight's own image, after a person approves it. It is a copy
of `Rothian-ai/rothian-gbp`, adapted for Serene; the two run separately, with
their own environment, routine and credentials, and cannot post to each
other's profile.

```
Weekly routine                        You                       Weekly routine (next run)
finds the next insight                look at the post and      posts it to the profile
writes hook, summary and CTA          its image in the PR,      with the image, and
fetches the insight's own image       then merge                tells Teams
opens a draft PR, tells Teams
```

A Business Profile post is public the moment it is made, so merging the pull
request is the approval: the routine only posts what is on `main`. Google takes
a post's image only as a public URL, and the images are served from `main`, so
**this repository must be public** (see ROUTINES.md).

Do not edit a published post on Google: change the post file in its pull
request before merging.

To skip an insight, close its pull request without merging and leave its
`post/<slug>` branch in place: the routine treats an existing branch as
handled. Deleting the branch puts the insight back in the queue.

## Files

| File | Purpose |
|---|---|
| `posts/` | One markdown file per post, images in `posts/images/` |
| `gbp_post.py` | `prepare`, `image`, `check`, `notify-pr`, `publish`, `list` |
| `gbp_auth.py` | One-time Google sign-in that produces the refresh token and IDs |
| `.claude/skills/gbp-insight-post/SKILL.md` | How the caption is written and the image checked |
| `ROUTINES.md` | Environment, setup script and the routine prompt |

## Google setup

Serene uses **Rothian's** Google Cloud project, which is already approved for
the Business Profile APIs and whose consent screen is In production. Nothing
new is needed there.

On your own computer, signed in to Google as **Serene's** account (the owner
or a manager of Serene's profile):

    python3 -m pip install requests
    python3 gbp_auth.py --client-id <Rothian's client id> --client-secret <secret>

It prints a refresh token and Serene's account and location IDs. They go in
Serene's environment (ROUTINES.md), never in Rothian's.

## The image

Each post uses the image the insight page shares (its `og:image`).
`gbp_post.py image` downloads it and converts it to a JPG Google accepts:
RGB, at most 2048 pixels on the long side, at least 250 pixels on each side
and 10 KB. Nothing is generated.

## What `check` enforces

- 80 to 1,500 characters of text.
- A hook of at most 150 characters, a summary, and a last paragraph that
  mentions the Learn more button; no `[HOOK]`/`[SUMMARY]` placeholder and no
  ellipsis left from a cut-off description.
- No phone numbers, email addresses, social handles, links or hashtags in the
  text: Google may remove posts with unverified contact details, and the
  button carries the link.
- The button links to a live serenebay.ae page.
- The image exists, is a JPG or PNG of at least 10 KB and 250 x 250 pixels.

`publish` also refuses to post if Google could not fetch the image from its
public URL.
