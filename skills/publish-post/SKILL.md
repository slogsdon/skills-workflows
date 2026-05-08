---
name: publish-post
description: >
  End-to-end publishing pipeline for a new article on shane.logsdon.io. Runs text quality passes
  (humanize → ms-style-pass), generates brand-system design artifacts (blog hero OG + in-page hero,
  LinkedIn post image), screenshots them at 1x, wires images into the article frontmatter and site,
  saves LinkedIn companion text to Obsidian, builds the site, bumps the service worker cache version,
  and commits + pushes. Unattended — no human checkpoints. Working directory must be the site repo root.
  Triggers: "publish <slug>", "publish the post", "run the publish pipeline", "/publish-post".
---

For drafting upstream of this skill, use `/workflow-stage-draft` — it covers the idea → draft → humanize → ms-style-pass chain that gets a piece ready for publish-post. This skill assumes the draft is already voice-corrected and style-checked.

# Skill: publish-post

Orchestrates the full publishing pipeline for a single article. One invocation, end to end.

## Input

- **Required:** article slug (kebab-case, e.g. `the-ax-shift`)
- Brand slug is always `shane-personal-v2`

## Steps

Run all steps in order. Do not skip steps. Do not pause for review.

---

### Step 1 — Locate the article

```bash
find ./pages/articles -name "<slug>.md" | head -1
```

Read the file. Extract and hold in context:
- `title` — full article title from frontmatter
- `date` — formatted as `YYYY.MM.DD` for artifact datelines
- `description` — from `resources/data/articles-list.json` (key = slug), used as dek
- `category` — derived from the file's directory path (e.g. `pages/articles/strategic-insights/` → `strategic insights`)

---

### Step 2 — Text pass: humanize

Invoke the `humanize` skill on the full article body (everything below the YAML frontmatter). The skill edits prose in place. Apply all its rules. Write the result back to the `.md` file using the Edit tool.

---

### Step 3 — Text pass: ms-style-pass

Invoke the `ms-style-pass` skill on the same file, after humanize has run. It operates on what humanize produced — do not re-run humanize after this step. Write the result back to the `.md` file.

---

### Step 4 — Generate blog hero

Invoke the `design-blog-hero` skill with brand slug `shane-personal-v2` and the following brief:

- **Headline:** article title, sentence-case, ending in a period (max ~12 words)
- **Dek:** article description (from step 1)
- **Category:** article category (from step 1)
- **Dateline:** article date formatted `YYYY.MM.DD`
- **Accent word:** the single most specific noun or subject word in the title (the word that names what the article is actually about)

The skill writes: `./design/shane-personal-v2/artifacts/blog-hero-YYYY-MM-DD-<slug>.html`

---

### Step 5 — Screenshot the blog hero (1x)

Find the artifact:
```bash
ls -t ./design/shane-personal-v2/artifacts/blog-hero-*-<slug>.html | head -1
```

Run this Node script, substituting the actual artifact path:

```javascript
const puppeteer = require('puppeteer');
const path = require('path');
const ARTIFACT = '<artifact-path-from-above>';
const OUT = path.resolve('./public/images');
(async () => {
  const browser = await puppeteer.launch({
    headless: true,
    args: ['--no-sandbox', '--disable-web-security']
  });
  const page = await browser.newPage();
  await page.goto('file://' + ARTIFACT, { waitUntil: 'networkidle0' });
  await page.setViewport({ width: 1200, height: 630, deviceScaleFactor: 1 });
  const og = await page.$('.canvas-og');
  await og.screenshot({ path: OUT + '/<slug>-og.png', type: 'png' });
  await page.setViewport({ width: 1440, height: 600, deviceScaleFactor: 1 });
  const hero = await page.$('.canvas-hero');
  await hero.screenshot({ path: OUT + '/<slug>-hero.png', type: 'png' });
  await browser.close();
  console.log('done');
})();
```

**Critical:** `deviceScaleFactor: 1` always. 2x PNGs cause LinkedIn CDN blur because the declared og:image dimensions won't match the actual file dimensions.

---

### Step 6 — Update article frontmatter

Edit the article's YAML frontmatter to set:
```yaml
image: <slug>-og.png
heroImage: <slug>-hero.png
```

If `image` already exists, replace it. If it doesn't, add both fields after `slug:`.

---

### Step 7 — Generate LinkedIn post

Invoke the `design-linkedin-post` skill with brand slug `shane-personal-v2` and the following brief:

- **Headline:** max 8 words from the title (legibility at feed scale)
- **Supporting line:** article description
- **CTA:** omit
- **Attribution:** omit
- **Companion text style:** insight

The skill writes: `./design/shane-personal-v2/artifacts/linkedin-YYYY-MM-DD-<slug>.html`

---

### Step 8 — Screenshot the LinkedIn image (1x)

Find the artifact:
```bash
ls -t ./design/shane-personal-v2/artifacts/linkedin-*-<slug>.html | head -1
```

Run:

```javascript
const puppeteer = require('puppeteer');
const path = require('path');
const ARTIFACT = '<artifact-path-from-above>';
const OUT = path.resolve('./design/shane-personal-v2/screenshots');
(async () => {
  const browser = await puppeteer.launch({
    headless: true,
    args: ['--no-sandbox', '--disable-web-security']
  });
  const page = await browser.newPage();
  await page.goto('file://' + ARTIFACT, { waitUntil: 'networkidle0' });
  await page.setViewport({ width: 1200, height: 627, deviceScaleFactor: 1 });
  const canvas = await page.$('.canvas');
  await canvas.screenshot({ path: OUT + '/linkedin-<slug>.png', type: 'png' });
  await browser.close();
  console.log('done');
})();
```

---

### Step 9 — Extract companion text and save to Obsidian

Extract the companion text from the HTML comment block at the end of the LinkedIn artifact:

```bash
awk '/COMPANION TEXT:/{found=1; next} /-->/{if(found) exit} found{print}' \
  ./design/shane-personal-v2/artifacts/linkedin-*-<slug>.html
```

Invoke the `obsidian` skill to create a note at `Publishing/LinkedIn/<slug>.md` with this content:

```markdown
# <title>

**Published:** <YYYY-MM-DD>
**Article:** https://shane.logsdon.io/<type>/<category>/<slug>/
**Image:** `design/shane-personal-v2/screenshots/linkedin-<slug>.png`

---

<companion text verbatim>
```

After writing, commit the vault:
```bash
VAULT="$HOME/Library/Mobile Documents/iCloud~md~obsidian/Documents/Personal"
git -C "$VAULT" add -A && git -C "$VAULT" commit -m "docs: linkedin companion for <slug>"
```

---

### Step 10 — Build the site

```bash
composer build
```

Verify the build succeeds (exit code 0) before continuing.

---

### Step 11 — Bump the service worker cache version

Read `public/sw.js`. Find the current cache name (pattern: `shane-logsdon-io-static-vN`). Increment N by 1. Use Edit with `replace_all: true` to update both occurrences (in `install` and `activate` handlers).

---

### Step 12 — Commit and push

Stage these files:
```
pages/articles/**/<slug>.md
public/images/<slug>-og.png
public/images/<slug>-hero.png
public/sw.js
design/shane-personal-v2/artifacts/blog-hero-*-<slug>.html
design/shane-personal-v2/artifacts/linkedin-*-<slug>.html
design/shane-personal-v2/screenshots/linkedin-<slug>.png
design/shane-personal-v2/components/components.html
```

Do not stage dist/.

Commit message:
```
feat: publish <slug>

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Push to origin.

---

## Rules

- **Order is fixed.** humanize before ms-style-pass. Hero before LinkedIn. Screenshots before frontmatter update. Build before commit.
- **1x screenshots always.** `deviceScaleFactor: 1` — never 2. Keeps declared og:image dimensions matching the actual PNG.
- **Companion text to Obsidian, not to the repo.** The companion text note goes to Obsidian only.
- **Stage all design artifacts and screenshots.** Design artifacts, screenshots, and components.html are committed for documentation. Only dist/ is excluded.
- **If design-blog-hero or design-linkedin-post need an accent word and none is obvious**, use the most specific noun in the title — the word that would answer "what is this article actually about?"
- **If the build fails**, stop. Do not commit. Report what failed.
