---
name: gbp-insight-post
description: Write the caption (hook, summary, call to action) for a Serene Bay insight's Business Profile post, fetch the insight's own image, and look at it before the pull request. Use whenever a post file in posts/ has been prepared.
---

# The caption and image for an insight post

`gbp_post.py prepare` writes the post file with the insight's title,
description and own image address in the frontmatter (`insight_title`,
`insight_description`, `source_image`) and a caption to fill in:

    [HOOK: ...]

    [SUMMARY: ...]

    Read the full insight on our website, using the Learn more button below.

You replace the two placeholders and fetch the image. `check` fails while a
placeholder is left.

## Read the insight first

Open the insight's page (`cta_url`) and read the whole of it, not just the
title. Find the one concrete thing it is about and what a reader should take
from it. The caption comes from that.

## The caption

Three paragraphs, separated by blank lines. Start from the insight's own title
and description, then sharpen them.

1. **Hook** (one line, 150 characters at most). Google shows only the first
   line or two before "More", so this decides whether anyone reads on. Give
   the sharpest true point of the insight: a clear claim, a consequence or a
   direct question to the reader. Not "Exciting news!".
2. **Summary** (two or three sentences, about 250 to 450 characters). What the
   insight says and why it matters to the reader, with facts only from the
   insight.
3. **Call to action** (one line). Keep the default, or make it more specific
   to what the article offers. It must mention the **Learn more** button,
   which carries the link, and point the reader down to it ("below").

Rules:

- Serene Bay's voice as it reads on serenebay.ae: an independent guide for
  people buying off-plan property in Dubai and Abu Dhabi, many of them from
  overseas. Calm, exact and on the buyer's side, with **no urgency**: never
  "don't miss out", "limited units", "act now" or anything that pressures a
  purchase. British spelling, and "per cent" in words as the site writes it.
- No hype, no exclamation marks, no emoji, no hashtags, no links, phone
  numbers or email addresses (`check` rejects these).
- Facts, figures, laws and authorities (DLD, RERA, Oqood, escrow) exactly as
  the insight gives them. No promises of returns, prices or yields, and
  nothing that reads as personal legal or financial advice.
- No relative dates such as "this month" or "next week": the post may go out
  days or weeks after the article was written. Name the month or leave the
  time out.
- Do not claim anything the article does not say: no prices, offers or
  promises that are not in it.
- Keep the whole caption under about 750 characters.

## The image

The post uses the insight's own image, the one the page shares as its
`og:image`. Fetch it:

    python3 gbp_post.py image posts/<file>.md

It downloads `source_image`, converts it to a JPG (at most 2048 pixels on the
long side) and writes `posts/images/<slug>.jpg`. Then open the file and look
at it:

- Is it the insight's real image, not a site logo or a placeholder? Some
  older insights share one of a few images (for example `ins-escrow.jpg`).
  That is fine: it is what the page itself shows. Say in the pull request
  body that the image is shared with other insights, so the reviewer knows.
- Is it sharp, the right way up and not cut off badly? Business Profile may
  trim the sides, so the subject should not sit at the very edge.
- Does any text on it carry a price, phone number or offer that might be out
  of date? If so, say so in the pull request body so the reviewer can decide.

Do not generate or edit images. If no usable image can be found, open the
pull request anyway and say in its body that the image needs a person.

Then run `python3 gbp_post.py check posts/<file>.md`; it must pass.
