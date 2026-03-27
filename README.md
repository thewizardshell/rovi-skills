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

### The idea behind it

If you look at the full picture, this is basically **Dependency Inversion applied across the entire stack** — not just inside a class, but between layers:

```
Data Layer        →  you own the schema, the ORM just reads it
Backend Layer     →  interfaces define contracts, classes implement them
API Contract      →  Swagger is the boundary between back and front
Frontend Layer    →  Orval consumes the contract, components consume Orval
```

Dependency Inversion says: high-level modules should not depend on low-level modules — both should depend on abstractions. Here, no layer depends on another layer's implementation. The service doesn't know if the repo uses Drizzle or Prisma. The DB doesn't know there's an ORM at all. Everyone depends on the contract in between.

The frontend is a good example. Sure, the front never talks to the backend directly — that's normal. But usually someone still has to write the types and fetch calls by hand, copying what the backend returns. That's an implicit, manual, fragile dependency. Here, Orval reads the Swagger spec and generates everything — types, hooks, query keys. The frontend doesn't need a human to tell it what the backend returns. The contract does it automatically. That's the difference.

I didn't plan it this way from the start. This structure came out naturally while building the agent — describing how I work forced me to see the pattern that was already there. It's not a coincidence. When you consistently separate "what something does" from "how it does it" at every layer, you end up with Dependency Inversion whether you meant to or not.

And in practice, it pays off. Clients change their minds, requirements shift, you swap a database or a framework — and the damage stays contained in one layer because nothing else was coupled to it.

### Who is this for

This is not for every project. If you're building an MVP, a quick prototype, or a small experiment, this is probably too much. The abstractions, the interfaces, the mandatory Swagger — it's overhead that doesn't pay off on something you might throw away next week.

Where it shines is on projects that grow, that change, that have clients who change their minds fast. That's where the contracts save you. You come back to the code three months later and the JSDoc tells you why, the Swagger tells you what, and the OOP structure tells you where.

### Why I built this

I got tired of two things:

**Rewriting types twice.** I'd define an interface in the backend, then manually recreate the same types in the frontend. When the backend changed a field, I had to go find every place in the frontend and update it by hand. Orval fixed that — the spec generates everything, and if something changes, TypeScript yells exactly where it broke.

**Not understanding my own code.** When I came back to a project after weeks, flat functions and loose files told me nothing. OOP with clear interfaces, classes with getters/setters, well-written JSDoc — that gives me context immediately. I read the contract and I know what it does without tracing through the implementation.

### What this actually is

This is my personal agent. It's how I see software development — a set of ideas that have worked for me with real clients who change requirements constantly. It gives me flexibility when things shift, and it gives me feedback when I return to code I haven't touched in a while.

These ideas will probably evolve. Maybe I'll drop something, maybe I'll add something new. But right now, this is what works.

I know it looks over-engineered to some people. That's fair. But every abstraction here exists because I needed it at some point — not because it looks good on paper. If you try it and some part doesn't fit your workflow, cut it. The skills are modular for a reason.

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
