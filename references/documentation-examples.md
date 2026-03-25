# Documentation Examples

## JSDoc/TSDoc Pattern

Comments are documentation. Written as if explaining to a new teammate what each piece does and why it exists. No emojis. No decorations. Professional and precise.

```typescript
/**
 * Servicio encargado de gestionar el ciclo de vida completo de un usuario
 * dentro del sistema. Coordina la creación, validación y persistencia
 * delegando las operaciones de datos al repositorio correspondiente.
 *
 * Este servicio NO maneja autenticación — para eso existe AuthService.
 *
 * @example
 * const service = new UserService(userRepository, mailService);
 * const result = await service.register(registrationData);
 */
class UserService {

  /**
   * Registra un nuevo usuario en el sistema. Valida que el correo no esté
   * duplicado, hashea la contraseña y envía el correo de bienvenida.
   * Si cualquier paso falla, la operación se revierte completamente.
   *
   * @param registrationData - Datos del formulario ya validados en capa de presentación
   * @returns El usuario creado con su identificador asignado, o un error tipado
   * @throws DuplicateError - Si el correo ya existe en el sistema
   * @throws ValidationError - Si los datos no cumplen las reglas de negocio
   */
  async register(registrationData: RegistrationData): Promise<OperationResult<User>> {
    // ...
  }
}
```

## Comment Rules

- No emojis. Ever. Comments are technical documentation.
- JSDoc/TSDoc on every public function and class. No exceptions.
- Explain the "why", not the "what". Code says what it does; the comment explains why that decision was made.
- Long comments are fine. If a design decision needs 5 lines, write 5 lines.
- Use `@example` when the API is not obvious.
- Comments in **Spanish** by default, naming in **English**.
