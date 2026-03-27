# Layer Implementation Examples

## Domain Entity

```typescript
// Pattern: domain class with validation logic, no infrastructure dependencies
export class Entity implements IEntity {
  id: number;
  name: string;
  email: string;
  createdAt: Date;

  constructor(id: number, name: string, email: string, createdAt: Date) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.createdAt = createdAt;
  }

  // Business rules live in the domain — not in services or infrastructure
  setEmail(newEmail: string): string {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(newEmail)) {
      throw new Error("Invalid email format");
    }
    this.email = newEmail;
    return this.email;
  }
}
```

## Repository Interface

```typescript
// Pattern: interface defines the contract, lives in domain or types/
export default interface IEntityRepository {
  create(entity: IEntity): Promise<IEntity>;
  findById(id: number): Promise<IEntity>;
  findAll(): Promise<IEntity[]>;
  delete(id: number): Promise<IEntity>;
  update(id: number, data: Partial<IEntity>): Promise<IEntity>;
}
```

## Use Case

```typescript
// Pattern: single-responsibility use case, receives repo interface via DI
@Injectable()
export class CreateEntity {
  constructor(
    @Inject('IEntityRepository') private readonly entityRepository: IEntityRepository,
  ) {}

  async execute(data: IEntity): Promise<IEntity> {
    try {
      // Business validation lives in the use case
      return await this.entityRepository.create(data);
    } catch (error: unknown) {
      throw new Error(`Failed to create entity: ${(error as Error).message}`);
    }
  }
}
```

## Concrete Repository

```typescript
// Pattern: implements interface, handles persistence details
@Injectable()
export class EntityRepositoryPrisma implements IEntityRepository {
  constructor(private prisma: PrismaService) {}

  async create(data: IEntity): Promise<IEntity> {
    return this.prisma.entity.create({
      data: {
        name: data.name,
        email: data.email,
        createdAt: new Date(),
      },
    });
  }

  async findById(id: number): Promise<IEntity> {
    return this.prisma.entity.findUnique({ where: { id: Number(id) } });
  }

  async findAll(): Promise<IEntity[]> {
    return this.prisma.entity.findMany();
  }

  async delete(id: number): Promise<IEntity> {
    return this.prisma.entity.delete({ where: { id: Number(id) } });
  }
}
```

## Factory

Encapsulates entity creation when there are variants (roles, types, different initial states). Only use when explicitly requested.

## Controller / Routes

```typescript
// Pattern: controller delegates to use case, handles HTTP concerns only
async createEntity(request, reply) {
  try {
    const result = await this.createEntityUseCase.execute(request.body);
    return reply.status(201).send({ data: result });
  } catch (error) {
    if (error instanceof HttpException) throw error;
    throw new DatabaseOperationException('Failed to create entity', error);
  }
}
```

## Middleware

```typescript
// Pattern: auth middleware extracts and validates token
export async function jwtAuth(request, reply) {
  const authHeader = request.headers['authorization'];
  if (!authHeader) {
    return reply.status(401).send({ message: 'Token not found' });
  }

  const token = authHeader.split(' ')[1];
  const payload = jwtService.verifyToken(token);

  if (!payload) {
    return reply.status(401).send({ message: 'Invalid token' });
  }

  request.user = payload;
}
```

## Auxiliary Services

```typescript
// Pattern: utility service with constructor injection
export class JwtService {
  private secretKey: string;

  constructor(secretKey: string) {
    this.secretKey = secretKey;
  }

  generateToken(payload: Record<string, any>, expiresInSeconds: number = 86400): string {
    return jwt.sign(payload, this.secretKey, { expiresIn: expiresInSeconds });
  }

  verifyToken(token: string): Record<string, any> | null {
    try {
      return jwt.verify(token, this.secretKey) as Record<string, any>;
    } catch {
      return null;
    }
  }
}
```

## Shared Configuration

Database connection, server setup, cloud clients, environment variables. Everything the application needs to boot that is not business logic. The folder name and location depend on the framework.
