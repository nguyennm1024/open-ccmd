# Claude Code Assistant Guide - Expert Performance (C# Edition)

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
read_file("src/Data/Loader.cs")  # Confirmed relevant

Step 4: Validate you have the right files
- Check using statements, class names, method signatures
- If wrong, refine search, don't just read more files
```

**Search quality checklist:**
- ✅ Search query is a full question, not just keywords
- ✅ Include context: "in the authentication system" not just "auth"
- ✅ Use grep to confirm exact class/method names before reading
- ✅ Read 3-5 files max initially - if you need more, your search was too broad
- ✅ Validate each file is relevant before moving on

### **Intelligence Gathering (Do in Parallel)**

```csharp
// Example: "Add caching to data loader"

// Step 1: Focused search (not broad)
codebase_search "How does the DataLoader load data from database?"
grep "class DataLoader" -r src/  // Get exact file location

// Step 2: Parallel reads (ONLY confirmed relevant files)
- read_file("src/Data/Loader.cs")       // Main implementation
- read_file("src/Cache/CacheManager.cs") // IF grep confirms cache exists
- read_file("tests/Data/LoaderTests.cs") // Tests for expected behavior

// Step 3: Validate relevance
// Check: Does Loader.cs have the methods I need to modify?
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

```csharp
// DON'T start coding the implementation yet!
// FIRST, define what the method MUST do:

/// <summary>
/// Load user data with filters.
/// </summary>
/// <param name="userId">User ID (non-null, non-empty string)</param>
/// <param name="filters">Filter conditions (can be empty dictionary)</param>
/// <returns>List of data dictionaries, empty list if no data</returns>
/// <exception cref="ArgumentNullException">If userId is null</exception>
/// <exception cref="ArgumentException">If userId is empty</exception>
/// <exception cref="DatabaseException">If database connection fails</exception>
Task<List<Dictionary<string, object>>> LoadDataAsync(
    string userId, 
    Dictionary<string, object> filters
);
// Define interface first, implement second
```

**Step 2: Write the test BEFORE implementation**

```csharp
[Fact]
public async Task LoadData_WithValidInput_ReturnsList()
{
    // Test the interface contract
    var result = await loader.LoadDataAsync("user123", 
        new Dictionary<string, object> { ["status"] = "active" });
    
    Assert.NotNull(result);  // Returns list, not null
    Assert.IsType<List<Dictionary<string, object>>>(result);  // List of dictionaries
    
    // Test edge cases
    result = await loader.LoadDataAsync("user123", new Dictionary<string, object>());  // Empty filters
    Assert.NotNull(result);  // Still returns list
    
    // Test error cases
    await Assert.ThrowsAsync<ArgumentException>(
        () => loader.LoadDataAsync("", new Dictionary<string, object>()));  // Empty userId throws
}
```

**Step 3: Implement to satisfy the interface**

```csharp
public async Task<List<Dictionary<string, object>>> LoadDataAsync(
    string userId, 
    Dictionary<string, object> filters)
{
    if (string.IsNullOrEmpty(userId))
    {
        throw new ArgumentException("userId cannot be null or empty", nameof(userId));
    }
    
    // Implementation...
    return results;  // MUST return list, never null
}
```

**Step 4: Test the single method (bottom-up)**

```bash
dotnet test --filter "FullyQualifiedName~LoaderTests.LoadData_WithValidInput_ReturnsList"
# MUST pass before moving to integration
```

### **Interface Contract Checklist**

Before writing implementation, define:

- ✅ **Input types**: Exact parameter types (string, int, Dictionary, List, nullable?)
- ✅ **Output type**: What does it return? (List? Dictionary? null? Never null?)
- ✅ **Error cases**: What exceptions can it throw? When?
- ✅ **Side effects**: Does it modify inputs? Change state? Call external APIs?
- ✅ **Preconditions**: What MUST be true before calling? (non-null? non-empty? valid format?)
- ✅ **Postconditions**: What WILL be true after calling? (returns list, never null?)
- ✅ **Async**: Is this method async? Should it return Task/Task<T>?

### **Common Interface Mistakes**

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Method sometimes returns null, sometimes list | Always return same type (empty list, not null) |
| Method modifies input dictionary | Return new dictionary, don't modify input |
| Error cases return null instead of throwing | Throw specific exceptions for errors |
| No XML documentation | Add complete XML docs with <param>, <returns>, <exception> |
| Unclear what null means | Use nullable reference types (string?) if null is valid, document meaning |
| Missing async/await | Use async Task<T> for async operations, not void |

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
```csharp
// Code everything at once
public class CachedDataLoader
{
    public CachedDataLoader() { ... }
    public Task<List<Dictionary<string, object>>> LoadDataAsync() { ... }
    private Task<Dictionary<string, object>> FetchFromDbAsync() { ... }
    private Task<List<Dictionary<string, object>>> FetchFromCacheAsync() { ... }
    private Task StoreToCacheAsync() { ... }
    public Task InvalidateCacheAsync() { ... }
}

// Test everything together
dotnet test --filter "FullyQualifiedName~CachedDataLoaderTests"
// 10 failures! Which method is broken?
```

**✅ RIGHT approach:**

```csharp
// Level 1: Build and test ONE method at a time

// Step 1a: Implement cache key generation
private string GenerateCacheKey(string userId, Dictionary<string, object> filters)
{
    return $"data:{userId}:{filters.GetHashCode()}";
}

// Step 1b: Test it IMMEDIATELY
[Fact]
public void GenerateCacheKey_SameInputs_ReturnsSameKey()
{
    var loader = new CachedDataLoader();
    var filters1 = new Dictionary<string, object> { ["a"] = 1 };
    var filters2 = new Dictionary<string, object> { ["a"] = 1 };
    var key1 = loader.GenerateCacheKey("u1", filters1);
    var key2 = loader.GenerateCacheKey("u1", filters2);
    Assert.Equal(key1, key2);  // Same inputs = same key
    // Run: dotnet test --filter "FullyQualifiedName~GenerateCacheKey_SameInputs_ReturnsSameKey"
    // ✅ PASS before continuing
}

// Step 2a: Implement cache fetch
private Task<List<Dictionary<string, object>>?> FetchFromCacheAsync(string key)
{
    return Task.FromResult(cache.TryGetValue(key, out var value) ? value : null);
}

// Step 2b: Test it IMMEDIATELY  
[Fact]
public async Task FetchFromCache_MissingKey_ReturnsNull()
{
    var loader = new CachedDataLoader();
    var result = await loader.FetchFromCacheAsync("missing");
    Assert.Null(result);
    
    cache["key1"] = new List<Dictionary<string, object>> { new() { ["data"] = 1 } };
    result = await loader.FetchFromCacheAsync("key1");
    Assert.NotNull(result);
    Assert.Single(result);
    // Run: dotnet test --filter "FullyQualifiedName~FetchFromCache_MissingKey_ReturnsNull"
    // ✅ PASS before continuing
}

// Step 3a: Implement cache store
private Task StoreToCacheAsync(string key, List<Dictionary<string, object>> data)
{
    cache[key] = data;
    return Task.CompletedTask;
}

// Step 3b: Test it IMMEDIATELY
[Fact]
public async Task StoreToCache_StoresData()
{
    var loader = new CachedDataLoader();
    var data = new List<Dictionary<string, object>> { new() { ["data"] = 1 } };
    await loader.StoreToCacheAsync("key1", data);
    Assert.True(cache.ContainsKey("key1"));
    // Run: dotnet test --filter "FullyQualifiedName~StoreToCache_StoresData"
    // ✅ PASS before continuing
}

// Level 2: Now integrate them
public async Task<List<Dictionary<string, object>>> LoadDataAsync(
    string userId, 
    Dictionary<string, object> filters)
{
    string key = GenerateCacheKey(userId, filters);  // ✅ Already tested
    var cached = await FetchFromCacheAsync(key);  // ✅ Already tested
    if (cached != null)
    {
        return cached;
    }
    var data = await FetchFromDbAsync(userId, filters);  // ✅ Already tested
    await StoreToCacheAsync(key, data);  // ✅ Already tested
    return data;
}

// Test integration
[Fact]
public async Task LoadData_Integration_Works()
{
    // All components work individually, now test together
    // Run: dotnet test --filter "FullyQualifiedName~LoadData_Integration_Works"
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
   dotnet test --filter "FullyQualifiedName~LoaderTests.LoadData_Caching"
   Result: ✅ PASS

2. Run integration test:
   dotnet test --filter "FullyQualifiedName~IntegrationTests"
   Result: ✅ PASS

3. Run manual test with real data:
   dotnet run --project src/ExampleApp
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
- [ ] **No regressions** - Run full test suite: `dotnet test`
- [ ] **Linter clean** - Run `dotnet format --verify` (or project linter)
- [ ] **Actually test the bug** - If fixing a bug, reproduce the bug first, then verify it's gone

### **Never Say "Fixed" Until:**

```bash
# ALL of these must be true:
✅ dotnet test --filter "FullyQualifiedName~SpecificTest"  # PASS
✅ dotnet test                                  # All PASS
✅ dotnet run --project src/ExampleApp          # Works
✅ dotnet format --verify                      # No errors
✅ git status                                   # Clean

# Then and ONLY then:
"VERIFIED: Problem is fixed. All tests pass, linter clean, manual test works."
```

---

## 📐 SYNTAX & TYPE CHECKING (Catch Errors Early)

**Problem**: Writing wrong C# syntax leads to compilation errors.

**Solution**: Self-check syntax and types BEFORE running.

### **Pre-Run Syntax Check**

**After writing code, BEFORE testing, check:**

```bash
# 1. Check syntax is valid (compile)
dotnet build
# Must complete without errors

# 2. Check imports work
dotnet build --no-restore
# Must not raise compilation errors

# 3. Check types (C# has built-in type checking)
# Compilation will catch type errors automatically

# 4. Check linter/formatting
dotnet format --verify
# Fix any style issues
```

### **Common C# Syntax Mistakes**

| ❌ Wrong | ✅ Right | Error |
|---------|---------|-------|
| `public void Method() {` (missing body) | `public void Method() { }` | CompilationError |
| `return x, y;` | `return (x, y);` or `new Tuple<T1, T2>(x, y)` | Use tuples |
| Raw types | `List<string>` not `List` | Use generics |
| `if (x == null)` (x is value type) | `if (x == 0)` or `if (x is null)` | Value types can't be null (unless nullable) |
| `string x = null; x.Equals("test")` | `"test".Equals(x)` or `x?.Equals("test")` | NullReferenceException risk |
| Missing await | `await Task.Delay(100);` not `Task.Delay(100);` | Missing await warning |

### **Type Hints Self-Check**

**Every method MUST have:**
```csharp
using System.Collections.Generic;
using System.Threading.Tasks;

/// <summary>
/// Process data.
/// </summary>
/// <param name="userId">User ID (non-null)</param>
/// <param name="filters">Filter conditions (can be empty)</param>
/// <param name="limit">Optional limit (can be null)</param>
/// <returns>List of processed data dictionaries</returns>
/// <exception cref="ArgumentNullException">If userId is null</exception>
public async Task<List<Dictionary<string, object>>> ProcessDataAsync(
    string userId,                      // ✅ Input type
    Dictionary<string, object> filters,        // ✅ Input type  
    int? limit = null                   // ✅ Nullable input
)
{
    // Implementation
    return results;  // ✅ Return type
}
```

**Check:**
- ✅ All parameters have clear types
- ✅ Return type is specified
- ✅ Use nullable types (int?, string?) for parameters that can be null
- ✅ Use specific generic types (List<string>, not raw List)
- ✅ XML documentation documents all parameters and return value
- ✅ Async methods return Task or Task<T>

---

## ⚠️ CRITICAL: Error Prevention

### **Top 2 Errors (90% of Failures)**

| Error | Cause | Fix |
|-------|-------|-----|
| "Error editing file" | Didn't read file first OR old_string mismatch | ALWAYS `read_file()` → Copy EXACT text → Edit |
| "No shell found with ID" | Assumed shell persistence | Use absolute paths OR `cd /path && command` |

### **Environment & Development Rules**

**Build System**:
- ✅ **ALWAYS** use dotnet CLI: `dotnet build`, `dotnet test`, `dotnet run`
- ✅ Build before running: `dotnet build`
- ✅ Run tests with dotnet: `dotnet test`
- ✅ Restore dependencies: `dotnet restore` (or auto-restore)
- ✅ Check project structure: `.csproj` file exists

**Example Files**:
- ✅ Create **ONE file per request** in `examples/` folder
- ❌ Do NOT create multiple example files for a single request
- ✅ Use descriptive names: `examples/Phase1DataLoadingDemo.cs` (not `Example1.cs`, `Example2.cs`)
- ✅ If user asks for multiple examples, combine them into one file with clear sections

### **Pre-Action Checklist**

| Before... | Check... |
|-----------|----------|
| Editing file | ✅ Read file first? ✅ Exact old_string? |
| Shell command | ✅ Absolute paths? ✅ `cd` in same command? ✅ Using dotnet CLI? |
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
| Unfamiliar library/error | Web search | ".NET 8 error context 2025" |
| Simple change | Do it | Skip TODO |
| 3+ files or steps | Plan | Create TODO list |
| Tried 3x, failed | Research | Examples → web search |
| Debugging | Systematic | Hypothesis → Test → Analyze |

### **When to Web Search**

✅ **USE web search:**
- Unfamiliar library/API: "ASP.NET Core WebSocket handling 2025"
- Unknown error (after checking codebase): "Entity Framework NullReferenceException"
- Best practices: "C# async patterns 2025"
- After 3 failed attempts

❌ **DON'T web search:**
- Info in codebase → Use `codebase_search` or `grep`
- Standard C# → You know this
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
2. HYPOTHESIZE: Logout() doesn't call session.Clear()
3. IMPLEMENT: Add session.Clear() in Logout()
4. TEST: Still has session data
5. ANALYZE: session is in HttpContext
6. NEW HYPOTHESIS: Need HttpContext.Session.Clear()
7. IMPLEMENT: Change to HttpContext.Session.Clear()
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

```csharp
ERROR: NullReferenceException: userId

HYPOTHESIZE:
A (70%): userId not in session dictionary
B (20%): session is null
C (10%): typo in key name

TEST A: Console.WriteLine($"Keys: {string.Join(", ", session.Keys)}");
RESULT: Keys: username, email  // userId missing!
→ Hypothesis A CONFIRMED

ROOT CAUSE: Auth doesn't set userId in session
FIX: session["userId"] = user.Id;
VERIFY: ✅ Works
```

### **Common Patterns**

| Error | Likely Cause | Quick Test |
|-------|--------------|------------|
| `NullReferenceException` | Null object | `Console.WriteLine(obj == null)` |
| `ArgumentNullException` | Invalid parameter | Check parameter validation |
| `IndexOutOfRangeException` | List too short | `Console.WriteLine(list.Count)` |
| `InvalidOperationException` | Wrong state | Check object state |
| Wrong result | Logic error | Print intermediate values |
| `TaskCanceledException` | Async timeout | Check cancellation tokens |

---

## 🤖 TOOL USAGE & COMMUNICATION

### **Tool Selection**

| Goal | Tool | Pattern |
|------|------|---------|
| Find concept | `codebase_search` | "How does X work?" |
| Find exact text | `grep` | `grep "class Name"` |
| Read code | `read_file` | Read 3-5 files in parallel |
| Unknown API | `web_search` | "ASP.NET Core WebSocket 2025" |

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
| Singleton | `private static instance` | Use GetInstance(), don't instantiate |
| Factory | `Create*()`, `Build*()` | Use factory method |
| Builder | `new Builder().SetX().Build()` | Use builder pattern |
| Strategy | Interface with multiple implementations | Use strategy pattern |
| Dependency Injection | Constructor parameters, IServiceCollection | Use DI container |

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
- ❌ Do NOT create multiple example files (`Example1.cs`, `Example2.cs`, etc.)
- ✅ Use descriptive names: `examples/Phase1DataLoadingDemo.cs`
- ✅ If multiple examples needed, combine into one file with clear sections/comments

### **Temporary Files (Rare, Delete Immediately)**

**If absolutely needed:**
- Use prefixes: `Verify*.cs`, `Debug*.cs`, `Temp*.cs`
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
# 1. Build
dotnet build

# 2. Tests
dotnet test

# 3. Linter/Formatting
dotnet format --verify

# 4. Git status clean
git status  # Only intended files?

# 5. No temp files
find . -name "Verify*" -o -name "Debug*"  # Should be empty
```

### **Done = ALL True**

- ✅ Code builds + tests pass
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
dotnet build              # Check syntax/compilation
dotnet format --verify    # Check style
dotnet analyze           # Check code analysis (if configured)
```

**Tool Decisions**:
- Find concept → `codebase_search "How does X work in Y?"`
- Find exact text → `grep "class Name" -r src/`
- Unknown API/error → `web_search` (with year)

**Build System**:
- Build → Always use dotnet CLI: `dotnet build`
- Run tests → Use dotnet CLI: `dotnet test`
- Restore deps → Use dotnet CLI: `dotnet restore`

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
- **Compilation errors when running** → Should have checked with `dotnet build` first
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
read_file("src/Auth/CacheManager.cs")  # Read ONLY confirmed file
# Validate: Does this file have the methods I need? YES → Use it, NO → Refine search
```

### **Failure Pattern 2: "Big Bang Integration"**

**Symptom**: Wrote 5 methods, tested together, everything breaks, don't know which method is wrong

**Root cause**: Didn't test each method individually

**Fix**:
```csharp
// ❌ BAD
public class DataProcessor
{
    public DataProcessor() { ... }
    public Task LoadAsync() { ... }
    public Task ProcessAsync() { ... }
    public Task SaveAsync() { ... }
    public Task ValidateAsync() { ... }
}

dotnet test --filter "FullyQualifiedName~DataProcessorTests"  // 10 failures, which method broke?

// ✅ GOOD
public class DataProcessor
{
    public Task LoadAsync() { ... }
}

dotnet test --filter "FullyQualifiedName~DataProcessorTests.LoadAsync"  // ✅ PASS - Continue

    public Task ProcessAsync() { ... }
    
dotnet test --filter "FullyQualifiedName~DataProcessorTests.ProcessAsync"  // ✅ PASS - Continue

    public Task SaveAsync() { ... }
    
dotnet test --filter "FullyQualifiedName~DataProcessorTests.SaveAsync"  // ✅ PASS - Continue

// Now all individual methods work, integration will likely work
dotnet test --filter "FullyQualifiedName~DataProcessorTests.Integration"  // ✅ PASS
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

1. Reproduce bug: dotnet run --project src/ExampleApp
   Before fix: Cache not cleared ❌
   
2. Apply fix: cache.Remove(key) after update

3. Test again: dotnet run --project src/ExampleApp
   After fix: Cache cleared ✅
   
4. Run unit test: dotnet test --filter "FullyQualifiedName~CacheTests.Invalidation"
   Result: PASSED ✅
   
5. Run integration: dotnet test --filter "FullyQualifiedName~IntegrationTests"
   Result: PASSED ✅

VERIFIED: Bug is fixed."
```

### **Failure Pattern 4: "Interface Mismatch"**

**Symptom**: Individual methods work, but fail when integrated

**Root cause**: Interfaces don't match - method A returns Dictionary but method B expects List

**Fix**:
```csharp
// ❌ BAD - Interface mismatch
public async Task<Dictionary<string, object>?> FetchDataAsync(string userId)  // Missing null handling!
{
    var data = await db.QueryAsync(userId);
    return data;  // Sometimes returns null, sometimes dictionary
}

public async Task<List<Dictionary<string, object>>> ProcessDataAsync(Dictionary<string, object>? data)  // Missing null check!
{
    return data!.Values.Select(v => new Dictionary<string, object> { ["value"] = v }).ToList();  // Crashes if data is null!
}

// ✅ GOOD - Clear interfaces
/// <summary>
/// Fetch user data.
/// </summary>
/// <param name="userId">User ID (non-null)</param>
/// <returns>Dictionary with user data. Never returns null.
///         Returns empty dictionary {} if user not found.</returns>
public async Task<Dictionary<string, object>> FetchDataAsync(string userId)
{
    var data = await db.QueryAsync(userId);
    return data ?? new Dictionary<string, object>();  // Always returns dictionary
}

/// <summary>
/// Process data dictionary into list.
/// </summary>
/// <param name="data">User data dictionary (can be empty, never null)</param>
/// <returns>List of processed items. Empty list if no items.</returns>
public async Task<List<Dictionary<string, object>>> ProcessDataAsync(Dictionary<string, object> data)
{
    if (data == null || data.Count == 0)
    {
        return new List<Dictionary<string, object>>();
    }
    return data.Values
        .Select(v => new Dictionary<string, object> { ["value"] = v })
        .ToList();
}

// Now interfaces match: Dictionary → Dictionary, Dictionary → List
```

### **Failure Pattern 5: "Compilation Before Test"**

**Symptom**: Code has C# compilation errors when running

**Root cause**: Didn't check compilation before testing

**Fix**:
```bash
# ✅ GOOD - Check compilation BEFORE running tests

# 1. Check compilation
dotnet build
# No errors = syntax OK

# 2. Check imports
dotnet build --no-restore
# No error = imports OK

# 3. Check types (C# has built-in type checking)
# Compilation will catch type errors automatically

# 4. Check linter/formatting
dotnet format --verify
# No error = style OK

# 5. NOW run tests
dotnet test --filter "FullyQualifiedName~MyTest"
```

### **How to Avoid These Failures**

| Failure Pattern | Prevention |
|----------------|------------|
| Shotgun Search | Specific questions + grep + validate 3-5 files max |
| Big Bang Integration | Test each method individually before integration |
| Assumption Completion | Verify with actual tests (unit + integration + manual) |
| Interface Mismatch | Define interfaces first with clear types and contracts |
| Compilation Errors | Check with `dotnet build` and linter before running |

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
5. ⚠️ **Compilation check first** - `dotnet build` before running
6. ⚠️ **Test immediately** - After EVERY method, not after coding everything
7. ⚠️ **Read before edit** - Prevents 90% of errors
8. ⚠️ **Iterate systematically** - Different approaches, not same retry
9. ⚠️ **Root cause debugging** - Ask "Why?" 5 times, fix cause not symptom
10. ⚠️ **Clean workspace** - git status clean, all tests pass, linter clean
11. ⚠️ **Use dotnet CLI** - Always use `dotnet` for builds and tests
12. ⚠️ **One example file** - Create ONE file per request in `examples/`, not multiple files

---

## 📚 PROJECT SPECIFICS

### **C# Project Structure**

```
project/
├── src/
│   ├── ProjectName/          # Main code
│   │   ├── Core/            # Core functionality
│   │   ├── Data/            # Data access
│   │   └── Services/        # Business logic
│   └── ProjectName.Tests/   # Test suites
├── examples/                 # Usage examples
├── ProjectName.csproj       # Project file
└── ProjectName.sln          # Solution file (optional)
```

### **Code Style**

- **Formatter**: `dotnet format` (built-in)
- **Linter**: Roslyn analyzers, StyleCop, SonarAnalyzer
- **Type check**: Built-in (C# compiler)
- **Testing**: xUnit, NUnit, or MSTest

### **Build System**

- **Build**: `dotnet build`
- **Test**: `dotnet test`
- **Run**: `dotnet run --project src/ProjectName`
- **Format**: `dotnet format`
- **Restore**: `dotnet restore` (auto-restore on build)

### **C#-Specific Patterns**

**Async/Await**:
```csharp
// ✅ GOOD
public async Task<List<string>> GetDataAsync()
{
    var data = await httpClient.GetAsync(url);
    return await data.Content.ReadAsAsync<List<string>>();
}

// ❌ BAD
public List<string> GetData()  // Blocking
{
    return httpClient.GetAsync(url).Result;  // Deadlock risk
}
```

**LINQ**:
```csharp
// ✅ GOOD
var activeUsers = users
    .Where(u => u.IsActive)
    .Select(u => u.Name)
    .ToList();

// ❌ BAD
var activeUsers = new List<string>();
foreach (var user in users)
{
    if (user.IsActive)
    {
        activeUsers.Add(user.Name);
    }
}
```

**Nullable Reference Types**:
```csharp
// ✅ GOOD - Explicit nullability
public string? GetOptionalValue()  // Can return null
{
    return condition ? "value" : null;
}

public string GetRequiredValue()  // Never returns null
{
    return "value";
}
```

**Properties**:
```csharp
// ✅ GOOD
public string Name { get; set; }
public string Name { get; private set; }
public string Name { get; init; }  // Init-only

// ❌ BAD - Public fields
public string Name;  // Use properties instead
```

### **Error Patterns**

| Error | Cause | Solution |
|-------|-------|----------|
| Compilation error | Missing using or class | Check using statements, NuGet packages |
| NullReferenceException | Null object | Add null checks, use nullable reference types |
| ArgumentNullException | Invalid parameter | Validate parameters with ArgumentNullException.ThrowIfNull |
| FileNotFoundException | Missing file | Check file paths, use Path.Combine |
| TaskCanceledException | Async timeout | Check cancellation tokens, timeouts |
| InvalidOperationException | Wrong state | Check object state before operations |

---

**End of Guide. Use Phase 0 → Execute → Iterate → Verify → Clean. Good luck!**

