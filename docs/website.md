# tiredithumans.com

The company's public front door. Source in [`site/`](../site); six files, one
stylesheet, no build step, no JavaScript, and no external assets.

## Why it exists

Two reasons, and the second one has a deadline attached:

1. It is where someone who meets a Tired IT Humans tool finds out who made it.
2. **Apple Developer Program enrolment as an *Organization* requires a publicly
   available website on a domain associated with the organization**, and the
   reviewer checks that the legal entity name on it matches the D-U-N-S record.
   Google Play's organization verification asks for the same thing.

That second reason is why `Tired IT Humans LLC` — the name exactly as filed —
appears three times on one short page: the eyebrow above the headline, the
contact block, and the copyright line. If the filed name ever changes, all
three move in the same commit. Nothing else on the page states it, so there is
nothing else to hunt down.

## Where it is published

**Cloudflare Pages, project `tiredithumans`** → <https://tiredithumans.com/>

```sh
npx wrangler pages deploy site --project-name tiredithumans --branch main
```

**Deploy `site/` directly. Do not build a staging copy.** Pages falls back to
`index.html` for unknown paths, so a file left out of a hand-maintained copy
list is not a 404 anyone would notice but the index page served under the wrong
content-type. This repo avoids that by keeping `site/` exhaustively deployable:
**everything in it is meant to be public, and so is everything else in this
repository, its history included.** Account, DNS and hosting-settings notes are
kept privately, not here.

Verify with a cache-busting query string rather than trusting a URL you already
fetched: `curl -sI 'https://tiredithumans.com/?v=1'`.

### What the host changes on the way out

**The page a visitor gets is not always the page in `site/`.** The host can
rewrite responses as they leave the edge, and because the policy here allows no
scripts at all, a rewrite that injects one fails silently in the source and
visibly on the live page at the same time: a rewritten `mailto:` whose decode
script is blocked renders as the literal text `[email protected]`.

`site/_headers` sends `Cache-Control: no-transform` on the HTML, which keeps
the served page byte-identical to the source; the comment in that file carries
the reasoning.

**`site/robots.txt` is not served verbatim.** A managed block (a content-signals
policy and `Disallow: /` for known AI-training crawlers) is prepended above the
file in this repo. It leaves search indexing intact. Fetch the file to see what
is served; do not read `site/robots.txt` and assume.

**A hosting setting is part of this site's behaviour and is not in this repo.**
Verify the served page, not the file you edited.

## Things that are deliberate

- **This site is indexable, and that is the one thing not to copy from a
  pre-launch product site.** A product site that has not shipped may carry
  `X-Robots-Tag: noindex` or sit behind a login, and the wrong half of that
  pattern is easy to carry across; both `_headers` and `robots.txt` here say so
  in place. A company website search engines have been told to forget cannot do
  the job described above. (`site/robots.txt` is not served verbatim; see
  "What the host changes on the way out" above.)
- **No availability claims, and no links a visitor cannot open.** Every card
  on the page is a shipped tool whose links resolve publicly: no store badges,
  no dates, no "coming soon", and nothing pointing at a page behind a login. A
  live page announcing a release that has not happened is the one kind of wrong
  that costs trust rather than just needing an edit; a link that dead-ends at
  an access wall is the same failure with a shorter fuse, because the visitor
  most likely to follow it is the enrolment reviewer. If an unreleased product
  is ever listed here, the card describes what it is and says nothing about
  when.
- **The "nothing of ours in the middle" claim is scoped to the two desktop
  tools — not the whole paragraph it sits in.** That is a factual claim about
  how software is built, which makes it the sort of sentence that has to move in
  the same change as the behaviour it describes — so it covers only the products
  whose behaviour this page can speak for. A product that maintains its own
  privacy statement keeps its version of the claim there; widening this one to
  cover "our software" would make this page a second, unwatched copy of a legal
  statement. The licence sentence beside it carries no such claim, so it names
  every open tool — crt-query included, which is MIT/Apache-2.0 but is a CLI
  querying a public third-party database rather than a desktop app under your
  credentials.
  A new tool goes in the licence sentence; it only joins the second one if the
  claim is actually true of it.
- **The Content-Security-Policy in `_headers` is as strict as the page is
  simple** — `default-src 'none'`, images and styles from `'self'`, nothing
  else. Adding a font, an embed, or an analytics tag will be blocked until the
  policy is widened, which is the intended order of events. The script that
  trips this policy is likelier to be one the host injected than one you added,
  and a blocked injection shows up nowhere in the source. See "What the host
  changes on the way out" above.
- **`site/mark.svg` is hand-drawn and is the only copy of the mark.** Nothing
  here has a second copy to drift from — so if one is ever cut (an
  `apple-touch-icon` PNG, say), generate it *from this file* rather than
  redrawing it, and this paragraph stops being true.
- **Third-party marks are attributed in the fine print.** Microsoft, NinjaOne
  and the rest are named nominatively, to say what the tools work with. Adding
  a product that names another company's system means adding it there too.

## Checklist when adding a product

1. A card in the grid in `index.html` — title, `.meta` line (platforms ·
   status), one paragraph, and links that resolve. Every card carries the same
   weight; there is no featured slot.
2. Its trademarks in the fine-print paragraph, if it names anyone else's system.
3. `curl` every link you added, unauthenticated. A landing page with a dead
   link — or one that lands on a login page — is the one an enrolment reviewer
   opens.
4. Deploy, then **diff the served page against the source** rather than trusting
   the deploy output:
   `curl -s 'https://tiredithumans.com/?v=1' | diff - site/index.html`
   An empty diff is the pass condition. Anything else is the host rewriting
   the page on the way out — and an unexpected `<script>` in that diff is being
   blocked by the CSP, which means whatever it was meant to do is silently not
   happening. This check is what turns a broken contact address into a
   two-minute fix instead of something a reviewer finds first.
