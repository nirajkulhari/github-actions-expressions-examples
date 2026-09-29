# fromJSON() vs toJSON() - Complete Comparison & Cheat Sheet

## Side-by-Side Comparison Table

| Aspect | `fromJSON()` | `toJSON()` |
|--------|--------------|-----------|
| **Direction** | JSON String → Parsed Object | Object → JSON String |
| **Purpose** | Parse/read JSON data | Serialize/convert to JSON |
| **Input Type** | String | Object, Array, Context |
| **Output Type** | Object, Array, Value | String |
| **Syntax** | `fromJSON('{"key":"value"}')` | `toJSON(github.event)` |
| **Example Input** | `'{"name":"Alice"}'` | `{name: "Alice"}` |
| **Example Output** | `{name: "Alice"}` | `'{"name":"Alice"}'` |
| **Use Case** | Parse API responses, config files | Log data, pass data between jobs |
| **Common with** | `join()`, conditionals | Logging, storage |

---

## Visual Flow

### fromJSON() Flow
```
JSON String (text)
        ↓
   fromJSON()
        ↓
Parsed Object (usable)
        ↓
Access properties with dot notation
```

**Example:**
```yaml
fromJSON('{"name":"Alice","age":30}')
              ↓
         {name: "Alice", age: 30}
              ↓
.name → "Alice"
.age → 30
```

### toJSON() Flow
```
Object/Context (usable)
        ↓
   toJSON()
        ↓
JSON String (text)
        ↓
Store, log, or pass as string
```

**Example:**
```yaml
toJSON(github.event)
        ↓
'{"action":"opened","issue":{...}}'
        ↓
Store in variable or log
```

---

## Complete Cheat Sheet

### fromJSON() Examples

```yaml
# Parse object
${{ fromJSON('{"name":"Alice"}').name }}
# → "Alice"

# Parse array
${{ fromJSON('["a","b","c"]')[0] }}
# → "a"

# Parse number
${{ fromJSON('42') }}
# → 42

# Parse boolean
${{ fromJSON('true') }}
# → true

# Parse null
${{ fromJSON('null') }}
# → null

# Nested object
${{ fromJSON('{"user":{"age":30}}').user.age }}
# → 30

# Array of objects
${{ fromJSON('[{"id":1}]')[0].id }}
# → 1

# Combined with join()
${{ join(fromJSON('["a","b","c"]'), '-') }}
# → "a-b-c"
```

### toJSON() Examples

```yaml
# Serialize context
${{ toJSON(github.event) }}
# → '{"action":"opened",...}'

# Serialize needs output
${{ toJSON(needs.setup.outputs) }}
# → '{"version":"1.0.0",...}'

# Serialize environment
${{ toJSON(env) }}
# → '{"PATH":"/usr/bin:...",...}'

# Serialize custom object (in some contexts)
${{ toJSON(matrix) }}
# → '{"os":"ubuntu-latest",...}'
```

---

## When to Use Each

### Use `fromJSON()` When:

✅ Parsing an API response stored as a string  
✅ Reading a JSON config file as text  
✅ Working with step outputs that contain JSON  
✅ Accessing nested properties in complex data  
✅ Using arrays with `join()`  
✅ Need to access specific fields from JSON data  

**Example:**
```yaml
- id: api_call
  run: curl https://api.example.com/status > result.json

- name: Parse response
  run: |
    RESPONSE='{"status":"active","version":"2.0"}'
    echo "Status: ${{ fromJSON(RESPONSE).status }}"
```

### Use `toJSON()` When:

✅ Logging entire context/object for debugging  
✅ Passing complex data between jobs  
✅ Storing object as environment variable  
✅ Creating JSON output for external tools  
✅ Serializing matrix strategy data  
✅ Need entire object as a string  

**Example:**
```yaml
- name: Log workflow context
  run: echo "${{ toJSON(github.event) }}"

- name: Create output
  run: |
    echo "metadata=${{ toJSON(matrix) }}" >> $GITHUB_OUTPUT
```

---

## Practical Scenarios

### Scenario 1: API Response Handling

**Without fromJSON() (Wrong):**
```yaml
RESPONSE='{"name":"Alice","role":"admin"}'
echo "Name: $RESPONSE.name"  # ❌ Prints: {name...}.name
```

**With fromJSON() (Correct):**
```yaml
RESPONSE='{"name":"Alice","role":"admin"}'
echo "Name: ${{ fromJSON(RESPONSE).name }}"  # ✅ Prints: Alice
```

---

### Scenario 2: Configuration Management

```yaml
# Store config
CONFIG: '{"db_host":"localhost","db_port":5432,"cache_ttl":3600}'

# Access with fromJSON()
- name: Use config
  run: |
    echo "DB Host: ${{ fromJSON(env.CONFIG).db_host }}"
    echo "DB Port: ${{ fromJSON(env.CONFIG).db_port }}"
    echo "Cache TTL: ${{ fromJSON(env.CONFIG).cache_ttl }}"
```

---

### Scenario 3: Job Communication

**Job 1 - Produce Data:**
```yaml
- id: build
  run: |
    BUILD_INFO='{"version":"1.2.3","sha":"abc123def456","timestamp":"2024-01-15"}'
    echo "info=$BUILD_INFO" >> $GITHUB_OUTPUT
```

**Job 2 - Consume Data:**
```yaml
needs: build
- name: Use build info
  run: |
    echo "Version: ${{ fromJSON(needs.build.outputs.info).version }}"
    echo "SHA: ${{ fromJSON(needs.build.outputs.info).sha }}"
```

---

### Scenario 4: Debug Logging

```yaml
# Log entire context for debugging
- name: Debug workflow
  run: echo "${{ toJSON(github) }}"

# Later, parse if needed
- name: Extract event details
  run: echo "${{ fromJSON(toJSON(github.event)).action }}"
```

---

### Scenario 5: Complex Array Processing

```yaml
TEAMS: '[{"name":"Frontend","members":3},{"name":"Backend","members":4}]'

- name: Process teams
  run: |
    echo "Team 1: ${{ fromJSON(TEAMS)[0].name }}"
    echo "Team 1 size: ${{ fromJSON(TEAMS)[0].members }}"
    echo "Team 2: ${{ fromJSON(TEAMS)[1].name }}"
    echo "Team 2 size: ${{ fromJSON(TEAMS)[1].members }}"
```

---

## Round-Trip Conversion

Sometimes you need to convert object → string → object:

```yaml
# Original context/object
github.event

# Convert to JSON string
${{ toJSON(github.event) }}

# Parse back to object
${{ fromJSON(toJSON(github.event)) }}

# Access property
${{ fromJSON(toJSON(github.event)).action }}
```

**Why do this?**
- Some contexts are already objects
- `toJSON()` ensures it's a string for storage
- `fromJSON()` makes it usable again

---

## Data Types Comparison

### fromJSON() - Parse All Types

```yaml
# String
fromJSON('"hello"')           # "hello"

# Number (integer)
fromJSON('42')                # 42

# Number (float)
fromJSON('3.14')              # 3.14

# Boolean
fromJSON('true')              # true
fromJSON('false')             # false

# Null
fromJSON('null')              # null

# Array
fromJSON('[1,2,3]')           # [1, 2, 3]

# Object
fromJSON('{"key":"val"}')     # {key: "val"}
```

### toJSON() - Serialize to String

```yaml
# String
toJSON('hello')               # "hello"

# Number
toJSON(42)                    # 42

# Boolean
toJSON(true)                  # true

# Array
toJSON([1,2,3])              # [1,2,3]

# Object
toJSON({key: "val"})         # {"key":"val"}

# Context
toJSON(github.event)          # Full event as JSON string
```

---

## Common Mistakes & Solutions

### ❌ Mistake 1: Forgetting Quotes Around JSON String

```yaml
# WRONG
${{ fromJSON({"name":"Alice"}) }}

# CORRECT
${{ fromJSON('{"name":"Alice"}') }}
```

---

### ❌ Mistake 2: Using toJSON() When You Need the Object

```yaml
# WRONG - toJSON() returns a string, not usable as object
value: ${{ toJSON(needs.job.outputs).version }}

# CORRECT - Just access the output directly
value: ${{ needs.job.outputs.version }}

# Or if stored as JSON string, parse it first
value: ${{ fromJSON(needs.job.outputs.data).version }}
```

---

### ❌ Mistake 3: Invalid JSON Syntax

```yaml
# WRONG - Single quotes inside (invalid JSON)
${{ fromJSON('{"name": \'Alice\'}') }}

# CORRECT - Double quotes
${{ fromJSON('{"name": "Alice"}') }}
```

---

### ❌ Mistake 4: Forgetting the Index for Arrays

```yaml
# WRONG - Trying to access array without index
${{ fromJSON('["a","b","c"]').0 }}

# CORRECT - Use bracket notation
${{ fromJSON('["a","b","c"]')[0] }}
```

---

## Quick Reference Matrix

```
╔════════════════════════════════════════════════════════════════════╗
║                   Operation Quick Reference                       ║
╠════════════════════════════════════════════════════════════════════╣
║ Task                          │ fromJSON()           │ toJSON()    ║
├───────────────────────────────┼──────────────────────┼─────────────┤
║ Parse JSON string             │ ✓ YES                │ ✗ NO        ║
║ Access object properties      │ ✓ YES                │ ✗ NO        ║
║ Use in array index            │ ✓ YES                │ ✗ NO        ║
║ Log entire context            │ ✗ NO                 │ ✓ YES       ║
║ Pass data between jobs        │ ✗ NO                 │ ✓ YES       ║
║ Convert to JSON string        │ ✗ NO                 │ ✓ YES       ║
║ Use with join()               │ ✓ YES                │ ✗ NO        ║
║ Use in conditionals           │ ✓ YES                │ ✗ NO        ║
╚════════════════════════════════════════════════════════════════════╝
```

---

## Decision Tree

```
Do you have a JSON string and want to use it?
  ↓
  YES → Use fromJSON()
  
  NO → Do you have an object and want to store/log it?
    ↓
    YES → Use toJSON()
    
    NO → You might not need either!
```

---

## Summary

| Aspect | fromJSON() | toJSON() |
|--------|-----------|---------|
| **What it does** | Parses JSON → Usable object | Converts object → JSON string |
| **When to use** | Parse API responses, configs | Log data, pass between jobs |
| **Input** | `'{"key":"value"}'` | `{key: "value"}` |
| **Output** | `{key: "value"}` | `'{"key":"value"}'` |
| **Pairs with** | `join()`, conditionals | Logging, storage |
| **Remember** | Always use single quotes around JSON string | No quotes needed, it's already an object |

---

## Examples in Workflow

```yaml
name: JSON Comparison

on: push

jobs:
  example:
    runs-on: ubuntu-latest
    
    outputs:
      data: ${{ toJSON(matrix) }}  # Convert to JSON string
    
    strategy:
      matrix:
        version: [1, 2, 3]
    
    steps:
      # Using fromJSON() to parse
      - name: Parse JSON
        run: |
          CONFIG='{"version":"1.2.3","active":true}'
          echo "Version: ${{ fromJSON(CONFIG).version }}"
      
      # Using toJSON() to serialize
      - name: Log context
        run: echo "${{ toJSON(github.event) }}"
      
      # Round-trip
      - name: Convert and parse
        run: |
          echo "${{ toJSON(matrix) }}"  # Serialize to JSON string
          echo "${{ fromJSON(toJSON(matrix)).version }}"  # Parse back

  use_output:
    needs: example
    runs-on: ubuntu-latest
    
    steps:
      # Using toJSON() output
      - name: Use serialized data
        run: |
          # Data comes as JSON string from previous job
          echo "Data: ${{ needs.example.outputs.data }}"
          
          # Parse it with fromJSON() if needed
          echo "Parsed: ${{ fromJSON(needs.example.outputs.data) }}"
```

---

**Key Takeaway:**
- **fromJSON()** = Parse JSON string to use it
- **toJSON()** = Convert object to JSON string to store/log it
- Use them together for round-trip conversion
- Always remember: fromJSON input is a STRING, toJSON output is a STRING
