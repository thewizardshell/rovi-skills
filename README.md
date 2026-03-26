# ROVI

Personal coding style and architecture skill for AI coding agents.

## What it does

When activated, the agent adopts a specific development philosophy: Clean Architecture with layered separation, interface-first design, strict typing, entity-colocated folder organization, centralized error handling, and direct no-analogy code explanations.

Includes a background documentation lookup agent (`docs-lookup`) that uses Context7 MCP to feed up-to-date docs while you work.

## Stack

TypeScript, JavaScript, Python, Go | React, Next.js, Vue | NestJS, Fastify, Flask, FastAPI, Gin | Jotai, Redux, Zustand, Pinia

## Installation

### Claude Code (personal)

```bash
# Skill
mkdir -p ~/.claude/skills
cp -r rovi ~/.claude/skills/

# Agent (optional — requires Context7 MCP)
mkdir -p ~/.claude/agents
cp .claude/agents/docs-lookup.md ~/.claude/agents/
```

### Claude Code (project-level, shared via git)

```bash
# Skill
cp -r rovi .claude/skills/
git add .claude/skills/rovi

# Agent (optional)
cp .claude/agents/docs-lookup.md .claude/agents/
git add .claude/agents/docs-lookup.md

git commit -m "Add rovi coding style skill"
```

## Structure

```
rovi-skills/
├── rovi/
│   ├── SKILL.md                          # Main skill file
│   └── references/
│       ├── layer-examples.md             # Domain, repository, use case, controller patterns
│       ├── documentation-examples.md     # JSDoc/TSDoc patterns
│       ├── store-examples.md             # State management patterns
│       └── testing-examples.md           # Testing patterns
├── .claude/
│   └── agents/
│       └── docs-lookup.md                # Background docs lookup agent (Context7)
└── README.md
```

## Context7 Setup

The `docs-lookup` agent requires Context7 MCP. If not configured, the skill will stop and ask.

1. Get a free API key at [context7.com/dashboard](https://context7.com/dashboard)
2. Add to `~/.claude/settings.json`:

```json
{
  "env": {
    "CONTEXT7_API_KEY": "your-key-here"
  },
  "permissions": {
    "allow": [
      "mcp__claude_ai_Context7__resolve-library-id",
      "mcp__claude_ai_Context7__query-docs"
    ]
  }
}
```

Without Context7, the skill will offer WebSearch as fallback.

## License

MIT
