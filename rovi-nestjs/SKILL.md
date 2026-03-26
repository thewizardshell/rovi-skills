---
name: rovi-nestjs
description: "NestJS backend patterns: module structure, controller with DI, service with constructor injection, repository pattern, decorators. Auto-loads when working with NestJS, creating modules, or building backend with NestJS."
---

# ROVI — NestJS

## Philosophy

Same architecture as Fastify (see rovi-fastify) but using NestJS's built-in DI system, decorators, and module structure. The principles are identical: interface-first, constructor injection, repository pattern, entity-colocated folders.

---

## Project Structure

```
src/
  config/
    database/
    env/
  modules/
    [EntityName]/
      [entityName].controller.ts
      [entityName].service.ts
      [entityName].module.ts
      repository/
        [entityName].repository.ts
      types/
        [entityName].interface.ts
        [entityName].dto.ts
  auth/
    guards/
    strategies/
    decorators/
  utils/
    errors/
  app.module.ts
  main.ts
```

---

## Controller

NestJS handles DI via decorators — no manual instantiation needed. But the same idea: you see the dependencies right there.

```typescript
@Controller("owners")
export class OwnerController {
  constructor(private readonly ownerService: OwnerService) {}

  @Get()
  @UseGuards(JwtAuthGuard)
  async findAll() {
    return this.ownerService.findAll();
  }

  @Post()
  async create(@Body() data: CreateOwnerDto) {
    return this.ownerService.create(data);
  }
}
```

---

## Service

```typescript
@Injectable()
export class OwnerService {
  constructor(
    @Inject("IOwnerRepository")
    private readonly ownerRepository: IOwnerRepository
  ) {}

  async findAll(): Promise<Owner[]> {
    return this.ownerRepository.findAll();
  }

  async create(data: CreateOwnerDto): Promise<Owner> {
    const existing = await this.ownerRepository.findByEmail(data.email);
    if (existing) {
      throw new ConflictException("Owner ya existe con ese email");
    }
    return this.ownerRepository.create(data);
  }
}
```

---

## Module (DI wiring)

```typescript
@Module({
  controllers: [OwnerController],
  providers: [
    OwnerService,
    {
      provide: "IOwnerRepository",
      useClass: OwnerRepositoryPrisma,
    },
  ],
  exports: [OwnerService],
})
export class OwnerModule {}
```

---

## Repository

Same pattern: interface in `types/`, implementation in `repository/`.

```typescript
// types/owner.interface.ts
export interface IOwnerRepository {
  findAll(): Promise<Owner[]>;
  findById(id: number): Promise<Owner>;
  findByEmail(email: string): Promise<Owner | null>;
  create(data: CreateOwnerDto): Promise<Owner>;
}

// repository/owner.repository.ts
@Injectable()
export class OwnerRepositoryPrisma implements IOwnerRepository {
  constructor(private prisma: PrismaService) {}

  async findAll(): Promise<Owner[]> {
    return this.prisma.owner.findMany();
  }
}
```

---

## Rules

- **NestJS's DI system.** Use `@Inject` for interfaces, no manual instantiation.
- **Constructor injection always.** Same philosophy as Fastify, NestJS just automates it.
- **Interface + class separated** in `types/` and `repository/`.
- **DTOs for validation.** Use `class-validator` + `class-transformer` with DTOs.
- **Guards, not middleware** for auth in NestJS.
- **No `fromPersistence` pattern.** Do not add static reconstruction methods unless explicitly requested.
