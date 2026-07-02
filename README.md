# Blog Engine for Cowork

A free, guided blog-writing skill for [Claude Cowork](https://claude.com). It sets itself up through a
short wizard, then researches, writes, humanises, makes a branded featured image, and delivers each
article either as a WordPress draft or as a document you upload yourself. It reads a topic queue, so it
always knows what to write next, and it runs mostly hands-off: you review, you publish.

Built by [Ikram Nagdawala](https://ikramnagdawala.com).

---

## What it does

- **Guided setup.** Say "set up my blog engine" and answer a few questions (niche, site, voice, brand,
  output mode, cadence). It writes your config and wires everything up. No files to edit by hand.
- **Real research, real facts.** Every article is written from live research, with dated, sourced
  numbers. It doesn't invent prices, stats, or sources.
- **Writes in your voice, in your English.** UK, US, Canadian, Australian, whatever you set. Zero em
  dashes, a built-in banned-words list you can extend, and a house style that reads like a person.
- **Built-in humaniser.** A mandatory pass strips the usual AI tells (flat cadence, no contractions,
  engineered punchlines, rule-of-three lists) so the writing doesn't read as generated. No separate
  humaniser skill needed; it's baked in.
- **SEO done right.** One H1, clean H2/H3, a 50-60 char title, a ~155 char meta, keyphrase placement,
  a cannibalisation guard (one keyphrase per URL), and answer-first structure for featured snippets
  and AI Overviews.
- **Branded images.** A 1200x630 featured image from your brand colour, logo, and (optionally) a real
  photo from your own library. Falls back to a clean brand-colour card if you have no photos.
- **Two output modes** (see below).
- **Remembers its worklist** in `content-queue.json`, not in chat, so it survives across sessions.

## The two output modes

1. **Publish to CMS (WordPress self-hosted).** Delivers each article as a WordPress **draft** with the
   body, featured and in-body images, category, tags, and Yoast SEO fields set. You review and publish.
   Publishing runs on your own computer with a small Node script (the Cowork sandbox can't reach your
   site). Uses a free WordPress Application Password.
2. **Document mode.** Delivers each finished article as a Markdown file **and** a `.docx`, for you to
   upload wherever you publish. No WordPress needed, nothing auto-publishes. This is the safe default.

## Install in Cowork

1. Download this repo (green **Code** button > **Download ZIP**, or `git clone`).
2. In the Claude desktop app: **Settings > Capabilities > Skills**, and add this folder as a skill.
   (If your Cowork build can't add a folder directly, ask Claude in a Cowork chat to package it as a
   `.plugin`/`.skill` bundle you can install in one click.)
3. In a Cowork chat, say: **"set up my blog engine."** The wizard takes it from there.

## Requirements

- **Cowork** (the desktop app with skills).
- **Node.js 18+** — for the Markdown-to-WordPress converter and the publisher. [nodejs.org](https://nodejs.org)
- **Python 3 with Pillow** — for the featured-image generator: `pip install pillow`
- **python-docx** — only for document mode's `.docx` output: `pip install python-docx`

You only need the WordPress bits if you choose CMS mode.

## How a run works

```
set up my blog engine     -> answer the wizard, config written
write my next blog        -> research -> write -> humanise -> image -> convert -> deliver
run this month's batch    -> the next few queue items in one go
```

In CMS mode, the last step on your PC is:
```
node scripts/publish.mjs <slug> --dry    # preview
node scripts/publish.mjs <slug>          # create the draft, then review + publish in WordPress
```
In document mode you just get `drafts/<slug>.md` and `drafts/<slug>.docx`.

## WordPress / Yoast / Elementor known issues

These trip up a lot of WordPress users, so they're worth knowing up front (full detail in
[`references/publishing.md`](references/publishing.md)):

- **The Cowork sandbox has no network to your site.** So publishing to a self-hosted site runs on your
  own PC (the Node script), not in the sandbox.
- **The WordPress.com connector needs a paid Jetpack plan** to publish to a self-hosted site. That's
  why the default route is a free Application Password + REST, not the connector.
- **Setting Yoast SEO via the API needs a Code Snippet.** Add `references/yoast-rest-snippet.php` via
  the free Code Snippets plugin, or the focus keyphrase / SEO title / meta come out blank and you set
  them by hand.
- **Some sites have a broken Yoast block-editor panel.** If yours does, publish via **Posts > Quick
  Edit** rather than the block editor. Updating the free Yoast plugin usually fixes it.
- **Elementor decouples a post from the script.** Don't open a script-managed post in "Edit with
  Elementor" if you want to keep updating it from the queue.

## What's in here

```
SKILL.md                     the orchestrator Cowork runs
config.example.json          copy to config.json (the wizard does this)
content-queue.json           your worklist, seeded with worked examples
references/
  setup-wizard.md            the guided setup flow
  writing-engine.md          research, archetypes, linking, SEO, and the built-in humaniser
  publishing.md              CMS publishing + the known issues above
  yoast-rest-snippet.php     exposes Yoast fields to the REST API
  keyword-map.template.md     optional: one-keyphrase-per-URL map + cannibalisation guard
  site-content-map.template.md optional: your real URLs for internal links
  content-calendar.template.md optional: a longer content plan
scripts/
  gen-featured-image.py      branded 1200x630 image (Pillow)
  md-to-blocks.mjs           Markdown -> WordPress Gutenberg blocks (network-free)
  wp-publish.mjs             create the WordPress draft (run on your PC)
  publish.mjs                friendly wrapper: publish.mjs <slug>
  md-to-docx.py              document mode: Markdown -> .docx
  lib/md-convert.mjs         shared converter used by both Node scripts
brand/                       drop your logo, photos, and fonts here
```

## Licence

MIT. See [LICENSE](LICENSE). Original concept acknowledged in [ACKNOWLEDGEMENTS.md](ACKNOWLEDGEMENTS.md).

## Enjoying it?

This is free. If it saves you time, connect with me on
[LinkedIn](https://www.linkedin.com/in/ikram-nagdawala/) and
[buy me a coffee](https://buymeacoffee.com/ikramn). Both keep me building tools like this.
