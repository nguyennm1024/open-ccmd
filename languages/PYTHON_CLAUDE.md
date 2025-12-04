# Claude Code Assistant Guide - Expert Performance

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

Step 2: If too many results, grep for exact class/function names
grep "class.*Loader" -r src/  # Find exact loader classes

Step 3: Read ONLY the files that match both searches
read_file("src/data/loader.py")  # Confirmed relevant

Step 4: Validate you have the right files
- Check imports, class names, function signatures
- If wrong, refine search, don't just read more files
```

**Search quality checklist:**
- ✅ Search query is a full question, not just keywords
- ✅ Include context: "in the authentication system" not just "auth"
- ✅ Use grep to confirm exact class/function names before reading
- ✅ Read 3-5 files max initially - if you need more, your search was too broad
- ✅ Validate each file is relevant before moving on

### **Intelligence Gathering (Do in Parallel)**

```python
# Example: "Add caching to data loader"

# Step 1: Focused search (not broad)
codebase_search "How does the DataLoader load data from database?"
grep "class DataLoader" -r src/  # Get exact file location

# Step 2: Parallel reads (ONLY confirmed relevant files)
- read_file("src/data/loader.py")       # Main implementation
- read_file("src/cache/manager.py")     # IF grep confirms cache exists
- read_file("tests/test_loader.py")     # Tests for expected behavior

# Step 3: Validate relevance
# Check: Does loader.py have the methods I need to modify?
# If NO → You searched wrong, refine and try again
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

```python
# DON'T start coding the implementation yet!
# FIRST, define what the function MUST do:

def load_data(user_id: str, filters: Dict[str, Any]) -> List[Dict[str, Any]]:
    """
    Load user data with filters.
    
    Args:
        user_id: User ID (non-empty string)
        filters: Filter conditions (can be empty dict)
    
    Returns:
        List of data dicts, empty list if no data
        
    Raises:
        ValueError: If user_id is empty
        DatabaseError: If database connection fails
    """
    pass  # Define interface first, implement second
```

**Step 2: Write the test BEFORE implementation**

```python
def test_load_data():
    # Test the interface contract
    result = load_data("user123", {"status": "active"})
    assert isinstance(result, list)  # Returns list, not None
    assert all(isinstance(item, dict) for item in result)  # List of dicts
    
    # Test edge cases
    result = load_data("user123", {})  # Empty filters
    assert isinstance(result, list)  # Still returns list
    
    # Test error cases
    with pytest.raises(ValueError):
        load_data("", {})  # Empty user_id raises ValueError
```

**Step 3: Implement to satisfy the interface**

```python
def load_data(user_id: str, filters: Dict[str, Any]) -> List[Dict[str, Any]]:
    """[Same docstring]"""
    if not user_id:
        raise ValueError("user_id cannot be empty")
    
    # Implementation...
    return results  # MUST return list, never None
```

**Step 4: Test the single function (bottom-up)**

```bash
pytest tests/test_loader.py::test_load_data -v
# MUST pass before moving to integration
```

### **Interface Contract Checklist**

Before writing implementation, define:

- ✅ **Input types**: Exact parameter types (str, int, Dict, List, Optional?)
- ✅ **Output type**: What does it return? (List? Dict? None? Never None?)
- ✅ **Error cases**: What exceptions can it raise? When?
- ✅ **Side effects**: Does it modify inputs? Change state? Call external APIs?
- ✅ **Preconditions**: What MUST be true before calling? (non-empty? valid format?)
- ✅ **Postconditions**: What WILL be true after calling? (returns list, never None?)

### **Common Interface Mistakes**

| ❌ Mistake | ✅ Fix |
|-----------|--------|
| Function sometimes returns None, sometimes list | Always return same type (empty list, not None) |
| Function modifies input dict | Return new dict, don't modify input |
| Error cases return None instead of raising | Raise specific exceptions for errors |
| No type hints | Add complete type hints |
| Unclear what None means | Use Optional[T] if None is valid, document meaning |

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
```python
# Code everything at once
class CachedDataLoader:
    def __init__(self): ...
    def load_data(self): ...
    def _fetch_from_db(self): ...
    def _fetch_from_cache(self): ...
    def _store_to_cache(self): ...
    def invalidate_cache(self): ...

# Test everything together
pytest tests/test_cached_loader.py
# 10 failures! Which function is broken?
```

**✅ RIGHT approach:**

```python
# Level 1: Build and test ONE function at a time

# Step 1a: Implement cache key generation
def _generate_cache_key(self, user_id: str, filters: Dict) -> str:
    """Generate cache key from user_id and filters."""
    return f"data:{user_id}:{hash(frozenset(filters.items()))}"

# Step 1b: Test it IMMEDIATELY
def test_generate_cache_key():
    loader = CachedDataLoader()
    key1 = loader._generate_cache_key("u1", {"a": 1})
    key2 = loader._generate_cache_key("u1", {"a": 1})
    assert key1 == key2  # Same inputs = same key
    # Run: pytest tests/test_cached_loader.py::test_generate_cache_key
    # ✅ PASS before continuing

# Step 2a: Implement cache fetch
def _fetch_from_cache(self, key: str) -> Optional[List[Dict]]:
    """Fetch data from cache, return None if not found."""
    return self.cache.get(key)

# Step 2b: Test it IMMEDIATELY  
def test_fetch_from_cache():
    loader = CachedDataLoader()
    assert loader._fetch_from_cache("missing") is None
    loader.cache.set("key1", [{"data": 1}])
    assert loader._fetch_from_cache("key1") == [{"data": 1}]
    # Run: pytest tests/test_cached_loader.py::test_fetch_from_cache
    # ✅ PASS before continuing

# Step 3a: Implement cache store
def _store_to_cache(self, key: str, data: List[Dict]) -> None:
    """Store data to cache."""
    self.cache.set(key, data, ttl=3600)

# Step 3b: Test it IMMEDIATELY
def test_store_to_cache():
    loader = CachedDataLoader()
    loader._store_to_cache("key1", [{"data": 1}])
    assert loader.cache.get("key1") == [{"data": 1}]
    # Run: pytest tests/test_cached_loader.py::test_store_to_cache
    # ✅ PASS before continuing

# Level 2: Now integrate them
def load_data(self, user_id: str, filters: Dict) -> List[Dict]:
    key = self._generate_cache_key(user_id, filters)  # ✅ Already tested
    cached = self._fetch_from_cache(key)              # ✅ Already tested
    if cached:
        return cached
    data = self._fetch_from_db(user_id, filters)      # ✅ Already tested
    self._store_to_cache(key, data)                   # ✅ Already tested
    return data

# Test integration
def test_load_data_integration():
    # All components work individually, now test together
    # Run: pytest tests/test_cached_loader.py::test_load_data_integration
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
   pytest tests/test_loader.py::test_caching -v
   Result: ✅ PASS

2. Run integration test:
   pytest tests/test_integration.py -v
   Result: ✅ PASS

3. Run manual test with real data:
   python -c "from loader import load_data; print(load_data('u1', {}))"
   Result: [{'id': 1, 'name': 'test'}] ✅ Correct format

4. Check cache is actually used:
   python -c "from loader import load_data; import time; 
   t1=time.time(); load_data('u1', {}); t2=time.time(); 
   load_data('u1', {}); t3=time.time(); 
   print(f'First: {t2-t1:.3f}s, Cached: {t3-t2:.3f}s')"
   Result: First: 0.150s, Cached: 0.002s ✅ Cache is working

VERIFIED: Problem is actually fixed."
```

### **Verification Checklist (ALL must pass)**

After claiming "it's fixed", you MUST:

- [ ] **Unit test passes** - Run specific test function
- [ ] **Integration test passes** - Run related integration tests  
- [ ] **Manual test works** - Run actual code with real input
- [ ] **Edge cases work** - Test with empty, None, invalid inputs
- [ ] **No regressions** - Run full test suite: `pytest tests/`
- [ ] **Linter clean** - Run `ruff check .` (or project linter)
- [ ] **Actually test the bug** - If fixing a bug, reproduce the bug first, then verify it's gone

### **Never Say "Fixed" Until:**

```python
# ALL of these must be true:
✅ pytest tests/specific_test.py::test_function -v  # PASS
✅ pytest tests/ -v                                 # All PASS
✅ python -c "from module import func; print(func(test_input))"  # Works
✅ ruff check .                                     # No errors
✅ git status                                       # Clean

# Then and ONLY then:
"VERIFIED: Problem is fixed. All tests pass, linter clean, manual test works."
```

---

## 📐 SYNTAX & TYPE CHECKING (Catch Errors Early)

**Problem**: Writing wrong Python syntax leads to runtime errors.

**Solution**: Self-check syntax and types BEFORE running.

### **Pre-Run Syntax Check**

**After writing code, BEFORE testing, check:**

```python
# 1. Check syntax is valid
python -m py_compile path/to/file.py
# Must complete without errors

# 2. Check imports work
python -c "import module_name"
# Must not raise ImportError

# 3. Check types (if using mypy)
mypy path/to/file.py --ignore-missing-imports
# Fix any type errors

# 4. Check linter
ruff check path/to/file.py
# Fix any style issues
```

### **Common Python Syntax Mistakes**

| ❌ Wrong | ✅ Right | Error |
|---------|---------|-------|
| `def func():` (missing body) | `def func():\n    pass` | IndentationError |
| `return x, y` (in type hint) | `-> Tuple[int, int]` | Use Tuple not comma |
| `dict[str, int]` (Python <3.9) | `Dict[str, int]` | Use typing.Dict |
| `list[int]` (Python <3.9) | `List[int]` | Use typing.List |
| `def func(x: int = None)` | `def func(x: Optional[int] = None)` | Use Optional |
| `if x:` (x can be 0) | `if x is not None:` | 0 is falsy but valid |

### **Type Hints Self-Check**

**Every function MUST have:**
```python
from typing import List, Dict, Optional, Any, Tuple

def process_data(
    user_id: str,                      # ✅ Input type
    filters: Dict[str, Any],            # ✅ Input type  
    limit: Optional[int] = None        # ✅ Optional input
) -> List[Dict[str, Any]]:             # ✅ Return type
    """Docstring with Args, Returns, Raises."""  # ✅ Documentation
    pass
```

**Check:**
- ✅ All parameters have type hints
- ✅ Return type is specified
- ✅ Use Optional[T] for parameters that can be None
- ✅ Use specific types (List[str], not just list)
- ✅ Docstring documents all parameters and return value

---

## ⚠️ CRITICAL: Error Prevention

### **Top 2 Errors (90% of Failures)**

| Error | Cause | Fix |
|-------|-------|-----|
| "Error editing file" | Didn't read file first OR old_string mismatch | ALWAYS `read_file()` → Copy EXACT text → Edit |
| "No shell found with ID" | Assumed shell persistence | Use absolute paths OR `cd /path && command` |

### **Environment & Development Rules**

**Virtual Environment**:
- ✅ **ALWAYS** use `.venv` when executing code
- ✅ Activate `.venv` before running Python commands: `source .venv/bin/activate` (or `.venv/bin/python`)
- ✅ Install dependencies in `.venv`: `pip install -e .` or `uv sync`
- ✅ Run tests in `.venv`: `.venv/bin/pytest` or `source .venv/bin/activate && pytest`
- ✅ Check `.venv` exists before executing: `test -d .venv || python -m venv .venv`

**Example Files**:
- ✅ Create **ONE file per request** in `examples/` folder
- ❌ Do NOT create multiple example files for a single request
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.py` (not `example1.py`, `example2.py`)
- ✅ If user asks for multiple examples, combine them into one file with clear sections

### **Pre-Action Checklist**

| Before... | Check... |
|-----------|----------|
| Editing file | ✅ Read file first? ✅ Exact old_string? |
| Shell command | ✅ Absolute paths? ✅ `cd` in same command? ✅ Using `.venv`? |
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
| Know exact text | Use grep | `grep "pattern" -r src/` |
| Unfamiliar library/error | Web search | "Library error context 2025" |
| Simple change | Do it | Skip TODO |
| 3+ files or steps | Plan | Create TODO list |
| Tried 3x, failed | Research | Examples → web search |
| Debugging | Systematic | Hypothesis → Test → Analyze |

### **When to Web Search**

✅ **USE web search:**
- Unfamiliar library/API: "FastAPI WebSocket handling 2025"
- Unknown error (after checking codebase): "SQLAlchemy DetachedInstanceError"
- Best practices: "Python async patterns 2025"
- After 3 failed attempts

❌ **DON'T web search:**
- Info in codebase → Use `codebase_search` or `grep`
- Standard Python → You know this
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
1. UNDERSTAND: After logout, session should be None
2. HYPOTHESIZE: logout() doesn't call session.clear()
3. IMPLEMENT: Add session.clear() in logout()
4. TEST: Still has session data
5. ANALYZE: session is in request context (g.session)
6. NEW HYPOTHESIS: Need g.pop('session', None)
7. IMPLEMENT: Change to g.pop('session', None)
8. TEST: ✅ Works! Session is None
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

```python
ERROR: KeyError: 'user_id'

HYPOTHESIZE:
A (70%): user_id not in session dict
B (20%): session is None
C (10%): typo in key name

TEST A: print(f"Keys: {session.keys()}")
RESULT: Keys: ['username', 'email']  # user_id missing!
→ Hypothesis A CONFIRMED

ROOT CAUSE: Auth doesn't set user_id in session
FIX: session['user_id'] = user.id
VERIFY: ✅ Works
```

### **Common Patterns**

| Error | Likely Cause | Quick Test |
|-------|--------------|------------|
| `KeyError` | Missing dict key | `print(dict.keys())` |
| `AttributeError` | Wrong type/None | `print(type(obj))` |
| `IndexError` | List too short | `print(len(list))` |
| Wrong result | Logic error | Print intermediate values |

---

## 🤖 TOOL USAGE & COMMUNICATION

### **Tool Selection**

| Goal | Tool | Pattern |
|------|------|---------|
| Find concept | `codebase_search` | "How does X work?" |
| Find exact text | `grep` | `grep "class Name"` |
| Read code | `read_file` | Read 3-5 files in parallel |
| Unknown API | `web_search` | "FastAPI WebSocket 2025" |

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
| Singleton | `_instance = None` | Use get_instance(), don't instantiate |
| Factory | `create_*()`, `make_*()` | Use factory method |
| Decorator | `@functools.wraps` | Follow same wrapping pattern |

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
- ❌ Do NOT create multiple example files (`example1.py`, `example2.py`, etc.)
- ✅ Use descriptive names: `examples/phase_1_data_loading_demo.py`
- ✅ If multiple examples needed, combine into one file with clear sections/comments

### **Temporary Files (Rare, Delete Immediately)**

**If absolutely needed:**
- Use prefixes: `verify_*.py`, `debug_*.py`, `temp_*.py`
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
# 1. Tests
pytest tests/ -v

# 2. Linter
ruff check .  # or project linter

# 3. Type check (if used)
mypy .

# 4. Git status clean
git status  # Only intended files?

# 5. No temp files
find . -name "verify_*" -o -name "debug_*"  # Should be empty
```

### **Done = ALL True**

- ✅ Code works + tests pass
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
python -m py_compile file.py  # Check syntax
mypy file.py                  # Check types (if used)
ruff check file.py            # Check style
```

**Tool Decisions**:
- Find concept → `codebase_search "How does X work in Y?"`
- Find exact text → `grep "class Name" -r src/`
- Unknown API/error → `web_search` (with year)

**Environment**:
- Execute code → Always use `.venv`: `source .venv/bin/activate && python script.py` or `.venv/bin/python script.py`
- Install deps → In `.venv`: `source .venv/bin/activate && pip install package` or `uv sync`
- Run tests → In `.venv`: `source .venv/bin/activate && pytest` or `.venv/bin/pytest`

**When Stuck**:
- After 3 fails → Check examples
- After 5 fails → Web search or ask

**Before Done**:
- ✅ Each function tested individually?
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
- **Coding multiple functions without testing** → Stop, test each function individually
- **Saying "fixed" without running tests** → Stop, verify with actual tests
- **Tried same fix 3 times** → Try completely different approach
- **Integration fails after functions work** → Check interface contracts match
- **Syntax errors when running** → Should have checked with `py_compile` first
- **Type errors** → Should have checked type hints match
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
read_file("src/auth/cache.py")     # Read ONLY confirmed file
# Validate: Does this file have the methods I need? YES → Use it, NO → Refine search
```

### **Failure Pattern 2: "Big Bang Integration"**

**Symptom**: Wrote 5 functions, tested together, everything breaks, don't know which function is wrong

**Root cause**: Didn't test each function individually

**Fix**:
```python
# ❌ BAD
class DataProcessor:
    def __init__(self): ...
    def load(self): ...
    def process(self): ...
    def save(self): ...
    def validate(self): ...

pytest test_processor.py  # 10 failures, which function broke?

# ✅ GOOD
class DataProcessor:
    def load(self): ...
    
pytest test_processor.py::test_load  # ✅ PASS - Continue

    def process(self): ...
    
pytest test_processor.py::test_process  # ✅ PASS - Continue

    def save(self): ...
    
pytest test_processor.py::test_save  # ✅ PASS - Continue

# Now all individual functions work, integration will likely work
pytest test_processor.py::test_integration  # ✅ PASS
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

1. Reproduce bug: python test_script.py
   Before fix: Cache not cleared ❌
   
2. Apply fix: cache.delete(key) after update

3. Test again: python test_script.py
   After fix: Cache cleared ✅
   
4. Run unit test: pytest tests/test_cache.py::test_invalidation -v
   Result: PASSED ✅
   
5. Run integration: pytest tests/test_integration.py -v
   Result: PASSED ✅

VERIFIED: Bug is fixed."
```

### **Failure Pattern 4: "Interface Mismatch"**

**Symptom**: Individual functions work, but fail when integrated

**Root cause**: Interfaces don't match - function A returns Dict but function B expects List

**Fix**:
```python
# ❌ BAD - Interface mismatch
def fetch_data(user_id: str):  # Missing return type!
    data = db.query(user_id)
    return data  # Sometimes returns None, sometimes dict

def process_data(data):  # Missing type hint!
    return [item for item in data]  # Crashes if data is None!

# ✅ GOOD - Clear interfaces
def fetch_data(user_id: str) -> Dict[str, Any]:
    """
    Fetch user data.
    
    Returns:
        Dict with user data. Never returns None.
        Returns empty dict {} if user not found.
    """
    data = db.query(user_id)
    return data if data else {}  # Always returns dict

def process_data(data: Dict[str, Any]) -> List[Dict[str, Any]]:
    """
    Process data dict into list.
    
    Args:
        data: User data dict (can be empty)
    
    Returns:
        List of processed items. Empty list if no items.
    """
    return [item for item in data.get('items', [])]  # Handles empty dict

# Now interfaces match: Dict → Dict, Dict → List
```

### **Failure Pattern 5: "Syntax Before Test"**

**Symptom**: Code has Python syntax errors when running

**Root cause**: Didn't check syntax before testing

**Fix**:
```bash
# ✅ GOOD - Check syntax BEFORE running tests

# 1. Check syntax
python -m py_compile src/module.py
# No output = syntax OK

# 2. Check imports
python -c "from src.module import MyClass"
# No error = imports OK

# 3. Check types (if using mypy)
mypy src/module.py --ignore-missing-imports
# No error = types OK

# 4. Check linter
ruff check src/module.py
# No error = style OK

# 5. NOW run tests
pytest tests/test_module.py -v
```

### **How to Avoid These Failures**

| Failure Pattern | Prevention |
|----------------|------------|
| Shotgun Search | Specific questions + grep + validate 3-5 files max |
| Big Bang Integration | Test each function individually before integration |
| Assumption Completion | Verify with actual tests (unit + integration + manual) |
| Interface Mismatch | Define interfaces first with clear types and contracts |
| Syntax Errors | Check with `py_compile` and linter before running |

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
5. ⚠️ **Syntax check first** - `python -m py_compile` before running
6. ⚠️ **Test immediately** - After EVERY function, not after coding everything
7. ⚠️ **Read before edit** - Prevents 90% of errors
8. ⚠️ **Iterate systematically** - Different approaches, not same retry
9. ⚠️ **Root cause debugging** - Ask "Why?" 5 times, fix cause not symptom
10. ⚠️ **Clean workspace** - git status clean, all tests pass, linter clean
11. ⚠️ **Use .venv** - Always activate `.venv` before executing Python code
12. ⚠️ **One example file** - Create ONE file per request in `examples/`, not multiple files

---

## 📚 PROJECT SPECIFICS

### **Python Project Structure**

```
project/
├── src/ or package_name/    # Main code
│   ├── core/                # Core functionality
│   └── utils/               # Utilities
├── tests/                   # Test suites
├── examples/                # Usage examples
├── requirements.txt         # Dependencies
└── pyproject.toml           # Config
```

### **Code Style**

- **Formatter**: Black (88 chars)
- **Linter**: Ruff (or project-specific)
- **Type check**: MyPy (if used)
- **Testing**: pytest

### **Error Patterns**

| Error | Cause | Solution |
|-------|-------|----------|
| Import error | Missing package or circular | Check requirements.txt, __init__.py |
| Type error | Wrong types | Add type hints, check compatibility |
| Attribute error | Missing attribute | Check object type, verify method exists |
| Key error | Missing dict key | Use .get() with default |

---

**End of Guide. Use Phase 0 → Execute → Iterate → Verify → Clean. Good luck!**
