---
name: rovi-fastify
description: "Fastify backend patterns: entities folder structure, controller with inline DI, service with constructor injection, repository pattern, route registration. Auto-loads when working with Fastify, creating API endpoints, or building backend with Fastify."
---

# ROVI — Fastify

## Project Structure

Use `entities/` instead of `modules/`. Each entity folder is self-contained:

```
src/
  config/
    db/
    server/
  entities/
    [EntityName]/
      controller/
        entityName.controller.ts
      routes/
        entityName.routes.ts
      service/
        entityName.service.ts
      repository/
        entityName.repository.ts
      types/
        entityName.interface.ts      > Interface defining the entity contract
        entityName.entity.ts         > Class implementing the interface with getters/setters
        entityName.type.ts
  auth/
    middleware/
      jwt.middleware.ts
  utils/
    errors/
```

---

## Controller Pattern

**Instantiate dependencies directly in the controller file.** No separate DI container file. The controller is where you see what depends on what.

```typescript
// Pattern: import repo + service, instantiate inline, export controller function
import { FastifyInstance } from "fastify";
import { EntityService } from "../service/entity.service";
import { EntityRepository } from "../repository/entity.repository";

const entityRepository = new EntityRepository();
const entityService = new EntityService(entityRepository);

export async function entityController(fastify: FastifyInstance) {
  fastify.get("/", async (request, reply) => {
    const items = await entityService.findAll();
    return reply.status(200).send({ data: items });
  });

  fastify.post("/", async (request, reply) => {
    const result = await entityService.create(request.body as CreateEntityData);
    return reply.status(201).send({ data: result });
  });
}
```

---

## Service Pattern

Services receive dependencies via **constructor injection**. No classes passed as method parameters.

```typescript
// Pattern: constructor receives interface, methods orchestrate business logic
export class EntityService {
  constructor(
    private readonly entityRepository: IEntityRepository
  ) {}

  async findAll(): Promise<Entity[]> {
    return this.entityRepository.findAll();
  }

  async create(data: CreateEntityData): Promise<Entity> {
    // Business validation lives here
    return this.entityRepository.create(data);
  }
}
```

---

## Entity Types

`types/` always contains two files: the **interface** defining the entity contract and a **class** implementing it with getters and setters.

```typescript
// Pattern: interface defines the entity shape
// types/entity.interface.ts
export interface IEntity {
  id: number;
  name: string;
  description: string;
  createdAt: Date;
  updatedAt: Date;
}

// Pattern: class implements interface with constructor, getters, and setters
// types/entity.entity.ts
import type { IEntity } from "./entity.interface";

export class Entity implements IEntity {
  constructor(
    private _id: number,
    private _name: string,
    private _description: string,
    private _createdAt: Date,
    private _updatedAt: Date,
  ) {}

  get id(): number { return this._id; }
  get name(): string { return this._name; }
  get description(): string { return this._description; }
  get createdAt(): Date { return this._createdAt; }
  get updatedAt(): Date { return this._updatedAt; }

  set name(value: string) { this._name = value; }
  set description(value: string) { this._description = value; }
}
```

The class is **not optional**. Every entity must have its interface and its implementing class with essential getters and setters.

---

## Repository Pattern

Interface in `types/`, implementation in `repository/`.

```typescript
// Pattern: interface defines contract, class implements it
// types/entity.interface.ts (repository interface, same file or separate)
export interface IEntityRepository {
  findAll(): Promise<Entity[]>;
  findById(id: number): Promise<Entity>;
  create(data: CreateEntityData): Promise<Entity>;
  delete(id: number): Promise<Entity>;
}

// repository/entity.repository.ts
export class EntityRepository implements IEntityRepository {
  // Implementation depends on the DB (Prisma, Drizzle, raw SQL, etc.)
}
```

---

## Route Registration

```typescript
// Pattern: register controller with prefix
import { FastifyInstance } from "fastify";
import { entityController } from "../controller/entity.controller";

export async function entityRoutes(fastify: FastifyInstance) {
  fastify.register(entityController, { prefix: "/entities" });
}
```

---

## Swagger / OpenAPI

**Swagger first.** Every Fastify backend must expose an OpenAPI spec. The frontend depends on it to auto-generate its entire API layer with Orval.

Use `@fastify/swagger` + `@fastify/swagger-ui`. Document every route with schemas. The OpenAPI spec is the contract between backend and frontend.

---

## Rules

- **Swagger first — non-negotiable.** Always expose OpenAPI spec. Without it, Orval cannot generate the frontend API layer. Do not skip or defer this step.
- **Entity class is mandatory.** Every entity in `types/` must have an interface file and a class file implementing it with getters and setters. The class cannot be omitted.
- **`entities/` not `modules/`.** Each folder is one entity.
- **Inline DI in controllers.** Instantiate repository → service right there. No separate DI file.
- **Constructor injection in services.** Dependencies via constructor, never as method params.
- **Interface + implementation separated** in `types/` (interface) and `repository/` or `service/` (class).
- **No `fromPersistence` pattern.** Do not add static reconstruction methods to entities unless explicitly requested.
- **Ask monorepo vs separate repos.** When the user asks for both frontend and backend, always ask first. Never assume.
