# Claude Code Assistant Guide - Expert Performance (C Edition)

**Guide for AI assistants to achieve expert-level performance through strategic thinking and systematic execution.**

**Core Philosophy**: Understand deeply → Plan strategically → Execute iteratively → Verify thoroughly

---

## 📖 Quick Start

**Every task follows this flow:**
1. **Phase 0** (5-Question Framework) → Understand deeply
2. **Avoid errors** (Read before edit, test immediately)
3. **Execute** (Implement → Test → Iterate)
4. **Verify** (Tests + Linter + Clean workspace)

**Key principles**: Think before acting • Read before editing • Test after every change • Iterate systematically • Clean workspace

---

## 🎯 PHASE 0: Strategic Analysis (START HERE)

### **The 5-Question Framework**

**Before writing ANY code, answer these:**

1. **WHAT** am I trying to achieve?
   - Success criteria? Edge cases? Constraints?

2. **WHERE** does this live in the codebase?
   - Which files? What dependencies? Data flow?

3. **HOW** does the current system work?
   - Read 3-5 files in parallel • Find existing patterns • Build mental model

4. **WHY** might this fail?
   - What could break? What assumptions am I making?

5. **WHICH** approach is best?
   - Simplest solution? Fits existing patterns?

### **Smart Code Search Strategy**

**❌ WRONG way to search:**
```
codebase_search "cache"  // Too broad, returns 100+ irrelevant files
→ Wastes time reading wrong files
```

**✅ RIGHT way to search:**
```
Step 1: Start specific with context
codebase_search "How is data caching implemented in the data loader?"

Step 2: If too many results, grep for exact struct/function names
grep "struct.*loader\|data_loader" -r src/  // Find exact loader structs/functions

Step 3: Read ONLY the files that match both searches
read_file("src/data/loader.h")  // Confirmed relevant

Step 4: Validate you have the right files
- Check includes, struct names, function signatures
- If wrong, refine search, don't just read more files
```

**Search quality checklist:**
- ✅ Search query is a full question, not just keywords
- ✅ Include context: "in the authentication system" not just "auth"
- ✅ Use grep to confirm exact struct/function names before reading
- ✅ Read 3-5 files max initially - if you need more, your search was too broad
- ✅ Validate each file is relevant before moving on

### **Intelligence Gathering (Do in Parallel)**

```c
// Example: "Add caching to data loader"

// Step 1: Focused search (not broad)
codebase_search "How does the data_loader load data from database?"
grep "data_loader\|DataLoader" -r src/ include/  // Get exact file location

// Step 2: Parallel reads (ONLY confirmed relevant files)
- read_file("include/data_loader.h")    // Main interface (header)
- read_file("src/data_loader.c")         // Main implementation
- read_file("include/cache_manager.h")    // IF grep confirms cache exists
- read_file("tests/test_data_loader.c")      // Tests for expected behavior

// Step 3: Validate relevance
// Check: Does loader.h have the functions I need to modify?
// Check: Are includes correct? Header guards present?
// If NO → You searched wrong, refine and try again
```

**Mental model checklist:**
- ✅ Entry points (how is this called?)
- ✅ Data flow (what data moves where?)
- ✅ Dependencies (what does this use?)
- ✅ Side effects (what else changes?)
- ✅ Error paths (what can go wrong?)
- ✅ **Interfaces** (what are the function signatures and contracts?)
- ✅ **Memory ownership** (who allocates? who frees?)

---

## 🔌 INTERFACE-FIRST DESIGN (Critical for Integration)

**Problem**: Coding everything then testing leads to interface mismatches and integration bugs.

**Solution**: Define interfaces FIRST, implement SECOND, test EACH level.

### **The Interface-First Process**

**Step 1: Define the interface (function signature + contract)**

```c
// DON'T start coding the implementation yet!
// FIRST, define what the function MUST do:

// In header file (data_loader.h):
#ifndef DATA_LOADER_H
#define DATA_LOADER_H

#include <stddef.h>
#include <stdint.h>

// Forward declarations
struct data_item;
struct filter_map;

/**
 * Load user data with filters.
 * 
 * @param userId User ID (non-null, non-empty string)
 * @param filters Filter conditions (can be NULL for no filters)
 * @param count Output parameter: number of items returned
 * @return Array of data items, NULL on error. Caller must free with data_loader_free_results()
 * @return count will be set to number of items (0 if no data)
 * 
 * Error codes:
 * - Returns NULL and sets errno to EINVAL if userId is NULL or empty
 * - Returns NULL and sets errno to ENOMEM if memory allocation fails
 * - Returns NULL and sets errno to EIO if database connection fails
 */
struct data_item* data_loader_load(const char* userId, 
                                   const struct filter_map* filters,
                                   size_t* count);

/**
 * Free results returned by data_loader_load().
 * 
 * @param items Array of items to free (can be NULL)
 * @param count Number of items in array
 */
void data_loader_free_results(struct data_item* items, size_t count);

#endif  // DATA_LOADER_H

// Define interface first, implement second
```

**Step 2: Write the test BEFORE implementation**

```c
// In test file (test_data_loader.c):
#include "data_loader.h"
#include <unity.h>

void test_data_loader_load_basic(void) {
    size_t count = 0;
    struct filter_map filters = {0};
    filters.status = "active";
    
    struct data_item* result = data_loader_load("user123", &filters, &count);
    
    TEST_ASSERT_NOT_NULL(result);  // Returns array, not NULL
    TEST_ASSERT_GREATER_THAN(0, count);  // Has items
    
    // Test edge cases
    result = data_loader_load("user123", NULL, &count);  // NULL filters
    TEST_ASSERT_NOT_NULL(result);  // Still returns array (may be empty)
    
    // Test error cases
    result = data_loader_load("", NULL, &count);  // Empty userId
    TEST_ASSERT_NULL(result);  // Returns NULL on error
    TEST_ASSERT_EQUAL(EINVAL, errno);  // Sets errno
    
    if (result) {
        data_loader_free_results(result, count);
    }
}
```

**Step 3: Implement to satisfy the interface**

```c
// In implementation file (data_loader.c):
#include "data_loader.h"
#include <string.h>
#include <errno.h>
#include <stdlib.h>

struct data_item* data_loader_load(const char* userId, 
                                   const struct filter_map* filters,
                                   size_t* count) {
    // Validate inputs
    if (userId == NULL || strlen(userId) == 0) {
        errno = EINVAL;
        return NULL;
    }
    
    if (count == NULL) {
        errno = EINVAL;
        return NULL;
    }
    
    // Implementation...
    *count = result_count;
    return results;  // MUST return array or NULL, set errno on error
}

void data_loader_free_results(struct data_item* items, size_t count) {
    if (items == NULL) {
        return;
    }
    
    for (size_t i = 0; i < count; i++) {
        // Free item fields if needed
        free(items[i].field);
    }
    free(items);
}
```

**Step 4: Test the single function (bottom-up)**

```bash
./build/tests/test_data_loader -v
# MUST pass before moving to integration
```

### **Interface Contract Checklist**

Before writing implementation, define:

- ✅ **Input types**: Exact parameter types (const char*, int, struct*, size_t*)
- ✅ **Output type**: What does it return? (struct*? int? void? NULL on error?)
- ✅ **Error handling**: How are errors reported? (return NULL? return error code? set errno?)
- ✅ **Memory ownership**: Who allocates? Who frees? Document clearly
- ✅ **Side effects**: Does it modify inputs? Change global state? Call external APIs?
- ✅ **Preconditions**: What MUST be true before calling? (non-null? non-empty? valid format?)
- ✅ **Postconditions**: What WILL be true after calling? (returns array, never invalid state?)
- ✅ **Thread safety**: Is function thread-safe? Document if not

### **Common Interface Mistakes**

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Function sometimes returns NULL, sometimes valid pointer | Always return same pattern (NULL on error, valid pointer on success) |
| Function modifies input without documentation | Document side effects, or make inputs const |
| Error cases return NULL without setting errno | Set errno to appropriate value (EINVAL, ENOMEM, etc.) |
| No documentation | Add complete Doxygen comments |
| Unclear memory ownership | Document who allocates and who must free |
| Missing error handling | Always check return values, handle errors |
| No input validation | Validate all inputs, return error if invalid |
| Memory leaks | Always provide cleanup functions, document ownership |

---

## 🏗️ BOTTOM-UP DEVELOPMENT (Build Solid Foundation)

**Problem**: Coding everything at once leads to many bugs.

**Solution**: Build and verify each level before moving up.

### **The Bottom-Up Process**

```
Level 1: Individual functions (test each)
    ↓ (all pass)
Level 2: Module integration (test together)
    ↓ (all pass)
Level 3: Component integration (test full flow)
    ↓ (all pass)
Level 4: System integration (test end-to-end)
```

### **Example: Add caching to data loader**

**❌ WRONG approach:**
```c
// Code everything at once
// cached_loader.h
struct cached_data_loader {
    struct cache* cache;
    // ... all fields at once
};

int cached_loader_load(struct cached_data_loader* loader, 
                       const char* userId, 
                       struct data_item** items, 
                       size_t* count);
void cached_loader_invalidate_cache(struct cached_data_loader* loader);
char* generate_cache_key(const char* userId, const struct filter_map* filters);
int fetch_from_cache(struct cached_data_loader* loader, const char* key, 
                     struct data_item** items, size_t* count);
int store_to_cache(struct cached_data_loader* loader, const char* key,
                   const struct data_item* items, size_t count);
// ... all functions at once

// Test everything together
./build/tests/test_cached_loader
// 10 failures! Which function is broken?
```

**✅ RIGHT approach:**

```c
// Level 1: Build and test ONE function at a time

// Step 1a: Implement cache key generation
char* generate_cache_key(const char* userId, const struct filter_map* filters) {
    if (userId == NULL) return NULL;
    size_t size = strlen("data:") + strlen(userId) + 1;
    if (filters != NULL) size += 32;
    char* key = malloc(size);
    if (key == NULL) return NULL;
    snprintf(key, size, "data:%s", userId);
    // Append filter info if needed
    return key;  // Caller must free
}

// Step 1b: Test it IMMEDIATELY
void test_generate_cache_key(void) {
    char* key1 = generate_cache_key("u1", NULL);
    char* key2 = generate_cache_key("u1", NULL);
    TEST_ASSERT_EQUAL_STRING(key1, key2);  // Same inputs = same key
    free(key1); free(key2);
    // Run: ./build/tests/test_cached_loader -v
    // ✅ PASS before continuing
}

// Step 2a: Implement cache fetch
int fetch_from_cache(struct cached_data_loader* loader, const char* key,
                     struct data_item** items, size_t* count) {
    if (loader == NULL || key == NULL || items == NULL || count == NULL) {
        errno = EINVAL; return -1;
    }
    struct cache_entry* entry = cache_lookup(loader->cache, key);
    if (entry == NULL) { *items = NULL; *count = 0; return 0; }
    *count = entry->count;
    *items = malloc(sizeof(struct data_item) * entry->count);
    if (*items == NULL) { errno = ENOMEM; return -1; }
    memcpy(*items, entry->items, sizeof(struct data_item) * entry->count);
    return 0;
}

// Step 2b: Test it IMMEDIATELY  
void test_fetch_from_cache(void) {
    struct cached_data_loader loader = {0};
    loader.cache = cache_create();
    struct data_item* items = NULL;
    size_t count = 0;
    int result = fetch_from_cache(&loader, "missing", &items, &count);
    TEST_ASSERT_EQUAL(0, result);
    TEST_ASSERT_NULL(items);
    // Run: ./build/tests/test_cached_loader -v
    // ✅ PASS before continuing
}

// Level 2: Now integrate them
int cached_loader_load(struct cached_data_loader* loader,
                       const char* userId,
                       struct data_item** items,
                       size_t* count) {
    char* key = generate_cache_key(userId, NULL);  // ✅ Already tested
    if (key == NULL) {
        return -1;
    }
    
    // Try cache first
    int result = fetch_from_cache(loader, key, items, count);  // ✅ Already tested
    if (result == 0 && *items != NULL) {
        free(key);
        return 0;  // Cache hit
    }
    
    // Cache miss, fetch from DB
    result = fetch_from_db(userId, NULL, items, count);  // ✅ Already tested
    if (result != 0) {
        free(key);
        return -1;
    }
    
    // Store in cache
    result = store_to_cache(loader, key, *items, *count);  // ✅ Already tested
    free(key);
    
    return result;
}

// Test integration
void test_cached_loader_load_integration(void) {
    // All components work individually, now test together
    // Run: ./build/tests/test_cached_loader -v
}
```

### **Bottom-Up Rules**

1. **Write ONE function at a time**
2. **Test that function IMMEDIATELY** (must pass 100%)
3. **Only after it passes, move to next function**
4. **Never write multiple functions before testing**
5. **Integration comes AFTER all pieces work individually**

---

## ✋ VERIFICATION PARANOIA (Never Trust, Always Verify)

**Problem**: AI concludes problem is fixed when it's still buggy.

**Solution**: Verify with ACTUAL tests, not assumptions.

### **The Verification Protocol**

**❌ WRONG - Assumption-based:**
```
"I've added the caching logic. The problem should be fixed now."
→ NO! You haven't verified anything!
```

**✅ RIGHT - Evidence-based:**
```
"I've added the caching logic. Let me verify:

1. Run the specific test:
   ./build/tests/test_data_loader -v
   Result: ✅ PASS

2. Run integration test:
   ./build/tests/test_integration -v
   Result: ✅ PASS

3. Run manual test with real data:
   ./build/bin/loader_demo
   Result: [{"id": 1, "name": "test"}] ✅ Correct format

4. Check cache is actually used:
   ./build/bin/loader_demo --benchmark
   Result: First: 150ms, Cached: 2ms ✅ Cache is working

5. Check for memory leaks:
   valgrind --leak-check=full ./build/tests/test_data_loader
   Result: ✅ No leaks

VERIFIED: Problem is actually fixed."
```

### **Verification Checklist (ALL must pass)**

After claiming "it's fixed", you MUST:

- [ ] **Unit test passes** - Run specific test function
- [ ] **Integration test passes** - Run related integration tests  
- [ ] **Manual test works** - Run actual code with real input
- [ ] **Edge cases work** - Test with NULL, empty, invalid inputs
- [ ] **No regressions** - Run full test suite: `./build/tests/all_tests`
- [ ] **Linter clean** - Run `splint` or project linter
- [ ] **Actually test the bug** - If fixing a bug, reproduce the bug first, then verify it's gone
- [ ] **Memory checks** - Run with valgrind: `valgrind --leak-check=full ./build/tests/test_loader`
- [ ] **Address sanitizer** - Compile with `-fsanitize=address` and run tests
- [ ] **Undefined behavior** - Compile with `-fsanitize=undefined` and run tests

### **Never Say "Fixed" Until:**

```bash
# ALL of these must be true:
✅ ./build/tests/test_loader -v                                    # PASS
✅ ./build/tests/all_tests -v                                       # All PASS
✅ ./build/bin/demo_app                                             # Works
✅ splint src/loader.c                                              # No errors
✅ valgrind --leak-check=full ./build/tests/test_loader            # No leaks
✅ git status                                                       # Clean

# Then and ONLY then:
"VERIFIED: Problem is fixed. All tests pass, linter clean, manual test works, no memory leaks."
```

---

## 📐 SYNTAX & TYPE CHECKING (Catch Errors Early)

**Problem**: Writing wrong C syntax leads to compilation errors.

**Solution**: Self-check syntax and types BEFORE running.

### **Pre-Run Syntax Check**

**After writing code, BEFORE testing, check:**

```bash
# 1. Check syntax is valid (compile)
gcc -std=c11 -Wall -Wextra -Wpedantic -c src/loader.c -o /tmp/test.o
# Must complete without errors

# 2. Check includes work
gcc -std=c11 -I./include -c src/loader.c -o /tmp/test.o
# Must not raise include errors

# 3. Check header guards (if modifying headers)
gcc -E include/data_loader.h | grep -q "#define.*_H" || echo "Missing header guard!"

# 4. Check types and static analysis (if using splint)
splint +posixlib src/loader.c -I./include
# Fix any type errors or warnings

# 5. Check linter/formatter
indent -npro -kr -i4 -ts4 -sob -l80 -ss -ncs src/loader.c
# Or use clang-format for C
clang-format --dry-run src/loader.c

# 6. Check for common issues
cppcheck --enable=all --suppress=missingIncludeSystem src/loader.c
```

### **Common C Syntax Mistakes**

| ❌ Wrong | ✅ Right | Error |
|---------|---------|-------|
| `void func();` (missing body) | `void func() { }` or implement in .c | Linker error |
| `return x, y` (comma operator) | Return struct or use output parameters | Wrong return type |
| `array[10]; array[10]` (out of bounds) | Check bounds: `if (i < 10) array[i]` | Undefined behavior |
| `free(ptr)` (double free) | Set `ptr = NULL` after free, check before free | Double free |
| `int* a, b` (b is int) | `int *a, *b` or `int* a; int* b;` | Type mismatch |
| `if (ptr)` (null check) | `if (ptr != NULL)` | Use NULL explicitly |
| Missing `const` | Use `const` for parameters you don't modify | Compiler warnings |
| `strcpy(dest, src)` (no bounds check) | `strncpy(dest, src, size)` or `snprintf` | Buffer overflow |
| Missing header guards | `#ifndef HEADER_H` / `#define HEADER_H` / `#endif` | Multiple definition errors |
| `#include <iostream>` (C++ header) | Use C headers: `<stdio.h>`, `<stdlib.h>` | Wrong language |

### **Type System Self-Check**

**Every function MUST have:**
```c
// In header file (processor.h):
#ifndef PROCESSOR_H
#define PROCESSOR_H

#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>

/**
 * Process data with filters.
 * 
 * @param userId User ID (non-null, non-empty string)
 * @param filters Filter conditions (can be NULL for no filters)
 * @param limit Optional limit (0 means no limit)
 * @param count Output parameter: number of items returned
 * @return Array of processed items, NULL on error. Caller must free.
 * @return count will be set to number of items (0 if no data)
 * 
 * Error codes:
 * - Returns NULL and sets errno to EINVAL if userId is NULL or empty
 * - Returns NULL and sets errno to ENOMEM if memory allocation fails
 */
struct processed_item* process_data(const char* userId,           // ✅ Input type
                                    const struct filter_map* filters,  // ✅ Input type
                                    size_t limit,                 // ✅ Input type
                                    size_t* count);              // ✅ Output parameter

#endif  // PROCESSOR_H
```

**Check:**
- ✅ All parameters have clear types
- ✅ Return type is specified
- ✅ Use output parameters (`size_t* count`) for multiple return values
- ✅ Use specific types (size_t, uint32_t, not just int)
- ✅ Use const for parameters that aren't modified
- ✅ Document all parameters and return value (Doxygen style)
- ✅ Document error handling (errno values, return codes)
- ✅ Document memory ownership (who allocates, who frees)

---

## ⚠️ CRITICAL: Error Prevention

### **Top 2 Errors (90% of Failures)**

| Error | Cause | Fix |
|-------|-------|-----|
| "Error editing file" | Didn't read file first OR old_string mismatch | ALWAYS `read_file()` → Copy EXACT text → Edit |
| "No shell found with ID" | Assumed shell persistence | Use absolute paths OR `cd /path && command` |

### **Environment & Development Rules**

**Build System**:
- ✅ **ALWAYS** use build system (Make, CMake, Autotools, etc.)
- ✅ Build before running: `make` or `cmake --build build`
- ✅ Run tests with build system: `make test` or `cd build && ctest`
- ✅ Use out-of-source builds: `mkdir -p build && cd build && cmake ..`
- ✅ Check build directory exists before executing: `test -d build || (mkdir -p build && cd build && cmake ..)`
- ✅ Use compiler flags: `-std=c11 -Wall -Wextra -Wpedantic -g`
- ✅ Enable sanitizers for debugging: `-fsanitize=address,undefined`

**Example Files**:
- ✅ Create **ONE file per request** in `examples/` folder
- ❌ Do NOT create multiple example files for a single request
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.c` (not `example1.c`, `example2.c`)
- ✅ If user asks for multiple examples, combine them into one file with clear sections

### **Pre-Action Checklist**

| Before... | Check... |
|-----------|----------|
| Editing file | ✅ Read file first? ✅ Exact old_string? |
| Shell command | ✅ Absolute paths? ✅ `cd` in same command? ✅ Using build system? |
| Implementing | ✅ Searched codebase? ✅ TODO if >3 steps? |
| Creating examples | ✅ One file per request? ✅ Descriptive filename? |

---

## 🧠 DECISION FRAMEWORK

### **Master Decision Tree**

```
Task received
    ↓
Understand what's needed? NO → Ask for clarification
    YES ↓
Simple (<3 steps, 1 file)? YES → Do it directly
    NO ↓ Create TODO
Understand codebase? NO → Phase 0: Read files + codebase_search
    YES ↓
Know how to implement? NO → Check examples → Web search if needed
    YES ↓
Form hypothesis → Assess risk → IMPLEMENT
    ↓
TEST immediately
    Works? YES → Verify & Done
    NO → Analyze → Different approach → Retry
        After 3 fails: Check examples
        After 5 fails: Web search or ask
```

### **Quick Decisions**

| Situation | Action | Tool/Method |
|-----------|--------|-------------|
| Unknown codebase area | Explore first | `codebase_search` + read files (parallel) |
| Know exact text | Use grep | `grep "struct Name" -r src/` |
| Unfamiliar library/error | Web search | "C library error context 2025" |
| Simple change | Do it | Skip TODO |
| 3+ files or steps | Plan | Create TODO list |
| Tried 3x, failed | Research | Examples → web search |
| Debugging | Systematic | Hypothesis → Test → Analyze |

### **When to Web Search**

✅ **USE web search:**
- Unfamiliar library/API: "C libcurl async handling 2025"
- Unknown error (after checking codebase): "C undefined reference linker error"
- Best practices: "C memory management patterns 2025"
- After 3 failed attempts

❌ **DON'T web search:**
- Info in codebase → Use `codebase_search` or `grep`
- Standard C → You know this
- Can infer from existing code → Follow patterns

**Query formula**: `[Library] [Specific Issue] [Context] [Year]`

---

## 🔧 ITERATIVE DEVELOPMENT (Core Loop)

**Never assume code works on first try. Always iterate.**

### **The Loop**

```
1. UNDERSTAND → What needs to work? Success criteria?
    ↓
2. HYPOTHESIZE → "Approach X will work because Y"
    ↓
3. IMPLEMENT → Small, focused change (read file first!)
    ↓
4. TEST → Immediately (don't wait!)
    ↓
5. EVALUATE → Works? YES → VERIFY & Done
              NO ↓
6. ANALYZE → Why failed? Wrong hypothesis?
    ↓
7. ITERATE → Try DIFFERENT approach (not same thing)
    → Loop to step 2
```

### **Example: Fix memory leak bug**

```
1. UNDERSTAND: Application leaks memory after processing data
2. HYPOTHESIZE: DataProcessor doesn't free allocated resources
3. IMPLEMENT: Add free() calls in cleanup function
4. TEST: Still leaks memory (valgrind shows leaks)
5. ANALYZE: Early return before cleanup, function doesn't free on error path
6. NEW HYPOTHESIS: Need to free in all code paths (use goto cleanup pattern)
7. IMPLEMENT: Use goto cleanup pattern for consistent cleanup
8. TEST: ✅ Works! No memory leaks (valgrind clean)
9. VERIFY: Unit tests pass, valgrind clean, integration works
```

### **When Stuck**

| After | Do This |
|-------|---------|
| 1-2 fails | Re-read error, check assumptions, add logging |
| 3 fails | Look for similar code in codebase, check examples/ |
| 5 fails | Web search specific error OR ask for help |

**Red flags**: Trying same fix repeatedly • Not reading errors • Testing at end only

---

## 🔍 DEBUGGING: Hypothesis-Driven

**Systematic debugging beats trial-and-error.**

### **The Protocol**

```
1. OBSERVE
   - Exact error? Expected vs actual? When does it happen?

2. HYPOTHESIZE (Rank 3 possibilities)
   A. Most likely cause
   B. Second possibility
   C. Edge case

3. TEST (Design minimal test for hypothesis A)
   - Add strategic logging
   - Run minimal reproduction

4. ANALYZE
   - Confirmed? → Fix it
   - Rejected? → Test hypothesis B
   - New info? → Refine hypothesis

5. FIX root cause (not symptom)
   - Ask "Why?" 5 times to find root cause
   - Verify fix works
```

### **Example**

```c
ERROR: Segmentation fault (core dumped)

HYPOTHESIZE:
A (70%): Null pointer dereference
B (20%): Out of bounds access
C (10%): Use after free

TEST A: Add null check before pointer use
printf("Pointer: %s\n", ptr ? "valid" : "null");
RESULT: Pointer: null  // ptr is null!
→ Hypothesis A CONFIRMED

ROOT CAUSE: Function doesn't check if ptr is null before use
FIX: if (ptr == NULL) { errno = EINVAL; return -1; }
VERIFY: ✅ Works
```

### **Common Patterns**

| Error | Likely Cause | Quick Test |
|-------|--------------|------------|
| `Segmentation fault` | Null pointer or out of bounds | `gdb ./build/bin/app`, `valgrind`, or `-fsanitize=address` |
| `Undefined reference` | Missing link or implementation | Check linker flags, check .c file exists, check Makefile |
| `Use after free` | Dangling pointer | Use valgrind, check lifetime, set pointer to NULL after free |
| `Memory leak` | Missing free or early return | Use `valgrind --leak-check=full`, use goto cleanup pattern |
| `Double free` | Freeing same pointer twice | Set pointer to NULL after free, use valgrind |
| Wrong result | Logic error | Print intermediate values, use `gdb`, add assertions |
| `Undefined behavior` | Uninitialized variables, etc. | Use `-fsanitize=undefined`, `-Wall -Wextra` |
| `Buffer overflow` | No bounds checking | Use `strncpy` instead of `strcpy`, use `snprintf` |

### **C Debugging Tools**

**Essential debugging commands:**
```bash
# GDB (GNU Debugger)
gdb ./build/bin/app
(gdb) break main
(gdb) run
(gdb) print variable_name
(gdb) backtrace
(gdb) next
(gdb) continue

# Valgrind (memory errors)
valgrind --leak-check=full --show-leak-kinds=all ./build/tests/test_app

# Address Sanitizer (compile with -fsanitize=address)
gcc -fsanitize=address -g -O1 src/app.c -o app
./app  # Will show memory errors at runtime

# Undefined Behavior Sanitizer
gcc -fsanitize=undefined -g -O1 src/app.c -o app

# Static analysis
splint +posixlib src/app.c -I./include
cppcheck --enable=all src/
```

---

## 🤖 TOOL USAGE & COMMUNICATION

### **Tool Selection**

| Goal | Tool | Pattern |
|------|------|---------|
| Find concept | `codebase_search` | "How does X work?" |
| Find exact text | `grep` | `grep "struct Name"` |
| Read code | `read_file` | Read 3-5 files in parallel |
| Unknown API | `web_search` | "C library API 2025" |

**Key principle**: Batch independent operations in parallel (3x faster)

### **Communication Template**

For every significant change, explain:

```
I'm [doing X] because [reason Y].
Following pattern from [file/code].
Risk: [potential issue] → Mitigation: [strategy].
Will verify by [test method].
```

**Show your reasoning**: WHAT you're doing, WHY, HOW it fits, WHAT could fail, HOW you'll verify.

---

## 🧠 COGNITIVE STRATEGIES

### **Pattern Recognition**

| Pattern | When You See | Action |
|---------|--------------|--------|
| Opaque pointer | `struct name;` in header, full definition in .c | Use opaque pointer pattern for encapsulation |
| Function pointer | `typedef int (*callback_t)(void*);` | Use function pointers for callbacks |
| Resource cleanup | `goto cleanup;` pattern | Use goto cleanup for consistent error handling |
| Memory pool | Pre-allocated array | Use memory pool for performance |

### **Mental Tricks**

**Rubber Duck**: Explain problem aloud → Often reveals solution  
**Backward Reasoning**: Start with goal, work backwards to identify all steps  
**Constraint Mapping**: List technical, business, architectural, data constraints  
**Assumption Validation**: Write assumptions, test each one explicitly

### **Risk Assessment**

| Risk | Indicators | Action |
|------|------------|--------|
| 🟢 Low | Single file, simple logic | Standard testing |
| 🟡 Medium | Multiple files | Extra careful, more tests |
| 🔴 High | Core system, many deps | Extensive planning + tests |
| ⚫ Critical | Memory safety, data loss | Maximum caution, get review |

---

## 📝 FILE MANAGEMENT

### **Modify vs Create**

| Action | Modify Existing | Create New |
|--------|----------------|------------|
| Fix bug | ✅ | ❌ |
| Add feature to module | ✅ | ❌ |
| Experiment | ✅ (git backup) | ❌ No temp files |
| New component | ❌ | ✅ |
| Unit tests | ❌ | ✅ |

### **Documentation Files**

⚠️ **NEVER create .md files unless explicitly asked**
- ❌ README.md, CHANGELOG.md, DESIGN.md (unless user asks)
- ✅ Only when user says: "Create a README" or "Write documentation"

### **Example Files**

⚠️ **Create ONE file per request in examples/**
- ✅ One comprehensive example file per user request
- ❌ Do NOT create multiple example files (`example1.c`, `example2.c`, etc.)
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.c`
- ✅ If multiple examples needed, combine into one file with clear sections/comments

### **Temporary Files (Rare, Delete Immediately)**

**If absolutely needed:**
- Use prefixes: `verify_*.c`, `debug_*.c`, `temp_*.c`
- Delete within minutes (same session)
- Add to .gitignore

**Lifecycle**: Create → Run → DELETE → Apply fix to real code

---

## 🚀 DEVELOPMENT WORKFLOW

### **6-Step Process**

1. **UNDERSTAND** → Phase 0: 5 questions + mental model
2. **EXPLORE** → Read files (parallel), codebase_search, grep
3. **PLAN** → TODO if >3 steps, assess risks
4. **IMPLEMENT** → Small changes, test immediately, iterate
5. **VERIFY** → Tests + linter + integration
6. **CLEANUP** → Delete temp files, git status clean

### **Core Principles**

- **KISS + YAGNI** → Simplest solution that works
- **Modify, don't create** → Edit existing files
- **Test immediately** → After every change
- **Iterate systematically** → Different approaches, not retries
- **Autonomous but transparent** → Make decisions, explain them

---

## ✅ QUALITY & DEFINITION OF DONE

### **Pre-Commit Checklist**

```bash
# 1. Compile
make  # or cmake --build build

# 2. Tests
./build/tests/all_tests -v
# OR: make test

# 3. Linter
splint +posixlib src/ -I./include
# OR: cppcheck --enable=all src/

# 4. Formatter
indent -npro -kr -i4 -ts4 -sob -l80 -ss -ncs src/*.c
# OR: clang-format --dry-run src/

# 5. Memory checks (recommended)
valgrind --leak-check=full --show-leak-kinds=all ./build/tests/all_tests

# 6. Git status clean
git status  # Only intended files?

# 7. No temp files
find . -name "verify_*" -o -name "debug_*" -o -name "temp_*"  # Should be empty
```

### **Done = ALL True**

- ✅ Code compiles + tests pass
- ✅ No linter errors
- ✅ ALL temp files deleted
- ✅ No .md files created (unless asked)
- ✅ `git status` clean - only intended changes
- ✅ Integration verified
- ✅ No memory leaks (valgrind clean)

---

## 🎴 QUICK REFERENCE CARD

### **The Essentials**

**5 Questions** (Always ask first):
1. WHAT → Success criteria?
2. WHERE → Which files?
3. HOW → Current system?
4. WHY → What could fail?
5. WHICH → Best approach?

**Search Strategy** (Avoid reading wrong files):
1. Specific question, not keywords: "How does X work in Y?"
2. Grep for exact names: `grep "struct Loader" -r src/`
3. Read 3-5 files max initially
4. Validate each file is relevant

**Interface-First Development**:
```
1. Define interface (signature + contract)
2. Write test for interface
3. Implement to satisfy interface
4. Test the single function ✅
5. Only then integrate with others
```

**Bottom-Up Building** (One function at a time):
```
Write function → Test it → MUST PASS → Next function
(Never write multiple functions before testing)
```

**Verification Protocol** (Never assume, always verify):
```
✅ Unit test pass
✅ Integration test pass  
✅ Manual test works
✅ Linter clean
✅ Full test suite pass
✅ No memory leaks (valgrind)
THEN say "fixed"
```

**Syntax Check** (Before running):
```bash
gcc -std=c11 -Wall -Wextra -Wpedantic -c src/module.c -o /tmp/test.o  # Check compilation
splint +posixlib src/module.c -I./include  # Check types/style
clang-format --dry-run src/module.c  # Check formatting
```

**Tool Decisions**:
- Find concept → `codebase_search "How does X work in Y?"`
- Find exact text → `grep "struct Name" -r src/`
- Unknown API/error → `web_search` (with year)

**Build System**:
- Compile → Always use build system: `make` or `cmake --build build`
- Run tests → Use build system: `make test` or `cd build && ctest`
- Install deps → Use package manager or build system
- Debug build → `make CFLAGS="-g -O0"`
- Release build → `make CFLAGS="-O2 -DNDEBUG"`
- Sanitizers → `make CFLAGS="-fsanitize=address,undefined"`

**When Stuck**:
- After 3 fails → Check examples
- After 5 fails → Web search or ask

**Before Done**:
- ✅ Each function tested individually?
- ✅ Integration tested?
- ✅ All tests pass?
- ✅ Linter clean?
- ✅ Git status clean?
- ✅ Code compiles without warnings?
- ✅ No memory leaks?

### **Communication Formula**

```
Doing: [X]
Why: [Y]
Pattern: [existing code]
Risk: [issue] → Mitigation: [plan]
Verify: [test method]
```

### **Red Flags → STOP**

- **Reading 10+ files** → Your search was too broad, refine it
- **Coding multiple functions without testing** → Stop, test each function individually
- **Saying "fixed" without running tests** → Stop, verify with actual tests
- **Tried same fix 3 times** → Try completely different approach
- **Integration fails after functions work** → Check interface contracts match
- **Compilation errors when running** → Should have checked with `gcc -c` first
- **Type errors** → Should have checked types match
- **Same test fails differently** → Fix root cause, not symptom
- **Don't understand error** → Use hypothesis-driven debugging
- **Creating temp files** → Edit existing code instead
- **Memory leaks** → Use valgrind, check all code paths free memory
- **Missing header guards** → Always use `#ifndef HEADER_H` / `#define HEADER_H` / `#endif`

---

## 🚨 COMMON FAILURE PATTERNS (Learn from These)

### **Failure Pattern 1: "Shotgun Search"**

**Symptom**: Reading 20 unrelated files, still can't find the right code

**Root cause**: Search query too broad, no validation of relevance

**Fix**:
```bash
# ❌ BAD
codebase_search "cache"  # Returns 100 files

# ✅ GOOD
codebase_search "How is user data cached in the authentication flow?"
grep "struct.*auth.*cache\|cache.*auth" -r src/  # Get exact file
read_file("src/auth/cache.h")     # Read ONLY confirmed file
# Validate: Does this file have the functions I need? YES → Use it, NO → Refine search
```

### **Failure Pattern 2: "Big Bang Integration"**

**Symptom**: Wrote 5 functions, tested together, everything breaks, don't know which function is wrong

**Root cause**: Didn't test each function individually

**Fix**:
```c
// ❌ BAD - All functions at once
int processor_load(...);
int processor_process(...);
int processor_save(...);
./build/tests/test_processor  // 10 failures, which broke?

// ✅ GOOD - One function at a time
int processor_load(...);
void test_processor_load(void) { /* test */ }
./build/tests/test_processor -v  // ✅ PASS - Continue

int processor_process(...);
void test_processor_process(void) { /* test */ }
./build/tests/test_processor -v  // ✅ PASS - Continue
```

### **Failure Pattern 3: "Assumption-Based Completion"**

**Symptom**: Said "fixed" but bug still exists

**Root cause**: Assumed code works without actually testing

**Fix**:
```bash
# ❌ BAD
"I've fixed the cache invalidation bug. It should work now."
# → No evidence provided!

# ✅ GOOD
"I've fixed the cache invalidation bug. Verification:

1. Reproduce bug: ./build/bin/demo_app
   Before fix: Cache not cleared ❌
   
2. Apply fix: cache_delete(key) after update

3. Test again: ./build/bin/demo_app
   After fix: Cache cleared ✅
   
4. Run unit test: ./build/tests/test_cache -v
   Result: PASSED ✅
   
5. Run integration: ./build/tests/test_integration -v
   Result: PASSED ✅
   
6. Check memory: valgrind --leak-check=full ./build/tests/test_cache
   Result: No leaks ✅

VERIFIED: Bug is fixed."
```

### **Failure Pattern 4: "Interface Mismatch"**

**Symptom**: Individual functions work, but fail when integrated

**Root cause**: Interfaces don't match - unclear return types, missing error handling

**Fix**:
```c
// ❌ BAD - Unclear interface
struct data_item* fetch_data(const char* userId);  // When is NULL? Unclear!
void process_data(struct data_item* data) {  // Crashes if NULL!

// ✅ GOOD - Clear interface with error handling
struct data_item* fetch_data(const char* userId, size_t* count) {
    if (userId == NULL || strlen(userId) == 0) { errno = EINVAL; return NULL; }
    if (count == NULL) { errno = EINVAL; return NULL; }
    // Implementation...
    *count = item_count;
    return items;  // NULL on error, valid pointer on success
}

int process_data(const struct data_item* data, size_t count) {
    if (data == NULL && count > 0) { errno = EINVAL; return -1; }
    for (size_t i = 0; i < count; i++) { /* process */ }
    return 0;
}
```

### **Failure Pattern 5: "Syntax Before Test"**

**Symptom**: Code has C compilation errors when running

**Root cause**: Didn't check compilation before testing

**Fix**:
```bash
# ✅ GOOD - Check compilation BEFORE running tests
gcc -std=c11 -Wall -Wextra -Wpedantic -c src/module.c -o /tmp/test.o  # Compile
splint +posixlib src/module.c -I./include  # Type check (if using)
cppcheck --enable=all src/module.c  # Linter
./build/tests/test_module  # NOW run tests
```

### **How to Avoid These Failures**

| Failure Pattern | Prevention |
|----------------|------------|
| Shotgun Search | Specific questions + grep + validate 3-5 files max |
| Big Bang Integration | Test each function individually before integration |
| Assumption Completion | Verify with actual tests (unit + integration + manual) |
| Interface Mismatch | Define interfaces first with clear types and contracts |
| Compilation Errors | Check with `gcc -c` and linter before running |
| Memory Leaks | Use valgrind, check all code paths, use goto cleanup pattern |

---

## 🏆 SUCCESS CRITERIA

**Expert-level performance means:**

✅ **Strategic**: Understand deeply before acting, build mental models, assess risks  
✅ **Systematic**: Follow workflows, test immediately, iterate with learning  
✅ **Quality**: Code works, tests pass, workspace clean, changes focused  
✅ **Improvement**: Learn from failures, adapt when stuck, recognize patterns

**Remember**: 
> *"Simple, working code beats complex, perfect code."*
> 
> **Think strategically. Act systematically. Iterate until it works. Always clean up.**

---

## 🔑 TOP 12 RULES

1. ⚠️ **Smart search** - Specific questions + grep exact names + read 3-5 files max
2. ⚠️ **Interface first** - Define signature/contract → Write test → Implement
3. ⚠️ **Bottom-up build** - One function → Test it → PASS → Next function
4. ⚠️ **Never assume fixed** - Verify with actual tests (unit + integration + manual)
5. ⚠️ **Compile check first** - `gcc -c` before running
6. ⚠️ **Test immediately** - After EVERY function, not after coding everything
7. ⚠️ **Read before edit** - Prevents 90% of errors
8. ⚠️ **Iterate systematically** - Different approaches, not same retry
9. ⚠️ **Root cause debugging** - Ask "Why?" 5 times, fix cause not symptom
10. ⚠️ **Clean workspace** - git status clean, all tests pass, linter clean
11. ⚠️ **Use build system** - Always use Make/CMake for compilation
12. ⚠️ **One example file** - Create ONE file per request in `examples/`, not multiple files
13. ⚠️ **Memory management** - Always free what you allocate, use valgrind, check all code paths
14. ⚠️ **Error handling** - Always check return values, set errno appropriately

---

## 📚 PROJECT SPECIFICS

### **C Project Structure**

```
project/
├── src/                    # Main code (.c files)
│   ├── core/               # Core functionality
│   └── utils/              # Utilities
├── include/                # Header files (.h files)
│   ├── project_name/       # Project namespace directory
│   │   ├── core/           # Core headers
│   │   └── utils/          # Utility headers
│   └── project_name.h      # Main header (optional)
├── tests/                  # Test suites
│   ├── unit/               # Unit tests
│   └── integration/        # Integration tests
├── examples/               # Usage examples
├── Makefile                # Make config (or CMakeLists.txt)
├── .clang-format           # Clang format config (optional)
└── build/                  # Build directory (out-of-source, gitignored)
```

**Include Path Convention:**
```c
// In src/core/loader.c:
#include "project_name/core/loader.h"  // Project headers use project_name/ prefix

// In include/project_name/core/loader.h:
#ifndef PROJECT_NAME_CORE_LOADER_H
#define PROJECT_NAME_CORE_LOADER_H

// 1. System headers first
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>
#include <errno.h>

// 2. Third-party headers
#include <curl/curl.h>

// 3. Project headers last
#include "project_name/utils/helper.h"

// Function declarations
struct data_item* loader_load(const char* userId, size_t* count);

#endif  // PROJECT_NAME_CORE_LOADER_H
```

### **Code Style**

- **Formatter**: `indent` (GNU style) or `clang-format`
- **Linter**: `splint`, `cppcheck`, or project-specific
- **Type check**: Compiler warnings + static analysis
- **Testing**: Unity, CUnit, Check, or custom framework
- **Build system**: Make (most common), CMake, Autotools

### **C Standards**

- **C11** or **C99** (prefer C11 for modern features)
- Use modern features: `_Generic`, `_Static_assert`, compound literals
- Avoid: Non-standard extensions, platform-specific code without abstraction

### **C Best Practices**

**Memory Management:**
- ✅ Always free what you allocate
- ✅ Use `goto cleanup` pattern for consistent error handling
- ✅ Set pointers to NULL after free
- ✅ Check return values of malloc/calloc/realloc
- ✅ Use valgrind to check for leaks
- ❌ Never use freed memory
- ❌ Never double-free

**Error Handling:**
- ✅ Always check return values
- ✅ Set `errno` appropriately (EINVAL, ENOMEM, EIO, etc.)
- ✅ Return error codes or NULL on error
- ✅ Document error conditions in function comments
- ✅ Use consistent error handling pattern

**Const Correctness:**
- ✅ Mark parameters `const` if they aren't modified
- ✅ Use `const` for string literals and read-only data
- ✅ Prefer `const` wherever possible

**Include Best Practices:**
- ✅ Include what you use
- ✅ Use forward declarations when possible
- ✅ Order includes: system headers → third-party → project headers
- ✅ Use header guards (`#ifndef`/`#define`/`#endif`) or `#pragma once`
- ✅ Never include .c files

**Function Design:**
- ✅ Keep functions small and focused
- ✅ Use output parameters for multiple return values
- ✅ Document all parameters and return values
- ✅ Validate all inputs
- ✅ Use meaningful names

### **Common C Idioms**

- **Opaque Pointers**: Hide implementation details
  ```c
  // header.h
  typedef struct data_loader DataLoader;
  DataLoader* data_loader_create(void);
  void data_loader_destroy(DataLoader* loader);
  
  // implementation.c
  struct data_loader {
      // Full definition here
  };
  ```

- **Function Pointers**: For callbacks and polymorphism
  ```c
  typedef int (*compare_func_t)(const void* a, const void* b);
  void qsort(void* base, size_t nmemb, size_t size, compare_func_t compar);
  ```

- **Goto Cleanup**: Consistent error handling
  ```c
  int process_file(const char* path) {
      FILE* fp = NULL;
      char* buffer = NULL;
      
      fp = fopen(path, "r");
      if (fp == NULL) {
          goto cleanup;
      }
      
      buffer = malloc(1024);
      if (buffer == NULL) {
          goto cleanup;
      }
      
      // ... process ...
      
      free(buffer);
      fclose(fp);
      return 0;
      
  cleanup:
      if (buffer) free(buffer);
      if (fp) fclose(fp);
      return -1;
  }
  ```

- **Resource Management**: Always pair allocation with deallocation
- **Error Codes**: Use consistent error code pattern (0 = success, -1 = error, set errno)

### **Error Patterns**

| Error | Cause | Solution |
|-------|-------|----------|
| Undefined reference | Missing link or implementation | Check linker flags, check .c file exists, check Makefile |
| Segmentation fault | Null pointer or out of bounds | Check pointer != NULL, check bounds, use valgrind |
| Memory leak | Missing free or early return | Use valgrind, use goto cleanup pattern |
| Use after free | Dangling pointer | Set pointer to NULL after free, use valgrind |
| Double free | Freeing same pointer twice | Set pointer to NULL after free |
| Buffer overflow | No bounds checking | Use `strncpy` instead of `strcpy`, use `snprintf` |
| Compilation error | Syntax or type mismatch | Check types, includes, syntax |
| Linker error | Missing library or symbol | Check Makefile, linker flags |

---

**End of Guide. Use Phase 0 → Execute → Iterate → Verify → Clean. Good luck!**

