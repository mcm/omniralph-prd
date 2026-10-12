# omniralph-prd

A Claude Code plugin marketplace with one plugin, `omniralph-prd`. The plugin
holds the `omniralph-prd` skill: it turns feature ideas into Linear PRDs (single
issues or epics with sub-issues) for omniralph to build.

## Layout

```
.claude-plugin/marketplace.json          # the marketplace
plugins/omniralph-prd/
  .claude-plugin/plugin.json             # the plugin
  skills/omniralph-prd/SKILL.md          # the skill
  skills/omniralph-prd/references/*.md
```

## Install

```sh
claude plugin marketplace add mcm/omniralph-prd
claude plugin install omniralph-prd@omniralph-prd
```

In a session, `/plugin marketplace add …` and `/plugin install …` do the same.

To install it everywhere from settings (`~/.claude/settings.json` or a
project's `.claude/settings.json`):

```json
{
  "extraKnownMarketplaces": {
    "omniralph-prd": {
      "source": { "source": "github", "repo": "mcm/omniralph-prd" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": { "omniralph-prd@omniralph-prd": true }
}
```

## Updates

`plugin.json` has no `version` on purpose: each commit is a new version, so
installs pick up every change pushed to the default branch. Turn on
auto-update for the marketplace (`/plugin` › Marketplaces › Enable
auto-update, or `autoUpdate` above); updates apply on the next launch or after
`/reload-plugins`. Without it, run
`claude plugin marketplace update omniralph-prd`.

Check the manifests with `claude plugin validate .` (the missing-version
warning is expected).
