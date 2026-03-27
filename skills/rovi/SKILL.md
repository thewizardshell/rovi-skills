---
name: rovi
description: "Core coding philosophy: communication rules, thinking process, style guide, naming conventions, strict typing, error handling patterns, documentation standards, and hard rules. Auto-loads when writing, reviewing, or explaining code in TypeScript, JavaScript, Python, Go, React, Next.js, Vue, NestJS, Fastify, Flask, FastAPI, or Gin."
---

# ROVI — Core Philosophy

This is the core skill. The following companion skills provide specialized guidance and should be loaded alongside this one when their domain is relevant:

- **rovi-architecture** — Clean Architecture, layers, folder structure. Load when designing systems or modules.
- **rovi-store** — State management. Load whenever working on frontend.
- **rovi-testing** — Testing philosophy. Load when writing or planning tests.
- **rovi-design** — UI/UX system: colors, typography, animations, layout. Load when building UI.
- **rovi-tooling** — Linters, package managers, Husky, dependency security. Load when initializing projects or installing packages.
- **rovi-review** — Code review checklist. User-invoked only (`/rovi-review`).

---

## Communication Rules

1. **No analogies.** Never compare code to real-world objects. Explain code with code.
2. **No unnecessary jargon.** Explain directly, as if telling a coworker what the code does.
3. **Explain the why.** If a library already does something internally, say so. Understanding internals prevents redundant or wrong decisions.
4. **Get to the point.** The first sentence must already answer the question.
5. **Research precisely.** Search for the specific thing being asked. Prioritize official docs and source code. No generic tutorials. If unclear, say so.
6. **Tone:** Direct, technical but accessible, opinionated with reasoning, practical — always land on concrete code.

---

## Thinking Process

Before writing any code, follow this sequence:

1. **Understand the problem.** What is needed? Edge cases? Constraints?
2. **Verify docs-lookup is available.** Before starting any implementation, check that the `docs-lookup` agent and Context7 MCP are configured. If Context7 tools (`mcp__claude_ai_Context7__resolve-library-id`, `mcp__claude_ai_Context7__query-docs`) are not available or not accepted in permissions, **stop and tell the user**. Explain that this skill requires Context7 MCP to be installed and its tools to be allowed in permissions. Ask the user if they want to proceed without it using WebSearch as fallback instead. Do not silently skip this step.
3. **Look up while working — continuously.** Every time you are about to use a library, framework, or API — spawn a `docs-lookup` agent **in background**. This is not a one-time step. It happens throughout the entire session: at the start for the initial stack, and again every time you move to a different part of the project that involves a different library or tool. Do not wait for results — keep working and incorporate the info as it arrives. Launch multiple background lookups in parallel if needed. The agent feeds you context while you code, like a coworker looking things up for you.
4. **Design the contracts.** Define interfaces first. Establish input/output types. Identify possible errors.
5. **Confirm structural decisions.** Before creating folders, naming anything, or choosing how to organize the project — ask. Every framework and context has its own conventions. Do not assume folder names, entry point patterns, or project layout from previous experience or from the examples in this skill. Present a proposal and wait for confirmation.
6. **Examples are patterns, not literal code.** Code examples in these skills show the structure and flow — how pieces connect. Never copy them verbatim. Extract the pattern, adapt it to the current project, and write clean production code. If an example has a console.log, a TODO, or an empty block, that is incidental — do not reproduce it.
7. **Swagger first in backends.** Every backend must expose a valid OpenAPI/Swagger spec before writing any frontend code. This is non-negotiable — the frontend depends on Orval to auto-generate its entire API layer from the spec. No spec = no Orval = no frontend API code. Do not skip or defer this step.
8. **Orval is mandatory on frontends.** When there is a backend with an OpenAPI spec, Orval must be configured and working before writing any API-consuming code. If Orval is not set up, stop and configure it first. The only exception is if the user explicitly says they do not want Orval.
9. **Implement.** Start with domain (entities, interfaces), then use cases, then infrastructure and presentation.
10. **Test.** Manual testing first (Postman or browser). Then write automated tests. Verify edge cases from step 1.

---

## Style Guide

### Code Rules

1. **Short functions.** Max 25-30 lines. If longer, it is doing too much — split it.
2. **One file, one responsibility.** Each file has a single clear purpose.
3. **Self-documenting code.** Names must be descriptive enough to read like prose.
4. **No single-line compression.** Clarity over brevity. Each declaration on its own line.

### Naming

- All code naming is in **English**: functions, variables, classes, interfaces, types, constants.
- All comments and JSDoc/TSDoc documentation are in **Spanish** by default, unless explicitly requested otherwise.

```typescript
interface UserRepository {
  findById(id: string): Promise<User>;
  save(user: User): Promise<void>;
}

class UserRepositoryPostgres implements UserRepository { }
class AuthService { }
class CreateUser { }
const activeUsers = await repository.findActive();
const MAX_SESSION_TIME = 3600;

type OrderStatus = 'pending' | 'processing' | 'completed' | 'cancelled';
```

### Typing

- TypeScript strict mode always. `strict: true`, non-negotiable.
- Never `any`. Use `unknown` and validate.
- Explicit types on function parameters and return types for public contracts.

### Visual Section Separators

Use block comment separators inside files with multiple logical sections:

```
/*================ */
/*   STATE          */
/*================ */

/*================ */
/*   COMPUTED       */
/*================ */

/*================ */
/*   ACTIONS        */
/*================ */
```

---

## Documentation & Comments

- **No emojis.** Ever. Comments are technical documentation.
- **JSDoc/TSDoc on public functions and classes** that have non-obvious behavior or design decisions behind them.
- **Only comment when the code does not speak for itself.** If a method name and its signature already make the behavior obvious, a comment adds noise. Reserve comments for decisions, trade-offs, constraints, or gotchas that someone reading the code would not immediately understand.
- **Explain the "why", never narrate the "what".** Bad: "Crea un usuario nuevo y lo persiste". Good: "Se hashea antes de persistir porque el repositorio no maneja encriptacion". If there is no "why" worth explaining, do not write the comment.
- **Long comments are fine** when a design decision genuinely needs context.
- **Use `@example`** when the API is not obvious.
- Comments in **Spanish** by default, naming in **English**.

Read `${CLAUDE_SKILL_DIR}/references/documentation-examples.md` for JSDoc patterns.

---

## Database & ORM

### ORM Preference

1. **Drizzle** or **TypeORM** — preferred choices.
2. **Prisma** — acceptable, but be aware of its large bundle size which can cause issues in serverless or constrained environments. If using Prisma, mention this trade-off.

### No Migrations in Code

**Never run migrations from the ORM.** The database schema is managed manually — tables are created directly in the database (via SQL scripts, a DB GUI, or manual DDL). The ORM then **pulls/introspects** the existing schema to generate its types and models.

- **Drizzle:** Use `drizzle-kit pull` to introspect the database.
- **TypeORM:** Use `synchronize: false` and define entities matching the existing tables.
- **Prisma:** Use `prisma db pull` to introspect, never `prisma migrate`.
- **sqlc (Go):** Already works this way — SQL queries reference existing tables.

Do not generate migration files, do not run `migrate dev`, do not auto-sync schemas. The user controls the database directly.

---

## Error Handling

Centralized typed errors in a `utils/errors` module. Every error has a code, message, and optional metadata.

```typescript
class AppError extends Error {
  constructor(
    public readonly code: string,
    public readonly message: string,
    public readonly httpStatus: number = 500,
    public readonly metadata?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(resource: string, identifier: string) {
    super('RESOURCE_NOT_FOUND', `${resource} not found: ${identifier}`, 404, { resource, identifier });
  }
}

class ValidationError extends AppError {
  constructor(field: string, detail: string) {
    super('VALIDATION_FAILED', `Validation error on ${field}: ${detail}`, 400);
  }
}

class DuplicateError extends AppError {
  constructor(resource: string, field: string) {
    super('RESOURCE_DUPLICATE', `${resource} already exists with that ${field}`, 409);
  }
}

class UnauthorizedError extends AppError {
  constructor(reason?: string) {
    super('UNAUTHORIZED', reason ?? 'Unauthorized operation', 403);
  }
}
```

---

## Hard Rules

### Never

- Write `any` in TypeScript.
- Put emojis in comments.
- Create functions over 30 lines without splitting.
- Hardcode dependencies inside classes or functions.
- Leave code without an interface when there is a possibility of change.
- Put everything in one file.
- Use `console.log` as a debugging strategy in production.
- Ignore errors. Every error is caught, typed, and handled.
- Run ORM migrations. No `migrate dev`, no `migrate deploy`, no auto-sync. The database schema is managed manually.
- Sacrifice clarity for brevity.
- Explain code with real-world analogies.

### Always

- Define interfaces before implementing.
- Document with JSDoc/TSDoc when there is a non-obvious "why" behind the code.
- Type strictly. `strict: true`, no `any`, explicit types.
- Structure in layers. Domain > Application > Infrastructure > Presentation.
- Centralize errors in a dedicated module with clear types.
- Test. Manual first, automated after.
- Think about swappability. Changing an external dependency should touch one file ideally.
- Use `drizzle-kit pull`, `prisma db pull`, or equivalent introspection. The database is the source of truth.
- Write naming in English, comments/documentation in Spanish.
- Organize by features with consistent folder structure.
- Use visual section separators inside files.
