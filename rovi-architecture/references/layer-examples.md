# Layer Implementation Examples

## Domain Entity

```typescript
export class User implements IUser {
  id: number;
  role: string;
  email: string;
  createdTime: Date;
  password: string;

  constructor(id: number, role: string, email: string, createdTime: Date, password: string) {
    this.id = id;
    this.role = role;
    this.email = email;
    this.createdTime = createdTime;
    this.password = password;
  }

  static fromPersistence(data: IUser): User {
    return new User(data.id, data.role, data.email, data.createdTime, data.password);
  }

  /** Valida formato antes de asignar — la regla vive aqui y no en infra */
  setEmail(newEmail: string): string {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(newEmail)) {
      throw new Error('El correo electrónico no es válido');
    }
    this.email = newEmail;
    return this.email;
  }
}
```

## Repository Interface

```typescript
export default interface IUserRepository {
  create(user: IUser): Promise<IUser>;
  findById(id: number): Promise<IUser>;
  findByEmail(email: string): Promise<IUser | null>;
  findAll(): Promise<IUser[]>;
  delete(id: number): Promise<IUser>;
  update(id: number, data: Partial<IUser>): Promise<IUser>;
}
```

## Use Case

```typescript
@Injectable()
export class CreateUser {
  constructor(
    @Inject('IUserRepository') private readonly userRepository: IUserRepository,
    private readonly encryptService: EncryptService,
  ) {}

  async execute(data: IUser): Promise<IUser> {
    try {
      const existingUser = await this.userRepository.findByEmail(data.email);
      if (existingUser) {
        throw new Error('El usuario ya existe');
      }

      const hashedPassword = await this.encryptService.encrypt(data.password);
      data.password = hashedPassword;

      const user = UserFactory.createClient(data.id, data.email, data.password);

      return await this.userRepository.create(user);
    } catch (error: unknown) {
      throw new Error(`Error al crear el usuario: ${(error as Error).message}`);
    }
  }
}
```

## Concrete Repository

```typescript
@Injectable()
export class UserRepositoryPrisma implements IUserRepository {
  constructor(private prisma: PrismaService) {}

  async create(data: IUser): Promise<IUser> {
    return this.prisma.user.create({
      data: {
        role: data.role,
        email: data.email,
        createdTime: new Date(),
        password: data.password,
      },
    });
  }

  async findById(id: number): Promise<IUser> {
    return this.prisma.user.findUnique({ where: { id: Number(id) } });
  }

  async findByEmail(email: string): Promise<IUser> {
    return this.prisma.user.findUnique({ where: { email } });
  }

  async findAll(): Promise<IUser[]> {
    return this.prisma.user.findMany();
  }

  async delete(id: number): Promise<IUser> {
    return this.prisma.user.delete({ where: { id: Number(id) } });
  }
}
```

## Factory

Encapsulates entity creation when there are variants (roles, types, different initial states).

## Controller / Routes

```typescript
async createUser(request, reply) {
  try {
    const result = await this.createUserUseCase.execute(request.body);
    return reply.status(201).send({ message: 'Usuario creado', data: result });
  } catch (error) {
    if (error instanceof HttpException) throw error;
    throw new DatabaseOperationException('No se pudo crear el usuario', error);
  }
}
```

## Middleware

```typescript
export async function jwtAuth(request, reply) {
  const authHeader = request.headers['authorization'];
  if (!authHeader) {
    return reply.status(401).send({ message: 'El token no ha sido encontrado' });
  }

  const token = authHeader.split(' ')[1];
  const payload = jwtService.verifyToken(token);

  if (!payload) {
    return reply.status(401).send({ message: 'Token inválido' });
  }

  request.user = payload;
}
```

## Auxiliary Services

```typescript
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
    } catch (error) {
      console.error('Error al verificar el token:', error);
      return null;
    }
  }
}
```

## Shared Configuration

Database connection, server setup, cloud clients, environment variables. Everything the application needs to boot that is not business logic. The folder name and location depend on the framework.
