---
name: plugin-manifest-reference
description: Ground truth for Claude Code plugin packaging — the plugin.json and marketplace.json field sets actually in use, how enabledPlugins and extraKnownMarketplaces are keyed, where installed plugins resolve on disk, and why a plugin-bundled CLAUDE.md does not load. Use when Shane asks "what fields does plugin.json support", "can a plugin declare dependencies on another plugin", "why isn't my plugin's CLAUDE.md loading", "where do installed plugin skills live", or is authoring or debugging a plugin manifest. Do NOT use for writing a skill's own SKILL.md (use /write-a-skill) or for marketplace listing/SEO copy.
---

# Skill: /plugin-manifest-reference

The answers to the packaging questions that keep getting re-researched from scratch — separated into what is verified against working manifests and what is still open.

**Don't:** answer from memory about plugin schema; the field set here is what is *observed working*, and anything past it needs the live docs. **Don't** present the open question in §Open as settled — five sessions across three repos have asked it and none produced a citable answer. **Don't** assume a plugin's CLAUDE.md loads.

## Verified: `.claude-plugin/plugin.json`

Observed across 11 shipping plugins (`slogsdon-claude-code-config` ×7, `loop-and-gate-*` ×4):

```json
{
  "name": "skills-vault-knowledge",
  "version": "0.3.0",
  "description": "One line. Shown in plugin listings and the marketplace card.",
  "author": { "name": "Shane Logsdon", "email": "shane@shanelogsdon.com" },
  "license": "MIT"
}
```

`name`, `version`, `description` are load-bearing. `author.email` is optional — present in `loop-and-gate-foundation`, absent from the rest, both install fine. No shipping manifest here declares anything else.

## Verified: `.claude-plugin/marketplace.json`

```json
{
  "name": "slogsdon-claude-code-config",
  "owner": { "name": "Shane Logsdon", "email": "shane@shanelogsdon.com" },
  "metadata": { "description": "Marketplace-level blurb" },
  "plugins": [
    {
      "name": "skills-design",
      "source": { "source": "github", "repo": "slogsdon/skills-design" },
      "description": "Shown on the plugin card — keep the skill count accurate."
    }
  ]
}
```

The `plugins[].description` is a *separate string* from the plugin's own `plugin.json` description. They drift. When you change one, change both.

## Verified: settings keys

- `enabledPlugins` — keyed `"<plugin>@<marketplace>"`, value `true`. Removing the key disables it; the cache stays on disk.
- `extraKnownMarketplaces` — `{ "<marketplace>": { "source": { "source": "github", "repo": "owner/repo" } } }`.
- These live in `~/.claude/settings.json`. A copy in a config repo is **not** deployed by anything unless a script does it — check before assuming edits take effect.

## Verified: where installed plugins resolve

`~/<CLAUDE_CONFIG_DIR>/plugins/cache/<marketplace>/<plugin>/<version>/`, a flat copy at the version tag.

- A `.claude/skills -> ../skills` compatibility symlink ships inside the cache, so `.claude/skills/*/SKILL.md` resolves *within the cache*.
- Cache files are owner-writable. "Read-only cache" is convention, not permission — an edit physically succeeds.
- A plugin session's cwd is the **user's project**, not the cache, so a relative `.claude/skills/...` path does not exist there. Use absolute paths.
- Orphaned plugins get an `.orphaned_at` marker rather than being deleted.
- **Observed divergence:** [claude-code-self-edit-gating](vault: `Knowledge/Projects/Loop & Gate/`) records the cache as "not a git repo". A GitHub-sourced plugin cache observed 2026-08-04 *did* contain `.git`. Verify per case rather than relying on either claim.

## Verified: a plugin's CLAUDE.md does not load

Bundling `CLAUDE.md` at a plugin root does **not** inject it into the user's session. The working pattern is a `SessionStart` hook that prints the content to stdout — this is what `loop-and-gate-foundation` does, and why its operating rules arrive as hook output rather than as a memory file.

Constraint that comes with it: hook stdout is capped near 10K per string. Split across multiple hooks rather than emitting one large block. Stderr does not enter context.

## Open: cross-plugin dependencies

**Unresolved.** "Can `plugin.json` declare a dependency on another plugin?" was asked in five sessions across `loop-and-gate-foundation`, `-build-kit`, and `-accountability-kit`, including two that resolved to "review all documentation, update plugin.json to include the plugin's dependencies". No dependency field appears in any of the four resulting manifests.

That is evidence the answer was *no field was applied* — it is **not** proof no field exists. If you need this settled, check the live docs and record the finding here rather than re-asking:

- `https://code.claude.com/docs/en/plugins`
- `https://code.claude.com/docs/en/plugins-reference`

Until then the working pattern is documentation: state the dependency in the README and in the setup skill, and have setup verify the other plugin is installed.
