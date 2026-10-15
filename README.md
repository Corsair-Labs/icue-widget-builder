# iCUE Widget Builder Skill

An AI-agent skill for building, reviewing, and packaging CORSAIR iCUE widgets.

This repository packages one universal iCUE widget-building skill that can be copied into Claude Code, Codex, OpenCode, or another compatible skill directory. The skill includes the workflow, technical documentation, examples, and reference checklists needed to create widgets for supported CORSAIR device screens such as Xeneon Edge, Pump LCD (all round screens, iCUE LINK TITAN II 360 RX LCD / 5" Module, NAUTILUS II RS LCD / iCUE LINK TITAN II ULTRA 360 LX LCD), and keyboard LCD displays.

## What this skill helps with

Use the iCUE Widget Builder skill when you want an AI coding agent to:

- create a new iCUE HTML widget from a user idea or product requirement;
- modify or debug an existing widget;
- choose the correct widget layout for a target device screen;
- use iCUE plugins such as sensors, media, link, FPS, device action, file system, notifications, and Stream Deck providers;
- declare and manage widget `permissions` in `manifest.json` (network hosts, local files/folders, hardware, clipboard, external applications such as Stream Deck) with least-privilege scoping and graceful handling of denied or revoked access;
- add settings controls, translations, lifecycle handling, and local storage;
- validate widget behavior against the bundled security and testing checklists;
- package the final widget for iCUE installation.

## Requirements

To build and package widgets successfully, install the following:

- **iCUE 5.52 or later** — the bundled documentation targets Widget API 1.9.0 / iCUE 5.52 (manifest `permissions`, File System and Notifications providers, Stream Deck Provider 1.2, Media Provider 1.1). Widgets that avoid these features may run on older iCUE versions; verify against the docs before lowering `min_app_version`.
- **iCUE Widget CLI** — required to validate and package widgets into `.icuewidget` files for installation or distribution.
- **A compatible CORSAIR device** — recommended for final device testing.

Download iCUE from the official CORSAIR downloads page: <https://www.corsair.com/downloads>.

The iCUE Widget CLI is distributed through npm ([`icuewidget-cli`](https://www.npmjs.com/package/icuewidget-cli)). It requires [Node.js](https://nodejs.org/). Install it globally:

```bash
npm install -g icuewidget-cli
```

To update it later, run `npm install -g icuewidget-cli@latest`.

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
Build a Pump LCD widget for NAUTILUS II RS LCD and round pump screens that shows CPU temperature, GPU temperature, and fan speed using iCUE sensor data.
```

```text
Review this iCUE widget package for security, lifecycle issues, and missing metadata before I ship it.
```

The skill is designed to ask clarifying questions first when the device type, data sources, visual style, or packaging requirements are unclear.

## What is included in the skill folder

- `SKILL.md` — the main agent instructions and workflow.
- `docs/` — iCUE technical documentation and control/plugin references, including `docs/permissions.md`.
- `references/` — templates, implementation notes, lifecycle guidance, responsive scaling, widget metadata, `permissions.md` (feature-to-permission map and denied-permission handling), and validation checklists.

## Maintenance notes

When updating the skill:

1. Preserve the skill name `icue-widget-builder` in frontmatter so installed environments can detect it consistently.
2. Keep bundled documentation close to the skill so agents do not rely on stale external knowledge.
3. Update this README whenever the repository structure, install paths, or supported formats change.
4. `docs/` is a mirror of the official iCUE widget docs (`docs/icue/widgets/` in the docs repository), adapted for the skill: Docusaurus front matter, `import` lines, and `<WidgetVersion />` are stripped, `.mdx` links become `.md`, and every void element inside an HTML `<head>` example is self-closed (`<meta ... />`) so the examples are valid XML. Skill-specific additions inside `docs/` are marked with a **Skill note** / **Version caveat** blockquote or live in a clearly labelled section — keep them when re-syncing.
5. When the upstream docs version changes (see the Widget API note in `SKILL.md` → Documentation Authority), re-sync `docs/`, then review `references/permissions.md`, `references/common-tools.md`, and the plugin-version strings in `SKILL.md` and `references/lifecycle-and-plugins.md`.
