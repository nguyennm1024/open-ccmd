# Claude Code Assistant Guide - Expert Performance (Rust Edition)

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

Step 2: If too many results, grep for exact struct/trait/function names
grep "struct.*Loader\|trait.*Loader" -r src/  // Find exact loader types

Step 3: Read ONLY the files that match both searches
read_file("src/data/loader.rs")  // Confirmed relevant

Step 4: Validate you have the right files
- Check use statements, struct names, function signatures
- If wrong, refine search, don't just read more files
```

**Search quality checklist:**
- ✅ Search query is a full question, not just keywords
- ✅ Include context: "in the authentication system" not just "auth"
- ✅ Use grep to confirm exact struct/trait/function names before reading
- ✅ Read 3-5 files max initially - if you need more, your search was too broad
- ✅ Validate each file is relevant before moving on

### **Intelligence Gathering (Do in Parallel)**

```rust
// Example: "Add caching to data loader"

// Step 1: Focused search (not broad)
codebase_search "How does the DataLoader load data from database?"
grep "struct DataLoader\|trait DataLoader" -r src/  // Get exact file location

// Step 2: Parallel reads (ONLY confirmed relevant files)
- read_file("src/data/loader.rs")       // Main implementation
- read_file("src/cache/mod.rs")          // IF grep confirms cache exists
- read_file("tests/data_loader_test.rs") // Tests for expected behavior
- read_file("Cargo.toml")                // Dependencies

// Step 3: Validate relevance
// Check: Does loader.rs have the methods I need to modify?
// Check: Are use statements correct? Ownership patterns?
// If NO → You searched wrong, refine and try again
```

**Mental model checklist:**
- ✅ Entry points (how is this called?)
- ✅ Data flow (what data moves where?)
- ✅ Dependencies (what does this use?)
- ✅ Side effects (what else changes?)
- ✅ Error paths (what can go wrong?)
- ✅ **Interfaces** (what are the function signatures and contracts?)
- ✅ **Ownership** (who owns the data? borrowing vs ownership?)
- ✅ **Lifetimes** (if any lifetime parameters exist)

---

## 🔌 INTERFACE-FIRST DESIGN (Critical for Integration)

**Problem**: Coding everything then testing leads to interface mismatches and integration bugs.

**Solution**: Define interfaces FIRST, implement SECOND, test EACH level.

### **The Interface-First Process**

**Step 1: Define the interface (function signature + contract)**

```rust
// DON'T start coding the implementation yet!
// FIRST, define what the function MUST do:

// In module file (src/data/loader.rs):
use std::collections::HashMap;
use crate::error::DataLoaderError;

/// Load user data with filters.
/// 
/// # Arguments
/// 
/// * `user_id` - User ID (non-empty string)
/// * `filters` - Filter conditions (can be empty map)
/// 
/// # Returns
/// 
/// * `Ok(Vec<HashMap<String, String>>)` - Vector of data maps, empty if no data
/// * `Err(DataLoaderError)` - Error if userId is empty or database fails
/// 
/// # Errors
/// 
/// * `DataLoaderError::InvalidInput` - If userId is empty
/// * `DataLoaderError::DatabaseError` - If database connection fails
pub fn load_data(
    user_id: &str,
    filters: &HashMap<String, String>,
) -> Result<Vec<HashMap<String, String>>, DataLoaderError> {
    // Define interface first, implement second
    todo!()
}
```

**Step 2: Write the test BEFORE implementation**

```rust
// In test file (tests/data_loader_test.rs or src/data/loader.rs with #[cfg(test)]):
#[cfg(test)]
mod tests {
    use super::*;
    use std::collections::HashMap;

    #[test]
    fn test_load_data_basic() {
        let mut filters = HashMap::new();
        filters.insert("status".to_string(), "active".to_string());
        
        let result = load_data("user123", &filters);
        
        assert!(result.is_ok());  // Returns Ok, not Err
        let data = result.unwrap();
        assert!(!data.is_empty() || data.is_empty());  // Returns Vec (may be empty)
        
        // Test edge cases
        let empty_filters = HashMap::new();
        let result = load_data("user123", &empty_filters);  // Empty filters
        assert!(result.is_ok());  // Still returns Ok
        
        // Test error cases
        let result = load_data("", &empty_filters);  // Empty userId
        assert!(result.is_err());  // Returns Err
        assert!(matches!(result.unwrap_err(), DataLoaderError::InvalidInput));
    }
}
```

**Step 3: Implement to satisfy the interface**

```rust
// In implementation file (src/data/loader.rs):
use crate::error::DataLoaderError;

pub fn load_data(
    user_id: &str,
    filters: &HashMap<String, String>,
) -> Result<Vec<HashMap<String, String>>, DataLoaderError> {
    // Validate inputs
    if user_id.is_empty() {
        return Err(DataLoaderError::InvalidInput("user_id cannot be empty".to_string()));
    }
    
    // Implementation...
    Ok(results)  // MUST return Ok(Vec) or Err, never panic
}
```

**Step 4: Test the single function (bottom-up)**

```bash
cargo test data_loader::tests::test_load_data_basic -- --nocapture
# MUST pass before moving to integration
```

### **Interface Contract Checklist**

Before writing implementation, define:

- ✅ **Input types**: Exact parameter types (&str, String, &[T], Vec<T>, Option<T>?)
- ✅ **Output type**: What does it return? (Result<T, E>? Option<T>? T? Never panics?)
- ✅ **Error cases**: What errors can it return? When? (Result::Err variants?)
- ✅ **Side effects**: Does it modify inputs? Change state? Call external APIs?
- ✅ **Preconditions**: What MUST be true before calling? (non-empty? valid format?)
- ✅ **Postconditions**: What WILL be true after calling? (returns Ok, never invalid state?)
- ✅ **Ownership**: Who owns the data? (owned, borrowed, mutable borrow?)
- ✅ **Lifetimes**: Are lifetime parameters needed? (`'a`, `'static`?)

### **Common Interface Mistakes**

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Function sometimes returns Ok(None), sometimes Err | Always return same pattern (Ok(Vec) even if empty, or always Err on error) |
| Function takes ownership when borrowing is sufficient | Use `&str` instead of `String`, `&[T]` instead of `Vec<T>` |
| Error cases panic instead of returning Err | Return `Result<T, E>` and handle errors properly |
| No documentation | Add complete rustdoc comments with `///` |
| Unclear what None means | Use `Option<T>` if None is valid, document meaning |
| Missing error types | Define custom error types, use `Result<T, E>` |
| Unnecessary cloning | Use references (`&str`, `&[T]`) when possible |
| Missing lifetime parameters | Add lifetime parameters when needed (`&'a str`) |

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
```rust
// Code everything at once
// src/data/cached_loader.rs
pub struct CachedDataLoader {
    cache: HashMap<String, Vec<Data>>,
}

impl CachedDataLoader {
    pub fn new() -> Self { ... }
    pub fn load_data(&mut self, user_id: &str, filters: &Filters) -> Result<Vec<Data>, Error> { ... }
    pub fn invalidate_cache(&mut self) { ... }
    fn generate_cache_key(&self, user_id: &str, filters: &Filters) -> String { ... }
    fn fetch_from_cache(&self, key: &str) -> Option<Vec<Data>> { ... }
    fn store_to_cache(&mut self, key: String, data: Vec<Data>) { ... }
    // ... all methods at once
}

// Test everything together
cargo test cached_loader
// 10 failures! Which function is broken?
```

**✅ RIGHT approach:**

```rust
// Level 1: Build and test ONE function at a time

// Step 1a: Implement cache key generation
fn generate_cache_key(user_id: &str, filters: &HashMap<String, String>) -> String {
    let mut key = format!("data:{}", user_id);
    for (k, v) in filters {
        key.push_str(&format!(":{}={}", k, v));
    }
    key
}

// Step 1b: Test it IMMEDIATELY
#[test]
fn test_generate_cache_key() {
    let mut filters = HashMap::new();
    filters.insert("a".to_string(), "1".to_string());
    let key1 = generate_cache_key("u1", &filters);
    let key2 = generate_cache_key("u1", &filters);
    assert_eq!(key1, key2);
    // Run: cargo test test_generate_cache_key
    // ✅ PASS before continuing
}

// Step 2a: Implement cache fetch
fn fetch_from_cache(cache: &HashMap<String, Vec<Data>>, key: &str) -> Option<Vec<Data>> {
    cache.get(key).cloned()
}

// Step 2b: Test it IMMEDIATELY  
#[test]
fn test_fetch_from_cache() {
    let mut cache = HashMap::new();
    cache.insert("key1".to_string(), vec![Data { id: 1 }]);
    assert!(fetch_from_cache(&cache, "missing").is_none());
    assert_eq!(fetch_from_cache(&cache, "key1").unwrap().len(), 1);
    // Run: cargo test test_fetch_from_cache
    // ✅ PASS before continuing
}

// Step 3a: Implement cache store
fn store_to_cache(cache: &mut HashMap<String, Vec<Data>>, key: String, data: Vec<Data>) {
    cache.insert(key, data);
}

// Step 3b: Test it IMMEDIATELY
#[test]
fn test_store_to_cache() {
    let mut cache = HashMap::new();
    store_to_cache(&mut cache, "key1".to_string(), vec![Data { id: 1 }]);
    assert!(cache.contains_key("key1"));
    // Run: cargo test test_store_to_cache
    // ✅ PASS before continuing
}

// Level 2: Now integrate them
impl CachedDataLoader {
    pub fn load_data(
        &mut self,
        user_id: &str,
        filters: &HashMap<String, String>,
    ) -> Result<Vec<Data>, DataLoaderError> {
        let key = generate_cache_key(user_id, filters);  // ✅ Already tested
        
        // Try cache first
        if let Some(cached) = fetch_from_cache(&self.cache, &key) {  // ✅ Already tested
            return Ok(cached);
        }
        
        // Cache miss, fetch from DB
        let data = fetch_from_db(user_id, filters)?;  // ✅ Already tested
        
        // Store in cache
        store_to_cache(&mut self.cache, key, data.clone());  // ✅ Already tested
        
        Ok(data)
    }
}

// Test integration
#[test]
fn test_load_data_integration() {
    // All components work individually, now test together
    // Run: cargo test test_load_data_integration -- --nocapture
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
   cargo test data_loader::tests::test_caching -- --nocapture
   Result: ✅ PASS

2. Run integration test:
   cargo test --test integration_test
   Result: ✅ PASS

3. Run manual test with real data:
   cargo run --bin loader_demo
   Result: [{"id": 1, "name": "test"}] ✅ Correct format

4. Check cache is actually used:
   cargo run --bin loader_demo -- --benchmark
   Result: First: 150ms, Cached: 2ms ✅ Cache is working

5. Check for warnings:
   cargo clippy
   Result: ✅ No warnings

VERIFIED: Problem is actually fixed."
```

### **Verification Checklist (ALL must pass)**

After claiming "it's fixed", you MUST:

- [ ] **Unit test passes** - Run specific test function
- [ ] **Integration test passes** - Run related integration tests  
- [ ] **Manual test works** - Run actual code with real input
- [ ] **Edge cases work** - Test with empty, None, invalid inputs
- [ ] **No regressions** - Run full test suite: `cargo test`
- [ ] **Clippy clean** - Run `cargo clippy` (no warnings)
- [ ] **Actually test the bug** - If fixing a bug, reproduce the bug first, then verify it's gone
- [ ] **Format check** - Run `cargo fmt --check` (code is formatted)
- [ ] **No panics** - Code should never panic in normal operation (use Result/Option)

### **Never Say "Fixed" Until:**

```bash
# ALL of these must be true:
✅ cargo test data_loader::tests::test_specific_test -- --nocapture  # PASS
✅ cargo test                                                         # All PASS
✅ cargo run --bin demo_app                                           # Works
✅ cargo clippy                                                       # No warnings
✅ cargo fmt --check                                                  # Formatted
✅ git status                                                         # Clean

# Then and ONLY then:
"VERIFIED: Problem is fixed. All tests pass, clippy clean, manual test works."
```

---

## 📐 SYNTAX & TYPE CHECKING (Catch Errors Early)

**Problem**: Writing wrong Rust syntax leads to compilation errors.

**Solution**: Self-check syntax and types BEFORE running.

### **Pre-Run Syntax Check**

**After writing code, BEFORE testing, check:**

```bash
# 1. Check syntax is valid (compile)
cargo check
# Must complete without errors

# 2. Check types
cargo check --message-format=short
# Must not raise type errors

# 3. Check for common issues (clippy)
cargo clippy -- -W clippy::all
# Fix any warnings

# 4. Check formatter
cargo fmt --check
# Fix any formatting issues

# 5. Check documentation
cargo doc --no-deps
# Ensure documentation builds
```

### **Common Rust Syntax Mistakes**

| ❌ Wrong | ✅ Right | Error |
|---------|---------|-------|
| `fn func();` (missing body) | `fn func() { }` or implement | Compilation error |
| `return x, y` (comma operator) | Return tuple `(x, y)` or struct | Wrong return type |
| `vec[0]` (might panic) | `vec.get(0)` or `vec[0]` with bounds check | Panic on out of bounds |
| `let x = null;` | `let x: Option<T> = None;` | No null in Rust |
| `let x = ptr;` (raw pointer) | Use references `&T` or `&mut T` | Unsafe code |
| Missing `&` for borrowing | Use `&str` instead of `String` when possible | Unnecessary clone |
| `mut` in wrong place | `let mut x = ...` not `let x mut = ...` | Syntax error |
| Missing `?` operator | Use `?` to propagate errors: `result?` | Compilation error |
| Wrong lifetime syntax | `&'a str` not `&str 'a` | Syntax error |

### **Type System Self-Check**

**Every function MUST have:**
```rust
/// Process data with filters.
/// 
/// # Arguments
/// 
/// * `user_id` - User ID (non-empty string)
/// * `filters` - Filter conditions (can be empty map)
/// * `limit` - Optional limit (None means no limit)
/// 
/// # Returns
/// 
/// * `Ok(Vec<HashMap<String, String>>)` - Vector of processed data, empty if no data
/// * `Err(ProcessorError)` - Error if processing fails
/// 
/// # Errors
/// 
/// * `ProcessorError::InvalidInput` - If userId is empty
pub fn process_data(
    user_id: &str,                      // ✅ Input type (borrowed)
    filters: &HashMap<String, String>,   // ✅ Input type (borrowed)
    limit: Option<usize>,                // ✅ Optional input
) -> Result<Vec<HashMap<String, String>>, ProcessorError> {  // ✅ Return type
    // Implementation
}
```

**Check:**
- ✅ All parameters have clear types
- ✅ Return type is specified (Result<T, E>, Option<T>, or T)
- ✅ Use `Option<T>` for parameters that can be None
- ✅ Use specific types (Vec<String>, not just Vec)
- ✅ Use references (`&str`, `&[T]`) when possible (avoid unnecessary clones)
- ✅ Use `Result<T, E>` for error handling (never panic in library code)
- ✅ Document all parameters and return value (rustdoc style)
- ✅ Consider lifetimes if needed (`&'a str`)

---

## ⚠️ CRITICAL: Error Prevention

### **Top 2 Errors (90% of Failures)**

| Error | Cause | Fix |
|-------|-------|-----|
| "Error editing file" | Didn't read file first OR old_string mismatch | ALWAYS `read_file()` → Copy EXACT text → Edit |
| "No shell found with ID" | Assumed shell persistence | Use absolute paths OR `cd /path && command` |

### **Environment & Development Rules**

**Build System**:
- ✅ **ALWAYS** use Cargo (Rust's build system)
- ✅ Build before running: `cargo build` or `cargo check`
- ✅ Run tests with Cargo: `cargo test`
- ✅ Check build before executing: `cargo check`
- ✅ Use release mode for performance: `cargo build --release`
- ✅ Use clippy for linting: `cargo clippy`
- ✅ Use fmt for formatting: `cargo fmt`

**Example Files**:
- ✅ Create **ONE file per request** in `examples/` folder
- ❌ Do NOT create multiple example files for a single request
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.rs` (not `example1.rs`, `example2.rs`)
- ✅ If user asks for multiple examples, combine them into one file with clear sections

### **Pre-Action Checklist**

| Before... | Check... |
|-----------|----------|
| Editing file | ✅ Read file first? ✅ Exact old_string? |
| Shell command | ✅ Absolute paths? ✅ `cd` in same command? ✅ Using cargo? |
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
| Unfamiliar library/error | Web search | "Rust library error context 2025" |
| Simple change | Do it | Skip TODO |
| 3+ files or steps | Plan | Create TODO list |
| Tried 3x, failed | Research | Examples → web search |
| Debugging | Systematic | Hypothesis → Test → Analyze |

### **When to Web Search**

✅ **USE web search:**
- Unfamiliar library/API: "Rust tokio async handling 2025"
- Unknown error (after checking codebase): "Rust lifetime error cannot infer"
- Best practices: "Rust ownership patterns 2025"
- After 3 failed attempts

❌ **DON'T web search:**
- Info in codebase → Use `codebase_search` or `grep`
- Standard Rust → You know this
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

### **Example: Fix ownership bug**

```
1. UNDERSTAND: Function fails with "value moved" error
2. HYPOTHESIZE: Function takes ownership when it should borrow
3. IMPLEMENT: Change parameter from String to &str
4. TEST: Still fails with lifetime error
5. ANALYZE: Need to add lifetime parameter
6. NEW HYPOTHESIS: Add 'a lifetime to function signature
7. IMPLEMENT: Change to fn process<'a>(data: &'a str) -> &'a str
8. TEST: ✅ Works! No ownership issues
9. VERIFY: Unit tests pass, clippy clean, integration works
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

```rust
ERROR: cannot move out of `data` which is behind a shared reference

HYPOTHESIZE:
A (70%): Trying to move value from behind & reference
B (20%): Need mutable reference
C (10%): Need to clone

TEST A: Check if function tries to take ownership
fn process(data: &Vec<String>) {
    let owned = *data;  // ERROR: trying to move
}
→ Hypothesis A CONFIRMED

ROOT CAUSE: Function tries to take ownership of borrowed data
FIX: Use reference or clone: let owned = data.clone();
VERIFY: ✅ Works
```

### **Common Patterns**

| Error | Likely Cause | Quick Test |
|-------|--------------|------------|
| `cannot move out of` | Trying to move borrowed value | Use reference `&T` or clone |
| `cannot borrow as mutable` | Already borrowed immutably | Restructure code, use `RefCell` if needed |
| `lifetime may not live long enough` | Lifetime mismatch | Add lifetime parameters `'a` |
| `expected type X, found type Y` | Type mismatch | Check types, use `as` or `Into`/`From` |
| `the trait bound is not satisfied` | Missing trait implementation | Implement required trait |
| `value used after move` | Value moved to another owner | Use reference or clone |
| Panic | Unwrapping None or out of bounds | Use `?` operator or `match`/`if let` |

### **Rust Debugging Tools**

**Essential debugging commands:**
```bash
# Cargo check (fast compilation check)
cargo check

# Cargo build (full compilation)
cargo build

# Cargo test (run tests)
cargo test -- --nocapture

# Clippy (linting)
cargo clippy -- -W clippy::all

# Format check
cargo fmt --check

# Documentation
cargo doc --open

# Debug with rust-gdb or rust-lldb
rust-gdb target/debug/app
(gdb) break main
(gdb) run
(gdb) print variable_name
(gdb) backtrace
```

---

## 🤖 TOOL USAGE & COMMUNICATION

### **Tool Selection**

| Goal | Tool | Pattern |
|------|------|---------|
| Find concept | `codebase_search` | "How does X work?" |
| Find exact text | `grep` | `grep "struct Name"` |
| Read code | `read_file` | Read 3-5 files in parallel |
| Unknown API | `web_search` | "Rust library API 2025" |

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
| Trait | `trait Name { ... }` | Implement trait for types |
| Result | `Result<T, E>` | Handle Ok/Err with match or `?` |
| Option | `Option<T>` | Handle Some/None with match or `?` |
| Iterator | `.iter()`, `.into_iter()` | Use iterator methods |
| Ownership | `String`, `Vec<T>` | Consider borrowing `&str`, `&[T]` |

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
| ⚫ Critical | Concurrency, unsafe code | Maximum caution, get review |

---

## 📝 FILE MANAGEMENT

### **Modify vs Create**

| Action | Modify Existing | Create New |
|--------|----------------|------------|
| Fix bug | ✅ | ❌ |
| Add feature to module | ✅ | ❌ |
| Experiment | ✅ (git backup) | ❌ No temp files |
| New module | ❌ | ✅ |
| Unit tests | ❌ | ✅ (or in same file with #[cfg(test)]) |

### **Documentation Files**

⚠️ **NEVER create .md files unless explicitly asked**
- ❌ README.md, CHANGELOG.md, DESIGN.md (unless user asks)
- ✅ Only when user says: "Create a README" or "Write documentation"

### **Example Files**

⚠️ **Create ONE file per request in examples/**
- ✅ One comprehensive example file per user request
- ❌ Do NOT create multiple example files (`example1.rs`, `example2.rs`, etc.)
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.rs`
- ✅ If multiple examples needed, combine them into one file with clear sections/comments

### **Temporary Files (Rare, Delete Immediately)**

**If absolutely needed:**
- Use prefixes: `verify_*.rs`, `debug_*.rs`, `temp_*.rs`
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
5. **VERIFY** → Tests + clippy + integration
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
cargo check

# 2. Tests
cargo test

# 3. Clippy
cargo clippy -- -W clippy::all

# 4. Formatter
cargo fmt --check

# 5. Documentation
cargo doc --no-deps

# 6. Git status clean
git status  # Only intended files?

# 7. No temp files
find . -name "verify_*" -o -name "debug_*" -o -name "temp_*"  # Should be empty
```

### **Done = ALL True**

- ✅ Code compiles + tests pass
- ✅ No clippy warnings
- ✅ ALL temp files deleted
- ✅ No .md files created (unless asked)
- ✅ `git status` clean - only intended changes
- ✅ Integration verified
- ✅ Code formatted (`cargo fmt`)
- ✅ No panics in normal operation

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
✅ Clippy clean
✅ Full test suite pass
✅ Formatted (cargo fmt)
THEN say "fixed"
```

**Syntax Check** (Before running):
```bash
cargo check                    # Check compilation
cargo clippy -- -W clippy::all # Check linting
cargo fmt --check              # Check formatting
```

**Tool Decisions**:
- Find concept → `codebase_search "How does X work in Y?"`
- Find exact text → `grep "struct Name" -r src/`
- Unknown API/error → `web_search` (with year)

**Cargo Commands**:
- Compile → Always use cargo: `cargo build` or `cargo check`
- Run tests → Use cargo: `cargo test`
- Lint → Use clippy: `cargo clippy`
- Format → Use fmt: `cargo fmt`
- Run binary → `cargo run --bin name`

**When Stuck**:
- After 3 fails → Check examples
- After 5 fails → Web search or ask

**Before Done**:
- ✅ Each function tested individually?
- ✅ Integration tested?
- ✅ All tests pass?
- ✅ Clippy clean?
- ✅ Git status clean?
- ✅ Code compiles without warnings?
- ✅ Code formatted?

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
- **Compilation errors when running** → Should have checked with `cargo check` first
- **Type errors** → Should have checked types match
- **Same test fails differently** → Fix root cause, not symptom
- **Don't understand error** → Use hypothesis-driven debugging
- **Creating temp files** → Edit existing code instead
- **Ownership errors** → Understand borrowing vs ownership, use references when possible
- **Lifetime errors** → Add lifetime parameters when needed

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
read_file("src/auth/cache.rs")     # Read ONLY confirmed file
# Validate: Does this file have the functions I need? YES → Use it, NO → Refine search
```

### **Failure Pattern 2: "Big Bang Integration"**

**Symptom**: Wrote 5 functions, tested together, everything breaks, don't know which function is wrong

**Root cause**: Didn't test each function individually

**Fix**:
```rust
// ❌ BAD - All functions at once
impl DataProcessor {
    pub fn load(...) -> Result<(), Error> { ... }
    pub fn process(...) -> Result<(), Error> { ... }
    pub fn save(...) -> Result<(), Error> { ... }
}
cargo test data_processor  // 10 failures, which broke?

// ✅ GOOD - One function at a time
pub fn load(...) -> Result<(), Error> { ... }
#[test]
fn test_load() { /* test */ }
cargo test test_load  // ✅ PASS - Continue
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
1. Reproduce bug: cargo run --bin demo_app → Cache not cleared ❌
2. Apply fix: cache.delete(key) after update
3. Test again: cargo run --bin demo_app → Cache cleared ✅
4. Run unit test: cargo test test_cache_invalidation → PASSED ✅
   Result: PASSED ✅
   
5. Run integration: cargo test --test integration_test
   Result: PASSED ✅
   
6. Check clippy: cargo clippy
   Result: No warnings ✅

VERIFIED: Bug is fixed."
```

### **Failure Pattern 4: "Interface Mismatch"**

**Symptom**: Individual functions work, but fail when integrated

**Root cause**: Interfaces don't match - function A returns Vec but function B expects different type

**Fix**:
```rust
// ❌ BAD - Interface mismatch
fn fetch_data(user_id: &str) -> Vec<Data> {  // Missing error handling!
    // Sometimes returns empty vec, sometimes panics
}

fn process_data(data: Vec<Data>) {  // Takes ownership unnecessarily!
    for item in data {  // Moves data, can't use again
        // ...
    }
}

// ✅ GOOD - Clear interfaces
/// Fetch user data.
/// 
/// # Returns
/// 
/// * `Ok(Vec<Data>)` - Vector with user data, empty if user not found
/// * `Err(DataError)` - Error if userId is empty or database fails
/// 
/// # Errors
/// 
/// * `DataError::InvalidInput` - If userId is empty
fn fetch_data(user_id: &str) -> Result<Vec<Data>, DataError> {
    if user_id.is_empty() {
        return Err(DataError::InvalidInput("user_id cannot be empty".to_string()));
    }
    
    // Implementation...
    Ok(results)  // Always returns Ok(Vec) or Err
}

/// Process data vector.
/// 
/// # Arguments
/// 
/// * `data` - User data vector (can be empty)
/// 
/// # Returns
/// 
/// * `Ok(Vec<ProcessedItem>)` - Vector of processed items, empty if no items
/// * `Err(ProcessError)` - Error if processing fails
fn process_data(data: &[Data]) -> Result<Vec<ProcessedItem>, ProcessError> {
    let mut result = Vec::new();
    for item in data {  // Borrows data, doesn't take ownership
        result.push(process_item(item)?);
    }
    Ok(result)
}

// Now interfaces match: Result<Vec> → Result<Vec>, borrows when possible
```

### **Failure Pattern 5: "Syntax Before Test"**

**Symptom**: Code has Rust compilation errors when running

**Root cause**: Didn't check compilation before testing

**Fix**:
```bash
# ✅ GOOD - Check compilation BEFORE running tests

# 1. Check compilation
cargo check
# No output = compilation OK

# 2. Check types
cargo check --message-format=short
# No error = types OK

# 3. Check clippy
cargo clippy -- -W clippy::all
# No error = linting OK

# 4. Check formatter
cargo fmt --check
# No error = formatting OK

# 5. NOW run tests
cargo test
```

### **How to Avoid These Failures**

| Failure Pattern | Prevention |
|----------------|------------|
| Shotgun Search | Specific questions + grep + validate 3-5 files max |
| Big Bang Integration | Test each function individually before integration |
| Assumption Completion | Verify with actual tests (unit + integration + manual) |
| Interface Mismatch | Define interfaces first with clear types and contracts |
| Compilation Errors | Check with `cargo check` and clippy before running |
| Ownership Errors | Understand borrowing vs ownership, use references when possible |

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
5. ⚠️ **Compile check first** - `cargo check` before running
6. ⚠️ **Test immediately** - After EVERY function, not after coding everything
7. ⚠️ **Read before edit** - Prevents 90% of errors
8. ⚠️ **Iterate systematically** - Different approaches, not same retry
9. ⚠️ **Root cause debugging** - Ask "Why?" 5 times, fix cause not symptom
10. ⚠️ **Clean workspace** - git status clean, all tests pass, clippy clean
11. ⚠️ **Use Cargo** - Always use Cargo for builds, tests, and dependencies
12. ⚠️ **One example file** - Create ONE file per request in `examples/`, not multiple files
13. ⚠️ **Ownership awareness** - Understand borrowing vs ownership, use references when possible
14. ⚠️ **Error handling** - Use Result<T, E> for errors, never panic in library code

---

## 📚 PROJECT SPECIFICS

### **Rust Project Structure**

```
project/
├── src/                    # Main code (.rs files)
│   ├── main.rs             # Binary entry point (if binary)
│   ├── lib.rs              # Library entry point (if library)
│   ├── data/               # Data module
│   │   ├── mod.rs          # Module declaration
│   │   └── loader.rs       # Loader implementation
│   └── utils/              # Utilities module
│       └── mod.rs
├── tests/                  # Integration tests
│   └── integration_test.rs
├── examples/               # Usage examples
│   └── demo.rs
├── benches/                # Benchmarks (optional)
│   └── bench.rs
├── Cargo.toml              # Dependencies and config
├── Cargo.lock              # Lock file (for binaries)
└── target/                 # Build directory (gitignored)
```

**Module Organization:**
```rust
// In src/data/mod.rs:
pub mod loader;
pub mod processor;

// In src/data/loader.rs:
use crate::error::DataLoaderError;

pub fn load_data(...) -> Result<..., DataLoaderError> {
    // Implementation
}

// In src/lib.rs or main.rs:
mod data;
mod error;

use data::loader;
```

### **Code Style**

- **Formatter**: `rustfmt` (via `cargo fmt`)
- **Linter**: `clippy` (via `cargo clippy`)
- **Type check**: Built-in (Rust compiler)
- **Testing**: Built-in `#[test]` attributes
- **Build system**: Cargo

### **Rust Editions**

- **Rust 2021** (current edition, prefer this)
- Use modern features: `?` operator, `match`, `if let`, `Option`, `Result`
- Avoid: `unwrap()` in library code (use `?` or proper error handling)

### **Rust Best Practices**

**Ownership & Borrowing:**
- ✅ Use references (`&str`, `&[T]`) when possible (avoid unnecessary clones)
- ✅ Use `String`/`Vec<T>` when you need ownership
- ✅ Understand the difference between `&T`, `&mut T`, and `T`
- ✅ Use `clone()` only when necessary
- ❌ Don't fight the borrow checker - restructure code instead

**Error Handling:**
- ✅ Always use `Result<T, E>` for fallible operations
- ✅ Use `?` operator to propagate errors
- ✅ Define custom error types using `thiserror` or `anyhow`
- ✅ Never panic in library code (use `Result` instead)
- ✅ Use `Option<T>` for nullable values (no null in Rust)

**Type Safety:**
- ✅ Use strong types (avoid `String` when `&str` works)
- ✅ Use `Option<T>` instead of null
- ✅ Use `Result<T, E>` instead of exceptions
- ✅ Leverage pattern matching (`match`, `if let`)

**Documentation:**
- ✅ Document all public APIs with `///` comments
- ✅ Include examples in documentation
- ✅ Document error conditions
- ✅ Use `# Examples` section in docs

**Testing:**
- ✅ Write unit tests in same file with `#[cfg(test)]`
- ✅ Write integration tests in `tests/` directory
- ✅ Test error cases, not just happy paths
- ✅ Use `assert!`, `assert_eq!`, etc. for assertions

### **Common Rust Idioms**

- **Result Propagation**: Use `?` operator
  ```rust
  fn process() -> Result<(), Error> {
      let data = load_data()?;  // Propagates error
      process_data(&data)?;
      Ok(())
  }
  ```

- **Option Handling**: Use `match` or `if let`
  ```rust
  match value {
      Some(v) => process(v),
      None => handle_none(),
  }
  
  // Or:
  if let Some(v) = value {
      process(v);
  }
  ```

- **Iterator Chains**: Use iterator methods
  ```rust
  let result: Vec<i32> = items
      .iter()
      .filter(|x| x > &0)
      .map(|x| x * 2)
      .collect();
  ```

- **Builder Pattern**: For complex construction
  ```rust
  struct Config {
      host: String,
      port: u16,
  }
  
  impl Config {
      fn new() -> Self { ... }
      fn host(mut self, host: String) -> Self { ... }
      fn port(mut self, port: u16) -> Self { ... }
      fn build(self) -> Result<Config, Error> { ... }
  }
  ```

- **Trait Objects**: For polymorphism
  ```rust
  trait Processor {
      fn process(&self, data: &str) -> Result<(), Error>;
  }
  
  fn use_processor(p: &dyn Processor) {
      p.process("data").unwrap();
  }
  ```

### **Error Patterns**

| Error | Cause | Solution |
|-------|-------|----------|
| `cannot move out of` | Trying to move borrowed value | Use reference `&T` or clone |
| `cannot borrow as mutable` | Already borrowed immutably | Restructure code, use `RefCell` if needed |
| `lifetime may not live long enough` | Lifetime mismatch | Add lifetime parameters `'a` |
| `expected type X, found type Y` | Type mismatch | Check types, use `as` or `Into`/`From` |
| `the trait bound is not satisfied` | Missing trait implementation | Implement required trait |
| `value used after move` | Value moved to another owner | Use reference or clone |
| Panic | Unwrapping None or out of bounds | Use `?` operator or `match`/`if let` |
| `cannot infer type` | Type cannot be inferred | Add type annotation |

---

**End of Guide. Use Phase 0 → Execute → Iterate → Verify → Clean. Good luck!**

