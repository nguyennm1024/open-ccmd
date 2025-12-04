# Claude Code Assistant Guide - Expert Performance (C++ Edition)

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

Step 2: If too many results, grep for exact class/function names
grep "class.*Loader" -r src/  // Find exact loader classes

Step 3: Read ONLY the files that match both searches
read_file("src/data/loader.hpp")  // Confirmed relevant

Step 4: Validate you have the right files
- Check includes, class names, function signatures
- If wrong, refine search, don't just read more files
```

**Search quality checklist:**
- ✅ Search query is a full question, not just keywords
- ✅ Include context: "in the authentication system" not just "auth"
- ✅ Use grep to confirm exact class/function names before reading
- ✅ Read 3-5 files max initially - if you need more, your search was too broad
- ✅ Validate each file is relevant before moving on

### **Intelligence Gathering (Do in Parallel)**

```cpp
// Example: "Add caching to data loader"

// Step 1: Focused search (not broad)
codebase_search "How does the DataLoader load data from database?"
grep "class DataLoader" -r src/ include/  // Get exact file location

// Step 2: Parallel reads (ONLY confirmed relevant files)
- read_file("include/data/loader.hpp")    // Main interface (header)
- read_file("src/data/loader.cpp")         // Main implementation
- read_file("include/cache/manager.hpp")    // IF grep confirms cache exists
- read_file("tests/test_loader.cpp")      // Tests for expected behavior

// Step 3: Validate relevance
// Check: Does loader.hpp have the methods I need to modify?
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

---

## 🔌 INTERFACE-FIRST DESIGN (Critical for Integration)

**Problem**: Coding everything then testing leads to interface mismatches and integration bugs.

**Solution**: Define interfaces FIRST, implement SECOND, test EACH level.

### **The Interface-First Process**

**Step 1: Define the interface (function signature + contract)**

```cpp
// DON'T start coding the implementation yet!
// FIRST, define what the function MUST do:

// In header file (loader.hpp):
#ifndef DATA_LOADER_HPP
#define DATA_LOADER_HPP

#include <string>
#include <vector>
#include <unordered_map>

/**
 * Load user data with filters.
 * 
 * @param userId User ID (non-empty string)
 * @param filters Filter conditions (can be empty map)
 * @return Vector of data maps, empty vector if no data
 * @throws std::invalid_argument If userId is empty
 * @throws DatabaseException If database connection fails
 */
std::vector<std::unordered_map<std::string, std::string>> 
loadData(const std::string& userId, 
         const std::unordered_map<std::string, std::string>& filters);

#endif  // DATA_LOADER_HPP

// Define interface first, implement second
```

**Step 2: Write the test BEFORE implementation**

```cpp
TEST(DataLoaderTest, LoadData) {
    DataLoader loader;
    
    // Test the interface contract
    auto result = loader.loadData("user123", {{"status", "active"}});
    ASSERT_FALSE(result.empty() || result.size() > 0);  // Returns vector, not empty
    ASSERT_TRUE(std::all_of(result.begin(), result.end(), 
        [](const auto& item) { return item.find("id") != item.end(); }));
    
    // Test edge cases
    result = loader.loadData("user123", {});  // Empty filters
    ASSERT_TRUE(result.empty() || result.size() >= 0);  // Still returns vector
    
    // Test error cases
    EXPECT_THROW(loader.loadData("", {}), std::invalid_argument);  // Empty userId throws
}
```

**Step 3: Implement to satisfy the interface**

```cpp
// In implementation file (loader.cpp):
#include "data/loader.hpp"
#include <stdexcept>

std::vector<std::unordered_map<std::string, std::string>> 
DataLoader::loadData(const std::string& userId, 
                     const std::unordered_map<std::string, std::string>& filters) {
    if (userId.empty()) {
        throw std::invalid_argument("userId cannot be empty");
    }
    
    // Implementation...
    return results;  // MUST return vector, never throw unless error
}
```

**Step 4: Test the single function (bottom-up)**

```bash
./build/tests/test_loader --gtest_filter=DataLoaderTest.LoadData
# MUST pass before moving to integration
```

### **Interface Contract Checklist**

Before writing implementation, define:

- ✅ **Input types**: Exact parameter types (const std::string&, int, std::vector, std::map, std::optional?)
- ✅ **Output type**: What does it return? (std::vector? std::map? void? Never returns invalid state?)
- ✅ **Error cases**: What exceptions can it throw? When? (std::invalid_argument, std::runtime_error?)
- ✅ **Side effects**: Does it modify inputs? Change state? Call external APIs?
- ✅ **Preconditions**: What MUST be true before calling? (non-empty? valid format? non-null pointer?)
- ✅ **Postconditions**: What WILL be true after calling? (returns vector, never invalid state?)
- ✅ **Ownership**: Who owns returned objects? (RAII, smart pointers, move semantics?)

### **Common Interface Mistakes**

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Function sometimes returns empty, sometimes throws | Always return same type (empty vector, or always throw on error) |
| Function modifies input map | Use const reference, return new map |
| Error cases return empty/null instead of throwing | Throw specific exceptions for errors |
| No documentation | Add complete Doxygen/Javadoc comments |
| Unclear what empty/null means | Use std::optional<T> if empty is valid, document meaning |
| Raw pointers without ownership | Use smart pointers (std::unique_ptr, std::shared_ptr) |
| Missing const correctness | Use const methods and const references where appropriate |

---

## 🏗️ BOTTOM-UP DEVELOPMENT (Build Solid Foundation)

**Problem**: Coding everything at once leads to many bugs.

**Solution**: Build and verify each level before moving up.

### **The Bottom-Up Process**

```
Level 1: Individual functions (test each)
    ↓ (all pass)
Level 2: Class/Module integration (test together)
    ↓ (all pass)
Level 3: Component integration (test full flow)
    ↓ (all pass)
Level 4: System integration (test end-to-end)
```

### **Example: Add caching to data loader**

**❌ WRONG approach:**
```cpp
// Code everything at once
// cached_loader.hpp
class CachedDataLoader {
public:
    CachedDataLoader();
    ~CachedDataLoader() = default;
    CachedDataLoader(const CachedDataLoader&) = delete;
    CachedDataLoader& operator=(const CachedDataLoader&) = delete;
    
    std::vector<Data> loadData(const std::string& userId, const Filters& filters);
    void invalidateCache();
private:
    std::string generateCacheKey(const std::string& userId, const Filters& filters);
    std::optional<std::vector<Data>> fetchFromCache(const std::string& key);
    void storeToCache(const std::string& key, const std::vector<Data>& data);
    std::unordered_map<std::string, std::vector<Data>> cache_;
    // ... all methods at once
};

// Test everything together
./build/tests/test_cached_loader
// 10 failures! Which function is broken?
```

**✅ RIGHT approach:**

```cpp
// Level 1: Build and test ONE function at a time

// Step 1a: Implement cache key generation
std::string CachedDataLoader::generateCacheKey(const std::string& userId, const Filters& filters) {
    std::ostringstream oss;
    oss << "data:" << userId;
    for (const auto& [key, value] : filters) {
        oss << ":" << key << "=" << value;
    }
    return oss.str();  // RVO optimization
}

// Step 1b: Test it IMMEDIATELY
TEST(CachedDataLoaderTest, GenerateCacheKey) {
    CachedDataLoader loader;
    auto key1 = loader.generateCacheKey("u1", {{"a", "1"}});
    auto key2 = loader.generateCacheKey("u1", {{"a", "1"}});
    ASSERT_EQ(key1, key2);  // Same inputs = same key
    // Run: ./build/tests/test_cached_loader --gtest_filter=CachedDataLoaderTest.GenerateCacheKey
    // ✅ PASS before continuing
}

// Step 2a: Implement cache fetch
std::optional<std::vector<Data>> CachedDataLoader::fetchFromCache(const std::string& key) {
    auto it = cache_.find(key);
    return (it != cache_.end()) ? std::make_optional(it->second) : std::nullopt;
}

// Step 2b: Test it IMMEDIATELY  
TEST(CachedDataLoaderTest, FetchFromCache) {
    CachedDataLoader loader;
    ASSERT_FALSE(loader.fetchFromCache("missing").has_value());
    loader.storeToCache("key1", {{"data", "1"}});
    ASSERT_TRUE(loader.fetchFromCache("key1").has_value());
    // Run: ./build/tests/test_cached_loader --gtest_filter=CachedDataLoaderTest.FetchFromCache
    // ✅ PASS before continuing
}

// Step 3a: Implement cache store
void CachedDataLoader::storeToCache(const std::string& key, const std::vector<Data>& data) {
    cache_[key] = data;
}

// Step 3b: Test it IMMEDIATELY
TEST(CachedDataLoaderTest, StoreToCache) {
    CachedDataLoader loader;
    loader.storeToCache("key1", {{"data", "1"}});
    ASSERT_TRUE(loader.fetchFromCache("key1").has_value());
    // Run: ./build/tests/test_cached_loader --gtest_filter=CachedDataLoaderTest.StoreToCache
    // ✅ PASS before continuing
}

// Level 2: Now integrate them
std::vector<Data> CachedDataLoader::loadData(
    const std::string& userId, 
    const Filters& filters) {
    auto key = generateCacheKey(userId, filters);  // ✅ Already tested
    auto cached = fetchFromCache(key);              // ✅ Already tested
    if (cached.has_value()) {
        return cached.value();
    }
    auto data = fetchFromDb(userId, filters);       // ✅ Already tested
    storeToCache(key, data);                        // ✅ Already tested
    return data;
}

// Test integration
TEST(CachedDataLoaderTest, LoadDataIntegration) {
    // All components work individually, now test together
    // Run: ./build/tests/test_cached_loader --gtest_filter=CachedDataLoaderTest.LoadDataIntegration
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
   ./build/tests/test_loader --gtest_filter=DataLoaderTest.Caching
   Result: ✅ PASS

2. Run integration test:
   ./build/tests/test_integration
   Result: ✅ PASS

3. Run manual test with real data:
   ./build/bin/loader_demo
   Result: [{"id": 1, "name": "test"}] ✅ Correct format

4. Check cache is actually used:
   ./build/bin/loader_demo --benchmark
   Result: First: 150ms, Cached: 2ms ✅ Cache is working

VERIFIED: Problem is actually fixed."
```

### **Verification Checklist (ALL must pass)**

After claiming "it's fixed", you MUST:

- [ ] **Unit test passes** - Run specific test function
- [ ] **Integration test passes** - Run related integration tests  
- [ ] **Manual test works** - Run actual code with real input
- [ ] **Edge cases work** - Test with empty, null, invalid inputs
- [ ] **No regressions** - Run full test suite: `./build/tests/all_tests`
- [ ] **Linter clean** - Run `clang-tidy` or project linter
- [ ] **Actually test the bug** - If fixing a bug, reproduce the bug first, then verify it's gone
- [ ] **Memory checks** - Run with valgrind or sanitizers: `valgrind --leak-check=full ./build/tests/test_loader`
- [ ] **Address sanitizer** - Compile with `-fsanitize=address` and run tests
- [ ] **Undefined behavior** - Compile with `-fsanitize=undefined` and run tests

### **Never Say "Fixed" Until:**

```bash
# ALL of these must be true:
✅ ./build/tests/test_loader --gtest_filter=DataLoaderTest.SpecificTest  # PASS
✅ ./build/tests/all_tests                                                # All PASS
✅ ./build/bin/demo_app                                                   # Works
✅ clang-tidy src/loader.cpp                                              # No errors
✅ git status                                                              # Clean

# Then and ONLY then:
"VERIFIED: Problem is fixed. All tests pass, linter clean, manual test works."
```

---

## 📐 SYNTAX & TYPE CHECKING (Catch Errors Early)

**Problem**: Writing wrong C++ syntax leads to compilation errors.

**Solution**: Self-check syntax and types BEFORE running.

### **Pre-Run Syntax Check**

**After writing code, BEFORE testing, check:**

```bash
# 1. Check syntax is valid (compile)
cmake --build build --target loader
# Must complete without errors

# 2. Check includes work (test compilation unit)
g++ -std=c++17 -I./include -c src/loader.cpp -o /tmp/test.o -Wall -Wextra
# Must not raise include errors or warnings

# 3. Check header guards (if modifying headers)
g++ -std=c++17 -I./include -E include/data/loader.hpp | grep -q "#define.*HPP" || echo "Missing header guard!"

# 4. Check types and static analysis (if using clang-tidy)
clang-tidy src/loader.cpp -- -std=c++17 -I./include -Wall -Wextra
# Fix any type errors or warnings

# 5. Check linter/formatter
clang-format --dry-run -Werror src/loader.cpp include/data/loader.hpp
# Fix any style issues

# 6. Check for common issues (if using include-what-you-use)
include-what-you-use src/loader.cpp -std=c++17 -I./include
```

### **Common C++ Syntax Mistakes**

| ❌ Wrong | ✅ Right | Error |
|---------|---------|-------|
| `void func();` (missing body in .cpp) | `void func() { }` or implement in .cpp | Linker error |
| `return x, y` (comma operator) | `return std::make_pair(x, y)` or `return {x, y}` | Wrong return type |
| `std::vector<int> v; v[0]` (empty) | `v.at(0)` or check `!v.empty()` | Undefined behavior |
| `delete ptr` (double delete) | Use smart pointers or RAII | Double free |
| `int* a, b` (b is int) | `int *a, *b` or `int* a; int* b;` | Type mismatch |
| `if (ptr)` (null check) | `if (ptr != nullptr)` | Use nullptr explicitly |
| Missing `const` correctness | Use `const` methods and `const&` params | Compiler warnings |
| `std::string s = "text" + "text"` | `std::string s = std::string("text") + "text"` or `"text"s + "text"s` | No operator+ for char* |
| Missing header guards | `#ifndef HEADER_HPP` / `#define HEADER_HPP` / `#endif` | Multiple definition errors |
| `#include <iostream>` in header | Forward declare or include in .cpp only | Slow compilation |
| `using namespace std;` in header | Use `std::` prefix or `using std::string;` | Name pollution |
| Missing `const` in getters | `int getValue() const { return value_; }` | Can't call on const objects |

### **Type System Self-Check**

**Every function MUST have:**
```cpp
// In header file (processor.hpp):
#ifndef PROCESSOR_HPP
#define PROCESSOR_HPP

#include <vector>
#include <string>
#include <unordered_map>
#include <optional>

/**
 * Process data with filters.
 * 
 * @param userId User ID (non-empty string)
 * @param filters Filter conditions (can be empty map)
 * @param limit Optional limit (nullopt if no limit)
 * @return Vector of processed data, empty if no data
 * @throws std::invalid_argument If userId is empty
 */
std::vector<std::unordered_map<std::string, std::string>> processData(
    const std::string& userId,                      // ✅ Input type (const ref)
    const std::unordered_map<std::string, std::string>& filters,  // ✅ Input type
    const std::optional<int>& limit = std::nullopt  // ✅ Optional input
);  // ✅ Return type

#endif  // PROCESSOR_HPP
```

**Check:**
- ✅ Header guards present (`#ifndef`, `#define`, `#endif`)
- ✅ All parameters have clear types
- ✅ Return type is specified
- ✅ Use `std::optional<T>` for parameters that can be empty/null
- ✅ Use specific types (std::vector<std::string>, not just vector)
- ✅ Use const references for large objects (avoid copying)
- ✅ Use const methods where appropriate (`const` after function name)
- ✅ Document all parameters and return value (Doxygen style)
- ✅ Includes are minimal (forward declare when possible)
- ✅ No `using namespace` in headers

---

## ⚠️ CRITICAL: Error Prevention

### **Top 2 Errors (90% of Failures)**

| Error | Cause | Fix |
|-------|-------|-----|
| "Error editing file" | Didn't read file first OR old_string mismatch | ALWAYS `read_file()` → Copy EXACT text → Edit |
| "No shell found with ID" | Assumed shell persistence | Use absolute paths OR `cd /path && command` |

### **Environment & Development Rules**

**Build System**:
- ✅ **ALWAYS** use build system (CMake, Make, Bazel, etc.)
- ✅ Build before running: `cmake --build build` or `make -C build`
- ✅ Run tests with build system: `cmake --build build --target test` or `cd build && ctest`
- ✅ Use out-of-source builds: `mkdir -p build && cd build && cmake ..`
- ✅ Check build directory exists before executing: `test -d build || (mkdir -p build && cd build && cmake ..)`
- ✅ Use CMake presets or configure with: `cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_STANDARD=17 ..`
- ✅ Enable sanitizers for debugging: `cmake -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined" ..`

**CMake Best Practices:**
```cmake
# CMakeLists.txt example
cmake_minimum_required(VERSION 3.15)
project(MyProject VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Add compiler warnings
if(MSVC)
    add_compile_options(/W4 /WX)
else()
    add_compile_options(-Wall -Wextra -Wpedantic -Werror)
endif()

# Include directories
target_include_directories(my_lib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# Link libraries
target_link_libraries(my_lib PUBLIC
    Threads::Threads
    ${CMAKE_DL_LIBS}
)

# Enable testing
enable_testing()
add_subdirectory(tests)
```

**Example Files**:
- ✅ Create **ONE file per request** in `examples/` folder
- ❌ Do NOT create multiple example files for a single request
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.cpp` (not `example1.cpp`, `example2.cpp`)
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
| Know exact text | Use grep | `grep "class Name" -r src/` |
| Unfamiliar library/error | Web search | "C++ library error context 2025" |
| Simple change | Do it | Skip TODO |
| 3+ files or steps | Plan | Create TODO list |
| Tried 3x, failed | Research | Examples → web search |
| Debugging | Systematic | Hypothesis → Test → Analyze |

### **When to Web Search**

✅ **USE web search:**
- Unfamiliar library/API: "C++ Boost Asio async handling 2025"
- Unknown error (after checking codebase): "C++ undefined reference linker error"
- Best practices: "C++ RAII patterns 2025"
- After 3 failed attempts

❌ **DON'T web search:**
- Info in codebase → Use `codebase_search` or `grep`
- Standard C++ → You know this
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
2. HYPOTHESIZE: DataProcessor doesn't clean up allocated resources
3. IMPLEMENT: Add delete statements in destructor
4. TEST: Still leaks memory (valgrind shows leaks)
5. ANALYZE: Exception thrown before delete, destructor not called
6. NEW HYPOTHESIS: Need RAII pattern with smart pointers
7. IMPLEMENT: Replace raw pointers with std::unique_ptr
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

```cpp
ERROR: Segmentation fault (core dumped)

HYPOTHESIZE:
A (70%): Null pointer dereference
B (20%): Out of bounds access
C (10%): Use after free

TEST A: Add null check before pointer use
std::cout << "Pointer: " << (ptr ? "valid" : "null") << std::endl;
RESULT: Pointer: null  // ptr is null!
→ Hypothesis A CONFIRMED

ROOT CAUSE: Function doesn't check if ptr is null before use
FIX: if (ptr == nullptr) { throw std::invalid_argument("ptr is null"); }
VERIFY: ✅ Works
```

### **Common Patterns**

| Error | Likely Cause | Quick Test |
|-------|--------------|------------|
| `Segmentation fault` | Null pointer or out of bounds | `gdb ./build/bin/app`, `valgrind`, or `-fsanitize=address` |
| `Undefined reference` | Missing link or implementation | Check linker flags, check .cpp file exists, check CMakeLists.txt |
| `Use after free` | Dangling pointer | Use smart pointers, check lifetime, use `-fsanitize=address` |
| `Memory leak` | Missing delete or exception | Use RAII, smart pointers, `valgrind --leak-check=full` |
| `Double free` | Deleting same pointer twice | Use smart pointers, `valgrind` or `-fsanitize=address` |
| Wrong result | Logic error | Print intermediate values, use `gdb`, add assertions |
| `Undefined behavior` | Uninitialized variables, etc. | Use `-fsanitize=undefined`, `-Wall -Wextra` |

### **C++ Debugging Tools**

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
g++ -fsanitize=address -g -O1 src/app.cpp -o app
./app  # Will show memory errors at runtime

# Undefined Behavior Sanitizer
g++ -fsanitize=undefined -g -O1 src/app.cpp -o app

# Static analysis
clang-tidy src/app.cpp -- -std=c++17 -I./include
cppcheck --enable=all src/
```

---

## 🤖 TOOL USAGE & COMMUNICATION

### **Tool Selection**

| Goal | Tool | Pattern |
|------|------|---------|
| Find concept | `codebase_search` | "How does X work?" |
| Find exact text | `grep` | `grep "class Name"` |
| Read code | `read_file` | Read 3-5 files in parallel |
| Unknown API | `web_search` | "C++ library API 2025" |

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
| RAII | Destructor cleanup | Use smart pointers, follow RAII pattern |
| PIMPL | Forward declaration + unique_ptr | Use pimpl pattern for ABI stability |
| Factory | `create_*()`, `make_*()` | Use factory method |
| Singleton | `getInstance()` | Use getInstance(), don't instantiate |

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
- ❌ Do NOT create multiple example files (`example1.cpp`, `example2.cpp`, etc.)
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.cpp`
- ✅ If multiple examples needed, combine into one file with clear sections/comments

### **Temporary Files (Rare, Delete Immediately)**

**If absolutely needed:**
- Use prefixes: `verify_*.cpp`, `debug_*.cpp`, `temp_*.cpp`
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
cmake --build build --config Debug  # or Release

# 2. Tests
cd build && ctest --output-on-failure
# OR: ./build/tests/all_tests

# 3. Linter
clang-tidy src/ -- -std=c++17 -I./include -Wall -Wextra

# 4. Formatter
clang-format --dry-run -Werror src/ include/

# 5. Static analysis (optional)
cppcheck --enable=all --suppress=missingIncludeSystem src/

# 6. Memory checks (optional but recommended)
valgrind --leak-check=full --show-leak-kinds=all ./build/tests/all_tests

# 7. Git status clean
git status  # Only intended files?

# 8. No temp files
find . -name "verify_*" -o -name "debug_*" -o -name "temp_*"  # Should be empty
```

### **Done = ALL True**

- ✅ Code compiles + tests pass
- ✅ No linter errors
- ✅ ALL temp files deleted
- ✅ No .md files created (unless asked)
- ✅ `git status` clean - only intended changes
- ✅ Integration verified
- ✅ No memory leaks (if checked with valgrind/sanitizers)

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
2. Grep for exact names: `grep "class Loader" -r src/`
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
THEN say "fixed"
```

**Syntax Check** (Before running):
```bash
cmake --build build --target module  # Check compilation
clang-tidy src/module.cpp -- -std=c++17 -I./include  # Check types/style
clang-format --dry-run -Werror src/module.cpp include/module.hpp  # Check formatting
g++ -std=c++17 -I./include -c src/module.cpp -o /tmp/test.o -Wall -Wextra  # Quick compile check
```

**Tool Decisions**:
- Find concept → `codebase_search "How does X work in Y?"`
- Find exact text → `grep "class Name" -r src/`
- Unknown API/error → `web_search` (with year)

**Build System**:
- Compile → Always use build system: `cmake --build build` or `make -C build`
- Run tests → Use build system: `cd build && ctest` or `cmake --build build --target test`
- Install deps → Use package manager (vcpkg, conan) or build system
- Debug build → `cmake -DCMAKE_BUILD_TYPE=Debug ..`
- Release build → `cmake -DCMAKE_BUILD_TYPE=Release ..`
- Sanitizers → `cmake -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined" ..`

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
- **Compilation errors when running** → Should have checked with `cmake --build` first
- **Type errors** → Should have checked types match
- **Same test fails differently** → Fix root cause, not symptom
- **Don't understand error** → Use hypothesis-driven debugging
- **Creating temp files** → Edit existing code instead
- **Memory leaks** → Use RAII and smart pointers, run valgrind or sanitizers
- **Missing header guards** → Always use `#ifndef HEADER_HPP` / `#define HEADER_HPP` / `#endif`
- **Slow compilation** → Forward declare in headers, include in .cpp files

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
grep "class.*Auth.*Cache" -r src/  # Get exact file
read_file("src/auth/cache.hpp")     # Read ONLY confirmed file
# Validate: Does this file have the methods I need? YES → Use it, NO → Refine search
```

### **Failure Pattern 2: "Big Bang Integration"**

**Symptom**: Wrote 5 functions, tested together, everything breaks, don't know which function is wrong

**Root cause**: Didn't test each function individually

**Fix**:
```cpp
// ❌ BAD - All functions at once
class DataProcessor {
    void load(); void process(); void save();
};
./build/tests/test_processor  // 10 failures, which broke?

// ✅ GOOD - One function at a time
void load();
TEST(DataProcessorTest, Load) { /* test */ }
./build/tests/test_processor --gtest_filter=DataProcessorTest.Load  // ✅ PASS - Continue
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
1. Reproduce bug: ./build/bin/demo_app → Cache not cleared ❌
2. Apply fix: cache.delete(key) after update
3. Test again: ./build/bin/demo_app → Cache cleared ✅
4. Run unit test: ./build/tests/test_cache --gtest_filter=CacheTest.Invalidation → PASSED ✅
5. Run integration: ./build/tests/test_integration → PASSED ✅
VERIFIED: Bug is fixed."
```

### **Failure Pattern 4: "Interface Mismatch"**

**Symptom**: Individual functions work, but fail when integrated

**Root cause**: Interfaces don't match - function A returns vector but function B expects map

**Fix**:
```cpp
// ❌ BAD - Interface mismatch
std::vector<Data> fetchData(const std::string& userId);  // Missing const correctness!
// Sometimes returns empty vector, sometimes throws

void processData(std::vector<Data> data) {  // Missing const reference!
    for (auto& item : data) {  // Crashes if data is empty!
        // ...
    }
}

// ✅ GOOD - Clear interfaces
/**
 * Fetch user data.
 * 
 * @param userId User ID (non-empty)
 * @return Vector with user data. Never returns invalid state.
 *         Returns empty vector {} if user not found.
 * @throws std::invalid_argument If userId is empty
 */
std::vector<Data> fetchData(const std::string& userId) {
    if (userId.empty()) {
        throw std::invalid_argument("userId cannot be empty");
    }
    auto data = db.query(userId);
    return data;  // Always returns vector (empty if not found)
}

/**
 * Process data vector.
 * 
 * @param data User data vector (can be empty)
 * @return Vector of processed items. Empty vector if no items.
 */
std::vector<ProcessedItem> processData(const std::vector<Data>& data) {
    std::vector<ProcessedItem> result;
    for (const auto& item : data) {  // Handles empty vector
        result.push_back(processItem(item));
    }
    return result;
}

// Now interfaces match: vector → vector, const reference
```

### **Failure Pattern 5: "Syntax Before Test"**

**Symptom**: Code has C++ compilation errors when running

**Root cause**: Didn't check compilation before testing

**Fix**:
```bash
# ✅ GOOD - Check compilation BEFORE running tests

# 1. Check compilation
cmake --build build --target module
# No output = compilation OK

# 2. Check includes
g++ -std=c++17 -I./include -c src/module.cpp -o /tmp/test.o
# No error = includes OK

# 3. Check types (if using clang-tidy)
clang-tidy src/module.cpp -- -std=c++17 -I./include
# No error = types OK

# 4. Check linter
clang-format --dry-run src/module.cpp
# No error = style OK

# 5. NOW run tests
./build/tests/test_module
```

### **How to Avoid These Failures**

| Failure Pattern | Prevention |
|----------------|------------|
| Shotgun Search | Specific questions + grep + validate 3-5 files max |
| Big Bang Integration | Test each function individually before integration |
| Assumption Completion | Verify with actual tests (unit + integration + manual) |
| Interface Mismatch | Define interfaces first with clear types and contracts |
| Compilation Errors | Check with `cmake --build` and linter before running |

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

## 🔑 TOP 10 RULES

1. ⚠️ **Smart search** - Specific questions + grep exact names + read 3-5 files max
2. ⚠️ **Interface first** - Define signature/contract → Write test → Implement
3. ⚠️ **Bottom-up build** - One function → Test it → PASS → Next function
4. ⚠️ **Never assume fixed** - Verify with actual tests (unit + integration + manual)
5. ⚠️ **Compile check first** - `cmake --build build` before running
6. ⚠️ **Test immediately** - After EVERY function, not after coding everything
7. ⚠️ **Read before edit** - Prevents 90% of errors
8. ⚠️ **Iterate systematically** - Different approaches, not same retry
9. ⚠️ **Root cause debugging** - Ask "Why?" 5 times, fix cause not symptom
10. ⚠️ **Clean workspace** - git status clean, all tests pass, linter clean
11. ⚠️ **Use build system** - Always use CMake/Make/Bazel for compilation
12. ⚠️ **One example file** - Create ONE file per request in `examples/`, not multiple files
13. ⚠️ **RAII and smart pointers** - Use RAII patterns, avoid raw pointers

---

## 📚 PROJECT SPECIFICS

### **C++ Project Structure**

```
project/
├── src/                    # Main code (.cpp files)
│   ├── core/               # Core functionality
│   └── utils/              # Utilities
├── include/                # Header files (.hpp/.h files)
│   ├── project_name/       # Project namespace directory
│   │   ├── core/           # Core headers
│   │   └── utils/          # Utility headers
│   └── project_name.hpp    # Main header (optional)
├── tests/                  # Test suites
│   ├── unit/               # Unit tests
│   └── integration/        # Integration tests
├── examples/               # Usage examples
├── CMakeLists.txt          # CMake config (or Makefile)
├── .clang-format           # Clang format config
├── .clang-tidy             # Clang tidy config
└── build/                  # Build directory (out-of-source, gitignored)
```

**Include Path Convention:**
```cpp
// In src/core/loader.cpp:
#include "project_name/core/loader.hpp"  // Project headers use project_name/ prefix

// In include/project_name/core/loader.hpp:
#ifndef PROJECT_NAME_CORE_LOADER_HPP
#define PROJECT_NAME_CORE_LOADER_HPP

// 1. System headers first
#include <string>
#include <vector>
#include <unordered_map>
#include <memory>

// 2. Third-party headers
#include <boost/optional.hpp>

// 3. Project headers last
#include "project_name/utils/helper.hpp"

namespace project_name {
namespace core {

class Loader {
    // Implementation
};

}  // namespace core
}  // namespace project_name

#endif  // PROJECT_NAME_CORE_LOADER_HPP
```

**Header File Template:**
```cpp
#ifndef PROJECT_NAME_MODULE_CLASS_HPP
#define PROJECT_NAME_MODULE_CLASS_HPP

// System includes
#include <vector>
#include <string>

// Forward declarations (prefer over includes when possible)
class OtherClass;

// Project includes
#include "project_name/base/common.hpp"

namespace project_name {
namespace module {

class MyClass {
public:
    MyClass();
    ~MyClass() = default;
    
    // Rule of Five
    MyClass(const MyClass&) = default;
    MyClass(MyClass&&) = default;
    MyClass& operator=(const MyClass&) = default;
    MyClass& operator=(MyClass&&) = default;
    
    // Public interface
    void doSomething() const;
    
private:
    std::unique_ptr<OtherClass> pimpl_;  // PIMPL pattern
};

}  // namespace module
}  // namespace project_name

#endif  // PROJECT_NAME_MODULE_CLASS_HPP
```

### **Code Style**

- **Formatter**: clang-format (Google, LLVM, or custom style)
- **Linter**: clang-tidy
- **Type check**: Compiler warnings + clang-tidy
- **Testing**: Google Test (gtest) or Catch2
- **Build system**: CMake (most common), Make, Bazel

### **C++ Standards**

- **C++17** or **C++20** (prefer modern C++)
- Use modern features: `auto`, range-based for, smart pointers, `std::optional`, structured bindings
- Avoid: Raw pointers (use smart pointers), C-style arrays (use std::array/vector), `new`/`delete` (use smart pointers)

### **C++ Best Practices**

**Memory Management:**
- ✅ Use `std::unique_ptr` for single ownership
- ✅ Use `std::shared_ptr` for shared ownership (sparingly)
- ✅ Use `std::weak_ptr` to break circular references
- ✅ Use RAII for all resources (files, locks, etc.)
- ❌ Never use raw `new`/`delete` in modern C++
- ❌ Never use raw pointers for ownership

**Const Correctness:**
- ✅ Mark methods `const` if they don't modify state
- ✅ Use `const&` for parameters you don't modify
- ✅ Use `const` variables when possible
- ✅ Prefer `const_iterator` when not modifying containers

**Move Semantics (C++11+):**
- ✅ Use `std::move` to transfer ownership
- ✅ Implement move constructor and move assignment
- ✅ Return by value (RVO/NRVO optimization)
- ✅ Use `std::make_unique` and `std::make_shared`

**Include Best Practices:**
- ✅ Include what you use
- ✅ Use forward declarations in headers when possible
- ✅ Order includes: system headers → third-party → project headers
- ✅ Use header guards (`#ifndef`/`#define`/`#endif`) or `#pragma once`
- ❌ Never `using namespace std;` in headers

### **Error Patterns**

| Error | Cause | Solution |
|-------|-------|----------|
| Undefined reference | Missing link or implementation | Check linker flags, check .cpp file exists |
| Segmentation fault | Null pointer or out of bounds | Check pointer != nullptr, check bounds |
| Memory leak | Missing delete or exception | Use RAII, smart pointers |
| Use after free | Dangling pointer | Use smart pointers, check lifetime |
| Compilation error | Syntax or type mismatch | Check types, includes, syntax |
| Linker error | Missing library or symbol | Check CMakeLists.txt, linker flags |

### **Common C++ Idioms**

- **RAII**: Resource Acquisition Is Initialization - use destructors for cleanup
  ```cpp
  class FileHandler {
      std::unique_ptr<FILE, decltype(&fclose)> file_;
  public:
      FileHandler(const char* path) : file_(fopen(path, "r"), fclose) {}
      // Destructor automatically closes file
  };
  ```

- **Smart Pointers**: `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`
  ```cpp
  auto ptr = std::make_unique<MyClass>(args);  // Prefer make_unique
  auto shared = std::make_shared<MyClass>(args);  // Prefer make_shared
  ```

- **Move Semantics**: Use `std::move` for efficient transfers
  ```cpp
  std::vector<int> data = std::move(other_data);  // Transfer ownership
  ```

- **const correctness**: Use `const` methods and `const&` parameters
  ```cpp
  int getValue() const { return value_; }  // const method
  void process(const std::string& data);   // const reference
  ```

- **Rule of Three/Five/Zero**: Copy/move constructors and assignment operators
  ```cpp
  class MyClass {
      // Rule of Zero: Let compiler generate everything
      // OR Rule of Five: Define all or none
      MyClass(const MyClass&) = default;
      MyClass(MyClass&&) = default;
      MyClass& operator=(const MyClass&) = default;
      MyClass& operator=(MyClass&&) = default;
      ~MyClass() = default;
  };
  ```

- **PIMPL**: Pointer to Implementation for ABI stability
  ```cpp
  // header.hpp
  class MyClass {
      class Impl;
      std::unique_ptr<Impl> pimpl_;
  public:
      MyClass();
      ~MyClass();
  };
  ```

- **CRTP**: Curiously Recurring Template Pattern for static polymorphism
- **SFINAE**: Substitution Failure Is Not An Error (or use `if constexpr` in C++17)
- **Type Traits**: Use `std::is_same`, `std::enable_if`, etc. for template metaprogramming

---

**End of Guide. Use Phase 0 → Execute → Iterate → Verify → Clean. Good luck!**

