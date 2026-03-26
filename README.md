# ROVI Skills

Modular coding style and architecture skills for Claude Code.

## Skills

| Skill | Type | Description |
|-------|------|-------------|
| `rovi` | Auto-invoked | Core philosophy: communication, thinking process, style, typing, errors, docs, hard rules |
| `rovi-architecture` | Auto-invoked | Clean Architecture, layered separation, folder structure, patterns |
| `rovi-store` | Auto-invoked | State management: one store per entity, loading/error state, optimistic updates |
| `rovi-testing` | Auto-invoked | Testing philosophy: manual-first, unit/integration, mock interfaces |
| `rovi-design` | Auto-invoked | UI/UX design system: solid colors, abstract shapes, framer-motion, CSS variables, smooth scroll |
| `rovi-tooling` | Auto-invoked | Linters (Biome > ESLint), package managers (bun/pnpm), .npmrc ignore-scripts, deps |
| `rovi-fastify` | Auto-invoked | Fastify: entities/ structure, inline DI in controllers, constructor injection |
| `rovi-react` | Auto-invoked | React: TanStack Query + Router, global store, feature-based structure |
| `rovi-nextjs` | Auto-invoked | Next.js: App Router, Server Components, TanStack, store global |
| `rovi-nestjs` | Auto-invoked | NestJS: same philosophy with decorators and built-in DI |
| `rovi-fastapi` | Auto-invoked | FastAPI: entities/ structure, Depends() DI, ABC interfaces, Pydantic |
| `rovi-go` | Auto-invoked | Go: cmd/internal layout, Gin, sqlc, handler/service/repository, New* constructors |
| `rovi-review` | User-invoked (`/rovi-review`) | Code review against rovi standards with checklist |

## Stack

TypeScript, JavaScript, Python, Go | React, Next.js, Vue | NestJS, Fastify, Flask, FastAPI, Gin | Jotai, Redux, Zustand, Pinia

## Installation

### Claude Code (personal)

```bash
# All skills
mkdir -p ~/.claude/skills
cp -r rovi rovi-architecture rovi-store rovi-testing rovi-design rovi-tooling rovi-review ~/.claude/skills/

# Agent (optional — requires Context7 MCP)
mkdir -p ~/.claude/agents
cp .claude/agents/docs-lookup.md ~/.claude/agents/
```

### Claude Code (project-level, shared via git)

```bash
# All skills
mkdir -p .claude/skills
cp -r rovi rovi-architecture rovi-store rovi-testing rovi-design rovi-review .claude/skills/

# Agent (optional)
mkdir -p .claude/agents
cp .claude/agents/docs-lookup.md .claude/agents/

git add .claude/
git commit -m "Add rovi skills"
```

## Structure

```
rovi-skills/
├── rovi/                           # Core philosophy
│   ├── SKILL.md
│   └── references/
│       └── documentation-examples.md
├── rovi-architecture/              # Clean Architecture + folders
│   ├── SKILL.md
│   └── references/
│       └── layer-examples.md
├── rovi-store/                     # State management
│   ├── SKILL.md
│   └── references/
│       └── store-examples.md
├── rovi-testing/                   # Testing
│   ├── SKILL.md
│   └── references/
│       └── testing-examples.md
├── rovi-design/                    # UI/UX design system
│   └── SKILL.md
├── rovi-tooling/                   # Linters, package managers, security
│   └── SKILL.md
├── rovi-review/                    # Code review (user-invoked)
│   └── SKILL.md
├── .claude/
│   └── agents/
│       └── docs-lookup.md          # Background docs lookup (Context7)
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

## Best Practices Applied

- **Modular skills** — one skill per concern, not a monolith
- **YAML frontmatter** — `name`, `description`, and skill-specific fields
- **`${CLAUDE_SKILL_DIR}`** — portable references to bundled files
- **`disable-model-invocation`** — task skills only run when user invokes them
- **`allowed-tools` scoped** — rovi-review restricted to Read, Glob, Grep
- **`context: fork`** — rovi-review runs in isolated subagent
- **Dynamic context** — rovi-review injects `git diff` output automatically

## License

MIT
