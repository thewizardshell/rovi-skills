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
        entityName.interface.ts
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
import { FastifyInstance } from "fastify";
import { OwnerService } from "../service/owner.service";
import { OwnerRepository } from "../repository/owner.repository";
import { jwtAuth } from "../../../auth/middleware/jwt.middleware";

const ownerRepository = new OwnerRepository();
const ownerService = new OwnerService(ownerRepository);

export async function ownerController(fastify: FastifyInstance) {
  fastify.get("/", { preHandler: [jwtAuth] }, async (request, reply) => {
    try {
      const owners = await ownerService.findAll();
      return reply.status(200).send({ data: owners });
    } catch (error) {
      if (error instanceof HttpException) throw error;
      throw new DatabaseOperationException("No se pudo obtener los owners", error);
    }
  });

  fastify.post("/create", async (request, reply) => {
    try {
      const result = await ownerService.create(request.body as CreateOwnerData);
      return reply.status(201).send({ message: "Owner creado", data: result });
    } catch (error) {
      if (error instanceof HttpException) throw error;
      throw new DatabaseOperationException("No se pudo crear el owner", error);
    }
  });
}
```

---

## Service Pattern

Services receive dependencies via **constructor injection**. No classes passed as method parameters.

```typescript
import { IOwnerRepository } from "../types/owner.interface";

export class OwnerService {
  constructor(
    private readonly ownerRepository: IOwnerRepository
  ) {}

  async findAll(): Promise<Owner[]> {
    return this.ownerRepository.findAll();
  }

  async create(data: CreateOwnerData): Promise<Owner> {
    const existing = await this.ownerRepository.findByEmail(data.email);
    if (existing) {
      throw new DuplicateError("Owner", "email");
    }
    return this.ownerRepository.create(data);
  }
}
```

---

## Repository Pattern

Interface in `types/`, implementation in `repository/`.

```typescript
// types/owner.interface.ts
export interface IOwnerRepository {
  findAll(): Promise<Owner[]>;
  findById(id: number): Promise<Owner>;
  findByEmail(email: string): Promise<Owner | null>;
  create(data: CreateOwnerData): Promise<Owner>;
  delete(id: number): Promise<Owner>;
}

// repository/owner.repository.ts
export class OwnerRepository implements IOwnerRepository {
  constructor(private prisma: PrismaClient) {}

  async findAll(): Promise<Owner[]> {
    return this.prisma.owner.findMany();
  }

  async findById(id: number): Promise<Owner> {
    return this.prisma.owner.findUnique({ where: { id } });
  }
}
```

---

## Route Registration

```typescript
// routes/owner.routes.ts
import { FastifyInstance } from "fastify";
import { ownerController } from "../controller/owner.controller";

export async function ownerRoutes(fastify: FastifyInstance) {
  fastify.register(ownerController, { prefix: "/owners" });
}
```

---

## Swagger / OpenAPI

**Swagger first.** Every Fastify backend must expose an OpenAPI spec. The frontend depends on it to auto-generate its entire API layer with Orval.

Use `@fastify/swagger` + `@fastify/swagger-ui`:

```typescript
import fastifySwagger from "@fastify/swagger";
import fastifySwaggerUi from "@fastify/swagger-ui";

await fastify.register(fastifySwagger, {
  openapi: {
    info: { title: "API", version: "1.0.0" },
    components: {
      securitySchemes: {
        Bearer: { type: "http", scheme: "bearer", bearerFormat: "JWT" },
      },
    },
  },
});

await fastify.register(fastifySwaggerUi, { routePrefix: "/docs" });
```

Document every route with schemas. The OpenAPI spec is the contract between backend and frontend.

---

## Rules

- **Swagger first.** Always expose OpenAPI spec. The frontend consumes it with Orval.
- **`entities/` not `modules/`.** Each folder is one entity.
- **Inline DI in controllers.** Instantiate repository → service right there. No separate DI file.
- **Constructor injection in services.** Dependencies via constructor, never as method params.
- **Interface + implementation separated** in `types/` (interface) and `repository/` or `service/` (class).
- **No `fromPersistence` pattern.** Do not add static reconstruction methods to entities unless explicitly requested.
