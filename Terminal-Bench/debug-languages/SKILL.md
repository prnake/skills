---
name: debug-languages
description: Task execution protocol with mandatory test-driven verification. Applies to ALL tasks - coding, debugging, data processing, file manipulation, git operations, and more. CRITICAL RULES (1) NEVER refuse a task - always attempt a solution using available tools, (2) MUST discover and run existing tests before declaring done, (3) MUST verify all outputs match exact expectations, (4) Single-round execution - no follow-up opportunities, output must be correct on first delivery.
---

# Task Execution & Verification Protocol

Strict execution protocol for all tasks. You have ONE chance to deliver correct results. There are NO follow-up rounds. Every output must be verified before delivery.

## Golden Rules (ALWAYS APPLY)

1. **NEVER REFUSE A TASK.** If a task seems hard, use tools creatively. Download videos with `yt-dlp`. Parse images with scripts. Use domain-specific engines (stockfish for chess, z3 for SAT, etc.) instead of guessing. There is always a way.
2. **ALWAYS FIND AND RUN TESTS BEFORE DECLARING DONE.** Tests exist at `/tests/` in almost every environment. Find them. Run them. If they fail, fix your work. Do not submit without passing tests.
3. **NEVER WRITE CODE WITHOUT TESTING IT.** Writing a file and immediately declaring success is forbidden. Every piece of code must be compiled/executed and verified.
4. **CLEAN UP BUILD ARTIFACTS.** If the task asks you to create a file, the output directory should contain ONLY what was requested. Remove compiled binaries, temp files, and intermediate outputs.
5. **PROTECT YOUR CONTEXT WINDOW.** Never read large binary files (images, PDFs, videos) directly into context. Write scripts to process them instead.

## Phase 1: Understand & Explore (REQUIRED)

Before writing any code or making any changes:

1. **Read the task carefully.** Identify ALL requirements, including implicit ones (file locations, output formats, edge cases).
2. **Build a Requirements Checklist.** List every requirement as a numbered checklist and mark each `[DONE]` or `[TODO]`. Maintain and update this checklist throughout the task — it must appear in every analysis until completion. Example:
   ```
   Requirements:
   1. [DONE] Parse input file at /app/data.csv
   2. [TODO] Output sorted results to /app/output.json
   3. [TODO] Handle empty input gracefully
   ```
3. **Explore the environment:**
   ```bash
   ls /app/                    # What files exist?
   ls /tests/ 2>/dev/null      # Are there test files?
   cat /tests/test_outputs.py  # What do the tests expect?
   ```
4. **Read existing test files FIRST.** The tests reveal the exact acceptance criteria - expected file locations, exact output values, data formats, and edge cases. This is more important than the task description.
5. **Read existing source files** referenced by the task before modifying or creating anything.

## Phase 2: Test Discovery (REQUIRED - DO NOT SKIP)

Locate ALL test and verification files. They define what "correct" means.

```bash
# Standard test locations (check ALL of these)
ls /tests/test_outputs.py 2>/dev/null
ls /tests/test_*.py 2>/dev/null
ls /app/test_outputs.py 2>/dev/null
ls /app/tests/ 2>/dev/null
find / -name "test_outputs.*" -o -name "test_*.py" 2>/dev/null | head -20

# Also look for verification scripts mentioned in the task prompt
# Read test files to understand exact expectations
```

**CRITICAL:** Read the test file contents. They tell you:
- Exact output file paths and names expected
- Exact data formats and values expected
- Edge cases and boundary conditions
- Which files should or should not exist after completion

If tests reference a different version of a file than what's in `/app/` (e.g., `/tests/filter.py` vs `/app/filter.py`), the `/tests/` version is what matters for grading. Test against BOTH.

## Phase 3: Implement Solution

### Strategy Selection

Choose your approach based on task type:

| Task Type | Strategy |
|-----------|----------|
| Code/algorithm | Write code, compile, test with multiple inputs |
| File processing | Write a script to process files; never read binaries into context |
| Image/video analysis | Use tools: `ffmpeg`, `tesseract`, `python-chess`, `stockfish`, specialized libraries |
| Git operations | Use git commands; verify with `git diff`, `git log`; check ALL changed files |
| Data retrieval | Write script to compute results; verify against expected output |
| Query writing (SQL/SPARQL) | Write query, execute it, compare results to expectations |

### Resource Management

- **Large files (images, PDFs, videos):** Write a Python/bash script to process them. NEVER use the Read tool on binary files to "look at" them.
- **Multiple files:** Process them in a loop script, not one-by-one in context.
- **Heavy dependencies:** Install via `pip install` or `apt-get` in a bash command, not interactively.
- **If approaching context limits:** Summarize findings so far, then continue with focused actions.

### Implementation Rules

- Write the minimum code needed. Don't over-engineer.
- Handle ALL edge cases revealed by the test files.
- For specialized domains (chess, math, crypto, ML), use domain-specific libraries and engines rather than attempting to reason through the problem manually.
- If creating output files, match EXACTLY the path, format, and content the tests expect.
- If moving/copying files, verify source and destination afterward.

## Phase 4: Pre-Delivery Verification (REQUIRED - DO NOT SKIP)

This is the most important phase. You MUST complete ALL steps.

### Step 1: Run Your Solution

```bash
# Execute what you built
python3 /app/solution.py        # or whatever you created
# Verify output files exist
ls -la /app/expected_output_file
cat /app/expected_output_file   # Check content is reasonable
```

### Step 2: Write and Run Unit Tests

Before relying on the project's test suite, write a dedicated verification script that tests each requirement from your checklist independently:

- Cover each requirement from Phase 1 with at least one test case.
- Include edge cases (empty input, boundary values, unexpected formats).
- Verify output format, types, and exact values — not just "it ran without error".
- Print clear `PASS`/`FAIL` for every test case.

```python
# Example structure for a verification script
def test_output_file_exists():
    assert os.path.exists("/app/output.json"), "FAIL: output file missing"
    print("PASS: output file exists")

def test_output_sorted():
    data = json.load(open("/app/output.json"))
    assert data == sorted(data, key=lambda x: x["score"]), "FAIL: not sorted"
    print("PASS: output is sorted")

# Run all tests; exit non-zero on any failure
```

Run your script and confirm every test prints `PASS`. Fix failures before continuing.

### Step 3: Run Existing Tests

```bash
# Run the project's test suite
cd /tests && python -m pytest test_outputs.py -v 2>&1
# OR
python -m pytest /tests/test_outputs.py -v 2>&1
```

If tests fail:
1. **Read the failure message carefully.** It tells you exactly what's wrong.
2. **Fix the issue.** Don't just try again - understand what the test expected vs what you produced.
3. **Re-run tests.** Repeat until ALL tests pass.
4. **Common failure patterns and fixes:**
   - "File does not exist" → Check your output path matches exactly what the test expects
   - "assert X == Y" → Your output value is wrong, recheck your logic
   - Directory not empty / extra files → Remove build artifacts: `rm -f /app/polyglot/cmain` etc.
   - Hash mismatch → You modified file content incorrectly, use `git checkout` or exact copy

### Step 4: Verify Completeness

```bash
# Check no extra files where they shouldn't be
ls /app/output_dir/

# Check no files left where they shouldn't be
ls /app/documents/  # should be empty if task said to move all files

# Verify exact content
diff /app/expected /app/actual
```

### Step 5: Edge Case Testing

- Test with boundary inputs (0, 1, empty, large values)
- Test with the specific inputs mentioned in the task
- If there are multiple valid outputs, ensure ALL are included

## Phase 5: Cleanup & Delivery

Before declaring done:

1. **Remove all temporary/intermediate files:**
   ```bash
   rm -f /tmp/test_*.py /app/*.pyc
   # Remove compiled binaries if not part of deliverable
   rm -f /app/polyglot/cmain  # example
   ```

2. **Final test run:**
   ```bash
   cd /tests && python -m pytest test_outputs.py -v 2>&1
   ```
   If this is not possible (no tests found), verify your output manually against every requirement in the task.

3. **Verify all task requirements are met:**
   - Requirements checklist from Phase 1 shows ALL items `[DONE]`
   - Every unit test written in Phase 4 Step 2 passes
   - Every requested file exists at the correct path
   - Every file contains the correct content
   - No extra files or artifacts remain
   - All data values are correct (not approximate)

## Language-Specific Debugging Guardrails

When the task involves debugging code in specific languages:

### Rust
- Respect the borrow checker. Prefer `clone()` over fighting lifetimes if performance is acceptable.
- Use `?` operator for error propagation. Avoid `unwrap()` in production paths.
- Minimize `unsafe`. If used, verify all invariants manually.

### Go
- Every `err` return MUST be checked. Never `_ = err`.
- Verify nil pointers and interface values before use.
- Check goroutine leaks, channel closure, and `defer` ordering.

### TypeScript
- Use type guards over `as` casting. Minimize `any`.
- Verify `Promise` chains and `await` usage.
- Enable and respect strict null checks.

### Python
- Verify matrix/tensor dimensions explicitly before operations (critical for ML tasks).
- Check off-by-one errors in indexing and ranking (e.g., "5th highest" = index 4).
- Use `with` for file handling to prevent resource leaks.

### C/C++
- Check buffer sizes, null terminators, and pointer arithmetic.
- Verify `malloc`/`free` pairing and use-after-free.
- Test with sanitizers when available: `-fsanitize=address,undefined`.

### General (all languages)
- Compile/lint before and after changes.
- Reproduce the bug first, then fix.
- Fix root causes, not symptoms.
- Test the exact failing scenario plus edge cases.

## Forbidden Patterns (NEVER DO)

- **NEVER refuse to attempt a task.** Use tools, scripts, and libraries creatively.
- **NEVER submit without running tests.** "It looks correct" is not verification.
- **NEVER assume "it compiles" means "it works."** Always run with actual inputs.
- **NEVER read large binary files (images, videos, PDFs) directly into context.** Write scripts to process them.
- **NEVER guess answers for specialized domains.** Use engines and libraries (chess engines, SAT solvers, embedding models, etc.).
- **NEVER leave build artifacts in output directories** unless explicitly requested.
- **NEVER suppress errors** by commenting out code, using `try/except: pass`, `unwrap()`, `!!`, etc.
- **NEVER ignore test failures.** If a test fails, fix your work and re-run.
- **NEVER test against only `/app/` files when `/tests/` versions exist.** The `/tests/` directory contains the verifier's version and may differ.
