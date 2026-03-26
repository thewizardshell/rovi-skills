---
name: rovi-store
description: "State management patterns: one store per entity, loading/error state, try-catch with finally, optimistic updates. Auto-loads when working on frontend, creating components, building UI, or modifying stores with Jotai, Redux, Zustand, or Pinia. Every frontend feature must have its own store."
---

# ROVI — State Management

## Rules

Consistent structure regardless of the state library:

- **One store per entity.** Never a mega-store.
- **Always `isLoading` and `error` as state.** Each store manages its own loading and error state.
- **Try-catch on every async action.** Always with `finally` to reset `isLoading`.
- **Visually separated sections:** state, computed/derived, actions.
- **Optimistic local state updates** when appropriate (push to array after create, filter after delete).

---

## Store Examples

Read `${CLAUDE_SKILL_DIR}/references/store-examples.md` for a full store pattern with state, computed, and actions sections.
