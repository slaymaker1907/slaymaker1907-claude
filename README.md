# slaymaker1907-claude

A collection of Claude skills for claude.ai.

Skills are account-wide — once installed they work across Chat and Cowork automatically, on any device. Available for Pro, Max, Team, and Enterprise plans. Mobile is not yet supported.

## Skills

### [`clauude-model-picker`](clauude-model-picker/) — claude.ai (general chat/writing)
Recommends the right model + reasoning mode for general workflows: writing, summarization, analysis, communications. Covers Sonnet and Opus with Thinking ON/OFF. Haiku is not recommended — Sonnet (Thinking OFF) is the reliable floor.

## Installation

### Option 1 — Upload via the web UI

1. Download the packaged `.skill` file *(ZIP format — if you have `clauude-model-picker.skill`, that's it)*
2. Go to **Settings → Customize → Skills → "+"**
3. Upload the `.skill` file

### Option 2 — Install from GitHub with npx

```bash
npx skills add slaymaker1907/slaymaker1907-claude --skill clauude-model-picker
```

If you want all skills in this repo:

```bash
npx skills add slaymaker1907/slaymaker1907-claude
```

### Team / Enterprise

On Team and Enterprise plans you can publish a skill to your organization's directory, making it discoverable by anyone on your org. Shared skills are view-only, and recipients automatically receive updates when the skill is revised.

## Updating an installed skill

After changes are pushed to this repo, update your installed version:

**Via web UI:** Go to **Settings → Customize → Skills**, remove the existing skill, then re-upload the new `.skill` file (see [AGENTS.md](AGENTS.md) for how to repackage it).

**Via npx:**
```bash
npx skills add slaymaker1907/slaymaker1907-claude --skill clauude-model-picker
```

## Contributing / editing

See [AGENTS.md](AGENTS.md) for instructions on editing skills, repackaging `.skill` files, and pushing updates.
