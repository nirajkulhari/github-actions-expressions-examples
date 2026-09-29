# GitHub Actions `join()` Function - Complete Explanation

## Function Signature

```
join( array, optionalSeparator )
```

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `array` | Array \| String | The input array or string to join |
| `optionalSeparator` | String | The separator to insert between elements. Default: `,` (comma) |

## Key Characteristics

✅ **Concatenates** all values in array into a single string  
✅ **Casts** all values to strings automatically  
✅ **Default separator** is comma (`,`) if not specified  
✅ **Works with** arrays, strings, numbers, booleans  
✅ **Returns** a single concatenated string  

---

## Examples Explained

### Example 1: Basic Array with Default Separator

```yaml
${{ join(fromJSON('["apple", "banana", "cherry"]')) }}
```

**Input:** Array of 3 fruit names  
**Separator:** `,` (comma - default)  
**Output:** `apple,banana,cherry`

**How it works:**
1. `fromJSON('["apple", "banana", "cherry"]')` creates array
2. Elements: `"apple"`, `"banana"`, `"cherry"`
3. Each element joined with comma
4. Result: All elements as single string separated by commas

---

### Example 2: Custom Separator - Hyphen

```yaml
${{ join(fromJSON('["red", "green", "blue"]'), '-') }}
```

**Input:** Array of 3 color names  
**Separator:** `-` (hyphen)  
**Output:** `red-green-blue`

**How it works:**
1. Create array with 3 elements
2. Use hyphen `-` as separator
3. Connect elements: `red` + `-` + `green` + `-` + `blue`
4. Result: `red-green-blue`

---

### Example 3: Space Separator

```yaml
${{ join(fromJSON('["Hello", "World", "from", "GitHub"]'), ' ') }}
```

**Input:** Array of 4 words  
**Separator:** ` ` (space)  
**Output:** `Hello World from GitHub`

**Visual breakdown:**
```
["Hello", "World", "from", "GitHub"]
    |         |         |        |
    └─ ─ ─ ─ ─┘         └─ ─ ─ ─┘
    
Hello World from GitHub
```

---

### Example 4: Empty Separator (Concatenation)

```yaml
${{ join(fromJSON('["Hello", "World"]'), '') }}
```

**Input:** Array of 2 words  
**Separator:** `` (empty string)  
**Output:** `HelloWorld`

**Why:** With empty separator, no character is inserted between elements

---

### Example 5: Numbers Are Cast to Strings

```yaml
${{ join(fromJSON('[1, 2, 3, 4, 5]'), '-') }}
```

**Input:** Array of numbers  
**Separator:** `-` (hyphen)  
**Output:** `1-2-3-4-5`

**Important:** 
- Numbers `1`, `2`, `3` are converted to strings `"1"`, `"2"`, `"3"`
- Then joined as if they were strings
- Result: A string representation

---

### Example 6: Mixed Types (Type Casting)

```yaml
${{ join(fromJSON('["v", 1, ".", 2, ".", 3]'), '') }}
```

**Input:** Mixed array (strings and numbers)  
**Separator:** `` (empty)  
**Output:** `v1.2.3`

**How casting works:**
```
Input array:  ["v", 1, ".", 2, ".", 3]
              ↓    ↓   ↓    ↓   ↓    ↓
Cast to str: ["v", "1", ".", "2", ".", "3"]
              └──────────────────────────┘
                  Concatenate (no sep)
                        ↓
              Result: "v1.2.3"
```

**Real-world use:** Building version strings like `v1.2.3`

---

### Example 7: Booleans Cast to Strings

```yaml
${{ join(fromJSON('[true, false, true]'), ', ') }}
```

**Input:** Array of booleans  
**Separator:** `, ` (comma-space)  
**Output:** `true, false, true`

**Type conversion:**
```
Input:  [true, false, true]
        ↓     ↓      ↓
String: ["true", "false", "true"]
        └─────────────────────────┘
         Joined with ", "
              ↓
Result: "true, false, true"
```

---

### Example 8: String Input (Treated as Character Array)

```yaml
${{ join('hello', '-') }}
```

**Input:** String `"hello"`  
**Separator:** `-` (hyphen)  
**Output:** `h-e-l-l-o`

**How strings are treated:**
```
String: "hello"
          ↓
Characters: ["h", "e", "l", "l", "o"]
            └────────────────────────┘
            Joined with "-"
                ↓
Result: "h-e-l-l-o"
```

**Note:** When you pass a string to `join()`, it splits into individual characters!

---

### Example 9: Empty Array

```yaml
${{ join(fromJSON('[]'), '-') }}
```

**Input:** Empty array  
**Separator:** `-` (hyphen)  
**Output:** `` (empty string)

**Reason:** No elements to join, so result is empty

---

## Real-World Use Cases

### Use Case 1: Generate CSV Output

```yaml
headers: ${{ join(fromJSON('["Name", "Email", "Status"]'), ', ') }}
row: ${{ join(fromJSON('["Alice", "alice@example.com", "active"]'), ', ') }}
```

**Output:**
```
Name, Email, Status
Alice, alice@example.com, active
```

---

### Use Case 2: Build URL Path

```yaml
path: ${{ join(fromJSON('["api", "v1", "users", "123"]'), '/') }}
```

**Output:** `api/v1/users/123`

**Full URL:** `https://example.com/api/v1/users/123`

---

### Use Case 3: Format Docker/Container Tags

```yaml
tags: ${{ join(fromJSON('["latest", "1.0.0", "prod"]'), ' ') }}
```

**Output:** `latest 1.0.0 prod`

**Docker command:**
```bash
docker tag myimage:latest myimage:1.0.0 myimage:prod
```

---

### Use Case 4: Build CLI Arguments

```yaml
args: ${{ join(fromJSON('["--config", "app.yml", "--verbose", "--dry-run"]'), ' ') }}
command: npm run deploy ${{ args }}
```

**Output:** `npm run deploy --config app.yml --verbose --dry-run`

---

### Use Case 5: Breadcrumb Navigation

```yaml
breadcrumb: ${{ join(fromJSON('["Home", "Products", "Electronics", "Phones"]'), ' > ') }}
```

**Output:** `Home > Products > Electronics > Phones`

---

## Common Mistakes & Solutions

### ❌ Mistake 1: Forgetting `fromJSON()`

```yaml
# WRONG
${{ join('[\"a\", \"b\", \"c\"]', '-') }}
```

**Why it's wrong:** Without `fromJSON()`, the string is treated as individual characters:
```
Input string: "[\"a\", \"b\", \"c\"]"
              ↓
Characters: ["[", "\"", "a", "\"", ",", " ", "\"", "b", "\"", ...]
            └────────────────────────────────────────────────────┘
Result: [-,",-,a,-,",-,,-," ...  (Not what we want!)
```

**✅ Solution:**
```yaml
${{ join(fromJSON('[\"a\", \"b\", \"c\"]'), '-') }}
```

---

### ❌ Mistake 2: Incorrect Separator

```yaml
# WRONG - Using numbers or booleans as separators
${{ join(array, 5) }}
${{ join(array, true) }}

# They get cast to strings: "5" and "true"
# Probably not what you intended!
```

**✅ Solution:**
```yaml
# Use string separators
${{ join(array, '-') }}
${{ join(array, ', ') }}
${{ join(array, ' | ') }}
```

---

### ❌ Mistake 3: Forgetting Order of Elements

```yaml
# Remember: join() concatenates IN ORDER
${{ join(fromJSON('[\"z\", \"a\", \"m\"]'), '-') }}
# Output: z-a-m  (NOT sorted!)
```

---

## Comparison with Other Functions

| Function | Input | Output | Use Case |
|----------|-------|--------|----------|
| `join()` | Array/String | Single String | Combine elements with separator |
| `split()` | String | Array | Break string into parts (not in Actions) |
| `format()` | Template | String | Format with placeholders |
| `contains()` | String | Boolean | Check if contains substring |

---

## Complete Workflow Example

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Join with different separators
        run: |
          # Comma separated
          echo "CSV: ${{ join(fromJSON('[\"Name\", \"Age\", \"City\"]'), ',') }}"
          
          # Space separated
          echo "Words: ${{ join(fromJSON('[\"The\", \"quick\", \"brown\", \"fox\"]'), ' ') }}"
          
          # Pipe separated
          echo "Options: ${{ join(fromJSON('[\"create\", \"read\", \"update\", \"delete\"]'), ' | ') }}"
          
          # Forward slash (paths)
          echo "Path: /${{ join(fromJSON('[\"api\", \"v1\", \"users\"]'), '/') }}"
          
          # No separator
          echo "Version: v${{ join(fromJSON('[\"1\", \"2\", \"3\"]'), '.') }}"
```

**Output:**
```
CSV: Name,Age,City
Words: The quick brown fox
Options: create | read | update | delete
Path: /api/v1/users
Version: v1.2.3
```

---

## Summary Cheat Sheet

```javascript
join(array)              // Default separator: comma
// Examples:
${{ join(['a','b'], ',') }}      // a,b
${{ join(['a','b'], '-') }}      // a-b
${{ join(['a','b'], ' ') }}      // a b
${{ join(['a','b'], '') }}       // ab
${{ join([1,2,3], ',') }}        // 1,2,3 (numbers cast to strings)
${{ join('hello', '-') }}        // h-e-l-l-o (string treated as chars)
${{ join([], ',') }}             // (empty)
```

---

## Key Takeaways

1. **Purpose:** Combine array elements into a single string
2. **Separator:** Default is comma; customize as needed
3. **Type Casting:** Automatically converts numbers, booleans to strings
4. **String Handling:** Strings are treated as character arrays
5. **Use `fromJSON()`:** Always convert JSON strings to arrays first
6. **Real-world:** Perfect for CSV, URLs, CLI args, tags, paths

---

For more examples, check out the workflow file: `.github/workflows/9-join-function-deep-dive.yml`
