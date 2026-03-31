# Cloudglue Agent Skills

Official [agent skills](https://agentskills.io) for coding agents working with the [Cloudglue](https://cloudglue.dev) API / SDK.

## Available Skills

| Skill | Description |
|-------|-------------|
| [`cloudglue`](skills/cloudglue/SKILL.md) | Turn video into LLM-ready data with the Cloudglue JS SDK |

## Installation

### Skills CLI (recommended)

```bash
npx skills add cloudglue/skills
```

### Direct Clone

```bash
git clone https://github.com/cloudglue/skills.git
```

## Usage

These skills follow the [agentskills.io](https://agentskills.io) specification. Compatible agents (Claude Code, Cursor, etc.) will automatically discover and load skills when they're relevant to your task.

The skills provide agents with:
- **Core concepts** — what Cloudglue is and how its APIs fit together
- **Documentation hierarchy** — where to find the most accurate, version-locked information
- **Code patterns** — correct method signatures, parameters, and end-to-end workflows
- **Troubleshooting** — common errors and how to fix them

## Documentation Hierarchy

The Cloudglue skill teaches agents to use a 3-tier documentation strategy:

1. **Embedded docs** in `node_modules/@cloudglue/cloudglue-js/docs/` — version-locked API reference (most reliable)
2. **Source code** in `node_modules/@cloudglue/cloudglue-js/src/api/` — full type information
3. **Remote docs** at `https://docs.cloudglue.dev/llms.txt` — conceptual guides and deep dives

## License

Apache-2.0
