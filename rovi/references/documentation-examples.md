# Documentation Examples

## JSDoc/TSDoc Pattern

Only comment when there is something non-obvious to explain. If the method name and signature already tell the story, skip the comment. When you do comment, explain **why** — not what the code does.

### Good — comments that add value

```typescript
/**
 * No maneja autenticacion — eso vive en AuthService.
 * Se separo porque el ciclo de vida del usuario y la sesion
 * tienen reglas de negocio independientes.
 */
class UserService {

  /**
   * La operacion es atomica: si falla el envio del correo,
   * se revierte la creacion para no dejar usuarios sin confirmar.
   *
   * @throws DuplicateError - El correo ya existe
   */
  async register(registrationData: RegistrationData): Promise<OperationResult<User>> {
    // ...
  }
}
```

### Bad — comments that narrate lo que el codigo ya dice

```typescript
// NO hacer esto:

/** Servicio encargado de gestionar el ciclo de vida completo de un usuario */
class UserService { }

/** Crea un usuario nuevo y lo persiste en la base de datos */
async execute(data: IUser): Promise<IUser> { }

/** Busca un usuario por su ID */
findById(id: number): Promise<IUser>;
```

## Comment Rules

- No emojis. Comments are technical documentation.
- Only add JSDoc/TSDoc when there is a non-obvious decision, constraint, or gotcha worth documenting.
- If the method name and types already explain everything, do not add a comment.
- Explain the **"why"**, never narrate the **"what"**. If there is no "why", there is no comment.
- Long comments are fine when a design decision genuinely needs 5 lines of context.
- Use `@example` when the API is not obvious.
- Comments in **Spanish** by default, naming in **English**.
