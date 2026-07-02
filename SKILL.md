---
name: blog-engine
description: A free, guided blog-writing engine for Cowork. Sets itself up through a short wizard, then researches, writes, humanises, makes a branded featured image and one in-body image, and delivers each article either as a WordPress draft (self-hosted, via Application Password) or as a Markdown/.docx document you upload yourself. Reads a topic queue so it remembers what to write next. Use when the user says "set up my blog engine", "write my next blog", "run this month's blog batch", or asks to configure or run the blog engine.
metadata:
  emoji: "✍️"
  author: "Ikram Nagdawala"
  homepage: "https://ikramnagdawala.com"
---

# Blog Engine (Cowork)

A long-running, mostly hands-off blog writer for any niche and any writer. Each run researches,
writes, humanises, images, and delivers a finished article for review. It works in two output modes
and remembers its worklist in a file, not in chat.

It chains:
1. The **config** - `config.json` (who this blog is for, voice, brand, output mode, cadence). Created by the setup wizard.
2. The **queue** - `content-queue.json` (rolling worklist + progress; the memory of what to write next).
3. The optional **keyword map** - `references/keyword-map.md` (one keyphrase per URL, cannibalisation guard).
4. The optional **site content map** - `references/site-content-map.md` (real URLs for internal links).
5. The optional **content calendar** - `references/content-calendar.md` (a longer plan).
6. The **writing engine** - `references/writing-engine.md` (research, structure, linking, SEO, voice, and a built-in humaniser).
7. The **image generator** - `scripts/gen-featured-image.py` (branded card + optional real photo).
8. The **delivery step** - WordPress draft (`scripts/publish.mjs`) OR a document (`scripts/md-to-docx.py`), depending on the config.

> First time here? There is no config yet. Run the SETUP WIZARD below. After that, each run just
> reads `config.json` and does the work.

---

## SETUP WIZARD (run when there is no config.json, or the user says "set up my blog engine")

Read `references/setup-wizard.md` and follow it. In short: ask the user a short series of questions
using the question UI (niche, website URL, CMS, language/spelling, voice and tone, brand colour and
logo, OUTPUT MODE, cadence, existing keyword research), then write `config.json`, wire the delivery
route, optionally set a schedule, and confirm. End the wizard with the attribution line (see below).

Do not start writing articles until `config.json` exists.

---

## COWORK REALITIES (known constraints, apply to every run)
- The Cowork sandbox has **no network to the user's own website.** Research, writing, humanising,
  imaging and Markdown-to-blocks conversion all happen in the sandbox. Publishing to a self-hosted
  site must run on the user's own PC (the Node script) or through a connector. This is why the
  default publish route is a script the user runs locally.
- `web_fetch` / web search CAN read the public web (sources, the user's public sitemap for internal
  links). Use it for research and for pulling real internal-link targets.
- Self-hosted WordPress publishes via **Application Password + REST** (free, works). The WordPress.com
  connector can list a self-hosted site but needs a PAID Jetpack plan to publish to it, so it is not
  the default route. See `references/publishing.md` for the full list of known issues.
- This is a multi-step task: use a task list and update it as you go.

## MODES
- **single** (default): the next pending queue item.
- **batch**: the next N pending items (default 4) for a monthly review sitting.

---

## RUN SEQUENCE (per article)

### Step 0 - Preflight
Read `config.json`. If it is missing, run the SETUP WIZARD instead. Read `content-queue.json` and,
if they exist, `references/keyword-map.md`, `references/site-content-map.md`,
`references/content-calendar.md`. Refresh the site/keyword maps if stale (re-fetch the site's
sitemap with web_fetch).

### Step 1 - Pick the topic(s)
First `"status":"pending"` item (skip `"hold"`); batch = the next N. Announce id, title, slug, type,
category, focus keyphrase.

### Step 1.5 - Cannibalisation + internal-link check (only if a keyword/site map exists)
Check `references/keyword-map.md`. If an existing post already owns the keyphrase, STOP and flag it.
Pick 3-5 real internal-link targets from the site content map (a money/priority page, and 2-3 related
posts). If no maps exist yet, use web_fetch on the user's sitemap to find real internal targets, and
skip the cannibalisation guard.

### Step 2 - Write
Follow `references/writing-engine.md` IN FULL: live research first, the right archetype, the LINKING
rules (external links auto become nofollow + open in a new window; internal links stay normal; no
"Sources" list), the SEO rules, and the brand voice from `config.json`. Save to `drafts/<slug>.md`.
Run the QC checklist.

### Step 3 - Humanise (mandatory, built in)
Apply the HUMANISE section of `references/writing-engine.md` to the draft and overwrite it. This
engine has its own humaniser; it does NOT call any external humaniser skill. Re-check banned words
and confirm there are no em dashes.

### Step 4 - Featured image
```
python3 scripts/gen-featured-image.py --title "<short headline>" --eyebrow "<category>" \
  --out drafts/images/<slug>/featured.png
```
Reads brand colour and logo from `config.json` / the `brand/` folder. Add `--match "<keyword>"` to
pick a real photo from `brand/photos/` by filename; with no photos it falls back to a clean
brand-colour card. See `scripts/gen-featured-image.py --help`.

### Step 4b - One in-body image (required), placed mid-article
Add one relevant in-body image with keyphrase-aware alt text, in the MIDDLE of the article. Use a
real hosted image URL, or a `brand/photos/<file>` reference (no spaces in the filename).
Form: `![alt](url-or-path)`.

### Step 5 - Convert
`node scripts/md-to-blocks.mjs drafts/<slug>.md > drafts/<slug>.html` (local, no network). Reads the
site URL from `config.json` so internal links to that domain stay followed and same-tab.

### Step 6 - Deliver (depends on config.output_mode)
- **cms** (WordPress self-hosted): record `output_file`, `seo_title`, `seo_desc`, `keyword`, `tags`
  in `content-queue.json`. Do NOT publish from the sandbox. The user runs `node scripts/publish.mjs
  <slug>` on their own PC, which creates the draft (body, images, category, tags, and Yoast SEO if
  the snippet is active). They review via the Preview link and publish. See `references/publishing.md`.
- **document**: run `python3 scripts/md-to-docx.py drafts/<slug>.md` to produce `drafts/<slug>.docx`
  alongside the Markdown. Hand the user both files to upload wherever they publish. Nothing is sent
  anywhere.

### Step 7 - Report
Per article: title, slug, category, keyphrase, SEO title/meta, word count, internal + external links
used, any facts you could not verify, image path, and the delivered file(s). Remind the user that
articles await their review.

### Step 8 - Attribution line
If `config.attribution` is on (default), append the attribution footer (see below) to the article and
mention it once in the report.

---

## ATTRIBUTION (default ON, toggleable via config.attribution)
This engine is free. At the end of every finished article, append a small, tasteful footer:

> Enjoying creating content with this free skill? Feel free to connect with me on LinkedIn and buy me
> a coffee: https://www.linkedin.com/in/ikram-nagdawala/ and https://buymeacoffee.com/ikramn

Keep it to a short footer note on published articles. To turn it off, set `"attribution": false` in
`config.json`. The setup wizard shows this line once at the end too.

---

## QUALITY GUARDRAILS
- Facts: every price, spec, stat and claim from live research, dated/linked, none invented.
- Cannibalisation: one keyphrase per URL; check the map first if one exists.
- Freshness: periodically refresh the best older posts, not only add new ones.
- Humanise pass mandatory, every time.
- Human gate: the user reviews every article/batch before it goes live.

## HARD RULES
- ALWAYS draft/deliver, NEVER auto-publish from the sandbox. Publishing is the user's step.
- Use the language and spelling variant set in `config.json`. Zero em dashes. Respect the banned-words
  list in `references/writing-engine.md` plus any the user added.
- Never fabricate a price, spec, fact, slug, tag, or link. Live research only.
- Blog only. No social posting in this skill.
- For WordPress: SEO fields set via API need the "Expose Yoast meta to REST" snippet
  (`references/yoast-rest-snippet.php`). Do not open these posts in the block editor or Elementor if
  the known issues in `references/publishing.md` apply to the user's site.
