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

| Step name       | What it does                                |
|-----------------|---------------------------------------------|
| `locate`        | Find the article and extract metadata       |
| `humanize`      | Voice pass on the article body              |
| `style`         | MS Writing Style Guide pass                 |
| `hero`          | Generate blog hero artifact                 |
| `hero-shot`     | Screenshot blog hero (OG + in-page)         |
| `frontmatter`   | Wire image filenames into article YAML      |
| `linkedin`      | Generate LinkedIn post artifact             |
| `linkedin-shot` | Screenshot LinkedIn image                   |
| `companion`     | Save companion text to Obsidian             |
| `build`         | Build the site                              |
| `sw-bump`       | Bump service worker cache version           |
| `commit`        | Stage files, commit, push                   |

---

### Step `locate` — Locate the article

```bash
find ./pages/articles -name "<slug>.md" | head -1
```

Read the file. Extract and hold in context:
- `title` — full article title from frontmatter
- `date` — formatted as `YYYY.MM.DD` for artifact datelines
- `description` — from `resources/data/articles-list.json` (key = slug), used as dek
- `category` — derived from the file's directory path (e.g. `pages/articles/strategic-insights/` → `strategic insights`)

---

### Step `humanize` — Text pass: humanize

Invoke the `humanize` skill on the full article body (everything below the YAML frontmatter). The skill edits prose in place. Apply all its rules. Write the result back to the `.md` file using the Edit tool.

---

### Step `style` — Text pass: ms-style-pass

Invoke the `ms-style-pass` skill on the same file, after humanize has run. It operates on what humanize produced — do not re-run humanize after this step. Write the result back to the `.md` file.

---

### Step `hero` — Generate blog hero

Invoke the `design-blog-hero` skill with brand slug `shane-personal-v2` and the following brief:

- **Headline:** article title, sentence-case, ending in a period (max ~12 words)
- **Dek:** article description (from `locate`)
- **Category:** article category (from `locate`)
- **Dateline:** article date formatted `YYYY.MM.DD`
- **Accent word:** the single most specific noun or subject word in the title (the word that names what the article is actually about)

The skill writes: `./design/shane-personal-v2/artifacts/blog-hero-YYYY-MM-DD-<slug>.html`

---

### Step `hero-shot` — Screenshot the blog hero (1x)

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

### Step `frontmatter` — Update article frontmatter

Edit the article's YAML frontmatter to set:
```yaml
image: <slug>-og.png
heroImage: <slug>-hero.png
```

If `image` already exists, replace it. If it doesn't, add both fields after `slug:`.

---

### Step `linkedin` — Generate LinkedIn post

Invoke the `design-linkedin-post` skill with brand slug `shane-personal-v2` and the following brief:

- **Headline:** max 8 words from the title (legibility at feed scale)
- **Supporting line:** article description
- **CTA:** omit
- **Attribution:** omit
- **Companion text style:** insight

The skill writes: `./design/shane-personal-v2/artifacts/linkedin-YYYY-MM-DD-<slug>.html`

---

### Step `linkedin-shot` — Screenshot the LinkedIn image (1x)

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

### Step `companion` — Extract companion text and save to Obsidian

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

### Step `build` — Build the site

```bash
composer build
```

Verify the build succeeds (exit code 0) before continuing.

---

### Step `sw-bump` — Bump the service worker cache version

Read `public/sw.js`. Find the current cache name (pattern: `shane-logsdon-io-static-vN`). Increment N by 1. Use Edit with `replace_all: true` to update both occurrences (in `install` and `activate` handlers).

---

### Step `commit` — Commit and push

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

## Resume protocol

Default: run all twelve steps in order, end-to-end, no checkpoints.

`--resume-from <step>`: skip every step before `<step>` and begin from `<step>`. The named step itself runs. The rest of the pipeline runs end-to-end from there.

If `<step>` is not in `{locate, humanize, style, hero, hero-shot, frontmatter, linkedin, linkedin-shot, companion, build, sw-bump, commit}`, print:

    Invalid resume-from step: <step>
    Valid steps: locate, humanize, style, hero, hero-shot, frontmatter, linkedin, linkedin-shot, companion, build, sw-bump, commit

…and stop.

### Required state when resuming

`locate` always runs first inside any resume — even when `--resume-from <later-step>` is passed — because every later step depends on the slug-derived metadata (title, date, description, category). Treat `locate` as implicit setup, not a step that can itself be skipped from the user's perspective.

State required to start at each step:

- `--resume-from locate` — same as a fresh run; needs the slug.
- `--resume-from humanize` — needs the article file (located via `locate`).
- `--resume-from style` — needs an article that has been through `humanize` (or that the user explicitly says is voice-correct). Confirm before running.
- `--resume-from hero` — needs the cleaned article body (text passes done). Pulls headline / dek / category / dateline from the located article.
- `--resume-from hero-shot` — needs `./design/shane-personal-v2/artifacts/blog-hero-*-<slug>.html`. If absent, prompt the user or run from `hero`.
- `--resume-from frontmatter` — needs `./public/images/<slug>-og.png` and `./public/images/<slug>-hero.png`. If absent, prompt or run from `hero-shot`.
- `--resume-from linkedin` — needs the article metadata; independent of hero artifacts.
- `--resume-from linkedin-shot` — needs `./design/shane-personal-v2/artifacts/linkedin-*-<slug>.html`. If absent, prompt or run from `linkedin`.
- `--resume-from companion` — needs the LinkedIn artifact HTML (companion text is embedded in the comment block) and the LinkedIn screenshot path. If either is absent, prompt or run from `linkedin-shot`.
- `--resume-from build` — independent of prior artifacts; just runs `composer build` against the working tree as it stands.
- `--resume-from sw-bump` — needs a successful build. If `composer build` has not run since the last edit to `public/`, ask whether to run `build` first.
- `--resume-from commit` — needs all final files in place (article, images, sw.js bump, design artifacts, screenshots, components.html). Show `git status` before committing so the user can verify the staged set; abort and ask for direction if the diff looks unexpected.

Examples:

    /publish-post the-ax-shift --resume-from hero          # text passes already done by hand
    /publish-post the-ax-shift --resume-from build         # everything generated, just rebuild + ship
    /publish-post the-ax-shift --resume-from commit        # final retry after a transient build hiccup

## Rules

- **Order is fixed.** humanize before ms-style-pass. Hero before LinkedIn. Screenshots before frontmatter update. Build before commit.
- **1x screenshots always.** `deviceScaleFactor: 1` — never 2. Keeps declared og:image dimensions matching the actual PNG.
- **Companion text to Obsidian, not to the repo.** The companion text note goes to Obsidian only.
- **Stage all design artifacts and screenshots.** Design artifacts, screenshots, and components.html are committed for documentation. Only dist/ is excluded.
- **If design-blog-hero or design-linkedin-post need an accent word and none is obvious**, use the most specific noun in the title — the word that would answer "what is this article actually about?"
- **If the build fails**, stop. Do not commit. Report what failed and suggest `--resume-from build` after the user fixes the underlying issue.
