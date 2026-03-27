---
name: rovi-tooling
description: "Project tooling setup: linters (Biome preferred, ESLint fallback), package managers (bun/pnpm preferred), .npmrc security with ignore-scripts, dependency management. Auto-loads when initializing projects, installing packages, configuring linters, or setting up dev tooling."
---

# ROVI — Tooling

## Linters

1. **Biome first.** Always propose Biome as the default linter/formatter.
2. **ESLint as fallback** only if the project already uses it or there is a specific reason.
3. If the project already has a linter configured, respect it. Do not switch without asking.

---

## Package Managers

### Preference order

1. **bun** — preferred for personal projects and new setups.
2. **pnpm** — preferred alternative, especially for monorepos.
3. **npm/yarn** — only if the project already uses it. Never switch a work project's package manager without asking.

### Rule

**Always respect the project's existing lockfile.** If you see `package-lock.json`, use `npm`. If `pnpm-lock.yaml`, use `pnpm`. If `bun.lockb`, use `bun`. Do not mix package managers.

---

## Security: .npmrc / bunfig.toml

Every project should have a base security config. The minimum recommended:

### npm / pnpm (.npmrc)

```ini
save-exact=true
package-lock=true
audit=true
audit-level=moderate
registry=https://registry.npmjs.org/
fund=false
update-notifier=false
```

### Optional: ignore-scripts

Adds `ignore-scripts=true` to block all lifecycle scripts (postinstall, prepare, etc.) from dependencies AND from the project itself. **Recommended for maximum security**, but has trade-offs:

- Husky's `prepare` script won't run automatically — requires `npm run prepare` after install
- Native addons (esbuild, sharp, bcrypt) need explicit rebuild

**If enabled**, add to `.npmrc`:

```ini
ignore-scripts=true
```

Or for bun, in `bunfig.toml`:

```toml
[install.lifecycle]
ignore-scripts = true
```

**Workarounds when ignore-scripts is enabled:**

```bash
# After install, manually run prepare (for Husky)
npm install && npm run prepare

# Rebuild native addons explicitly
npx prisma generate
```

**Alternative: `ignore-dependency-scripts`** — only blocks dependency scripts, lets your own project scripts (like Husky `prepare`) run normally:

```json
{
  "prepare": "npx --yes ignore-dependency-scripts \"husky\""
}
```

For pnpm, allowlist specific packages that need scripts:

```json
{
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild", "sharp", "bcrypt"]
  }
}
```

Choose the approach that fits the project. Both are valid.

---

## Vetting Packages Before Installing

**Never install a package without checking it first.** Supply chain attacks are real and frequent (compromised tokens, typosquatting, malicious postinstall scripts).

### Quick CLI checks

```bash
# View maintainers, repo, last publish date
npm view <pkg> maintainers repository time

# Check for install scripts (red flag if unexpected)
npm view <pkg> scripts

# Check dependency tree
npm view <pkg> dependencies

# Download and inspect without installing
npm pack <pkg> --dry-run
```

### Red flags

- Package has `postinstall`/`preinstall` scripts that are not native addons
- Very few downloads but similar name to a popular package (typosquatting)
- No linked GitHub repo or repo is empty
- Recently published by a new maintainer on an established package
- Obfuscated code in the tarball

### Typosquatting protection

- Always copy package names from official docs or npmjs.com, never type from memory
- Use scoped packages (`@org/pkg`) when available — harder to typosquat
- Common tricks: swapped chars (`axois`), missing hyphens (`reactrouterdom`), scope confusion

---

## CI Security Pipeline

Always include these checks in CI:

```bash
# 1. Frozen lockfile — no surprise dependency changes
npm ci                              # npm
pnpm install --frozen-lockfile      # pnpm
bun install --frozen-lockfile       # bun

# 2. Lockfile integrity check
npx lockfile-lint --path package-lock.json --type npm \
  --allowed-hosts npm \
  --validate-https \
  --validate-integrity \
  --validate-package-names

# 3. Audit for known CVEs
npm audit --audit-level=high --omit=dev

# 4. Verify package signatures/provenance
npm audit signatures
```

### Recommended tools

| Tool | What it catches |
|------|----------------|
| `npm audit` | Known CVEs in dependencies |
| `npm audit signatures` | Unsigned or tampered packages |
| `lockfile-lint` | Lockfile injection, HTTP registries, missing integrity |
| `Socket.dev` | Malicious code, typosquatting, install scripts, obfuscation |
| `Snyk` | Vulnerabilities + license issues |
| `Renovate` / `Dependabot` | Outdated deps with known vulns |

---

## Git Hooks: Husky + Pre-commit

**Optional but recommended.** Set up Husky with a pre-commit hook that runs the linter with auto-fix.

**Note:** If using `ignore-scripts=true`, Husky's `prepare` script won't run automatically. See the "Security: ignore-scripts" section for workarounds.

```bash
# Setup
npx husky init  # or bunx husky init
echo "npx biome check --fix --staged" > .husky/pre-commit
```

If using ESLint instead of Biome:

```bash
echo "npx eslint --fix ." > .husky/pre-commit
```

For staged-only files, prefer `lint-staged`:

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx,js,jsx}": ["biome check --fix"],
    "*.{json,css,md}": ["biome format --fix"]
  }
}
```

```bash
echo "npx lint-staged" > .husky/pre-commit
```

---

## Dependency Management

- **Minimize dependencies.** Before adding a package, check if the functionality can be achieved with what is already installed or with a few lines of code.
- **No random packages.** Every dependency must be justified. If adding a new one, state why.
- **Check before installing.** Look at the package's size, maintenance status, and weekly downloads. Avoid abandoned or bloated packages.
- **Pin versions.** `save-exact=true` in `.npmrc`. No `^` or `~` ranges in production.
- **Audit regularly.** Run `npm audit` / `pnpm audit` and address vulnerabilities.
- **Verify provenance.** Prefer packages with npm provenance (green checkmark on npmjs.com). Run `npm audit signatures` to validate.

---

## Error Logging (Compounding Engineering)

When a mistake is made and corrected during a session — a wrong pattern, a bad package choice, a misconfigured tool — save a concise memory entry:

- **1-3 lines max.** Core problem + what was wrong + the fix.
- No verbose explanations. Scannable.
- This builds institutional knowledge that prevents repeating the same mistakes.

Format:
```
Wrong: [what was done incorrectly]
Fix: [what the correct approach is]
```
