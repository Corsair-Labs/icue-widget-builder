# iCUE Widget Builder Skill

An AI-agent skill for building, reviewing, and packaging CORSAIR iCUE widgets.

This repository packages one universal iCUE widget-building skill that can be copied into Claude Code, Codex, OpenCode, or another compatible skill directory. The skill includes the workflow, technical documentation, examples, and reference checklists needed to create widgets for supported CORSAIR device screens such as Xeneon Edge, Pump LCD, and keyboard LCD displays.

## What this skill helps with

Use the iCUE Widget Builder skill when you want an AI coding agent to:

- create a new iCUE HTML widget from a user idea or product requirement;
- modify or debug an existing widget;
- choose the correct widget layout for a target device screen;
- use iCUE plugins such as sensors, media, and link providers;
- add settings controls, translations, lifecycle handling, and local storage;
- validate widget behavior against the bundled security and testing checklists;
- package the final widget for iCUE installation.

## Requirements

To build and package widgets successfully, install the following:

- **iCUE 5.44 or later** — required to run and test iCUE widgets.
- **iCUE Widget CLI** — required to validate and package widgets into `.icuewidget` files for installation or distribution.
- **A code editor** — any editor works, but a full code editor is recommended.
- **A compatible CORSAIR device** — recommended for final device testing.

Download iCUE and the iCUE Widget CLI from the official CORSAIR downloads page: <https://www.corsair.com/downloads>.

After installing the CLI, verify it is available in your terminal:

```bash
icuewidget --version
```

The skill can still help create widget files manually, but packaging requires the CLI:

```bash
icuewidget validate MyWidget
icuewidget package MyWidget
```

## Repository structure

```text
icue-widget-skill/
├── README.md
└── skills/
    └── icue-widget-builder/
        ├── SKILL.md
        ├── docs/
        └── references/
```

### Skill format

The repository keeps a single universal skill folder: `skills/icue-widget-builder/`.

The skill uses the common Markdown skill shape supported by Claude Code and Codex-style skill loaders:

```text
icue-widget-builder/
├── SKILL.md
├── docs/
└── references/
```

## Other formats that are possible

The skill content is plain Markdown plus bundled reference files, so it can be adapted to other agent systems. Common options include:

- **OpenCode skill** — install the skill under `.opencode/skills/icue-widget-builder/` or `~/.config/opencode/skills/icue-widget-builder/` with a `SKILL.md` file. OpenCode can also auto-load external skills from common Claude/Codex skill locations in some configurations.
- **Generic prompt pack** — use `SKILL.md` as a reusable system/developer prompt and keep `docs/` plus `references/` beside it.
- **Custom agent profile** — convert the workflow into an agent instruction file for tools that support named agents rather than skills.
- **MCP-backed workflow** — expose iCUE widget packaging or validation as MCP tools, while keeping this skill as the agent-facing operating guide.

This repository currently ships one ready-to-copy skill folder. Platform-specific install paths are documented below; the skill content stays the same.

## Installation

### Claude Code

Copy the Claude Code skill folder into your Claude skills directory:

```bash
mkdir -p ~/.claude/skills
cp -R skills/icue-widget-builder ~/.claude/skills/
```

Restart Claude Code so it reloads the skill list.

### Codex

Copy the Codex skill folder into your Codex skills directory:

```bash
mkdir -p ~/.agents/skills
cp -R skills/icue-widget-builder ~/.agents/skills/
```

Restart Codex so it reloads the skill list.

### OpenCode

Copy the same skill folder into an OpenCode skill directory:

```bash
mkdir -p ~/.config/opencode/skills
cp -R skills/icue-widget-builder ~/.config/opencode/skills/
```

Restart OpenCode so it reloads the skill list.

## Example prompts

After installation, ask your agent for iCUE widget work in natural language:

```text
Create a Xeneon Edge weather widget with the current temperature, 3-day forecast, and a subtle animated background.
```

```text
Build a Pump LCD widget that shows CPU temperature, GPU temperature, and fan speed using iCUE sensor data.
```

```text
Review this iCUE widget package for security, lifecycle issues, and missing metadata before I ship it.
```

The skill is designed to ask clarifying questions first when the device type, data sources, visual style, or packaging requirements are unclear.

## What is included in the skill folder

- `SKILL.md` — the main agent instructions and workflow.
- `docs/` — iCUE technical documentation and control/plugin references.
- `references/` — templates, implementation notes, lifecycle guidance, responsive scaling, widget metadata, and validation checklists.

## Maintenance notes

When updating the skill:

1. Preserve the skill name `icue-widget-builder` in frontmatter so installed environments can detect it consistently.
2. Keep bundled documentation close to the skill so agents do not rely on stale external knowledge.
3. Update this README whenever the repository structure, install paths, or supported formats change.

## License and ownership

This repository is intended for internal iCUE widget-building workflows. Add project-specific license, contribution, and ownership details here before publishing publicly.
