# Store Pattern Examples

## Consistent Store Structure

Every store follows this structure regardless of the state library (Zustand, Pinia, Jotai, Redux). Visually separated sections, always with loading and error state, try-catch on every async action.

```typescript
// Pattern: one store per entity, separated sections, loading + error state

/*================ */
/*   STATE          */
/*================ */
const entities = ref<Array<Entity>>([])
const isLoading = ref<boolean>(false)
const error = ref<string | null>(null)

/*================ */
/*   COMPUTED       */
/*================ */
const hasEntities = computed(() => entities.value.length > 0)
const totalEntities = computed(() => entities.value.length)

/*================ */
/*   ACTIONS        */
/*================ */

async function fetchEntities(): Promise<void> {
  try {
    isLoading.value = true
    error.value = null
    const response = await api.entities.getAll()
    entities.value = response
  } catch (err: unknown) {
    const message = err instanceof Error ? err.message : 'Failed to fetch entities'
    error.value = message
    console.error('Failed to fetch entities:', err)
  } finally {
    isLoading.value = false
  }
}

async function createEntity(data: CreateEntityData): Promise<Entity> {
  try {
    isLoading.value = true
    error.value = null
    const newEntity = await api.entities.create(data)
    entities.value.push(newEntity)
    return newEntity
  } catch (err: unknown) {
    const message = err instanceof Error ? err.message : 'Failed to create entity'
    error.value = message
    console.error('Failed to create entity:', err)
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
