# padre

Personal Claude skills, packaged as a single plugin (`padre`) in a single-plugin marketplace, so the same skills reach Claude Code, Claude Desktop and Claude in the cloud.

## Layout

- `skills/<name>/SKILL.md` holds one skill each.
- `.claude-plugin/plugin.json` is the plugin manifest.
- `.claude-plugin/marketplace.json` makes this repo installable as a marketplace.

`humanizer` is a copy of [blader/humanizer](https://github.com/blader/humanizer) (MIT, licence alongside it). Refresh it by hand when upstream changes.

## Install

In Claude Code:

```
/plugin marketplace add giorgiodoueihi/padre
/plugin install padre@padre
```

On a machine with the repo already cloned, `/plugin marketplace add /path/to/padre` works too.

In Claude Desktop and Claude on the web, add the same GitHub repo as a plugin marketplace from the plugin settings, then install `padre`.

## Adding or changing a skill

Create `skills/<name>/SKILL.md`, bump `version` in `.claude-plugin/plugin.json`, commit and push. Each client picks up the change on its next marketplace update (`/plugin marketplace update padre` in Claude Code).

Skills from this plugin are namespaced, for example `padre:slop-check`.

## Do not symlink this repo to `~/.claude/skills`

Claude's account sync writes its own `synced/` folder into `~/.claude/skills`, which would land inside this repo. Leave `~/.claude/skills` as a normal directory.
