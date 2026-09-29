# GitHub Actions `hashFiles()` Function - Complete Guide

## Function Signature

```
hashFiles( path )
```

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `path` | String | File path pattern(s) to hash (supports wildcards) |

## Return Value

| Type | Description |
|------|-------------|
| String | SHA256 hash of the specified files |

## Key Characteristics

✅ **Generates** a SHA256 hash of file contents  
✅ **Updates** automatically when file contents change  
✅ **Supports** glob patterns for multiple files  
✅ **Perfect** for caching strategies  
✅ **Deterministic** - same files always produce same hash  
✅ **Case-sensitive** on Linux, case-insensitive on Windows/macOS  

---

## Basic Concept

`hashFiles()` computes a checksum (hash) of one or more files. Think of it as a fingerprint:

- File changes → Hash changes
- Same file contents → Same hash
- Different file contents → Different hash

```
File: package.json
Contents: {"dependencies": {...}}
         ↓
    hashFiles()
         ↓
Hash: 3f7d8c2a1b9e4f6d5c8a2b1e3f7d9c6a
```

When you change the file:
```
File: package.json
Contents: {"dependencies": {...updated...}}
         ↓
    hashFiles()
         ↓
Hash: 9c6a2b1e3f7d8c5a4f2e1d3c9b8a7f6e (DIFFERENT!)
```

---

## Examples Explained

### Example 1: Hash a Single File

```yaml
${{ hashFiles('package.json') }}
```

**Output:**
```
3f7d8c2a1b9e4f6d5c8a2b1e3f7d9c6a
```

**What it does:**
- Reads the contents of `package.json`
- Computes SHA256 hash
- Returns the hash as a string

---

### Example 2: Hash Multiple Files with Wildcard

```yaml
${{ hashFiles('*.json') }}
```

**Matches:**
- `package.json`
- `package-lock.json`
- `tsconfig.json`
- `config.json`

**Output:**
```
5e8c1a3f7d2b9c6a1e4f8d3b2c9a7f6e
```

**How it works:**
- Finds all `.json` files in root directory
- Hashes all their contents together
- Returns single hash

---

### Example 3: Hash All Files in a Directory

```yaml
${{ hashFiles('config/**') }}
```

**Matches:**
- `config/app.yaml`
- `config/database.yml`
- `config/settings/security.yaml`

**Output:**
```
7f2e1d3c9b8a4f6e5d8c1a3f7d2b9c6a
```

---

### Example 4: Hash Multiple Patterns

```yaml
${{ hashFiles('package.json', 'yarn.lock') }}
```

**Matches:**
- `package.json`
- `yarn.lock`

**Output:**
```
2b9c6a1e4f8d3b2c9a7f6e5d8c1a3f7d
```

**Note:** When you provide multiple paths, it hashes both files together

---

### Example 5: Complex Glob Pattern

```yaml
${{ hashFiles('src/**/*.ts', 'src/**/*.tsx') }}
```

**Matches:**
- `src/app.ts`
- `src/index.ts`
- `src/components/Button.tsx`
- `src/hooks/useAuth.tsx`
- `src/utils/helpers.ts`

**Output:**
```
8a7f6e5d8c1a3f7d2b9c6a1e4f8d3b2c
```

---

## Real-World Use Cases

### Use Case 1: Cache NPM Dependencies

```yaml
name: Test

on: push

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Cache node modules
        uses: actions/cache@v3
        with:
          path: node_modules
          key: node-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            node-
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
```

**How it works:**
1. `hashFiles('package-lock.json')` creates a unique key
2. If `package-lock.json` hasn't changed, uses cached `node_modules`
3. If `package-lock.json` changes, hash changes, creates new cache
4. Saves CI time by avoiding reinstalls

---

### Use Case 2: Cache Pip Dependencies (Python)

```yaml
- name: Cache pip packages
  uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: pip-${{ hashFiles('requirements.txt') }}
    restore-keys: |
      pip-
```

**Benefit:** Python dependencies only re-downloaded if `requirements.txt` changes

---

### Use Case 3: Cache Maven Packages (Java)

```yaml
- name: Cache Maven
  uses: actions/cache@v3
  with:
    path: ~/.m2/repository
    key: maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      maven-
```

**Benefit:** Java packages only re-downloaded if any `pom.xml` changes

---

### Use Case 4: Cache Docker Build Layers

```yaml
- name: Cache Docker layers
  uses: actions/cache@v3
  with:
    path: /tmp/.buildx-cache
    key: docker-${{ hashFiles('Dockerfile', 'package.json') }}
    restore-keys: |
      docker-
```

---

### Use Case 5: Environment-Specific Caching

```yaml
- name: Cache with environment
  uses: actions/cache@v3
  with:
    path: build/
    key: build-${{ runner.os }}-${{ hashFiles('Makefile', 'src/**') }}
    restore-keys: |
      build-${{ runner.os }}-
```

**Benefit:** Separate caches for Windows, macOS, Linux

---

### Use Case 6: Conditional Steps Based on Hash Change

```yaml
- id: hash
  name: Calculate file hash
  run: echo "current_hash=${{ hashFiles('config.yaml') }}" >> $GITHUB_OUTPUT

- name: Config changed
  if: steps.hash.outputs.current_hash != env.previous_hash
  run: echo "Configuration has changed, rebuilding..."

- name: Config unchanged
  if: steps.hash.outputs.current_hash == env.previous_hash
  run: echo "No changes detected, skipping rebuild"
```

---

### Use Case 7: Cache Build Artifacts

```yaml
- name: Build application
  run: npm run build

- name: Cache build output
  uses: actions/cache@v3
  with:
    path: dist/
    key: build-${{ hashFiles('src/**', 'webpack.config.js', 'package.json') }}
    restore-keys: |
      build-
```

---

## Glob Pattern Examples

| Pattern | Matches |
|---------|---------|
| `*.json` | All JSON files in root only |
| `**/*.json` | All JSON files recursively |
| `src/**/*.ts` | All TypeScript files in src |
| `config/*` | Files directly in config (not subdirs) |
| `config/**` | All files in config and subdirs |
| `src/{utils,hooks}/**/*.ts` | Files in utils and hooks folders |
| `package*.json` | package.json, package-lock.json, etc |
| `requirements{,-dev}.txt` | requirements.txt, requirements-dev.txt |

---

## Common Patterns for Popular Languages

### JavaScript/Node.js
```yaml
# Hash both package files
${{ hashFiles('package.json', 'package-lock.json') }}

# Or just one
${{ hashFiles('package-lock.json') }}
```

### Python
```yaml
# Single requirements file
${{ hashFiles('requirements.txt') }}

# Multiple requirements files
${{ hashFiles('requirements.txt', 'requirements-dev.txt') }}
```

### Java/Maven
```yaml
# All pom.xml files
${{ hashFiles('**/pom.xml') }}
```

### Go
```yaml
# Go modules
${{ hashFiles('go.sum') }}
```

### Rust
```yaml
# Cargo lock file
${{ hashFiles('Cargo.lock') }}
```

### Ruby
```yaml
# Gemfile lock
${{ hashFiles('Gemfile.lock') }}
```

---

## Workflow Example: Complete Caching Strategy

```yaml
name: Build with Caching

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      # Cache 1: npm dependencies
      - name: Cache npm dependencies
        uses: actions/cache@v3
        id: npm-cache
        with:
          path: node_modules
          key: npm-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            npm-${{ runner.os }}-
      
      # Cache 2: build output
      - name: Cache build output
        uses: actions/cache@v3
        id: build-cache
        with:
          path: dist/
          key: build-${{ runner.os }}-${{ hashFiles('src/**', 'webpack.config.js', 'package.json') }}
          restore-keys: |
            build-${{ runner.os }}-
      
      # Install if not cached
      - name: Install dependencies
        if: steps.npm-cache.outputs.cache-hit != 'true'
        run: npm ci
      
      # Build if not cached
      - name: Build
        if: steps.build-cache.outputs.cache-hit != 'true'
        run: npm run build
      
      # Always run tests (don't cache)
      - name: Run tests
        run: npm test
      
      # Report cache status
      - name: Cache status
        run: |
          echo "NPM cache hit: ${{ steps.npm-cache.outputs.cache-hit }}"
          echo "Build cache hit: ${{ steps.build-cache.outputs.cache-hit }}"
```

---

## How Cache Keys Work

```
Cache Key Structure:
key: prefix-${{ hashFiles('file') }}
      ↑        ↑                   ↑
   Prefix   Separator         Hash of file
```

**Example:**
```
key: node-3f7d8c2a1b9e4f6d5c8a2b1e3f7d9c6a
key: build-5e8c1a3f7d2b9c6a1e4f8d3b2c9a7f6e
key: pip-2b9c6a1e4f8d3b2c9a7f6e5d8c1a3f7d
```

**Restore keys** (fallback):
```yaml
restore-keys: |
  npm-${{ runner.os }}-
  npm-
```

If exact key not found, tries partial matches in order

---

## Common Mistakes & Solutions

### ❌ Mistake 1: Wrong File Path

```yaml
# WRONG - file doesn't exist at root
key: npm-${{ hashFiles('dependencies/package-lock.json') }}

# CORRECT - relative to repo root
key: npm-${{ hashFiles('package-lock.json') }}
```

---

### ❌ Mistake 2: Hashing Wrong File for Python

```yaml
# WRONG - this file changes frequently
key: pip-${{ hashFiles('setup.py') }}

# CORRECT - this locks versions
key: pip-${{ hashFiles('requirements.txt') }}
```

---

### ❌ Mistake 3: Missing Wildcard

```yaml
# WRONG - only hashes files in root
key: src-${{ hashFiles('src/*.ts') }}

# CORRECT - hashes all TS files recursively
key: src-${{ hashFiles('src/**/*.ts') }}
```

---

### ❌ Mistake 4: Not Hashing All Dependencies

```yaml
# WRONG - misses package-lock.json changes
key: npm-${{ hashFiles('package.json') }}

# CORRECT - includes both
key: npm-${{ hashFiles('package*.json') }}
```

---

## Hash Output Example

When you use `hashFiles()`, you get a 64-character SHA256 hash:

```
3f7d8c2a1b9e4f6d5c8a2b1e3f7d9c6a5e8c1a3f7d2b9c6a1e4f8d3b2c9a7f
```

This hash:
- Changes if ANY file content changes
- Is identical for same file contents
- Is deterministic (always same for same input)
- Is used as cache key identifier

---

## Performance Impact

### Without Caching
```
Time: 1m 30s

Steps:
├─ Checkout: 5s
├─ Setup Node: 15s
├─ Install deps: 45s ← Slow!
├─ Build: 20s
└─ Test: 5s
```

### With Caching (first run)
```
Time: 1m 30s (same as above)

Steps:
├─ Checkout: 5s
├─ Setup Node: 15s
├─ Install deps: 45s (creates cache)
├─ Build: 20s
└─ Test: 5s
```

### With Caching (subsequent runs)
```
Time: 30s ← Much faster!

Steps:
├─ Checkout: 5s
├─ Setup Node: 15s
├─ Install deps: 3s ← Uses cache!
├─ Build: 20s
└─ Test: 5s

SAVES: 42 seconds per run!
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Purpose** | Hash file contents for cache keys |
| **Returns** | SHA256 hash string (64 chars) |
| **Use with** | actions/cache@v3 |
| **Changes when** | File contents change |
| **Supports** | Wildcards and glob patterns |
| **Common files** | package-lock.json, requirements.txt, go.sum, Cargo.lock |
| **Saves** | Time by caching dependencies and build artifacts |

---

## Key Takeaways

1. **hashFiles()** generates a fingerprint of file(s)
2. **Use it** in cache keys to invalidate cache when files change
3. **Hash lockfiles** not source files (they're more stable)
4. **Combine** with actions/cache@v3 for CI/CD efficiency
5. **Multiple patterns** can be used separated by commas
6. **Glob patterns** supported: *, **, /

---

For complete workflow examples, see:  
`.github/workflows/11-hashFiles-function-guide.yml`
