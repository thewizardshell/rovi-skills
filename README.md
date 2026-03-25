# ROVI

Personal coding style and architecture skill for AI coding agents.

## What it does

When activated, the agent adopts a specific development philosophy: Clean Architecture with layered separation, interface-first design, strict typing, feature-based folder organization, centralized error handling, and direct no-analogy code explanations.

## Stack

TypeScript, JavaScript, Python, Go | React, Next.js, Vue | NestJS, Fastify, Flask, FastAPI, Gin | Jotai, Redux, Zustand, Pinia

## Installation

### Claude Code (personal)

```bash
mkdir -p ~/.claude/skills
cp -r rovi ~/.claude/skills/
```

### Claude Code (project-level, shared via git)

```bash
cp -r rovi .claude/skills/
git add .claude/skills/rovi
git commit -m "Add rovi coding style skill"
```

### OpenCode / Codex

```bash
cp -r rovi ~/.codex/skills/
```

## Structure

```
rovi/
├── SKILL.md                          # Main skill file
├── references/
│   ├── layer-examples.md             # Domain, repository, use case, controller patterns
│   ├── documentation-examples.md     # JSDoc/TSDoc patterns
│   ├── store-examples.md             # State management patterns
│   └── testing-examples.md           # Testing patterns
└── README.md
```

## License

MIT
