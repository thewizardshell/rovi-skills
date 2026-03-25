# Layer Implementation Examples

## Domain Entity

Entities have their own logic. They validate data internally, expose getters/setters with business rules, and have static methods for rehydration from persistence.

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

  /**
   * Reconstruye la entidad desde los datos que vienen de la base de datos.
   * Separa la lógica de creación de la lógica de rehidratación.
   */
  static fromPersistence(data: IUser): User {
    return new User(data.id, data.role, data.email, data.createdTime, data.password);
  }

  /**
   * Cambia el email validando el formato antes de asignarlo.
   * La validación vive en la entidad porque es una regla de negocio,
   * no de infraestructura.
   */
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

Defines the complete contract of operations on an entity. Each method with a short comment. The use case depends on the interface, never on the implementation.

```typescript
export default interface IUserRepository {
  // Create a new user
  create(user: IUser): Promise<IUser>;
  // Find user by ID
  findById(id: number): Promise<IUser>;
  // Find user by email
  findByEmail(email: string): Promise<IUser | null>;
  // Get all users
  findAll(): Promise<IUser[]>;
  // Delete user
  delete(id: number): Promise<IUser>;
  // Update user
  update(id: number, data: Partial<IUser>): Promise<IUser>;
}
```

## Use Case

One use case = one system action. Receives dependencies via constructor (injection), has an `execute` method as entry point, orchestrates logic without knowing concrete implementations.

```typescript
@Injectable()
export class CreateUser {
  constructor(
    @Inject('IUserRepository') private readonly userRepository: IUserRepository,
    private readonly encryptService: EncryptService,
  ) {}

  /**
   * Crea un usuario nuevo. Verifica duplicados, valida datos,
   * encripta la contraseña y persiste a través del repositorio.
   */
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

The implementation does data operations only. Zero business logic. Each method is direct — receives parameters, executes the query, returns the result.

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

Encapsulates entity creation when there are variants (roles, types, different initial states). The use case does not instantiate the entity directly — it uses the factory.

## Controller / Routes

The controller is pure delegation. Receives the request, calls the corresponding use case or service, and returns the response. No business logic. If there is an error, it catches it and transforms it into an appropriate HTTP response.

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

Controller rules:
- Each endpoint delegates to the use case. The controller does not validate business, does not query the DB, does not encrypt anything.
- API documentation separate. Swagger/OpenAPI config lives in separate files, not polluting the controller.
- Layer-specific exceptions. The controller uses HTTP-specific exceptions (`UserNotFoundException`, `InvalidCredentialsException`), not generic errors.

## Middleware

Middlewares are simple functions that do one thing. Authentication, permission validation, logging — each is an independent function hooked to the route.

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

Services like JWT, encryption, mail sending, etc. are classes with single responsibility. They receive configuration via constructor and are injected where needed. Framework-independent — portable.

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

## Common Folder

Shared configuration that does not belong to any module: database connection, server configuration, cloud service clients, environment variables. Everything the application needs to boot that is not business logic.
