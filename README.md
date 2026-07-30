# padre

Personal Claude Code skills, kept in one repo so they can be synced between machines.

## Setup on a new machine

Clone this repo, then point `~/.claude/skills` at it with a symlink instead of copying files:

```
git clone git@github.com:<you>/padre.git
rm -rf ~/.claude/skills
ln -s /path/to/padre ~/.claude/skills
```

Claude Code will pick up skills straight from the repo, so edits just need a commit and push to reach the other machine, then a `git pull` there.

## Adding a skill

Create a new directory with a `SKILL.md` inside, commit it, and push. On the other machine, pull to get it.
