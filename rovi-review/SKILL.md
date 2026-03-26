---
name: rovi-review
description: "Review code against rovi standards: architecture, typing, naming, error handling, documentation."
argument-hint: "[file or directory path]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep
context: fork
---

# ROVI — Code Review

Review the code at `$ARGUMENTS` against rovi standards. If no path is provided, review recently changed files.

## Changed files context

!`git diff --name-only HEAD~1 2>/dev/null || echo "No git history available"`

## Review Checklist

For each file, check the following:

### Architecture
- [ ] Layers respected: domain has zero external dependencies
- [ ] Interfaces defined for anything that might change
- [ ] Dependencies injected, never instantiated inside classes
- [ ] Repository pattern used for data access
- [ ] Single responsibility: no function doing more than 2 things

### Typing
- [ ] No `any` — uses `unknown` with validation instead
- [ ] `strict: true` in tsconfig (if TypeScript)
- [ ] Explicit types on public function parameters and return types

### Naming & Style
- [ ] Code naming in English
- [ ] Comments/documentation in Spanish
- [ ] Functions under 30 lines
- [ ] One file, one responsibility
- [ ] Visual section separators where applicable

### Error Handling
- [ ] Errors are typed (extend AppError or equivalent)
- [ ] No swallowed errors — every catch block handles or rethrows
- [ ] No `console.log` as production debugging strategy

### Documentation
- [ ] No emojis in comments
- [ ] JSDoc/TSDoc only where there is a non-obvious "why"
- [ ] No narrating the "what" — only the "why"

### State Management (if applicable)
- [ ] One store per entity
- [ ] `isLoading` and `error` in state
- [ ] Try-catch with `finally` on async actions

## Instructions

1. Read all files in the target path (or changed files if no path given).
2. For each file, evaluate against the checklist above.
3. Report findings grouped by file, with specific line references.
4. Severity levels: **violation** (must fix), **suggestion** (should fix), **note** (consider).
5. If a file passes all checks, say so briefly. Do not pad the review.
6. End with a summary: total files reviewed, violations, suggestions, notes.
