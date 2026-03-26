---
name: rovi-testing
description: "Testing philosophy: manual-first then automated, unit tests for domain/use-cases, integration tests for repos/endpoints, mock interfaces. Auto-loads when writing or planning tests."
---

# ROVI — Testing

## Philosophy

1. Implement the feature completely.
2. Test manually — endpoints with Postman or similar, UI in the browser.
3. Once it works, write automated tests.

## Rules

- **Unit tests** for business logic (domain and use cases).
- **Integration tests** for repositories and endpoints.
- **Mock the interfaces** — this is why everything has an interface.
- Test names describe the expected behavior, not the method name.
- Each test covers one scenario. No multi-assertion mega-tests.

---

## Test Examples

Read `${CLAUDE_SKILL_DIR}/references/testing-examples.md` for unit test patterns with mocked interfaces.
