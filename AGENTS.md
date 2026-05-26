# Agent Instructions

This repo contains Claude skills for claude.ai.

## Repo structure

```
clauude-model-picker/
  SKILL.md    ← the skill definition (edit this)
README.md
AGENTS.md
```

## Making changes

Edit the relevant `SKILL.md` directly. The YAML frontmatter `description` field controls when the skill auto-triggers — keep it precise.

## Repackaging as a .skill file

Skills for claude.ai are distributed as `.skill` files (ZIP archives). After editing `SKILL.md`, repackage from the repo root:

```bash
zip clauude-model-picker.skill clauude-model-picker/SKILL.md
```

The zip must contain the file at `clauude-model-picker/SKILL.md` (not flat at the root).

## Installing / updating on claude.ai

**Via web UI (re-upload):**
1. Go to **Settings → Customize → Skills**
2. Remove the existing `clauude-model-picker` skill
3. Click **"+"** and upload the new `.skill` file

**Via npx (first install or update):**
```bash
npx skills add slaymaker1907/slaymaker1907-claude --skill clauude-model-picker
```

## Pushing updates to GitHub

After editing and repackaging:

```bash
git add clauude-model-picker/SKILL.md
git commit -m "Update skill: <description of change>"
git push
```
