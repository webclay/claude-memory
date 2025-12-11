# Bun

Bun is an all-in-one JavaScript runtime, package manager, bundler, and test runner.

## Key Facts

- **Bun 1.3+** uses `bun.lock` (text-based JSONC format) instead of the old `bun.lockb` (binary)
- Drop-in replacement for Node.js with most npm packages working out of the box
- Significantly faster than npm/pnpm for package installation

## Common Commands

```bash
# Install dependencies
bun install

# Add a package
bun add <package>

# Add dev dependency
bun add -d <package>

# Remove a package
bun remove <package>

# Run scripts from package.json
bun run dev
bun run build
bun run test

# Run a file directly
bun run script.ts
```

## Migrating from pnpm/npm

1. Install Bun: `curl -fsSL https://bun.sh/install | bash`
2. Run `bun install` to create `bun.lock`
3. Delete old lockfile (`pnpm-lock.yaml` or `package-lock.json`)
4. Use `bun` commands going forward

## Compatibility Notes

- Most Node.js APIs are supported but not 100%
- Some packages with native bindings may have issues
- Works with Appwrite SDK (confirmed)
