# Claude Code Assistant Guide - Expert Performance (JavaScript Edition)

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
grep "class.*Loader\|function.*load" -r src/  // Find exact loader classes/functions

Step 3: Read ONLY the files that match both searches
read_file("src/data/loader.js")  // Confirmed relevant

Step 4: Validate you have the right files
- Check imports/exports, function signatures
- If wrong, refine search, don't just read more files
```

**Search quality checklist:**
- ✅ Search query is a full question, not just keywords
- ✅ Include context: "in the authentication system" not just "auth"
- ✅ Use grep to confirm exact class/function names before reading
- ✅ Read 3-5 files max initially - if you need more, your search was too broad
- ✅ Validate each file is relevant before moving on

### **Intelligence Gathering (Do in Parallel)**

```javascript
// Example: "Add caching to data loader"

// Step 1: Focused search (not broad)
codebase_search "How does the DataLoader load data from database?"
grep "class DataLoader\|function loadData\|export.*loadData" -r src/  // Get exact file location

// Step 2: Parallel reads (ONLY confirmed relevant files)
- read_file("src/data/loader.js")       // Main implementation
- read_file("src/cache/manager.js")      // IF grep confirms cache exists
- read_file("src/data/loader.test.js")  // Tests for expected behavior
- read_file("package.json")              // Dependencies

// Step 3: Validate relevance
// Check: Does loader.js have the functions I need to modify?
// Check: Are imports/exports correct? Module system?
// If NO → You searched wrong, refine and try again
```

**Mental model checklist:**
- ✅ Entry points (how is this called?)
- ✅ Data flow (what data moves where?)
- ✅ Dependencies (what does this use?)
- ✅ Side effects (what else changes?)
- ✅ Error paths (what can go wrong?)
- ✅ **Interfaces** (what are the function signatures and contracts?)
- ✅ **Async handling** (Promises, async/await, callbacks?)
- ✅ **Module system** (ES modules, CommonJS?)

---

## 🔌 INTERFACE-FIRST DESIGN (Critical for Integration)

**Problem**: Coding everything then testing leads to interface mismatches and integration bugs.

**Solution**: Define interfaces FIRST, implement SECOND, test EACH level.

### **The Interface-First Process**

**Step 1: Define the interface (function signature + contract)**

```javascript
// DON'T start coding the implementation yet!
// FIRST, define what the function MUST do:

// In module file (src/data/loader.js):
/**
 * Load user data with filters.
 * 
 * @param {string} userId - User ID (non-empty string)
 * @param {Object<string, string>} filters - Filter conditions (can be empty object)
 * @returns {Promise<Array<Object<string, string>>>} Promise that resolves to array of data objects, empty array if no data
 * @throws {Error} If userId is empty
 * @throws {DatabaseError} If database connection fails
 * 
 * @example
 * const data = await loadData('user123', { status: 'active' });
 * console.log(data); // [{ id: '1', name: 'test' }]
 */
export async function loadData(userId, filters = {}) {
    // Define interface first, implement second
    throw new Error('not implemented');
}
```

**Step 2: Write the test BEFORE implementation**

```javascript
// In test file (src/data/loader.test.js):
import { loadData } from './loader.js';

describe('loadData', () => {
    test('should load data with filters', async () => {
        const filters = { status: 'active' };
        
        const result = await loadData('user123', filters);
        
        expect(result).toBeDefined();
        expect(Array.isArray(result)).toBe(true);  // Returns array, not null/undefined
        expect(result.length).toBeGreaterThanOrEqual(0);  // May be empty
        
        // Test edge cases
        const emptyResult = await loadData('user123', {});  // Empty filters
        expect(Array.isArray(emptyResult)).toBe(true);  // Still returns array
        
        // Test error cases
        await expect(loadData('', {})).rejects.toThrow();  // Empty userId throws
        await expect(loadData('', {})).rejects.toThrow('userID cannot be empty');
    });
});
```

**Step 3: Implement to satisfy the interface**

```javascript
// In implementation file (src/data/loader.js):
export async function loadData(userId, filters = {}) {
    // Validate inputs
    if (!userId || userId.trim() === '') {
        throw new Error('userID cannot be empty');
    }
    
    // Implementation...
    return results;  // MUST return Promise<Array>, never throw unless error
}
```

**Step 4: Test the single function (bottom-up)**

```bash
npm test -- loader.test.js -t "should load data with filters"
# MUST pass before moving to integration
```

### **Interface Contract Checklist**

Before writing implementation, define:

- ✅ **Input types**: Exact parameter types (string, number, Object, Array, Promise?, optional?)
- ✅ **Output type**: What does it return? (Promise<T>? T? void? Never returns undefined?)
- ✅ **Error cases**: What errors can it throw? When? (Error, custom error types?)
- ✅ **Side effects**: Does it modify inputs? Change state? Call external APIs?
- ✅ **Preconditions**: What MUST be true before calling? (non-empty? valid format? non-null?)
- ✅ **Postconditions**: What WILL be true after calling? (returns array, never undefined?)
- ✅ **Async behavior**: Is it async? Does it return Promise? Use async/await?
- ✅ **Module exports**: How is it exported? (export, module.exports, default?)

### **Common Interface Mistakes**

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Function sometimes returns undefined, sometimes array | Always return same type (empty array [], or always throw on error) |
| Function modifies input object | Return new object, don't modify input (use spread operator) |
| Error cases return undefined instead of throwing | Throw specific errors for failures |
| No JSDoc comments | Add complete JSDoc comments with @param, @returns, @throws |
| Unclear what undefined/null means | Use optional chaining, document meaning |
| Missing async/await | Use async/await for Promises, handle errors with try/catch |
| No input validation | Validate all inputs, throw error if invalid |
| Mixing callbacks and Promises | Use async/await consistently, avoid callback hell |

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
```javascript
// Code everything at once
// src/data/cached-loader.js
export class CachedDataLoader {
    constructor() {
        this.cache = new Map();
    }
    
    async loadData(userId, filters) { ... }
    invalidateCache() { ... }
    generateCacheKey(userId, filters) { ... }
    fetchFromCache(key) { ... }
    storeToCache(key, data) { ... }
    // ... all methods at once
}

// Test everything together
npm test
// 10 failures! Which function is broken?
```

**✅ RIGHT approach:**

```javascript
// Level 1: Build and test ONE function at a time

// Step 1a: Implement cache key generation
function generateCacheKey(userId, filters) {
    let key = `data:${userId}`;
    for (const [k, v] of Object.entries(filters)) {
        key += `:${k}=${v}`;
    }
    return key;
}

// Step 1b: Test it IMMEDIATELY
test('generateCacheKey should generate same key for same inputs', () => {
    const filters = { a: '1' };
    const key1 = generateCacheKey('u1', filters);
    const key2 = generateCacheKey('u1', filters);
    expect(key1).toBe(key2);
    // Run: npm test -- -t "generateCacheKey"
    // ✅ PASS before continuing
});

// Step 2a: Implement cache fetch
fetchFromCache(key) {
    return this.cache.get(key) || null;
}

// Step 2b: Test it IMMEDIATELY  
test('fetchFromCache should return cached data', () => {
    const loader = new CachedDataLoader();
    loader.cache.set('key1', [{ id: '1' }]);
    expect(loader.fetchFromCache('missing')).toBeNull();
    expect(loader.fetchFromCache('key1')).toEqual([{ id: '1' }]);
    // Run: npm test -- -t "fetchFromCache"
    // ✅ PASS before continuing
});

// Step 3a: Implement cache store
storeToCache(key, data) {
    this.cache.set(key, JSON.parse(JSON.stringify(data)));  // Defensive copy
}

// Step 3b: Test it IMMEDIATELY
test('storeToCache should store data', () => {
    const loader = new CachedDataLoader();
    loader.storeToCache('key1', [{ id: '1' }]);
    expect(loader.cache.get('key1')).toEqual([{ id: '1' }]);
    // Run: npm test -- -t "storeToCache"
    // ✅ PASS before continuing
});

// Level 2: Now integrate them
async loadData(userId, filters = {}) {
    const key = generateCacheKey(userId, filters);  // ✅ Already tested
    
    // Try cache first
    const cached = this.fetchFromCache(key);  // ✅ Already tested
    if (cached) {
        return cached;
    }
    
    // Cache miss, fetch from DB
    const data = await this.fetchFromDB(userId, filters);  // ✅ Already tested
    
    // Store in cache
    this.storeToCache(key, data);  // ✅ Already tested
    
    return data;
}

// Test integration
test('loadData integration', async () => {
    // All components work individually, now test together
    // Run: npm test -- -t "loadData integration"
});
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
   npm test -- loader.test.js -t "caching"
   Result: ✅ PASS

2. Run integration test:
   npm test
   Result: ✅ PASS

3. Run manual test with real data:
   node src/bin/loader.js
   Result: [{id: '1', name: 'test'}] ✅ Correct format

4. Check cache is actually used:
   node src/bin/loader.js --benchmark
   Result: First: 150ms, Cached: 2ms ✅ Cache is working

5. Check linting:
   npm run lint
   Result: ✅ No errors

VERIFIED: Problem is actually fixed."
```

### **Verification Checklist (ALL must pass)**

After claiming "it's fixed", you MUST:

- [ ] **Unit test passes** - Run specific test function
- [ ] **Integration test passes** - Run related integration tests  
- [ ] **Manual test works** - Run actual code with real input
- [ ] **Edge cases work** - Test with empty, null, undefined, invalid inputs
- [ ] **No regressions** - Run full test suite: `npm test`
- [ ] **Linter clean** - Run `npm run lint` or `eslint`
- [ ] **Actually test the bug** - If fixing a bug, reproduce the bug first, then verify it's gone
- [ ] **Type check** - Run `npm run type-check` (if using TypeScript)
- [ ] **Format check** - Run `npm run format:check` (if using Prettier)

### **Never Say "Fixed" Until:**

```bash
# ALL of these must be true:
✅ npm test -- loader.test.js -t "specific test"  # PASS
✅ npm test                                       # All PASS
✅ node src/bin/app.js                            # Works
✅ npm run lint                                   # No errors
✅ npm run type-check                             # No errors (if TS)
✅ git status                                     # Clean

# Then and ONLY then:
"VERIFIED: Problem is fixed. All tests pass, linter clean, manual test works."
```

---

## 📐 SYNTAX & TYPE CHECKING (Catch Errors Early)

**Problem**: Writing wrong JavaScript syntax leads to runtime errors.

**Solution**: Self-check syntax and types BEFORE running.

### **Pre-Run Syntax Check**

**After writing code, BEFORE testing, check:**

```bash
# 1. Check syntax is valid (if using Node.js)
node --check src/loader.js
# Must complete without errors

# 2. Check types (if using TypeScript)
npm run type-check
# OR: tsc --noEmit

# 3. Check linter
npm run lint
# OR: eslint src/

# 4. Check formatter
npm run format:check
# OR: prettier --check src/

# 5. Check for common issues
npm run lint -- --fix
```

### **Common JavaScript Syntax Mistakes**

| ❌ Wrong | ✅ Right | Error |
|---------|---------|-------|
| `function func()` (missing body) | `function func() { }` | Syntax error |
| `return x, y` (comma operator) | Return object `{x, y}` or array `[x, y]` | Wrong return type |
| `array[10]` (might be undefined) | Check bounds: `if (i < array.length)` or use `array.at(i)` | Undefined access |
| `obj.property` (might be undefined) | Use optional chaining: `obj?.property` | Cannot read property |
| Missing `await` | Use `await` for Promises: `await promise` | Unhandled Promise |
| `==` instead of `===` | Use strict equality: `===` | Type coercion bugs |
| Missing error handling | Use try/catch for async: `try { await ... } catch (e) {}` | Unhandled errors |
| `var` instead of `const`/`let` | Use `const` by default, `let` if reassignment needed | Hoisting issues |

### **Type System Self-Check**

**Every function MUST have:**
```javascript
/**
 * Process data with filters.
 * 
 * @param {string} userId - User ID (non-empty string)
 * @param {Object<string, string>} filters - Filter conditions (can be empty object)
 * @param {number} [limit] - Optional limit (undefined means no limit)
 * @returns {Promise<Array<Object<string, string>>>} Promise that resolves to array of processed data, empty array if no data
 * @throws {Error} If userId is empty
 * 
 * @example
 * const result = await processData('user123', { status: 'active' }, 10);
 */
export async function processData(userId, filters = {}, limit) {
    // Validate inputs
    if (!userId || userId.trim() === '') {
        throw new Error('userId cannot be empty');
    }
    
    // Implementation...
    return results;  // MUST return Promise<Array>, never undefined
}
```

**Check:**
- ✅ All parameters have JSDoc types
- ✅ Return type is specified (Promise<T>?, T?, void?)
- ✅ Use JSDoc for optional parameters `@param {type} [name]`
- ✅ Use specific types (Array<string>, not just Array)
- ✅ Document async behavior (async/await, Promises)
- ✅ Document all parameters and return value (JSDoc style)
- ✅ Include examples in JSDoc

---

## ⚠️ CRITICAL: Error Prevention

### **Top 2 Errors (90% of Failures)**

| Error | Cause | Fix |
|-------|-------|-----|
| "Error editing file" | Didn't read file first OR old_string mismatch | ALWAYS `read_file()` → Copy EXACT text → Edit |
| "No shell found with ID" | Assumed shell persistence | Use absolute paths OR `cd /path && command` |

### **Environment & Development Rules**

**Package Manager**:
- ✅ **ALWAYS** use package manager (npm, yarn, pnpm)
- ✅ Install dependencies: `npm install` or `yarn install`
- ✅ Run scripts: `npm run script-name` or `yarn script-name`
- ✅ Check package.json for available scripts
- ✅ Use `npm ci` for CI/CD (clean install)

**Example Files**:
- ✅ Create **ONE file per request** in `examples/` folder
- ❌ Do NOT create multiple example files for a single request
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.js` (not `example1.js`, `example2.js`)
- ✅ If user asks for multiple examples, combine them into one file with clear sections

### **Pre-Action Checklist**

| Before... | Check... |
|-----------|----------|
| Editing file | ✅ Read file first? ✅ Exact old_string? |
| Shell command | ✅ Absolute paths? ✅ `cd` in same command? ✅ Using npm/yarn? |
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
| Know exact text | Use grep | `grep "function Name" -r src/` |
| Unfamiliar library/error | Web search | "JavaScript library error context 2025" |
| Simple change | Do it | Skip TODO |
| 3+ files or steps | Plan | Create TODO list |
| Tried 3x, failed | Research | Examples → web search |
| Debugging | Systematic | Hypothesis → Test → Analyze |

### **When to Web Search**

✅ **USE web search:**
- Unfamiliar library/API: "JavaScript Express routing 2025"
- Unknown error (after checking codebase): "JavaScript async await error"
- Best practices: "JavaScript async patterns 2025"
- After 3 failed attempts

❌ **DON'T web search:**
- Info in codebase → Use `codebase_search` or `grep`
- Standard JavaScript → You know this
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

### **Example: Fix async bug**

```
1. UNDERSTAND: Function returns Promise instead of data
2. HYPOTHESIZE: Missing await in function call
3. IMPLEMENT: Add await before function call
4. TEST: Still returns Promise
5. ANALYZE: Function itself is not async, but calls async function
6. NEW HYPOTHESIS: Need to make function async and await the call
7. IMPLEMENT: Change to async function and await the call
8. TEST: ✅ Works! Returns data correctly
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

```javascript
ERROR: Cannot read property 'id' of undefined

HYPOTHESIZE:
A (70%): Object is undefined before accessing property
B (20%): Property doesn't exist
C (10%): Wrong object reference

TEST A: Add null check before property access
console.log('obj:', obj);
if (obj && obj.id) { ... }
RESULT: obj: undefined  // obj is undefined!
→ Hypothesis A CONFIRMED

ROOT CAUSE: Function doesn't check if obj is defined before use
FIX: if (!obj) { throw new Error('obj cannot be undefined'); }
VERIFY: ✅ Works
```

### **Common Patterns**

| Error | Likely Cause | Quick Test |
|-------|--------------|------------|
| `Cannot read property of undefined` | Object is undefined | Check `if (obj)` before access, use optional chaining `obj?.property` |
| `Unhandled Promise rejection` | Missing await or catch | Use `await` or `.catch()`, wrap in try/catch |
| `TypeError: X is not a function` | Wrong type or undefined | Check `typeof x === 'function'` |
| `ReferenceError: X is not defined` | Variable not in scope | Check imports/exports, scope |
| Wrong result | Logic error | Print intermediate values, use debugger |
| `undefined` return | Missing return or async issue | Check return statement, await Promises |

### **JavaScript Debugging Tools**

**Essential debugging commands:**
```bash
# Node.js syntax check
node --check src/file.js

# Run tests
npm test

# Linting
npm run lint

# Type checking (if TypeScript)
npm run type-check

# Debug with Node.js inspector
node --inspect src/file.js
# Then open chrome://inspect in Chrome

# Debug with VS Code
# Set breakpoints and use F5 to debug
```

---

## 🤖 TOOL USAGE & COMMUNICATION

### **Tool Selection**

| Goal | Tool | Pattern |
|------|------|---------|
| Find concept | `codebase_search` | "How does X work?" |
| Find exact text | `grep` | `grep "function Name"` |
| Read code | `read_file` | Read 3-5 files in parallel |
| Unknown API | `web_search` | "JavaScript library API 2025" |

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
| Promise | `.then()`, `.catch()` | Use async/await instead |
| Callback | Function taking callback | Consider Promise/async-await |
| Class | `class Name { }` | Use for stateful objects |
| Module | `export`/`import` or `module.exports`/`require` | Follow existing module system |
| Async | `async function` | Always await Promises |

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
| ⚫ Critical | Async/concurrency, data loss | Maximum caution, get review |

---

## 📝 FILE MANAGEMENT

### **Modify vs Create**

| Action | Modify Existing | Create New |
|--------|----------------|------------|
| Fix bug | ✅ | ❌ |
| Add feature to module | ✅ | ❌ |
| Experiment | ✅ (git backup) | ❌ No temp files |
| New module | ❌ | ✅ |
| Unit tests | ❌ | ✅ (in *.test.js file) |

### **Documentation Files**

⚠️ **NEVER create .md files unless explicitly asked**
- ❌ README.md, CHANGELOG.md, DESIGN.md (unless user asks)
- ✅ Only when user says: "Create a README" or "Write documentation"

### **Example Files**

⚠️ **Create ONE file per request in examples/**
- ✅ One comprehensive example file per user request
- ❌ Do NOT create multiple example files (`example1.js`, `example2.js`, etc.)
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.js`
- ✅ If multiple examples needed, combine them into one file with clear sections/comments

### **Temporary Files (Rare, Delete Immediately)**

**If absolutely needed:**
- Use prefixes: `verify_*.js`, `debug_*.js`, `temp_*.js`
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
# 1. Syntax check
node --check src/

# 2. Tests
npm test

# 3. Linter
npm run lint

# 4. Formatter (if using Prettier)
npm run format:check

# 5. Type check (if TypeScript)
npm run type-check

# 6. Git status clean
git status  # Only intended files?

# 7. No temp files
find . -name "verify_*" -o -name "debug_*" -o -name "temp_*"  # Should be empty
```

### **Done = ALL True**

- ✅ Code runs + tests pass
- ✅ No linter errors
- ✅ ALL temp files deleted
- ✅ No .md files created (unless asked)
- ✅ `git status` clean - only intended changes
- ✅ Integration verified
- ✅ Code formatted (if using formatter)
- ✅ No unhandled Promise rejections

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
2. Grep for exact names: `grep "function Loader" -r src/`
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
✅ Formatted (if using formatter)
THEN say "fixed"
```

**Syntax Check** (Before running):
```bash
node --check src/file.js     # Check syntax
npm run lint                  # Check linting
npm run type-check            # Check types (if TS)
```

**Tool Decisions**:
- Find concept → `codebase_search "How does X work in Y?"`
- Find exact text → `grep "function Name" -r src/`
- Unknown API/error → `web_search` (with year)

**Package Manager Commands**:
- Install deps → `npm install` or `yarn install`
- Run tests → `npm test` or `yarn test`
- Lint → `npm run lint` or `yarn lint`
- Format → `npm run format` or `yarn format`
- Run script → `npm run script-name`

**When Stuck**:
- After 3 fails → Check examples
- After 5 fails → Web search or ask

**Before Done**:
- ✅ Each function tested individually?
- ✅ Integration tested?
- ✅ All tests pass?
- ✅ Linter clean?
- ✅ Git status clean?
- ✅ Code runs without errors?
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
- **Runtime errors when running** → Should have checked with `node --check` first
- **Type errors** → Should have checked types match
- **Same test fails differently** → Fix root cause, not symptom
- **Don't understand error** → Use hypothesis-driven debugging
- **Creating temp files** → Edit existing code instead
- **Unhandled Promises** → Always await or catch Promises
- **Missing error handling** → Use try/catch for async code

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
grep "class.*auth.*cache\|cache.*auth" -r src/  # Get exact file
read_file("src/auth/cache.js")     # Read ONLY confirmed file
# Validate: Does this file have the functions I need? YES → Use it, NO → Refine search
```

### **Failure Pattern 2: "Big Bang Integration"**

**Symptom**: Wrote 5 functions, tested together, everything breaks, don't know which function is wrong

**Root cause**: Didn't test each function individually

**Fix**:
```javascript
// ❌ BAD - All functions at once
class DataProcessor {
    async load(...) { ... }
    async process() { ... }
    async save(...) { ... }
}
npm test  // 10 failures, which broke?

// ✅ GOOD - One function at a time
async load(...) { ... }
test('load should load data', async () => { /* test */ });
npm test -- -t "load"  // ✅ PASS - Continue
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
1. Reproduce bug: node src/bin/demo.js → Cache not cleared ❌
2. Apply fix: cache.delete(key) after update
3. Test again: node src/bin/demo.js → Cache cleared ✅
4. Run unit test: npm test -- -t "cache invalidation" → PASSED ✅
5. Run integration: npm test → PASSED ✅
6. Check linting: npm run lint → No errors ✅
VERIFIED: Bug is fixed."
```

### **Failure Pattern 4: "Interface Mismatch"**

**Symptom**: Individual functions work, but fail when integrated

**Root cause**: Interfaces don't match - function A returns array but function B expects different type

**Fix**:
```javascript
// ❌ BAD - Interface mismatch
function fetchData(userId) {  // Missing error handling!
    // Sometimes returns undefined, sometimes array
}

function processData(data) {  // Missing null check!
    for (const item of data) {  // Crashes if data is undefined!
        // ...
    }
}

// ✅ GOOD - Clear interfaces
/**
 * Fetch user data.
 * 
 * @param {string} userId - User ID (non-empty)
 * @returns {Promise<Array<Object>>} Promise that resolves to array of data, empty array if no data
 * @throws {Error} If userId is empty
 */
async function fetchData(userId) {
    if (!userId || userId.trim() === '') {
        throw new Error('userId cannot be empty');
    }
    
    // Implementation...
    return results || [];  // Always returns array (empty if not found)
}

/**
 * Process data array.
 * 
 * @param {Array<Object>} data - Data array (can be empty, but not undefined)
 * @returns {Promise<Array<Object>>} Promise that resolves to array of processed items
 * @throws {Error} If data is undefined
 */
async function processData(data) {
    if (!Array.isArray(data)) {
        throw new Error('data must be an array');
    }
    
    const result = [];
    for (const item of data) {  // Handles empty array
        result.push(await processItem(item));
    }
    return result;
}

// Now interfaces match: Promise<Array> → Promise<Array>
```

### **Failure Pattern 5: "Syntax Before Test"**

**Symptom**: Code has JavaScript syntax/runtime errors when running

**Root cause**: Didn't check syntax before testing

**Fix**:
```bash
# ✅ GOOD - Check syntax BEFORE running tests

# 1. Check syntax
node --check src/module.js
# No output = syntax OK

# 2. Check types (if TypeScript)
npm run type-check
# No error = types OK

# 3. Check linter
npm run lint
# No error = linting OK

# 4. NOW run tests
npm test
```

### **How to Avoid These Failures**

| Failure Pattern | Prevention |
|----------------|------------|
| Shotgun Search | Specific questions + grep + validate 3-5 files max |
| Big Bang Integration | Test each function individually before integration |
| Assumption Completion | Verify with actual tests (unit + integration + manual) |
| Interface Mismatch | Define interfaces first with clear types and contracts |
| Runtime Errors | Check with `node --check` and linter before running |
| Async Errors | Always await Promises, use try/catch for error handling |

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
5. ⚠️ **Syntax check first** - `node --check` before running
6. ⚠️ **Test immediately** - After EVERY function, not after coding everything
7. ⚠️ **Read before edit** - Prevents 90% of errors
8. ⚠️ **Iterate systematically** - Different approaches, not same retry
9. ⚠️ **Root cause debugging** - Ask "Why?" 5 times, fix cause not symptom
10. ⚠️ **Clean workspace** - git status clean, all tests pass, linter clean
11. ⚠️ **Use package manager** - Always use npm/yarn for dependencies and scripts
12. ⚠️ **One example file** - Create ONE file per request in `examples/`, not multiple files
13. ⚠️ **Async handling** - Always await Promises, use try/catch for errors
14. ⚠️ **Error handling** - Handle all errors, don't let Promises reject unhandled

---

## 📚 PROJECT SPECIFICS

### **JavaScript Project Structure**

```
project/
├── src/                    # Main code (.js files)
│   ├── data/
│   │   ├── loader.js
│   │   └── loader.test.js
│   └── utils/
│       └── helper.js
├── lib/                    # Compiled code (if using build step)
├── examples/               # Usage examples
│   └── demo.js
├── tests/                  # Integration tests (optional)
│   └── integration.test.js
├── package.json            # Dependencies and scripts
├── package-lock.json       # Lock file (npm)
├── .eslintrc.js            # ESLint config
├── .prettierrc             # Prettier config (optional)
└── tsconfig.json           # TypeScript config (if using TS)
```

**Module Organization:**
```javascript
// In src/data/loader.js (ES modules):
export async function loadData(userId, filters) {
    // Implementation
}

// In src/data/loader.js (CommonJS):
module.exports = {
    loadData: async function(userId, filters) {
        // Implementation
    }
};

// In src/app.js:
import { loadData } from './data/loader.js';  // ES modules
// OR:
const { loadData } = require('./data/loader');  // CommonJS
```

### **Code Style**

- **Formatter**: Prettier (recommended) or project-specific
- **Linter**: ESLint (most common)
- **Type check**: TypeScript (if using TS) or JSDoc
- **Testing**: Jest (most common), Mocha, Vitest, or project-specific
- **Build system**: npm/yarn scripts, webpack, rollup, or esbuild

### **JavaScript Versions**

- **ES2020+** (prefer modern JavaScript)
- Use modern features: async/await, arrow functions, destructuring, optional chaining
- Avoid: `var` (use `const`/`let`), `==` (use `===`), callback hell

### **JavaScript Best Practices**

**Async/Await:**
- ✅ Use async/await instead of callbacks or `.then()/.catch()`
- ✅ Always handle errors with try/catch
- ✅ Don't forget `await` keyword
- ✅ Use `Promise.all()` for parallel async operations
- ❌ Don't mix callbacks and Promises

**Error Handling:**
- ✅ Always handle errors (try/catch for async, .catch() for Promises)
- ✅ Throw meaningful errors with messages
- ✅ Use custom error classes when appropriate
- ❌ Never ignore errors silently

**Code Organization:**
- ✅ Use ES modules (`import`/`export`) or CommonJS consistently
- ✅ Keep functions small and focused
- ✅ Use meaningful variable and function names
- ✅ Follow existing code style in project

**Type Safety:**
- ✅ Use JSDoc comments for type documentation
- ✅ Consider TypeScript for larger projects
- ✅ Validate inputs, especially from external sources
- ✅ Use optional chaining (`?.`) for safe property access

**Testing:**
- ✅ Write unit tests for all functions
- ✅ Test error cases, not just happy paths
- ✅ Use descriptive test names
- ✅ Mock external dependencies
- ✅ Test async functions properly

### **Common JavaScript Idioms**

- **Async/Await**: Prefer over callbacks
  ```javascript
  async function fetchData() {
      try {
          const response = await fetch(url);
          return await response.json();
      } catch (error) {
          throw new Error(`Failed to fetch: ${error.message}`);
      }
  }
  ```

- **Optional Chaining**: Safe property access
  ```javascript
  const value = obj?.property?.nested?.value;
  ```

- **Nullish Coalescing**: Default values
  ```javascript
  const value = input ?? 'default';
  ```

- **Destructuring**: Extract values
  ```javascript
  const { name, age } = user;
  const [first, second] = array;
  ```

- **Spread Operator**: Copy/merge objects/arrays
  ```javascript
  const newObj = { ...oldObj, newProp: 'value' };
  const newArray = [...oldArray, newItem];
  ```

- **Arrow Functions**: Concise function syntax
  ```javascript
  const add = (a, b) => a + b;
  const process = async (data) => { /* ... */ };
  ```

### **Error Patterns**

| Error | Cause | Solution |
|-------|-------|----------|
| `Cannot read property of undefined` | Object is undefined | Use optional chaining `obj?.property` or check `if (obj)` |
| `Unhandled Promise rejection` | Missing await or catch | Use `await` or `.catch()`, wrap in try/catch |
| `TypeError: X is not a function` | Wrong type or undefined | Check `typeof x === 'function'` |
| `ReferenceError: X is not defined` | Variable not in scope | Check imports/exports, scope |
| `undefined` return | Missing return or async issue | Check return statement, await Promises |
| Compilation error | Syntax or type mismatch | Check syntax, types, imports |
| Module not found | Missing import/export | Check module system, file paths |

---

**End of Guide. Use Phase 0 → Execute → Iterate → Verify → Clean. Good luck!**

