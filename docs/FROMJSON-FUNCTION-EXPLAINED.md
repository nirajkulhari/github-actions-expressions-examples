# GitHub Actions `fromJSON()` Function - Complete Explanation

## Function Signature

```
fromJSON( value )
```

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `value` | String | A JSON string to be parsed and converted to a JSON object or data type |

## Return Value

| Type | Description |
|------|-------------|
| Object \| Array \| String \| Number \| Boolean \| null | The parsed JSON data structure |

## Key Characteristics

✅ **Parses** JSON strings into usable objects/arrays  
✅ **Supports** all JSON data types (objects, arrays, strings, numbers, booleans, null)  
✅ **Enables** accessing nested properties with dot notation  
✅ **Works with** environment variables and GitHub context  
✅ **Pairs well** with `toJSON()` for round-trip serialization  

---

## Basic Concept: What is JSON?

JSON (JavaScript Object Notation) is a text format for storing data:

```json
{
  "name": "Alice",
  "age": 30,
  "city": "New York",
  "skills": ["JavaScript", "Python", "Go"],
  "active": true
}
```

When you have JSON **as a string**, you need to **parse it** to use it. That's what `fromJSON()` does.

---

## Examples Explained

### Example 1: Parse a Simple Object

```yaml
${{ fromJSON('{"name": "Alice", "age": 30}') }}
```

**Input (string):** `'{"name": "Alice", "age": 30}'`  
**Output (object):** A JSON object you can access

**Why:** The input is a JSON string. `fromJSON()` converts it to an object.

---

### Example 2: Access Nested Properties

```yaml
person: ${{ fromJSON('{"name": "Alice", "age": 30}') }}
name: ${{ fromJSON('{"name": "Alice", "age": 30}').name }}
```

**Input:** `'{"name": "Alice", "age": 30}'`  
**Output:** 
- Full object: `{"name": "Alice", "age": 30}`
- Just name: `Alice`
- Just age: `30`

**How it works:**
1. `fromJSON('...')` converts the string to an object
2. `.name` accesses the `name` property
3. `.age` accesses the `age` property

---

### Example 3: Parse a JSON Array

```yaml
colors: ${{ fromJSON('["red", "green", "blue"]') }}
first_color: ${{ fromJSON('["red", "green", "blue"]')[0] }}
```

**Input:** `'["red", "green", "blue"]'`  
**Output:**
- Full array: `["red", "green", "blue"]`
- First element: `red` (index 0)
- Second element: `green` (index 1)
- Third element: `blue` (index 2)

**How indexing works:**
```
Array: ["red", "green", "blue"]
Index:   [0]     [1]      [2]

Access first: [0] → red
Access second: [1] → green
```

---

### Example 4: Parse Numbers

```yaml
count: ${{ fromJSON('42') }}
pi: ${{ fromJSON('3.14159') }}
sum: ${{ fromJSON('42') + 8 }}
```

**Input:** String numbers `'42'` and `'3.14159'`  
**Output:** Actual numbers that can be used in math

**Why:** You need `fromJSON()` to convert string `"42"` to number `42`

---

### Example 5: Parse Booleans

```yaml
enabled: ${{ fromJSON('true') }}
disabled: ${{ fromJSON('false') }}
```

**Input:** String booleans `'true'`, `'false'`  
**Output:** Actual booleans `true`, `false`

**Usage in conditions:**
```yaml
if: ${{ fromJSON('true') }}  # Job runs
if: ${{ fromJSON('false') }} # Job skipped
```

---

### Example 6: Parse Null

```yaml
empty: ${{ fromJSON('null') }}
```

**Input:** String `'null'`  
**Output:** `null` (empty/no value)

**Use case:** When a value is intentionally empty

---

### Example 7: Complex Nested Object

```yaml
config: ${{ fromJSON('{"database": {"host": "localhost", "port": 5432}, "debug": true}') }}
host: ${{ fromJSON('{"database": {"host": "localhost", "port": 5432}, "debug": true}').database.host }}
port: ${{ fromJSON('{"database": {"host": "localhost", "port": 5432}, "debug": true}').database.port }}
```

**Input:**
```json
{
  "database": {
    "host": "localhost",
    "port": 5432
  },
  "debug": true
}
```

**Accessing nested properties:**
- `.database` → `{"host": "localhost", "port": 5432}`
- `.database.host` → `localhost`
- `.database.port` → `5432`
- `.debug` → `true`

**How it works:**
```
config object
  ├── database object
  │   ├── host: "localhost"
  │   └── port: 5432
  └── debug: true
```

---

### Example 8: Array of Objects

```yaml
${{ fromJSON('[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]') }}

first_person: ${{ fromJSON('[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]')[0].name }}
second_person: ${{ fromJSON('[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]')[1].name }}
```

**Input:**
```json
[
  {"name": "Alice", "age": 30},
  {"name": "Bob", "age": 25}
]
```

**Output:**
- Full array: Both objects
- `[0].name` → `Alice`
- `[0].age` → `30`
- `[1].name` → `Bob`
- `[1].age` → `25`

**How it works:**
```
Array of objects:
[
  0: {name: "Alice", age: 30},
  1: {name: "Bob", age: 25}
]

Access [0] → first object
Access [0].name → "Alice"
Access [1].name → "Bob"
```

---

### Example 9: Using with Environment Variables

```yaml
env:
  CONFIG: '{"environment": "production", "timeout": 30}'

steps:
  - name: Parse environment variable
    run: |
      ENV_VAR='${{ env.CONFIG }}'
      echo "Config: ${{ fromJSON(env.CONFIG).environment }}"
      echo "Timeout: ${{ fromJSON(env.CONFIG).timeout }}"
```

**Output:**
```
Config: production
Timeout: 30
```

---

### Example 10: Using with GitHub Context

```yaml
steps:
  - name: Parse GitHub event payload
    run: |
      echo "Full event: ${{ toJSON(github.event) }}"
      # Parse it back if needed
      EVENT_ACTION: ${{ fromJSON(toJSON(github.event)).action }}
```

---

## Common Use Cases

### Use Case 1: Parse Configuration File Content

```yaml
- name: Read config
  id: config
  run: echo "::set-output name=data::$(cat config.json)"

- name: Use config
  run: |
    echo "Database: ${{ fromJSON(steps.config.outputs.data).database }}"
    echo "Port: ${{ fromJSON(steps.config.outputs.data).port }}"
```

---

### Use Case 2: Parse API Response

```yaml
- name: Call API
  id: api
  run: |
    curl -s https://api.example.com/status > response.json
    echo "::set-output name=response::$(cat response.json)"

- name: Use API response
  run: |
    STATUS="${{ fromJSON(steps.api.outputs.response).status }}"
    echo "API Status: $STATUS"
```

---

### Use Case 3: Pass Complex Data Between Jobs

```yaml
job1:
  outputs:
    metadata: ${{ steps.gather.outputs.meta }}

job2:
  needs: job1
  steps:
    - name: Use job1 metadata
      run: |
        echo "Version: ${{ fromJSON(needs.job1.outputs.metadata).version }}"
        echo "Build: ${{ fromJSON(needs.job1.outputs.metadata).build_id }}"
```

---

### Use Case 4: Conditional Logic Based on JSON

```yaml
- name: Parse and decide
  run: |
    CONFIG='{"deploy": true, "target": "production"}'
    SHOULD_DEPLOY=${{ fromJSON(CONFIG).deploy }}
    
    if [[ "$SHOULD_DEPLOY" == "true" ]]; then
      echo "Deploying..."
    fi
```

---

## Comparison: String vs JSON vs fromJSON()

| Type | Format | Example | Usable As |
|------|--------|---------|-----------|
| String | Quoted text | `'{"name": "Alice"}'` | Just text |
| JSON | Text format | `{"name": "Alice"}` | Still text, needs parsing |
| Parsed Object | After fromJSON() | Object with `.name` property | Usable object |

**Visual flow:**

```
JSON String in YAML
    ↓
'{"name": "Alice", "age": 30}'
    ↓
fromJSON()
    ↓
{name: "Alice", age: 30}
    ↓
Access: .name → "Alice"
Access: .age → 30
```

---

## Real-World Workflow Example

```yaml
name: Parse JSON Example

on: push

jobs:
  example:
    runs-on: ubuntu-latest
    
    steps:
      - name: Simple object
        run: |
          PERSON='{"name": "Alice", "age": 30}'
          echo "Name: ${{ fromJSON(PERSON).name }}"
          echo "Age: ${{ fromJSON(PERSON).age }}"
      
      - name: Array parsing
        run: |
          FRUITS='["apple", "banana", "cherry"]'
          echo "First: ${{ fromJSON(FRUITS)[0] }}"
          echo "Second: ${{ fromJSON(FRUITS)[1] }}"
          echo "All: ${{ join(fromJSON(FRUITS), ', ') }}"
      
      - name: Nested object
        run: |
          CONFIG='{"db": {"host": "localhost", "port": 5432}}'
          echo "Host: ${{ fromJSON(CONFIG).db.host }}"
          echo "Port: ${{ fromJSON(CONFIG).db.port }}"
      
      - name: Array of objects
        run: |
          USERS='[{"name": "Alice", "role": "admin"}, {"name": "Bob", "role": "user"}]'
          echo "First user: ${{ fromJSON(USERS)[0].name }}"
          echo "First role: ${{ fromJSON(USERS)[0].role }}"
          echo "Second user: ${{ fromJSON(USERS)[1].name }}"
      
      - name: Numbers and booleans
        run: |
          DATA='{"count": 42, "active": true, "ratio": 3.14}'
          echo "Count: ${{ fromJSON(DATA).count }}"
          echo "Active: ${{ fromJSON(DATA).active }}"
          echo "Ratio: ${{ fromJSON(DATA).ratio }}"
```

---

## Common Mistakes & Solutions

### ❌ Mistake 1: Forgetting Quotes Around JSON String

```yaml
# WRONG - JSON not quoted
value: ${{ fromJSON({"name": "Alice"}) }}

# CORRECT - JSON is quoted as a string
value: ${{ fromJSON('{"name": "Alice"}') }}
```

**Why:** YAML needs to see it as a string, not a raw object.

---

### ❌ Mistake 2: Invalid JSON Syntax

```yaml
# WRONG - Single quotes inside (invalid JSON)
DATA='{"name": 'Alice'}'

# CORRECT - Double quotes (valid JSON)
DATA='{"name": "Alice"}'
```

**Valid JSON rules:**
- Properties must be quoted: `"name"`, not `name`
- Values must be properly quoted or typed
- Use double quotes for strings

---

### ❌ Mistake 3: Accessing Non-existent Property

```yaml
# WRONG - Accessing property that doesn't exist
${{ fromJSON('{"name": "Alice"}').age }}
# Result: null (no error, but returns null)

# CORRECT - Access existing properties
${{ fromJSON('{"name": "Alice", "age": 30}').age }}
# Result: 30
```

---

### ❌ Mistake 4: Escaping Issues

```yaml
# WRONG - Improper escaping
data: ${{ fromJSON(\"{"name": "Alice"}\") }}

# CORRECT - Proper single quotes
data: ${{ fromJSON('{"name": "Alice"}') }}

# Or with escaped quotes if needed
data: ${{ fromJSON('{\"name\": \"Alice\"}') }}
```

---

## Comparison with toJSON()

| Function | Input | Output | Use |
|----------|-------|--------|-----|
| `fromJSON()` | JSON string | Parsed object | Parse JSON to use it |
| `toJSON()` | Object/context | JSON string | Convert to JSON for storage |

**Round-trip example:**

```yaml
# Original object
github.event

# Convert to JSON string
toJSON(github.event)

# Parse back to object
fromJSON(toJSON(github.event))
```

---

## Data Types Supported

```yaml
# String
${{ fromJSON('"hello"') }}  # → "hello"

# Number
${{ fromJSON('42') }}       # → 42
${{ fromJSON('3.14') }}     # → 3.14

# Boolean
${{ fromJSON('true') }}     # → true
${{ fromJSON('false') }}    # → false

# Null
${{ fromJSON('null') }}     # → null

# Array
${{ fromJSON('[1,2,3]') }}  # → [1, 2, 3]

# Object
${{ fromJSON('{"key":"value"}') }}  # → {"key": "value"}

# Complex nested
${{ fromJSON('{"arr":[1,2],"obj":{"name":"test"}}') }}
# → {arr: [1, 2], obj: {name: "test"}}
```

---

## Summary Cheat Sheet

```javascript
// Strings
fromJSON('"hello"')                    // "hello"

// Numbers
fromJSON('42')                         // 42
fromJSON('3.14')                       // 3.14

// Booleans
fromJSON('true')                       // true
fromJSON('false')                      // false

// Null
fromJSON('null')                       // null

// Arrays
fromJSON('[1, 2, 3]')                  // [1, 2, 3]
fromJSON('[1, 2, 3]')[0]               // 1

// Objects
fromJSON('{"name": "Alice"}')          // {name: "Alice"}
fromJSON('{"name": "Alice"}').name     // "Alice"

// Nested
fromJSON('{"user": {"name": "Alice"}}').user.name  // "Alice"

// Array of objects
fromJSON('[{"id": 1}, {"id": 2}]')[0].id  // 1
```

---

## Key Takeaways

1. **Purpose:** Convert JSON strings to usable objects/values
2. **Input:** Must be a valid JSON string (quoted)
3. **Output:** Parsed data (object, array, string, number, boolean, or null)
4. **Access:** Use dot notation for objects, brackets for arrays
5. **Validation:** JSON must be valid - double quotes for strings
6. **Pairing:** Often used with `join()` to work with arrays
7. **Real-world:** Essential for parsing configs, API responses, and passing data between jobs

---

For complete workflow examples with all data types, see:  
`.github/workflows/10-fromJSON-function-deep-dive.yml`
