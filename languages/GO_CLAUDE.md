# Claude Code Assistant Guide - Expert Performance (Go Edition)

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

Step 2: If too many results, grep for exact struct/interface/function names
grep "type.*Loader\|func.*Load" -r .  // Find exact loader types/functions

Step 3: Read ONLY the files that match both searches
read_file("data/loader.go")  // Confirmed relevant

Step 4: Validate you have the right files
- Check imports, struct names, function signatures
- If wrong, refine search, don't just read more files
```

**Search quality checklist:**
- ✅ Search query is a full question, not just keywords
- ✅ Include context: "in the authentication system" not just "auth"
- ✅ Use grep to confirm exact struct/interface/function names before reading
- ✅ Read 3-5 files max initially - if you need more, your search was too broad
- ✅ Validate each file is relevant before moving on

### **Intelligence Gathering (Do in Parallel)**

```go
// Example: "Add caching to data loader"

// Step 1: Focused search (not broad)
codebase_search "How does the DataLoader load data from database?"
grep "type.*DataLoader\|func.*LoadData" -r .  // Get exact file location

// Step 2: Parallel reads (ONLY confirmed relevant files)
- read_file("data/loader.go")       // Main implementation
- read_file("cache/manager.go")      // IF grep confirms cache exists
- read_file("data/loader_test.go")   // Tests for expected behavior
- read_file("go.mod")                // Dependencies

// Step 3: Validate relevance
// Check: Does loader.go have the functions I need to modify?
// Check: Are imports correct? Package structure?
// If NO → You searched wrong, refine and try again
```

**Mental model checklist:**
- ✅ Entry points (how is this called?)
- ✅ Data flow (what data moves where?)
- ✅ Dependencies (what does this use?)
- ✅ Side effects (what else changes?)
- ✅ Error paths (what can go wrong?)
- ✅ **Interfaces** (what are the function signatures and contracts?)
- ✅ **Error handling** (how are errors returned and handled?)
- ✅ **Concurrency** (goroutines, channels, mutexes?)

---

## 🔌 INTERFACE-FIRST DESIGN (Critical for Integration)

**Problem**: Coding everything then testing leads to interface mismatches and integration bugs.

**Solution**: Define interfaces FIRST, implement SECOND, test EACH level.

### **The Interface-First Process**

**Step 1: Define the interface (function signature + contract)**

```go
// DON'T start coding the implementation yet!
// FIRST, define what the function MUST do:

// In package file (data/loader.go):
package data

import (
    "context"
    "errors"
)

// DataItem represents a single data item
type DataItem struct {
    ID   string
    Name string
}

// LoadData loads user data with filters.
//
// Parameters:
//   - ctx: Context for cancellation and timeout
//   - userID: User ID (non-empty string)
//   - filters: Filter conditions (can be nil for no filters)
//
// Returns:
//   - []DataItem: Slice of data items, empty slice if no data
//   - error: Error if userID is empty or database fails
//
// Errors:
//   - ErrInvalidInput: If userID is empty
//   - ErrDatabaseError: If database connection fails
func LoadData(ctx context.Context, userID string, filters map[string]string) ([]DataItem, error) {
    // Define interface first, implement second
    panic("not implemented")
}
```

**Step 2: Write the test BEFORE implementation**

```go
// In test file (data/loader_test.go):
package data

import (
    "context"
    "testing"
)

func TestLoadData_Basic(t *testing.T) {
    ctx := context.Background()
    filters := map[string]string{
        "status": "active",
    }
    
    result, err := LoadData(ctx, "user123", filters)
    
    if err != nil {
        t.Fatalf("LoadData() error = %v, want nil", err)
    }
    if result == nil {
        t.Fatal("LoadData() returned nil slice, want non-nil")
    }
    // Returns slice (may be empty)
    
    // Test edge cases
    result, err = LoadData(ctx, "user123", nil)  // nil filters
    if err != nil {
        t.Fatalf("LoadData() with nil filters error = %v, want nil", err)
    }
    
    // Test error cases
    result, err = LoadData(ctx, "", nil)  // Empty userID
    if err == nil {
        t.Fatal("LoadData() with empty userID error = nil, want error")
    }
    if !errors.Is(err, ErrInvalidInput) {
        t.Fatalf("LoadData() error = %v, want ErrInvalidInput", err)
    }
}
```

**Step 3: Implement to satisfy the interface**

```go
// In implementation file (data/loader.go):
package data

import (
    "context"
    "errors"
    "fmt"
)

var (
    ErrInvalidInput  = errors.New("invalid input")
    ErrDatabaseError = errors.New("database error")
)

func LoadData(ctx context.Context, userID string, filters map[string]string) ([]DataItem, error) {
    // Validate inputs
    if userID == "" {
        return nil, fmt.Errorf("%w: userID cannot be empty", ErrInvalidInput)
    }
    
    // Check context cancellation
    select {
    case <-ctx.Done():
        return nil, ctx.Err()
    default:
    }
    
    // Implementation...
    return results, nil  // MUST return ([]DataItem, error), never panic
}
```

**Step 4: Test the single function (bottom-up)**

```bash
go test -v -run TestLoadData_Basic ./data
# MUST pass before moving to integration
```

### **Interface Contract Checklist**

Before writing implementation, define:

- ✅ **Input types**: Exact parameter types (string, int, []T, map[K]V, interface{}, context.Context?)
- ✅ **Output type**: What does it return? (T, error)? (T, bool)? Multiple return values?
- ✅ **Error cases**: What errors can it return? When? (error interface, custom error types?)
- ✅ **Side effects**: Does it modify inputs? Change state? Call external APIs?
- ✅ **Preconditions**: What MUST be true before calling? (non-empty? valid format? non-nil?)
- ✅ **Postconditions**: What WILL be true after calling? (returns slice, never nil?)
- ✅ **Context**: Does it accept context.Context? For cancellation/timeout?
- ✅ **Concurrency**: Is it safe for concurrent use? Document if not

### **Common Interface Mistakes**

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Function sometimes returns nil slice, sometimes error | Always return same pattern (empty slice []T{}, or always error on failure) |
| Function modifies input slice/map | Document side effects, or make defensive copies |
| Error cases return nil error | Always return error for failures, use errors.New or fmt.Errorf |
| No documentation | Add complete godoc comments |
| Unclear what nil means | Use pointer *T if nil is valid, document meaning |
| Missing error wrapping | Use fmt.Errorf with %w to wrap errors |
| Missing context | Accept context.Context for cancellation/timeout |
| No input validation | Validate all inputs, return error if invalid |

---

## 🏗️ BOTTOM-UP DEVELOPMENT (Build Solid Foundation)

**Problem**: Coding everything at once leads to many bugs.

**Solution**: Build and verify each level before moving up.

### **The Bottom-Up Process**

```
Level 1: Individual functions (test each)
    ↓ (all pass)
Level 2: Package integration (test together)
    ↓ (all pass)
Level 3: Component integration (test full flow)
    ↓ (all pass)
Level 4: System integration (test end-to-end)
```

### **Example: Add caching to data loader**

**❌ WRONG approach:**
```go
// Code everything at once
// data/cached_loader.go
type CachedDataLoader struct {
    cache map[string][]DataItem
    mu    sync.RWMutex
}

func NewCachedDataLoader() *CachedDataLoader { ... }
func (l *CachedDataLoader) LoadData(ctx context.Context, userID string, filters map[string]string) ([]DataItem, error) { ... }
func (l *CachedDataLoader) InvalidateCache() { ... }
func (l *CachedDataLoader) generateCacheKey(userID string, filters map[string]string) string { ... }
func (l *CachedDataLoader) fetchFromCache(key string) ([]DataItem, bool) { ... }
func (l *CachedDataLoader) storeToCache(key string, data []DataItem) { ... }
// ... all methods at once

// Test everything together
go test ./data
// 10 failures! Which function is broken?
```

**✅ RIGHT approach:**

```go
// Level 1: Build and test ONE function at a time

// Step 1a: Implement cache key generation
func generateCacheKey(userID string, filters map[string]string) string {
    key := fmt.Sprintf("data:%s", userID)
    for k, v := range filters {
        key += fmt.Sprintf(":%s=%s", k, v)
    }
    return key
}

// Step 1b: Test it IMMEDIATELY
func TestGenerateCacheKey(t *testing.T) {
    filters := map[string]string{"a": "1"}
    key1 := generateCacheKey("u1", filters)
    key2 := generateCacheKey("u1", filters)
    if key1 != key2 {
        t.Errorf("generateCacheKey() = %v, want %v", key1, key2)
    }
    // Run: go test -v -run TestGenerateCacheKey ./data
    // ✅ PASS before continuing
}

// Step 2a: Implement cache fetch
func (l *CachedDataLoader) fetchFromCache(key string) ([]DataItem, bool) {
    l.mu.RLock()
    defer l.mu.RUnlock()
    data, ok := l.cache[key]
    return data, ok
}

// Step 2b: Test it IMMEDIATELY  
func TestFetchFromCache(t *testing.T) {
    loader := &CachedDataLoader{cache: make(map[string][]DataItem)}
    loader.cache["key1"] = []DataItem{{ID: "1"}}
    _, ok := loader.fetchFromCache("missing")
    if ok {
        t.Error("fetchFromCache() found missing key")
    }
    data, ok := loader.fetchFromCache("key1")
    if !ok || len(data) != 1 {
        t.Errorf("fetchFromCache() = %v, want found", ok)
    }
    // Run: go test -v -run TestFetchFromCache ./data
    // ✅ PASS before continuing
}

// Step 3a: Implement cache store
func (l *CachedDataLoader) storeToCache(key string, data []DataItem) {
    l.mu.Lock()
    defer l.mu.Unlock()
    
    // Make defensive copy
    cached := make([]DataItem, len(data))
    copy(cached, data)
    l.cache[key] = cached
}

// Step 3b: Test it IMMEDIATELY
func TestStoreToCache(t *testing.T) {
    loader := &CachedDataLoader{
        cache: make(map[string][]DataItem),
    }
    data := []DataItem{{ID: "1"}}
    
    loader.storeToCache("key1", data)
    
    if len(loader.cache) != 1 {
        t.Errorf("storeToCache() cache len = %d, want 1", len(loader.cache))
    }
    
    // Run: go test -v -run TestStoreToCache ./data
    // ✅ PASS before continuing
}

// Level 2: Now integrate them
func (l *CachedDataLoader) LoadData(ctx context.Context, userID string, filters map[string]string) ([]DataItem, error) {
    key := generateCacheKey(userID, filters)  // ✅ Already tested
    
    // Try cache first
    if data, ok := l.fetchFromCache(key); ok {  // ✅ Already tested
        return data, nil
    }
    
    // Cache miss, fetch from DB
    data, err := l.fetchFromDB(ctx, userID, filters)  // ✅ Already tested
    if err != nil {
        return nil, err
    }
    
    // Store in cache
    l.storeToCache(key, data)  // ✅ Already tested
    
    return data, nil
}

// Test integration
func TestLoadData_Integration(t *testing.T) {
    // All components work individually, now test together
    // Run: go test -v -run TestLoadData_Integration ./data
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
   go test -v -run TestLoadData_Caching ./data
   Result: ✅ PASS

2. Run integration test:
   go test -v ./data
   Result: ✅ PASS

3. Run manual test with real data:
   go run cmd/loader/main.go
   Result: [{id: 1, name: test}] ✅ Correct format

4. Check cache is actually used:
   go run cmd/loader/main.go --benchmark
   Result: First: 150ms, Cached: 2ms ✅ Cache is working

5. Check for race conditions:
   go test -race ./data
   Result: ✅ No race conditions

VERIFIED: Problem is actually fixed."
```

### **Verification Checklist (ALL must pass)**

After claiming "it's fixed", you MUST:

- [ ] **Unit test passes** - Run specific test function
- [ ] **Integration test passes** - Run related integration tests  
- [ ] **Manual test works** - Run actual code with real input
- [ ] **Edge cases work** - Test with empty, nil, invalid inputs
- [ ] **No regressions** - Run full test suite: `go test ./...`
- [ ] **Linter clean** - Run `golangci-lint` or `golint`
- [ ] **Actually test the bug** - If fixing a bug, reproduce the bug first, then verify it's gone
- [ ] **Race detector** - Run `go test -race` (no race conditions)
- [ ] **Format check** - Run `gofmt -l .` (code is formatted)
- [ ] **Vet check** - Run `go vet ./...` (no suspicious constructs)

### **Never Say "Fixed" Until:**

```bash
# ALL of these must be true:
✅ go test -v -run TestSpecificTest ./package  # PASS
✅ go test ./...                                # All PASS
✅ go run cmd/app/main.go                       # Works
✅ golangci-lint run                            # No errors
✅ gofmt -l .                                   # Formatted
✅ go test -race ./...                          # No races
✅ git status                                   # Clean

# Then and ONLY then:
"VERIFIED: Problem is fixed. All tests pass, linter clean, manual test works, no race conditions."
```

---

## 📐 SYNTAX & TYPE CHECKING (Catch Errors Early)

**Problem**: Writing wrong Go syntax leads to compilation errors.

**Solution**: Self-check syntax and types BEFORE running.

### **Pre-Run Syntax Check**

**After writing code, BEFORE testing, check:**

```bash
# 1. Check syntax is valid (compile)
go build ./...
# Must complete without errors

# 2. Check types
go build ./...
# Must not raise type errors

# 3. Check for common issues (vet)
go vet ./...
# Fix any suspicious constructs

# 4. Check formatter
gofmt -l .
# Fix any formatting issues (or use gofmt -w .)

# 5. Check linter
golangci-lint run
# Fix any linting issues
```

### **Common Go Syntax Mistakes**

| ❌ Wrong | ✅ Right | Error |
|---------|---------|-------|
| `func func() {}` (missing name) | `func name() {}` | Syntax error |
| `return x, y` (wrong types) | Return matching types or use named returns | Type mismatch |
| `slice[10]` (might panic) | Check bounds: `if i < len(slice)` | Panic on out of bounds |
| `var x = nil` | `var x *Type = nil` or `var x interface{} = nil` | Cannot infer type |
| Missing error handling | Always check errors: `if err != nil { return err }` | Silent failures |
| `defer` in wrong place | `defer` should be near resource acquisition | Resource leaks |
| Wrong pointer usage | Use `*T` for pointer, `T` for value | Type mismatch |
| Missing context | Accept `context.Context` for cancellation | No cancellation support |

### **Type System Self-Check**

**Every function MUST have:**
```go
// ProcessData processes data with filters.
//
// Parameters:
//   - ctx: Context for cancellation and timeout
//   - userID: User ID (non-empty string)
//   - filters: Filter conditions (can be nil for no filters)
//   - limit: Optional limit (0 means no limit)
//
// Returns:
//   - []ProcessedItem: Slice of processed items, empty slice if no data
//   - error: Error if processing fails
//
// Errors:
//   - ErrInvalidInput: If userID is empty
func ProcessData(
    ctx context.Context,              // ✅ Input type (context)
    userID string,                     // ✅ Input type
    filters map[string]string,         // ✅ Input type
    limit int,                         // ✅ Input type
) ([]ProcessedItem, error) {          // ✅ Return types
    // Implementation
}
```

**Check:**
- ✅ All parameters have clear types
- ✅ Return types are specified (multiple return values)
- ✅ Use appropriate types (string, int, []T, map[K]V, interface{})
- ✅ Use context.Context for cancellation/timeout
- ✅ Always return error as last return value
- ✅ Document all parameters and return values (godoc style)
- ✅ Handle errors properly (check if err != nil)

---

## ⚠️ CRITICAL: Error Prevention

### **Top 2 Errors (90% of Failures)**

| Error | Cause | Fix |
|-------|-------|-----|
| "Error editing file" | Didn't read file first OR old_string mismatch | ALWAYS `read_file()` → Copy EXACT text → Edit |
| "No shell found with ID" | Assumed shell persistence | Use absolute paths OR `cd /path && command` |

### **Environment & Development Rules**

**Build System**:
- ✅ **ALWAYS** use Go toolchain (go build, go test, go mod)
- ✅ Build before running: `go build ./...` or `go build ./cmd/app`
- ✅ Run tests with Go: `go test ./...` or `go test ./package`
- ✅ Check build before executing: `go build ./...`
- ✅ Use modules: `go mod tidy` to manage dependencies
- ✅ Use vet: `go vet ./...` for static analysis
- ✅ Use race detector: `go test -race ./...` for concurrency bugs

**Example Files**:
- ✅ Create **ONE file per request** in `examples/` folder
- ❌ Do NOT create multiple example files for a single request
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.go` (not `example1.go`, `example2.go`)
- ✅ If user asks for multiple examples, combine them into one file with clear sections

### **Pre-Action Checklist**

| Before... | Check... |
|-----------|----------|
| Editing file | ✅ Read file first? ✅ Exact old_string? |
| Shell command | ✅ Absolute paths? ✅ `cd` in same command? ✅ Using go commands? |
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
    Works? YES → VERIFY & Done
    NO → Analyze → Different approach → Retry
        After 3 fails: Check examples
        After 5 fails: Web search or ask
```

### **Quick Decisions**

| Situation | Action | Tool/Method |
|-----------|--------|-------------|
| Unknown codebase area | Explore first | `codebase_search` + read files (parallel) |
| Know exact text | Use grep | `grep "type Name" -r .` |
| Unfamiliar library/error | Web search | "Go library error context 2025" |
| Simple change | Do it | Skip TODO |
| 3+ files or steps | Plan | Create TODO list |
| Tried 3x, failed | Research | Examples → web search |
| Debugging | Systematic | Hypothesis → Test → Analyze |

### **When to Web Search**

✅ **USE web search:**
- Unfamiliar library/API: "Go gorilla mux routing 2025"
- Unknown error (after checking codebase): "Go interface satisfaction error"
- Best practices: "Go error handling patterns 2025"
- After 3 failed attempts

❌ **DON'T web search:**
- Info in codebase → Use `codebase_search` or `grep`
- Standard Go → You know this
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

### **Example: Fix goroutine leak bug**

```
1. UNDERSTAND: Application leaks goroutines after processing
2. HYPOTHESIZE: Goroutines not properly cleaned up on error
3. IMPLEMENT: Add defer cleanup in goroutine
4. TEST: Still leaks goroutines (go test -race shows leaks)
5. ANALYZE: Context not cancelled, goroutines wait forever
6. NEW HYPOTHESIS: Need to cancel context on error
7. IMPLEMENT: Use context.WithCancel and cancel on error
8. TEST: ✅ Works! No goroutine leaks
9. VERIFY: Unit tests pass, race detector clean, integration works
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

```go
ERROR: nil pointer dereference

HYPOTHESIZE:
A (70%): Pointer is nil before use
B (20%): Pointer set to nil after initialization
C (10%): Wrong pointer usage

TEST A: Add nil check before pointer use
if ptr == nil {
    return fmt.Errorf("ptr is nil")
}
RESULT: ptr is nil!
→ Hypothesis A CONFIRMED

ROOT CAUSE: Function doesn't check if ptr is nil before use
FIX: if ptr == nil { return fmt.Errorf("ptr cannot be nil") }
VERIFY: ✅ Works
```

### **Common Patterns**

| Error | Likely Cause | Quick Test |
|-------|--------------|------------|
| `nil pointer dereference` | Using nil pointer | Check `if ptr == nil` before use |
| `index out of range` | Array/slice bounds | Check `if i < len(slice)` |
| `interface conversion` | Wrong type assertion | Use type switch or `v, ok := x.(Type)` |
| `goroutine leak` | Goroutine not cleaned up | Use context cancellation, check with `go test -race` |
| `race condition` | Concurrent access without sync | Use mutex, channels, or `go test -race` |
| `deadlock` | Circular wait | Use timeout, check channel operations |
| Wrong result | Logic error | Print intermediate values, use debugger |

### **Go Debugging Tools**

**Essential debugging commands:**
```bash
# Go build (compile check)
go build ./...

# Go test (run tests)
go test -v ./...

# Go vet (static analysis)
go vet ./...

# Race detector
go test -race ./...

# Delve debugger
dlv debug ./cmd/app
(dlv) break main.main
(dlv) continue
(dlv) print variable_name
(dlv) next

# pprof (profiling)
go test -cpuprofile=cpu.prof -memprofile=mem.prof ./...
go tool pprof cpu.prof
```

---

## 🤖 TOOL USAGE & COMMUNICATION

### **Tool Selection**

| Goal | Tool | Pattern |
|------|------|---------|
| Find concept | `codebase_search` | "How does X work?" |
| Find exact text | `grep` | `grep "type Name"` |
| Read code | `read_file` | Read 3-5 files in parallel |
| Unknown API | `web_search` | "Go library API 2025" |

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
| Interface | `type Interface interface { ... }` | Implement interface implicitly |
| Error handling | `if err != nil { return err }` | Always check errors |
| Defer | `defer resource.Close()` | Use defer for cleanup |
| Context | `context.Context` parameter | Accept context for cancellation |
| Goroutine | `go func() { ... }()` | Use channels for communication |
| Mutex | `sync.Mutex` or `sync.RWMutex` | Protect shared state |

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
| ⚫ Critical | Concurrency, data races | Maximum caution, get review |

---

## 📝 FILE MANAGEMENT

### **Modify vs Create**

| Action | Modify Existing | Create New |
|--------|----------------|------------|
| Fix bug | ✅ | ❌ |
| Add feature to package | ✅ | ❌ |
| Experiment | ✅ (git backup) | ❌ No temp files |
| New package | ❌ | ✅ |
| Unit tests | ❌ | ✅ (in *_test.go file) |

### **Documentation Files**

⚠️ **NEVER create .md files unless explicitly asked**
- ❌ README.md, CHANGELOG.md, DESIGN.md (unless user asks)
- ✅ Only when user says: "Create a README" or "Write documentation"

### **Example Files**

⚠️ **Create ONE file per request in examples/**
- ✅ One comprehensive example file per user request
- ❌ Do NOT create multiple example files (`example1.go`, `example2.go`, etc.)
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.go`
- ✅ If multiple examples needed, combine them into one file with clear sections/comments

### **Temporary Files (Rare, Delete Immediately)**

**If absolutely needed:**
- Use prefixes: `verify_*.go`, `debug_*.go`, `temp_*.go`
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
go build ./...

# 2. Tests
go test ./...

# 3. Vet
go vet ./...

# 4. Linter
golangci-lint run
# OR: golint ./...

# 5. Formatter
gofmt -l .
# OR: gofmt -w .

# 6. Race detector
go test -race ./...

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
- ✅ No race conditions (`go test -race`)
- ✅ Code formatted (`gofmt`)

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
2. Grep for exact names: `grep "type Loader" -r .`
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
✅ No race conditions
✅ Formatted (gofmt)
THEN say "fixed"
```

**Syntax Check** (Before running):
```bash
go build ./...              # Check compilation
go vet ./...                # Check static analysis
golangci-lint run            # Check linting
gofmt -l .                   # Check formatting
```

**Tool Decisions**:
- Find concept → `codebase_search "How does X work in Y?"`
- Find exact text → `grep "type Name" -r .`
- Unknown API/error → `web_search` (with year)

**Go Commands**:
- Compile → Always use go: `go build ./...` or `go build ./cmd/app`
- Run tests → Use go: `go test ./...` or `go test ./package`
- Lint → Use golangci-lint: `golangci-lint run`
- Format → Use gofmt: `gofmt -w .`
- Run binary → `go run ./cmd/app/main.go`
- Race detector → `go test -race ./...`

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
- ✅ Code formatted?
- ✅ No race conditions?

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
- **Compilation errors when running** → Should have checked with `go build` first
- **Type errors** → Should have checked types match
- **Same test fails differently** → Fix root cause, not symptom
- **Don't understand error** → Use hypothesis-driven debugging
- **Creating temp files** → Edit existing code instead
- **Race conditions** → Use `go test -race`, protect shared state
- **Missing error handling** → Always check errors, use `if err != nil`

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
grep "type.*auth.*cache\|cache.*auth" -r .  # Get exact file
read_file("auth/cache.go")     # Read ONLY confirmed file
# Validate: Does this file have the functions I need? YES → Use it, NO → Refine search
```

### **Failure Pattern 2: "Big Bang Integration"**

**Symptom**: Wrote 5 functions, tested together, everything breaks, don't know which function is wrong

**Root cause**: Didn't test each function individually

**Fix**:
```go
// ❌ BAD - All functions at once
func (p *DataProcessor) Load(...) error { ... }
func (p *DataProcessor) Process() error { ... }
func (p *DataProcessor) Save(...) error { ... }
go test ./data  // 10 failures, which broke?

// ✅ GOOD - One function at a time
func (p *DataProcessor) Load(...) error { ... }
func TestLoad(t *testing.T) { /* test */ }
go test -v -run TestLoad ./data  // ✅ PASS - Continue
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
1. Reproduce bug: go run cmd/demo/main.go → Cache not cleared ❌
2. Apply fix: cache.Delete(key) after update
3. Test again: go run cmd/demo/main.go → Cache cleared ✅
4. Run unit test: go test -v -run TestCacheInvalidation ./cache → PASSED ✅
5. Run integration: go test ./... → PASSED ✅
6. Check race conditions: go test -race ./... → No races ✅
VERIFIED: Bug is fixed."
```

### **Failure Pattern 4: "Interface Mismatch"**

**Symptom**: Individual functions work, but fail when integrated

**Root cause**: Interfaces don't match - function A returns slice but function B expects different type

**Fix**:
```go
// ❌ BAD - Interface mismatch
func FetchData(userID string) []DataItem {  // Missing error handling!
    // Sometimes returns nil, sometimes empty slice, unclear when
}

func ProcessData(data []DataItem) {  // Missing nil check!
    for _, item := range data {  // Panics if data is nil!
        // ...
    }
}

// ✅ GOOD - Clear interfaces
// FetchData fetches user data.
//
// Returns:
//   - []DataItem: Slice of data items, empty slice if no data
//   - error: Error if userID is empty or database fails
//
// Errors:
//   - ErrInvalidInput: If userID is empty
func FetchData(ctx context.Context, userID string) ([]DataItem, error) {
    if userID == "" {
        return nil, fmt.Errorf("%w: userID cannot be empty", ErrInvalidInput)
    }
    
    // Implementation...
    return items, nil  // Always returns ([]DataItem, error)
}

// ProcessData processes data slice.
//
// Parameters:
//   - data: Data slice (can be empty, but not nil)
//
// Returns:
//   - []ProcessedItem: Slice of processed items, empty if no items
//   - error: Error if processing fails
func ProcessData(ctx context.Context, data []DataItem) ([]ProcessedItem, error) {
    if data == nil {
        return nil, fmt.Errorf("data cannot be nil")
    }
    
    result := make([]ProcessedItem, 0, len(data))
    for _, item := range data {  // Handles empty slice
        processed, err := processItem(ctx, item)
        if err != nil {
            return nil, err
        }
        result = append(result, processed)
    }
    return result, nil
}

// Now interfaces match: ([]DataItem, error) → ([]ProcessedItem, error)
```

### **Failure Pattern 5: "Syntax Before Test"**

**Symptom**: Code has Go compilation errors when running

**Root cause**: Didn't check compilation before testing

**Fix**:
```bash
# ✅ GOOD - Check compilation BEFORE running tests

# 1. Check compilation
go build ./...
# No output = compilation OK

# 2. Check types
go build ./...
# No error = types OK

# 3. Check vet
go vet ./...
# No error = static analysis OK

# 4. Check linter
golangci-lint run
# No error = linting OK

# 5. NOW run tests
go test ./...
```

### **How to Avoid These Failures**

| Failure Pattern | Prevention |
|----------------|------------|
| Shotgun Search | Specific questions + grep + validate 3-5 files max |
| Big Bang Integration | Test each function individually before integration |
| Assumption Completion | Verify with actual tests (unit + integration + manual) |
| Interface Mismatch | Define interfaces first with clear types and contracts |
| Compilation Errors | Check with `go build` and vet before running |
| Race Conditions | Use `go test -race`, protect shared state with mutexes |

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
5. ⚠️ **Compile check first** - `go build ./...` before running
6. ⚠️ **Test immediately** - After EVERY function, not after coding everything
7. ⚠️ **Read before edit** - Prevents 90% of errors
8. ⚠️ **Iterate systematically** - Different approaches, not same retry
9. ⚠️ **Root cause debugging** - Ask "Why?" 5 times, fix cause not symptom
10. ⚠️ **Clean workspace** - git status clean, all tests pass, linter clean
11. ⚠️ **Use Go toolchain** - Always use go build/test/mod for builds and dependencies
12. ⚠️ **One example file** - Create ONE file per request in `examples/`, not multiple files
13. ⚠️ **Error handling** - Always check errors, use `if err != nil { return err }`
14. ⚠️ **Concurrency safety** - Use `go test -race`, protect shared state

---

## 📚 PROJECT SPECIFICS

### **Go Project Structure**

```
project/
├── cmd/                    # Application entry points
│   └── app/
│       └── main.go
├── internal/               # Private application code
│   ├── data/
│   │   ├── loader.go
│   │   └── loader_test.go
│   └── cache/
│       └── manager.go
├── pkg/                    # Public library code
│   └── utils/
│       └── helper.go
├── api/                    # API definitions (optional)
│   └── proto/
├── web/                    # Web assets (optional)
│   └── static/
├── examples/               # Usage examples
│   └── demo.go
├── go.mod                  # Module definition
├── go.sum                  # Dependency checksums
└── README.md
```

**Package Organization:**
```go
// In internal/data/loader.go:
package data

import (
    "context"
    "fmt"
)

// LoadData loads user data.
func LoadData(ctx context.Context, userID string) ([]DataItem, error) {
    // Implementation
}

// In cmd/app/main.go:
package main

import (
    "context"
    "project/internal/data"
)

func main() {
    ctx := context.Background()
    items, err := data.LoadData(ctx, "user123")
    // ...
}
```

### **Code Style**

- **Formatter**: `gofmt` (standard Go formatter)
- **Linter**: `golangci-lint` (recommended) or `golint`
- **Type check**: Built-in (Go compiler)
- **Testing**: Built-in `testing` package
- **Build system**: Go toolchain (go build, go test, go mod)

### **Go Versions**

- **Go 1.21+** (prefer latest stable version)
- Use modern features: generics (Go 1.18+), error wrapping, context
- Avoid: Deprecated features, platform-specific code without abstraction

### **Go Best Practices**

**Error Handling:**
- ✅ Always check errors: `if err != nil { return err }`
- ✅ Wrap errors with context: `fmt.Errorf("failed to load: %w", err)`
- ✅ Use custom error types: `var ErrNotFound = errors.New("not found")`
- ✅ Return error as last return value: `(result, error)`
- ❌ Never ignore errors silently

**Concurrency:**
- ✅ Use `context.Context` for cancellation and timeout
- ✅ Protect shared state with mutexes or channels
- ✅ Use `go test -race` to detect race conditions
- ✅ Clean up goroutines properly (use defer, context cancellation)
- ❌ Don't leak goroutines

**Memory Management:**
- ✅ Use `defer` for cleanup (files, mutexes, etc.)
- ✅ Make defensive copies when needed
- ✅ Use slices efficiently (pre-allocate capacity when known)
- ✅ Be aware of pointer vs value semantics

**Code Organization:**
- ✅ Keep packages focused and cohesive
- ✅ Export only what's needed (capitalize exported names)
- ✅ Use `internal/` for private code
- ✅ Use `pkg/` for public library code
- ✅ Follow Go naming conventions

**Documentation:**
- ✅ Document all exported functions/types with godoc
- ✅ Include examples in documentation
- ✅ Document error conditions
- ✅ Use `// Package` comment for package documentation

**Testing:**
- ✅ Write unit tests in `*_test.go` files
- ✅ Use table-driven tests for multiple cases
- ✅ Test error cases, not just happy paths
- ✅ Use `t.Helper()` in test helpers
- ✅ Use subtests with `t.Run()`

### **Common Go Idioms**

- **Error Wrapping**: Use `fmt.Errorf` with `%w`
  ```go
  if err != nil {
      return fmt.Errorf("failed to process: %w", err)
  }
  ```

- **Defer for Cleanup**: Always use defer for resource cleanup
  ```go
  file, err := os.Open(path)
  if err != nil {
      return err
  }
  defer file.Close()
  ```

- **Context for Cancellation**: Accept context.Context
  ```go
  func Process(ctx context.Context, data string) error {
      select {
      case <-ctx.Done():
          return ctx.Err()
      default:
      }
      // Process...
  }
  ```

- **Table-Driven Tests**: For multiple test cases
  ```go
  func TestProcess(t *testing.T) {
      tests := []struct {
          name    string
          input   string
          want    string
          wantErr bool
      }{
          {"valid", "input", "output", false},
          {"invalid", "", "", true},
      }
      for _, tt := range tests {
          t.Run(tt.name, func(t *testing.T) {
              got, err := Process(tt.input)
              // Assertions...
          })
      }
  }
  ```

- **Interface Satisfaction**: Implicit interface implementation
  ```go
  type Reader interface {
      Read([]byte) (int, error)
  }
  
  type MyReader struct {}
  func (r *MyReader) Read(p []byte) (int, error) { ... }
  // MyReader automatically satisfies Reader interface
  ```

### **Error Patterns**

| Error | Cause | Solution |
|-------|-------|----------|
| `nil pointer dereference` | Using nil pointer | Check `if ptr == nil` before use |
| `index out of range` | Array/slice bounds | Check `if i < len(slice)` |
| `interface conversion` | Wrong type assertion | Use type switch or `v, ok := x.(Type)` |
| `goroutine leak` | Goroutine not cleaned up | Use context cancellation, check with `go test -race` |
| `race condition` | Concurrent access without sync | Use mutex, channels, or `go test -race` |
| `deadlock` | Circular wait | Use timeout, check channel operations |
| Compilation error | Syntax or type mismatch | Check types, imports, syntax |
| Linker error | Missing package or symbol | Check imports, go.mod |

---

**End of Guide. Use Phase 0 → Execute → Iterate → Verify → Clean. Good luck!**

