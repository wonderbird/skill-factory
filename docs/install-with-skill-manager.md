# Installing skills with skill-manager

This fork installs skills per project with [skill-manager](https://github.com/omrikais/skill-manager) (`sm`). Never deploy globally: always pass `--project` to `sm add`.

## How sm gets skills

`sm` imports skills from this repository's default branch on GitHub (`fork`) into its store at `~/.skill-manager/skills/`. A project then symlinks the skills it needs from that store. Your working tree is never read directly.

## First-time setup

Import every skill from this repository into the sm store. This deploys nothing:

```bash
sm source add --install https://github.com/wonderbird/skill-factory
```

The import also stores the vendored skills under `docs/knowledge/` (e.g. `anthropic-skill-creator`). They are harmless and stay undeployed unless you deploy them with `sm add <name> --cc --project`.

## Use a skill in a project

```bash
cd /path/to/your/project
sm add <skill-name> --cc --project   # symlinks into ./.claude/skills/<skill-name>
sm doctor                            # check the deployment
```

`<skill-name>` is the skill's directory name, e.g. `c4-diagrams` for `output_skills/design/c4-diagrams/`.

## After you change a skill

1. Commit and push to `fork` on GitHub.
2. In an interactive terminal, run again:

   ```bash
   sm source add --install https://github.com/wonderbird/skill-factory
   ```

   It asks before updating each changed skill. Without a terminal it skips the updates silently. `sm source sync` alone only pulls, and does not update the store.

3. If you changed bundled files (e.g. `scripts/`), re-importing updates only `SKILL.md`. Purge and re-import the skill:

   ```bash
   sm backup
   sm remove <skill-name> --purge  --force  # deletes it from the store and undeploys it from every project
   sm source add --install https://github.com/wonderbird/skill-factory
   cd /path/to/your/project && sm add <skill-name> --cc --project
   ```

   Re-run `sm add ... --project` in every project that used the skill.

## Do not use

- `sm add` without `--project`: deploys globally.
- `sm import <path>`: deploys globally to Claude Code and Codex.
- `sm import` without a path: bulk-imports everything in `~/.claude` and `~/.agents`.
