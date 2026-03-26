---
name: rovi-react
description: "React patterns: TanStack (React Query + Router), global store with Zustand/Jotai, component structure, hooks. Auto-loads when working with React, creating components, or building frontend with React and TanStack."
---

# ROVI — React

## Stack

- **TanStack Query** for server state (fetching, caching, mutations)
- **TanStack Router** for routing (type-safe)
- **Zustand or Jotai** for global client state
- **framer-motion** for animations (see rovi-design skill)

---

## Project Structure

```
src/
  assets/
  components/              > Global reusable components
  core/
    layout/
    store/                 > Global store (Zustand/Jotai)
    providers/             > QueryClient, Router, Theme providers
  features/
    [FeatureName]/
      components/          > Feature-specific components
      hooks/               > Feature-specific hooks (useQueries, mutations)
      store/               > Feature-specific store (if needed)
      views/               > Feature pages/views
      types/               > Feature types
  hooks/                   > Global shared hooks
  utils/
  styles/
    global.css             > CSS variables (see rovi-design)
```

---

## Global Store

Every React project has a global store. One store per entity, never a mega-store. See `rovi-store` skill for the full pattern.

---

## TanStack Query Pattern

```typescript
// features/Users/hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { userApi } from "../api/userApi";

export function useUsers() {
  return useQuery({
    queryKey: ["users"],
    queryFn: userApi.findAll,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["users"] });
    },
  });
}
```

---

## Component Pattern

```typescript
// features/Users/components/UserCard.tsx
interface UserCardProps {
  user: User;
  onDelete: (id: string) => void;
}

export function UserCard({ user, onDelete }: UserCardProps) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true }}
    >
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onDelete(user.id)}>Eliminar</button>
    </motion.div>
  );
}
```

---

## Rules

- **TanStack Query for server state.** No fetching in useEffect. No manual cache management.
- **Global store for client state.** UI state, filters, selections — not server data.
- **One store per entity.** See rovi-store skill.
- **Function components only.** No class components.
- **Custom hooks for logic.** Components are for rendering, hooks are for logic.
- **No `fromPersistence` pattern.** Do not add static reconstruction methods unless explicitly requested.
