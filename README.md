# Astro Skills

Claude Code plugin providing stack-specific skills and agents for Astro framework development.

## Installation

### Add the Marketplace

```bash
/plugin marketplace add pddhkt/astro-skills
```

### Install the Plugin

```bash
# Project scope (recommended)
/plugin install astro@astro-skills --scope project

# User scope
/plugin install astro@astro-skills
```

## Available Skills

| Skill | Usage | Description |
|-------|-------|-------------|
| astro | `/astro:astro` | Astro framework patterns, routing, components |
| content | `/astro:content` | Content collections and markdown |
| islands | `/astro:islands` | Islands architecture and interactive components |
| reflect | `/astro:reflect` | Push skill improvements back to source repo |

## Included Agents

| Agent | Description |
|-------|-------------|
| astro-impl | Astro framework implementation specialist |
| content-impl | Content collections specialist |
| api-impl | API endpoints specialist |
| islands-impl | Islands architecture specialist |

## Stack Detection

This stack is auto-detected when a project contains:
- `astro.config.mjs` or `astro.config.ts` file
- `astro` package in dependencies

## Updating

```bash
/plugin marketplace update astro-skills
```

## Repository Structure

```
astro-skills/
├── .claude-plugin/
│   └── marketplace.json
├── stack.json
│
└── plugins/
    └── astro/
        ├── .claude-plugin/
        │   └── plugin.json
        ├── agents/
        │   ├── astro-impl.md
        │   ├── content-impl.md
        │   ├── api-impl.md
        │   └── islands-impl.md
        └── skills/
            ├── astro/
            ├── content/
            ├── islands/
            └── reflect/
```

## License

MIT
