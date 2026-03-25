# Testing Pattern Examples

## Unit Test for Use Cases

Mock the interfaces — this is why everything has an interface.

```typescript
/**
 * Suite de tests para el caso de uso CreateUser.
 * Verifica el flujo completo de registro incluyendo
 * validaciones de negocio, persistencia y notificaciones.
 */
describe('CreateUser', () => {
  let useCase: CreateUser;
  let repositoryMock: UserRepository;
  let mailServiceMock: MailService;

  beforeEach(() => {
    repositoryMock = {
      findByEmail: jest.fn(),
      save: jest.fn(),
    };
    mailServiceMock = {
      send: jest.fn(),
    };
    useCase = new CreateUser(repositoryMock, mailServiceMock);
  });

  it('debería crear un usuario cuando los datos son válidos', async () => {
    repositoryMock.findByEmail = jest.fn().mockResolvedValue(null);

    const result = await useCase.execute(validData);

    expect(result.success).toBe(true);
    expect(repositoryMock.save).toHaveBeenCalledTimes(1);
  });

  it('debería rechazar cuando el correo ya existe', async () => {
    repositoryMock.findByEmail = jest.fn().mockResolvedValue(existingUser);

    const result = await useCase.execute(validData);

    expect(result.success).toBe(false);
    expect(result.error).toBeInstanceOf(DuplicateError);
  });
});
```

## Testing Rules

1. Implement the feature completely.
2. Test manually — endpoints with Postman, UI in the browser.
3. Once it works, write automated tests.
4. Unit tests for business logic (domain and use cases).
5. Integration tests for repositories and endpoints.
6. Mock the interfaces.
