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

Same account as the IFO product site (`ifo-app` → ifo.tiredithumans.com), which
keeps the two under one bill and one DNS zone.

```sh
npx wrangler pages deploy site --project-name tiredithumans --branch main
```

**Deploy `site/` directly. Do not build a staging copy.** The IFO site needs
one because its source directory also holds an internal README, so its runbook
carries a hand-maintained list of which files to copy — and Pages falls back to
`index.html` for unknown paths, which means a file left out of that list is not
a 404 anyone would notice but the index page served under the wrong
content-type. This repo dodges the whole failure mode by keeping `site/`
exhaustively deployable: **everything in it is meant to be public, and anything
that is not public goes here in `docs/` instead.** Keep it that way.

### DNS

The zone is on Cloudflare. Before this site existed, `tiredithumans.com` and
`www.tiredithumans.com` both answered **error 1016** — proxied records pointing
at an origin that no longer resolves. Attaching the Pages custom domain
replaces them.

The wrangler OAuth token in use has `pages (write)` but only `zone (read)` and
no DNS write, so **the custom-domain step is a dashboard action**, not a
scriptable one: Pages → `tiredithumans` → Custom domains → add
`tiredithumans.com`, then `www.tiredithumans.com`. Cloudflare rewrites the
records itself once the domain is attached. Add a redirect rule from `www` to
the apex if you want one — the page already declares the apex as its
`<link rel="canonical">`, which settles the search-engine half either way.

Verify with a cache-busting query string rather than trusting a URL you already
fetched: `curl -sI 'https://tiredithumans.com/?v=1'`.

## Things that are deliberate

- **This site is indexable; the IFO site is not.** ifo.tiredithumans.com ships
  `X-Robots-Tag: noindex` until that app launches. Both `_headers` and
  `robots.txt` here say why in place, because the two sites are built the same
  way and the wrong half of that pattern is easy to copy across. A company
  website search engines have been told to forget cannot do the job described
  above.
- **No availability claims for IFO.** No store badges, no dates, no "coming
  soon". The card says *in development* and links to the product site. A live
  page announcing a release that has not happened is the one kind of wrong that
  costs trust rather than just needing an edit — the IFO site is written under
  the same rule, and when there is a store link to make, both change together.
- **The "nothing of ours in the middle" claim is scoped to the two desktop
  tools — not the whole paragraph it sits in.** That is a factual claim about
  how software is built, which makes it the sort of sentence that has to move in
  the same change as the behaviour it describes. IFO's version of it lives on
  IFO's own privacy page where it is maintained; widening this one to cover "our
  software" would make this page a second, unwatched copy of a legal statement.
  The licence sentence beside it carries no such claim, so it names every open
  tool — crt-query included, which is MIT/Apache-2.0 but is a CLI querying a
  public third-party database rather than a desktop app under your credentials.
  A new tool goes in the licence sentence; it only joins the second one if the
  claim is actually true of it.
- **The Content-Security-Policy in `_headers` is as strict as the page is
  simple** — `default-src 'none'`, images and styles from `'self'`, nothing
  else. Adding a font, an embed, or an analytics tag will be blocked until the
  policy is widened, which is the intended order of events.
- **`site/mark.svg` is hand-drawn and is the only copy of the mark.** IFO's
  icon is generated from the script that also emits its app-icon PNG,
  specifically so the site's copy cannot drift from the app's. Nothing here has
  a second copy to drift from — so if one is ever cut (an `apple-touch-icon`
  PNG, say), generate it *from this file* rather than redrawing it, and this
  paragraph stops being true.
- **Third-party marks are attributed in the fine print.** Microsoft, NinjaOne
  and the rest are named nominatively, to say what the tools work with. Adding
  a product that names another company's system means adding it there too.

## Checklist when adding a product

1. A card in `index.html` — title, `.meta` line (platforms · status), one
   paragraph, and links that resolve. Featured card for the flagship, the grid
   for the rest.
2. Its trademarks in the fine-print paragraph, if it names anyone else's system.
3. `curl` every link you added. A landing page with a dead link is the one an
   enrolment reviewer opens.
