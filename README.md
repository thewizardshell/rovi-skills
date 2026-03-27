# Rovi

Opinionated Claude Code plugin for building full-stack apps. Architecture, frameworks, design system, testing, and tooling — with strong conventions baked in.

## Philosophy: Contract-Driven Development

Rovi follows a simple idea: **contracts go down, implementations go up.**

Every layer defines a contract before anyone implements anything. The layer above only knows the contract — never the implementation. This is how the full stack connects:

```
DB schema (you create it manually)
  ↓ contract
ORM pull (generates models from your DB)
  ↓ contract
Interface (defines what something can do)
  ↓ contract
Class + Repository (implements the interface)
  ↓ contract
Service (orchestrates business logic)
  ↓ contract
Controller + Swagger (exposes the API)
  ↓ contract
Orval (generates typed hooks from the spec)
  ↓ contract
React component (consumes the hooks)
```

Each layer doesn't know how the one below works. The DB doesn't know there's an ORM. The service doesn't know if the repo uses Prisma or Drizzle. The frontend doesn't know if the backend is Fastify or Go. They only know the contract.

You can swap any piece without the rest noticing — as long as the contract holds.

### Why this works

Every restriction has a reason, and every reason connects to the next:

- **No migrations** — because I control the DB directly. The schema is mine, not the ORM's.
- **ORM pulls the schema** — because the DB is the source of truth, not the code.
- **Interface + class mandatory** — because the interface alone doesn't exist at runtime. The class gives it shape with getters and setters.
- **Swagger first** — because without it, Orval can't generate the frontend API layer.
- **Orval mandatory** — because I'm not writing API code by hand when a machine can do it typed and correct.
- **Utils is a real module** — because a project has security, validation, formatting, logging — not just errors.

This is how I personally like to work. It's not the only way, and I'm flexible when the context calls for it. But this flow has consistently worked for me — so I built it into a plugin.

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
| `rovi-react` | Auto-invoked | React: Orval + TanStack Query + Router, global store, feature-based structure |
| `rovi-nextjs` | Auto-invoked | Next.js: App Router, Server Components, TanStack, store global |
| `rovi-nestjs` | Auto-invoked | NestJS: same philosophy with decorators and built-in DI |
| `rovi-fastapi` | Auto-invoked | FastAPI: entities/ structure, Depends() DI, ABC interfaces, Pydantic |
| `rovi-go` | Auto-invoked | Go: cmd/internal layout, Gin, sqlc, handler/service/repository, New* constructors |
| `rovi-review` | User-invoked (`/rovi-review`) | Code review against rovi standards with checklist |

## Stack

TypeScript, JavaScript, Python, Go | React, Next.js, Vue | NestJS, Fastify, FastAPI, Gin | Drizzle, TypeORM, Prisma | Orval, TanStack Query, Axios | Zustand, Jotai

## Installation

```bash
claude plugin install rovi-skills
```

## Structure

```
rovi-skills/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── rovi/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── rovi-architecture/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── rovi-store/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── rovi-testing/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── rovi-design/
│   ├── rovi-tooling/
│   ├── rovi-fastify/
│   ├── rovi-react/
│   ├── rovi-nextjs/
│   ├── rovi-nestjs/
│   ├── rovi-fastapi/
│   ├── rovi-go/
│   └── rovi-review/
├── agents/
│   └── docs-lookup.md
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
