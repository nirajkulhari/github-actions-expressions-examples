# GitHub Actions Expressions - Complete Guide

## Table of Contents
1. [Syntax](#syntax)
2. [Literals](#literals)
3. [Operators](#operators)
4. [Functions](#functions)
5. [Context Objects](#context-objects)
6. [Common Patterns](#common-patterns)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

## Syntax

All expressions must be enclosed in `${{ }}` format:

```yaml
${{ expression }}
```

### Important Notes

- Expressions are evaluated as strings or booleans depending on context
- You cannot use more than 65,536 characters in an expression
- Functions are case-sensitive
- Expressions can be used in conditionals, environment variables, and step outputs

## Literals

### String Literals

```yaml
# Single quoted strings
message: ${{ 'Hello World' }}

# Strings in functions
echo ${{ format('Hello {0}', 'Alice') }}
```

### Number Literals

```yaml
count: ${{ 42 }}
sum: ${{ 5 + 10 }}  # = 15
```

### Boolean Literals

```yaml
enabled: ${{ true }}
disabled: ${{ false }}
```

### Null

```yaml
empty: ${{ null }}
```

### Arrays (using fromJSON)

```yaml
colors: ${{ fromJSON('[\"red\", \"green\", \"blue\"]') }}
```

### Objects (using fromJSON)

```yaml
config: ${{ fromJSON('{\"version\": \"1.0\"}') }}
```

## Operators

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `==` | Equal | `${{ 5 == 5 }}` → `true` |
| `!=` | Not equal | `${{ 5 != 3 }}` → `true` |
| `<` | Less than | `${{ 3 < 5 }}` → `true` |
| `<=` | Less than or equal | `${{ 3 <= 3 }}` → `true` |
| `>` | Greater than | `${{ 5 > 3 }}` → `true` |
| `>=` | Greater than or equal | `${{ 5 >= 5 }}` → `true` |

### Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `&&` | AND | `${{ true && false }}` → `false` |
| `\|\|` | OR | `${{ true \|\| false }}` → `true` |
| `!` | NOT | `${{ !false }}` → `true` |

### Grouping

Use parentheses to group conditions:

```yaml
if: ${{ (github.event_name == 'push' || github.event_name == 'pull_request') && github.ref == 'refs/heads/main' }}
```

## Functions

### String Functions

#### contains(search, item)
```yaml
contains('hello world', 'world')  # → true
contains(github.ref, 'main')      # → true/false
```

#### startsWith(searchString, searchValue)
```yaml
startsWith('hello world', 'hello')        # → true
startsWith(github.ref, 'refs/heads/')    # → true/false
```

#### endsWith(searchString, searchValue)
```yaml
endsWith('hello world', 'world')          # → true
endsWith(github.event.head_commit.message, '[skip ci]')  # → true/false
```

#### format(string, replaceValue0, replaceValue1, ...)
```yaml
format('Hello {0}', 'Alice')                    # → "Hello Alice"
format('Name: {0}, Age: {1}', 'Bob', 25)       # → "Name: Bob, Age: 25"
format('Repo {0}/{1}', 'owner', 'repo')        # → "Repo owner/repo"
```

### Array Functions

#### join(array, separator)
```yaml
join(fromJSON('[\"a\", \"b\", \"c\"]'), '-')    # → "a-b-c"
```

### Type Functions

#### toJSON(value)
Converts a value to JSON format:
```yaml
toJSON(github.event)       # Outputs entire event as JSON
toJSON(needs.setup.outputs) # Outputs outputs as JSON
```

#### fromJSON(value)
Parses JSON string into an object:
```yaml
fromJSON('{\"key\": \"value\"}')
fromJSON(needs.setup.outputs.config)
```

### Built-in Functions

#### always()
Job continues even if previous steps failed:
```yaml
if: ${{ always() }}
```

#### success()
All previous steps succeeded:
```yaml
if: ${{ success() }}
```

#### failure()
Any previous step failed:
```yaml
if: ${{ failure() }}
```

#### cancelled()
Job was cancelled:
```yaml
if: ${{ cancelled() }}
```

## Context Objects

### github Context

Most commonly used context object with GitHub event and repository information.

```yaml
github.repository              # owner/repo
github.repository_owner        # owner
github.repository_name         # repo
github.actor                   # Username who triggered workflow
github.event_name              # Event type (push, pull_request, etc.)
github.event                   # Full event payload
github.ref                     # Full ref (refs/heads/main)
github.ref_name                # Short ref name (main, PR number, etc.)
github.ref_protected           # Whether ref is protected
github.sha                     # Commit SHA
github.head_ref                # PR head branch (PR events only)
github.base_ref                # PR base branch (PR events only)
github.workspace               # Checkout directory
github.server_url              # GitHub server URL
github.api_url                 # GitHub API URL
```

### env Context

Access environment variables:

```yaml
env:
  NODE_VERSION: '18'

steps:
  - run: echo ${{ env.NODE_VERSION }}
```

### secrets Context

Access repository secrets (requires explicit reference):

```yaml
- run: curl -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" ...
```

### jobs Context

Access outputs from previous jobs:

```yaml
needs: setup
steps:
  - run: echo ${{ needs.setup.outputs.version }}
```

### steps Context

Access step outputs and outcomes:

```yaml
- id: build
  run: npm run build
  
- run: echo "Outcome: ${{ steps.build.outcome }}"  # success/failure
```

Available properties:
- `steps.<id>.outputs.<output_name>` - Step output value
- `steps.<id>.outcome` - Result of step (success/failure)
- `steps.<id>.conclusion` - Result including skipped/cancelled

### runner Context

Information about the runner:

```yaml
runner.os              # Linux, Windows, macOS
runner.arch            # x64, x86, arm64, arm
runner.name            # Name of runner machine
runner.tool_cache      # Tools cache directory
runner.temp            # Temp directory
runner.debug           # true if debug logging enabled
```

## Common Patterns

### Conditional Deployment

```yaml
deploy:
  if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
  runs-on: ubuntu-latest
  steps:
    - run: echo "Deploying to production..."
```

### Branch-based Logic

```yaml
- name: Deploy staging
  if: ${{ github.ref == 'refs/heads/develop' }}
  run: ./deploy-staging.sh

- name: Deploy production
  if: ${{ github.ref == 'refs/heads/main' }}
  run: ./deploy-production.sh
```

### Using Step Outputs

```yaml
- id: version
  run: echo "version=$(cat VERSION)" >> $GITHUB_OUTPUT

- name: Use version
  run: echo "Building v${{ steps.version.outputs.version }}"
```

### Matrix with Conditions

```yaml
strategy:
  matrix:
    node: [16, 18, 20]
    
steps:
  - name: Test on Node ${{ matrix.node }}
    if: ${{ matrix.node != '16' || runner.os != 'windows-latest' }}
    run: npm test
```

### Job Dependencies

```yaml
build:
  runs-on: ubuntu-latest
  outputs:
    artifact: ${{ steps.build.outputs.path }}

deploy:
  needs: build
  runs-on: ubuntu-latest
  steps:
    - run: echo "Using ${{ needs.build.outputs.artifact }}"
```

## Best Practices

1. **Always quote expressions in YAML**: `if: ${{ condition }}`
2. **Use step IDs** for referencing step outputs: `- id: step-name`
3. **Escape special characters** when needed in strings
4. **Prefer `contains()` over regex** for simple string matching
5. **Use `always()` for cleanup steps** that should run regardless
6. **Document complex expressions** with inline comments
7. **Test expressions locally** using `act` tool
8. **Use context objects** instead of hardcoding values
9. **Break complex conditions** into multiple simpler ones
10. **Use `fromJSON()` and `toJSON()`** for working with complex data

## Troubleshooting

### Expression Not Evaluating

**Problem**: Expression shows literal text like `${{ github.ref }}` in logs

**Solution**: 
- Ensure you're using expressions in a context where they're evaluated (job `if`, step `if`, environment variables, etc.)
- Make sure the YAML syntax is correct with proper quotes

### Context Not Available

**Problem**: Getting `Null` when accessing a context variable

**Solution**:
- Check context is available for your workflow trigger
- PR contexts (like `github.head_ref`) only available on `pull_request` events
- Verify the exact property path is correct

### Variable Not Found

**Problem**: `Null` or empty value from `env.VARIABLE`

**Solution**:
- Check environment variable is defined at the job or step level
- Environment variables must be referenced with `env.` prefix in expressions
- Check for typos in variable name (case-sensitive)

### Complex Expression Errors

**Problem**: Expression works locally but fails in workflow

**Solution**:
- Check character limit (65,536 max)
- Verify operator precedence
- Test with simpler expressions first
- Check YAML quoting

### Output Not Passing to Next Job

**Problem**: `needs.job.outputs.variable` is empty

**Solution**:
- Declare `outputs:` in the job
- Use `$GITHUB_OUTPUT` to set output values
- Ensure step has an `id:` attribute
- Reference step output: `steps.step-id.outputs.name`

---

For more information, see [GitHub Actions Documentation](https://docs.github.com/en/actions)
