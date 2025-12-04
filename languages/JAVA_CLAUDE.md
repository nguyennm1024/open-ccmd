# Claude Code Assistant Guide - Expert Performance (Java Edition)

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
codebase_search "cache"  # Too broad, returns 100+ irrelevant files
→ Wastes time reading wrong files
```

**✅ RIGHT way to search:**
```
Step 1: Start specific with context
codebase_search "How is data caching implemented in the data loader?"

Step 2: If too many results, grep for exact class/method names
grep "class.*Loader" -r src/  # Find exact loader classes

Step 3: Read ONLY the files that match both searches
read_file("src/data/Loader.java")  # Confirmed relevant

Step 4: Validate you have the right files
- Check imports, class names, method signatures
- If wrong, refine search, don't just read more files
```

**Search quality checklist:**
- ✅ Search query is a full question, not just keywords
- ✅ Include context: "in the authentication system" not just "auth"
- ✅ Use grep to confirm exact class/method names before reading
- ✅ Read 3-5 files max initially - if you need more, your search was too broad
- ✅ Validate each file is relevant before moving on

### **Intelligence Gathering (Do in Parallel)**

```java
// Example: "Add caching to data loader"

// Step 1: Focused search (not broad)
codebase_search "How does the DataLoader load data from database?"
grep "class DataLoader" -r src/  // Get exact file location

// Step 2: Parallel reads (ONLY confirmed relevant files)
- read_file("src/data/Loader.java")       // Main implementation
- read_file("src/cache/CacheManager.java") // IF grep confirms cache exists
- read_file("test/data/LoaderTest.java")   // Tests for expected behavior

// Step 3: Validate relevance
// Check: Does Loader.java have the methods I need to modify?
// If NO → You searched wrong, refine and try again
```

**Mental model checklist:**
- ✅ Entry points (how is this called?)
- ✅ Data flow (what data moves where?)
- ✅ Dependencies (what does this use?)
- ✅ Side effects (what else changes?)
- ✅ Error paths (what can go wrong?)
- ✅ **Interfaces** (what are the method signatures and contracts?)

---

## 🔌 INTERFACE-FIRST DESIGN (Critical for Integration)

**Problem**: Coding everything then testing leads to interface mismatches and integration bugs.

**Solution**: Define interfaces FIRST, implement SECOND, test EACH level.

### **The Interface-First Process**

**Step 1: Define the interface (method signature + contract)**

```java
// DON'T start coding the implementation yet!
// FIRST, define what the method MUST do:

/**
 * Load user data with filters.
 * 
 * @param userId User ID (non-null, non-empty string)
 * @param filters Filter conditions (can be empty map)
 * @return List of data maps, empty list if no data
 * @throws IllegalArgumentException If userId is null or empty
 * @throws DatabaseException If database connection fails
 */
List<Map<String, Object>> loadData(String userId, Map<String, Object> filters);
// Define interface first, implement second
```

**Step 2: Write the test BEFORE implementation**

```java
@Test
void testLoadData() {
    // Test the interface contract
    List<Map<String, Object>> result = loader.loadData("user123", 
        Map.of("status", "active"));
    assertNotNull(result);  // Returns list, not null
    assertTrue(result instanceof List);  // List of maps
    
    // Test edge cases
    result = loader.loadData("user123", Map.of());  // Empty filters
    assertNotNull(result);  // Still returns list
    
    // Test error cases
    assertThrows(IllegalArgumentException.class, 
        () -> loader.loadData("", Map.of()));  // Empty userId throws
}
```

**Step 3: Implement to satisfy the interface**

```java
@Override
public List<Map<String, Object>> loadData(String userId, Map<String, Object> filters) {
    if (userId == null || userId.isEmpty()) {
        throw new IllegalArgumentException("userId cannot be null or empty");
    }
    
    // Implementation...
    return results;  // MUST return list, never null
}
```

**Step 4: Test the single method (bottom-up)**

```bash
./mvnw test -Dtest=LoaderTest#testLoadData
# MUST pass before moving to integration
```

### **Interface Contract Checklist**

Before writing implementation, define:

- ✅ **Input types**: Exact parameter types (String, int, Map, List, Optional?)
- ✅ **Output type**: What does it return? (List? Map? null? Never null?)
- ✅ **Error cases**: What exceptions can it throw? When?
- ✅ **Side effects**: Does it modify inputs? Change state? Call external APIs?
- ✅ **Preconditions**: What MUST be true before calling? (non-null? non-empty? valid format?)
- ✅ **Postconditions**: What WILL be true after calling? (returns list, never null?)

### **Common Interface Mistakes**

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Method sometimes returns null, sometimes list | Always return same type (empty list, not null) |
| Method modifies input map | Return new map, don't modify input |
| Error cases return null instead of throwing | Throw specific exceptions for errors |
| No JavaDoc | Add complete JavaDoc with @param, @return, @throws |
| Unclear what null means | Use Optional<T> if null is valid, document meaning |

---

## 🏗️ BOTTOM-UP DEVELOPMENT (Build Solid Foundation)

**Problem**: Coding everything at once leads to many bugs.

**Solution**: Build and verify each level before moving up.

### **The Bottom-Up Process**

```
Level 1: Individual methods (test each)
    ↓ (all pass)
Level 2: Class integration (test together)
    ↓ (all pass)
Level 3: Component integration (test full flow)
    ↓ (all pass)
Level 4: System integration (test end-to-end)
```

### **Example: Add caching to data loader**

**❌ WRONG approach:**
```java
// Code everything at once
public class CachedDataLoader {
    public CachedDataLoader() { ... }
    public List<Map<String, Object>> loadData() { ... }
    private Map<String, Object> fetchFromDb() { ... }
    private List<Map<String, Object>> fetchFromCache() { ... }
    private void storeToCache() { ... }
    public void invalidateCache() { ... }
}

// Test everything together
./mvnw test -Dtest=CachedDataLoaderTest
// 10 failures! Which method is broken?
```

**✅ RIGHT approach:**

```java
// Level 1: Build and test ONE method at a time

// Step 1a: Implement cache key generation
private String generateCacheKey(String userId, Map<String, Object> filters) {
    return "data:" + userId + ":" + filters.hashCode();
}

// Step 1b: Test it IMMEDIATELY
@Test
void testGenerateCacheKey() {
    CachedDataLoader loader = new CachedDataLoader();
    String key1 = loader.generateCacheKey("u1", Map.of("a", 1));
    String key2 = loader.generateCacheKey("u1", Map.of("a", 1));
    assertEquals(key1, key2);  // Same inputs = same key
    // Run: ./mvnw test -Dtest=CachedDataLoaderTest#testGenerateCacheKey
    // ✅ PASS before continuing
}

// Step 2a: Implement cache fetch
private Optional<List<Map<String, Object>>> fetchFromCache(String key) {
    return Optional.ofNullable(cache.get(key));
}

// Step 2b: Test it IMMEDIATELY  
@Test
void testFetchFromCache() {
    CachedDataLoader loader = new CachedDataLoader();
    assertTrue(loader.fetchFromCache("missing").isEmpty());
    cache.put("key1", List.of(Map.of("data", 1)));
    assertEquals(List.of(Map.of("data", 1)), 
        loader.fetchFromCache("key1").orElse(List.of()));
    // Run: ./mvnw test -Dtest=CachedDataLoaderTest#testFetchFromCache
    // ✅ PASS before continuing
}

// Step 3a: Implement cache store
private void storeToCache(String key, List<Map<String, Object>> data) {
    cache.put(key, data);
}

// Step 3b: Test it IMMEDIATELY
@Test
void testStoreToCache() {
    CachedDataLoader loader = new CachedDataLoader();
    loader.storeToCache("key1", List.of(Map.of("data", 1)));
    assertEquals(List.of(Map.of("data", 1)), cache.get("key1"));
    // Run: ./mvnw test -Dtest=CachedDataLoaderTest#testStoreToCache
    // ✅ PASS before continuing
}

// Level 2: Now integrate them
public List<Map<String, Object>> loadData(String userId, Map<String, Object> filters) {
    String key = generateCacheKey(userId, filters);  // ✅ Already tested
    Optional<List<Map<String, Object>>> cached = fetchFromCache(key);  // ✅ Already tested
    if (cached.isPresent()) {
        return cached.get();
    }
    List<Map<String, Object>> data = fetchFromDb(userId, filters);  // ✅ Already tested
    storeToCache(key, data);  // ✅ Already tested
    return data;
}

// Test integration
@Test
void testLoadDataIntegration() {
    // All components work individually, now test together
    // Run: ./mvnw test -Dtest=CachedDataLoaderTest#testLoadDataIntegration
}
```

### **Bottom-Up Rules**

1. **Write ONE method at a time**
2. **Test that method IMMEDIATELY** (must pass 100%)
3. **Only after it passes, move to next method**
4. **Never write multiple methods before testing**
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
   ./mvnw test -Dtest=LoaderTest#testCaching
   Result: ✅ PASS

2. Run integration test:
   ./mvnw test -Dtest=IntegrationTest
   Result: ✅ PASS

3. Run manual test with real data:
   java -cp target/classes com.example.LoaderTest
   Result: [{"id": 1, "name": "test"}] ✅ Correct format

4. Check cache is actually used:
   // Add timing logs and verify second call is faster
   Result: First: 150ms, Cached: 2ms ✅ Cache is working

VERIFIED: Problem is actually fixed."
```

### **Verification Checklist (ALL must pass)**

After claiming "it's fixed", you MUST:

- [ ] **Unit test passes** - Run specific test method
- [ ] **Integration test passes** - Run related integration tests  
- [ ] **Manual test works** - Run actual code with real input
- [ ] **Edge cases work** - Test with null, empty, invalid inputs
- [ ] **No regressions** - Run full test suite: `./mvnw test`
- [ ] **Linter clean** - Run `./mvnw checkstyle:check` (or project linter)
- [ ] **Actually test the bug** - If fixing a bug, reproduce the bug first, then verify it's gone

### **Never Say "Fixed" Until:**

```bash
# ALL of these must be true:
✅ ./mvnw test -Dtest=SpecificTest#testMethod  # PASS
✅ ./mvnw test                                  # All PASS
✅ java -cp target/classes com.example.Test     # Works
✅ ./mvnw checkstyle:check                      # No errors
✅ git status                                   # Clean

# Then and ONLY then:
"VERIFIED: Problem is fixed. All tests pass, linter clean, manual test works."
```

---

## 📐 SYNTAX & TYPE CHECKING (Catch Errors Early)

**Problem**: Writing wrong Java syntax leads to compilation errors.

**Solution**: Self-check syntax and types BEFORE running.

### **Pre-Run Syntax Check**

**After writing code, BEFORE testing, check:**

```bash
# 1. Check syntax is valid (compile)
./mvnw compile
# Must complete without errors

# 2. Check imports work
./mvnw compile -DskipTests
# Must not raise compilation errors

# 3. Check types (Java has built-in type checking)
# Compilation will catch type errors automatically

# 4. Check linter
./mvnw checkstyle:check  # or spotbugs:check, pmd:check
# Fix any style issues
```

### **Common Java Syntax Mistakes**

| ❌ Wrong | ✅ Right | Error |
|---------|---------|-------|
| `public void method() {` (missing body) | `public void method() { }` | CompilationError |
| `return x, y;` | `return new Pair<>(x, y);` or separate returns | Use Pair/Tuple |
| Raw types | `List<String>` not `List` | Use generics |
| `if (x == null)` (x is primitive) | `if (x == 0)` | Primitives can't be null |
| `String x = null; x.equals("test")` | `"test".equals(x)` or `Objects.equals(x, "test")` | NPE risk |

### **Type Hints Self-Check**

**Every method MUST have:**
```java
import java.util.List;
import java.util.Map;
import java.util.Optional;

/**
 * Process data.
 * 
 * @param userId User ID (non-null)
 * @param filters Filter conditions (can be empty)
 * @param limit Optional limit (can be null)
 * @return List of processed data maps
 * @throws IllegalArgumentException If userId is null
 */
public List<Map<String, Object>> processData(
    String userId,                      // ✅ Input type
    Map<String, Object> filters,        // ✅ Input type  
    Optional<Integer> limit            // ✅ Optional input
) {
    // Implementation
    return results;  // ✅ Return type
}
```

**Check:**
- ✅ All parameters have clear types
- ✅ Return type is specified
- ✅ Use Optional<T> for parameters that can be null
- ✅ Use specific generic types (List<String>, not raw List)
- ✅ JavaDoc documents all parameters and return value

---

## ⚠️ CRITICAL: Error Prevention

### **Top 2 Errors (90% of Failures)**

| Error | Cause | Fix |
|-------|-------|-----|
| "Error editing file" | Didn't read file first OR old_string mismatch | ALWAYS `read_file()` → Copy EXACT text → Edit |
| "No shell found with ID" | Assumed shell persistence | Use absolute paths OR `cd /path && command` |

### **Environment & Development Rules**

**Build System**:
- ✅ **ALWAYS** use Maven or Gradle wrapper: `./mvnw` or `./gradlew`
- ✅ Compile before running: `./mvnw compile` or `./gradlew compileJava`
- ✅ Run tests with wrapper: `./mvnw test` or `./gradlew test`
- ✅ Install dependencies: `./mvnw install` or `./gradlew build`
- ✅ Check project structure: `pom.xml` (Maven) or `build.gradle` (Gradle) exists

**Example Files**:
- ✅ Create **ONE file per request** in `examples/` folder
- ❌ Do NOT create multiple example files for a single request
- ✅ Use descriptive names: `examples/Phase1DataLoadingDemo.java` (not `Example1.java`, `Example2.java`)
- ✅ If user asks for multiple examples, combine them into one file with clear sections

### **Pre-Action Checklist**

| Before... | Check... |
|-----------|----------|
| Editing file | ✅ Read file first? ✅ Exact old_string? |
| Shell command | ✅ Absolute paths? ✅ `cd` in same command? ✅ Using Maven/Gradle wrapper? |
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
| Unfamiliar library/error | Web search | "Spring Boot error context 2025" |
| Simple change | Do it | Skip TODO |
| 3+ files or steps | Plan | Create TODO list |
| Tried 3x, failed | Research | Examples → web search |
| Debugging | Systematic | Hypothesis → Test → Analyze |

### **When to Web Search**

✅ **USE web search:**
- Unfamiliar library/API: "Spring Boot WebSocket handling 2025"
- Unknown error (after checking codebase): "Hibernate LazyInitializationException"
- Best practices: "Java async patterns 2025"
- After 3 failed attempts

❌ **DON'T web search:**
- Info in codebase → Use `codebase_search` or `grep`
- Standard Java → You know this
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

### **Example: Fix logout bug**

```
1. UNDERSTAND: After logout, session should be null
2. HYPOTHESIZE: logout() doesn't call session.invalidate()
3. IMPLEMENT: Add session.invalidate() in logout()
4. TEST: Still has session data
5. ANALYZE: session is in HttpServletRequest
6. NEW HYPOTHESIS: Need request.getSession(false).invalidate()
7. IMPLEMENT: Change to request.getSession(false).invalidate()
8. TEST: ✅ Works! Session is null
9. VERIFY: Unit tests pass, integration works
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

```java
ERROR: NullPointerException: userId

HYPOTHESIZE:
A (70%): userId not in session map
B (20%): session is null
C (10%): typo in key name

TEST A: System.out.println("Keys: " + session.getAttributeNames());
RESULT: Keys: [username, email]  // userId missing!
→ Hypothesis A CONFIRMED

ROOT CAUSE: Auth doesn't set userId in session
FIX: session.setAttribute("userId", user.getId());
VERIFY: ✅ Works
```

### **Common Patterns**

| Error | Likely Cause | Quick Test |
|-------|--------------|------------|
| `NullPointerException` | Null object | `System.out.println(obj == null)` |
| `IllegalArgumentException` | Invalid parameter | Check parameter validation |
| `IndexOutOfBoundsException` | List too short | `System.out.println(list.size())` |
| Wrong result | Logic error | Print intermediate values |

---

## 🤖 TOOL USAGE & COMMUNICATION

### **Tool Selection**

| Goal | Tool | Pattern |
|------|------|---------|
| Find concept | `codebase_search` | "How does X work?" |
| Find exact text | `grep` | `grep "class Name"` |
| Read code | `read_file` | Read 3-5 files in parallel |
| Unknown API | `web_search` | "Spring Boot WebSocket 2025" |

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
| Singleton | `private static instance` | Use getInstance(), don't instantiate |
| Factory | `create*()`, `build*()` | Use factory method |
| Builder | `new Builder().setX().build()` | Use builder pattern |
| Strategy | Interface with multiple implementations | Use strategy pattern |

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
| ⚫ Critical | Auth/security, data loss | Maximum caution, get review |

---

## 📝 FILE MANAGEMENT

### **Modify vs Create**

| Action | Modify Existing | Create New |
|--------|----------------|------------|
| Fix bug | ✅ | ❌ |
| Add feature to class | ✅ | ❌ |
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
- ❌ Do NOT create multiple example files (`Example1.java`, `Example2.java`, etc.)
- ✅ Use descriptive names: `examples/Phase1DataLoadingDemo.java`
- ✅ If multiple examples needed, combine into one file with clear sections/comments

### **Temporary Files (Rare, Delete Immediately)**

**If absolutely needed:**
- Use prefixes: `Verify*.java`, `Debug*.java`, `Temp*.java`
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
./mvnw compile  # or ./gradlew compileJava

# 2. Tests
./mvnw test  # or ./gradlew test

# 3. Linter
./mvnw checkstyle:check  # or spotbugs:check, pmd:check

# 4. Git status clean
git status  # Only intended files?

# 5. No temp files
find . -name "Verify*" -o -name "Debug*"  # Should be empty
```

### **Done = ALL True**

- ✅ Code compiles + tests pass
- ✅ No linter errors
- ✅ ALL temp files deleted
- ✅ No .md files created (unless asked)
- ✅ `git status` clean - only intended changes
- ✅ Integration verified

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
4. Test the single method ✅
5. Only then integrate with others
```

**Bottom-Up Building** (One method at a time):
```
Write method → Test it → MUST PASS → Next method
(Never write multiple methods before testing)
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
./mvnw compile              # Check syntax/compilation
./mvnw checkstyle:check    # Check style
./mvnw spotbugs:check      # Check bugs (if configured)
```

**Tool Decisions**:
- Find concept → `codebase_search "How does X work in Y?"`
- Find exact text → `grep "class Name" -r src/`
- Unknown API/error → `web_search` (with year)

**Build System**:
- Compile → Always use wrapper: `./mvnw compile` or `./gradlew compileJava`
- Run tests → Use wrapper: `./mvnw test` or `./gradlew test`
- Install deps → Use wrapper: `./mvnw install` or `./gradlew build`

**When Stuck**:
- After 3 fails → Check examples
- After 5 fails → Web search or ask

**Before Done**:
- ✅ Each method tested individually?
- ✅ Integration tested?
- ✅ All tests pass?
- ✅ Linter clean?
- ✅ Git status clean?

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
- **Coding multiple methods without testing** → Stop, test each method individually
- **Saying "fixed" without running tests** → Stop, verify with actual tests
- **Tried same fix 3 times** → Try completely different approach
- **Integration fails after methods work** → Check interface contracts match
- **Compilation errors when running** → Should have checked with `mvnw compile` first
- **Type errors** → Should have checked generics and types match
- **Same test fails differently** → Fix root cause, not symptom
- **Don't understand error** → Use hypothesis-driven debugging
- **Creating temp files** → Edit existing code instead

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
read_file("src/auth/CacheManager.java")  # Read ONLY confirmed file
# Validate: Does this file have the methods I need? YES → Use it, NO → Refine search
```

### **Failure Pattern 2: "Big Bang Integration"**

**Symptom**: Wrote 5 methods, tested together, everything breaks, don't know which method is wrong

**Root cause**: Didn't test each method individually

**Fix**:
```java
// ❌ BAD
public class DataProcessor {
    public DataProcessor() { ... }
    public void load() { ... }
    public void process() { ... }
    public void save() { ... }
    public void validate() { ... }
}

./mvnw test -Dtest=DataProcessorTest  // 10 failures, which method broke?

// ✅ GOOD
public class DataProcessor {
    public void load() { ... }
}

./mvnw test -Dtest=DataProcessorTest#testLoad  // ✅ PASS - Continue

    public void process() { ... }
    
./mvnw test -Dtest=DataProcessorTest#testProcess  // ✅ PASS - Continue

    public void save() { ... }
    
./mvnw test -Dtest=DataProcessorTest#testSave  // ✅ PASS - Continue

// Now all individual methods work, integration will likely work
./mvnw test -Dtest=DataProcessorTest#testIntegration  // ✅ PASS
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

1. Reproduce bug: java -cp target/classes com.example.Test
   Before fix: Cache not cleared ❌
   
2. Apply fix: cache.remove(key) after update

3. Test again: java -cp target/classes com.example.Test
   After fix: Cache cleared ✅
   
4. Run unit test: ./mvnw test -Dtest=CacheTest#testInvalidation
   Result: PASSED ✅
   
5. Run integration: ./mvnw test -Dtest=IntegrationTest
   Result: PASSED ✅

VERIFIED: Bug is fixed."
```

### **Failure Pattern 4: "Interface Mismatch"**

**Symptom**: Individual methods work, but fail when integrated

**Root cause**: Interfaces don't match - method A returns Map but method B expects List

**Fix**:
```java
// ❌ BAD - Interface mismatch
public Map<String, Object> fetchData(String userId) {  // Missing null handling!
    Map<String, Object> data = db.query(userId);
    return data;  // Sometimes returns null, sometimes map
}

public List<Map<String, Object>> processData(Map<String, Object> data) {  // Missing null check!
    return data.values().stream().collect(Collectors.toList());  // Crashes if data is null!
}

// ✅ GOOD - Clear interfaces
/**
 * Fetch user data.
 * 
 * @param userId User ID (non-null)
 * @return Map with user data. Never returns null.
 *         Returns empty map {} if user not found.
 */
public Map<String, Object> fetchData(String userId) {
    Map<String, Object> data = db.query(userId);
    return data != null ? data : Map.of();  // Always returns map
}

/**
 * Process data map into list.
 * 
 * @param data User data map (can be empty, never null)
 * @return List of processed items. Empty list if no items.
 */
public List<Map<String, Object>> processData(Map<String, Object> data) {
    if (data == null || data.isEmpty()) {
        return List.of();
    }
    return data.values().stream()
        .map(v -> Map.of("value", v))
        .collect(Collectors.toList());
}

// Now interfaces match: Map → Map, Map → List
```

### **Failure Pattern 5: "Compilation Before Test"**

**Symptom**: Code has Java compilation errors when running

**Root cause**: Didn't check compilation before testing

**Fix**:
```bash
# ✅ GOOD - Check compilation BEFORE running tests

# 1. Check compilation
./mvnw compile
# No errors = syntax OK

# 2. Check imports
./mvnw compile -DskipTests
# No error = imports OK

# 3. Check types (Java has built-in type checking)
# Compilation will catch type errors automatically

# 4. Check linter
./mvnw checkstyle:check
# No error = style OK

# 5. NOW run tests
./mvnw test -Dtest=MyTest
```

### **How to Avoid These Failures**

| Failure Pattern | Prevention |
|----------------|------------|
| Shotgun Search | Specific questions + grep + validate 3-5 files max |
| Big Bang Integration | Test each method individually before integration |
| Assumption Completion | Verify with actual tests (unit + integration + manual) |
| Interface Mismatch | Define interfaces first with clear types and contracts |
| Compilation Errors | Check with `mvnw compile` and linter before running |

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
3. ⚠️ **Bottom-up build** - One method → Test it → PASS → Next method
4. ⚠️ **Never assume fixed** - Verify with actual tests (unit + integration + manual)
5. ⚠️ **Compilation check first** - `./mvnw compile` before running
6. ⚠️ **Test immediately** - After EVERY method, not after coding everything
7. ⚠️ **Read before edit** - Prevents 90% of errors
8. ⚠️ **Iterate systematically** - Different approaches, not same retry
9. ⚠️ **Root cause debugging** - Ask "Why?" 5 times, fix cause not symptom
10. ⚠️ **Clean workspace** - git status clean, all tests pass, linter clean
11. ⚠️ **Use build wrapper** - Always use `./mvnw` or `./gradlew` for builds and tests
12. ⚠️ **One example file** - Create ONE file per request in `examples/`, not multiple files

---

## 📚 PROJECT SPECIFICS

### **Java Project Structure**

```
project/
├── src/
│   ├── main/java/          # Main code
│   │   ├── com/example/
│   │   │   ├── core/       # Core functionality
│   │   │   └── util/       # Utilities
│   └── test/java/          # Test suites
├── examples/                # Usage examples
├── pom.xml                 # Maven config (or build.gradle)
└── .mvn/ or gradle/        # Build wrapper
```

### **Code Style**

- **Formatter**: Google Java Format or Checkstyle
- **Linter**: Checkstyle, SpotBugs, PMD
- **Type check**: Built-in (Java compiler)
- **Testing**: JUnit 5

### **Error Patterns**

| Error | Cause | Solution |
|-------|-------|----------|
| Compilation error | Missing import or class | Check imports, classpath |
| NullPointerException | Null object | Add null checks, use Optional |
| IllegalArgumentException | Invalid parameter | Validate parameters |
| ClassNotFoundException | Missing dependency | Check pom.xml/build.gradle |

---

**End of Guide. Use Phase 0 → Execute → Iterate → Verify → Clean. Good luck!**

