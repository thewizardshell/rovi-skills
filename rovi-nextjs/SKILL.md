---
name: rovi-nextjs
description: "Next.js patterns: App Router, Server Components, TanStack Query for client state, global store, server actions. Auto-loads when working with Next.js, creating pages, or building fullstack with Next.js."
---

# ROVI — Next.js

## Stack

- **App Router** (not Pages Router)
- **TanStack Query** for client-side server state
- **Zustand or Jotai** for global client state
- **Server Components** by default, `"use client"` only when needed
- **framer-motion** for animations (see rovi-design skill)

---

## Project Structure

```
src/
  app/
    layout.tsx
    page.tsx
    [feature]/
      page.tsx
      loading.tsx
      error.tsx
  components/                > Global reusable components
  features/
    [FeatureName]/
      components/            > Feature-specific components
      hooks/                 > Client hooks (useQuery, mutations)
      store/                 > Feature client store
      actions/               > Server actions
      types/
  core/
    store/                   > Global store
    providers/               > Client providers (QueryClient, etc.)
  utils/
  styles/
    global.css               > CSS variables (see rovi-design)
```

---

## Server vs Client

- **Server Components** (default): data fetching, layout, static content. No `"use client"`.
- **Client Components** (`"use client"`): interactivity, hooks, event handlers, stores, animations.
- Keep `"use client"` boundary as low as possible. The parent can be server, only the interactive child is client.

---

## Data Fetching

```typescript
// Server Component — direct fetch
async function UsersPage() {
  const users = await fetchUsers();
  return <UserList users={users} />;
}

// Client Component — TanStack Query
"use client";
function UserSearch() {
  const { data, isLoading } = useUsers();
  // ...
}
```

---

## Global Store

Same as React — one store per entity with Zustand/Jotai. Wrap provider at layout level. See `rovi-store` skill.

---

## Rules

- **App Router only.** No Pages Router.
- **Server Components by default.** `"use client"` only for interactivity.
- **TanStack Query for client fetching.** Server Components use direct fetch/server actions.
- **Global store for client state.** Same pattern as rovi-react.
- **No `fromPersistence` pattern.** Do not add static reconstruction methods unless explicitly requested.
