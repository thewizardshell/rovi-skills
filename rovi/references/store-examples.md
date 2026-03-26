# Store Pattern Examples

## Consistent Store Structure

Every store follows this structure regardless of the state library (Zustand, Pinia, Jotai, Redux). Visually separated sections, always with loading and error state, try-catch on every async action.

```typescript
/*================ */
/*   STATE          */
/*================ */
const clients = ref<Array<Client>>([])
const isLoading = ref<boolean>(false)
const error = ref<string | null>(null)

/*================ */
/*   COMPUTED       */
/*================ */
const hasClients = computed(() => clients.value.length > 0)
const totalClients = computed(() => clients.value.length)

/*================ */
/*   ACTIONS        */
/*================ */

/**
 * Obtiene todos los clientes desde el servidor y actualiza el estado local.
 * Si falla, guarda el mensaje de error en el estado para que la UI lo muestre.
 */
async function getClients(): Promise<void> {
  try {
    isLoading.value = true
    error.value = null
    const response = await api.clients.getAll()
    clients.value = response
  } catch (err: unknown) {
    const message = err instanceof Error ? err.message : 'Error al obtener los clientes'
    error.value = message
    console.error('Hubo un error al obtener los clientes:', err)
  } finally {
    isLoading.value = false
  }
}

/**
 * Crea un nuevo cliente y lo agrega al estado local sin necesidad
 * de volver a pedir toda la lista al servidor.
 * Si falla, lanza el error para que el componente que llamó pueda reaccionar.
 */
async function createClient(data: CreateClientData): Promise<Client> {
  try {
    isLoading.value = true
    error.value = null
    const newClient = await api.clients.create(data)
    clients.value.push(newClient)
    return newClient
  } catch (err: unknown) {
    const message = err instanceof Error ? err.message : 'Error al crear el cliente'
    error.value = message
    console.error('Hubo un error al crear el cliente:', err)
    throw err
  } finally {
    isLoading.value = false
  }
}
```

## Store Rules

- One store per entity. Never a mega-store.
- Always `isLoading` and `error` as state.
- Try-catch on every async action with `finally` to reset `isLoading`.
- Visually separated sections: state, computed, actions.
- Optimistic local state updates when it makes sense.
