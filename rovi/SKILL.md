---
name: rovi
description: "Full-stack coding style, Clean Architecture patterns, and development philosophy. Use when writing code, designing systems, reviewing or refactoring code, planning architecture, structuring folders, creating interfaces, implementing repositories, building use cases, setting up state management, writing tests, handling errors, or explaining technical concepts. Enforces interface-first design, layered separation, strict typing, feature-based folder organization, centralized error handling, visual section separators in files, and direct no-analogy explanations. Applies to any language or framework in the stack: TypeScript, JavaScript, Python, Go, React, Next.js, Vue, NestJS, Fastify, Flask, FastAPI, Gin."
---

# ROVI — Style & Architecture Skill

## Overview

This skill defines a complete development philosophy: how to think before coding, how to structure projects, how to write and document code, and how to communicate technical concepts. It is framework-agnostic — the same principles apply regardless of the stack.

**Tech stack:** TypeScript, JavaScript, Python, Go | React, Next.js, Vue | NestJS, Fastify, Flask, FastAPI, Gin | Jotai, Redux, Zustand, Pinia

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
6. **Implement.** Start with domain (entities, interfaces), then use cases, then infrastructure and presentation.
7. **Test.** Manual testing first (Postman or browser). Then write automated tests. Verify edge cases from step 1.

---

## Architecture

### Clean Architecture (Layered)

These are dependency layers, not folder names. All layers coexist inside each entity's folder.

```
domain layer          → Entities, interfaces, business types. Zero external dependencies.
application layer     → Use cases. Orchestrate the domain. Depend only on interfaces.
infrastructure layer  → Concrete implementations: repositories, external APIs, DB.
presentation layer    → Controllers, routes, UI. The outermost layer.
```

### Core Principles

- **Interfaces for everything that might change.** Swapping the database or a provider should only change the infrastructure layer.
- **Repository pattern mandatory** for data access.
- **Dependency injection.** Never instantiate dependencies inside a class. Always received via constructor or parameter.
- **Single responsibility:** if a function does more than 2 things, split it.
- **Purposeful abstractions:** every interface exists to enable swapping implementations or facilitate testing. No "just in case" interfaces.

### Patterns

Repository, Service, UseCase/Interactor, Ports & Adapters, Factory, Strategy, Observer/EventEmitter.

### Layer Implementation

Read `references/layer-examples.md` for code examples of each layer.

---

## Folder Structure

### Principles

- **Feature-based organization.** Each feature is autonomous. Never mix code from one feature into another.
- **Global vs local.** If something is used by only one feature, it lives inside that feature. If shared, it goes up to the global level.
- **Shared infrastructure separated.** DB config, server setup, cloud clients — separated from business features.
- **One store per entity** (frontend). Never a mega-store.
- **File naming:** `camelCase` for stores and utilities, `PascalCase` for components and views.

### Important

The examples below are **references**, not rigid templates. Folder names, nesting depth, and grouping depend on the framework and the project. Always confirm the structure before creating it. What matters is the layered separation principle, not the specific names.

### Frontend (reference)

```
src/
  assets/
  components/               → Global reusable components
  core/
    layout/
    store/                  → Base store configuration
  [feature-group]/
    [FeatureName]/
      components/           → Feature-specific components
      store/                → Feature-specific stores
      views/                → Feature pages
  utils/
```

### Backend (reference)

Everything related to an entity lives inside that entity's folder. You work on one entity, you stay in one place.

```
src/
  [shared-config]/
    db/
    server/
  modules/
    [EntityName]/
      entities/
      interfaces/
      types/
      use-cases/
      dto/
      repositories/
      services/
      controllers/
      middleware/
  utils/
    errors/
```

Names in `[brackets]` are placeholders — they change per project and framework. The key rule: everything for one entity is colocated, no jumping between folders to find related code.

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

Read `references/documentation-examples.md` for JSDoc patterns.

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

## State Management (Frontend)

Consistent structure regardless of the state library:

- One store per entity. Never a mega-store.
- Always `isLoading` and `error` as state. Each store manages its own loading and error state.
- Try-catch on every async action. Always with `finally` to reset `isLoading`.
- Visually separated sections: state, computed/derived, actions.
- Optimistic local state updates when appropriate (push to array after create, filter after delete).

Read `references/store-examples.md` for a full store pattern.

---

## Testing

1. Implement the feature completely.
2. Test manually — endpoints with Postman or similar, UI in the browser.
3. Once it works, write automated tests.

- Unit tests for business logic (domain and use cases).
- Integration tests for repositories and endpoints.
- Mock the interfaces — another reason everything has an interface.

Read `references/testing-examples.md` for test patterns.

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
- Sacrifice clarity for brevity.
- Explain code with real-world analogies.

### Always

- Define interfaces before implementing.
- Document with JSDoc/TSDoc when there is a non-obvious "why" behind the code.
- Type strictly. `strict: true`, no `any`, explicit types.
- Structure in layers. Domain → Application → Infrastructure → Presentation.
- Centralize errors in a dedicated module with clear types.
- Test. Manual first, automated after.
- Think about swappability. Changing an external dependency should touch one file ideally.
- Write naming in English, comments/documentation in Spanish.
- Organize by features with consistent folder structure.
- Use visual section separators inside files.
