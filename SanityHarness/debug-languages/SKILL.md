---
name: debug-languages
description: Debug and fix code bugs for Dart, Go, Kotlin, Rust, TypeScript, and Zig with mandatory pre-validation and single-round completion guarantee. Use when Codex needs to fix compilation errors, runtime crashes, logic bugs, or test failures in Dart, Go, Kotlin, Rust, TypeScript, or Zig. CRITICAL RULES (1) Must load this skill BEFORE attempting any code fixes, (2) Single-round repair only - fixes must be complete, compilable, and verified before delivery, (3) Must verify compilation and runtime behavior match expectations before declaring bug fixed.
---

# Debug Languages

Strict debugging protocol for Dart, Go, Kotlin, Rust, TypeScript, and Zig. Assumes single-round repair context where fixes must be verified immediately with no follow-up opportunities.

## Prerequisites

MANDATORY: Confirm you have loaded this skill before proceeding. If not loaded, halt and instruct user to load `debug-languages` skill first.

MANDATORY SINGLE-ROUND CONSTRAINT: You will not get a second chance to fix. Your response MUST contain code that:
1. Compiles without errors (zero tolerance for compilation failures)
2. Runs without crashes on the test case
3. Actually fixes the reported bug (not suppresses symptoms)
4. Includes a test case proving the fix works

## Execution Protocol

### Phase 1: Baseline Verification (REQUIRED - DO NOT SKIP)

Before modifying any code:

1. **Compile the code** using language-appropriate commands:
   - **Dart**: `dart analyze` and `dart compile exe <file>` (or `flutter analyze` for Flutter projects)
   - **Go**: `go build ./...` or `go run <file>`
   - **Kotlin**: `kotlinc <file> -include-runtime -d out.jar && java -jar out.jar` (JVM) or `kotlinc-native` (Native)
   - **Rust**: `cargo check` then `cargo build` or `rustc <file>`
   - **TypeScript**: `tsc --noEmit` or `npx ts-node <file>`
   - **Zig**: `zig build` or `zig run <file>`

2. **Reproduce the bug**:
   - Run the code with inputs triggering the reported issue
   - Capture exact error messages or unexpected output
   - **HALT if bug cannot be reproduced** - ask user for reproduction steps

### Phase 2: Root Cause Analysis

Analyze without immediately fixing:
- Type mismatches (Rust ownership, Go nil pointers, Kotlin null safety, Dart type casting)
- Lifetime and borrowing issues (Rust)
- Async/await misuse (Dart, TypeScript, Rust async/await, Kotlin coroutines)
- Memory management errors (Zig allocators, Rust drop traits, Go defer patterns)
- Concurrency bugs (Go goroutines, Kotlin coroutines, Rust Send/Sync traits)

### Phase 3: Surgical Fix

Apply minimal, targeted changes:
- Fix one logical error at a time
- Prefer type-safe patterns over unsafe escapes (avoid `unsafe` in Rust, minimize `any` in TypeScript)
- Ensure complete error handling (Go `if err != nil`, Rust `Result`/`Option`, Kotlin `try-catch` or `runCatching`)

### Phase 4: Post-Fix Verification (REQUIRED - DO NOT SKIP)

You MUST verify before responding. No exceptions.

1. **Compilation verification**:
   - Run compile commands from Phase 1
   - Must produce zero errors (treat warnings as errors if they indicate logic issues)
   - Fix all compiler warnings related to unsafe operations or unused results

2. **Runtime verification**:
   - Execute the exact scenario that previously failed
   - Verify output matches expected behavior exactly
   - Test boundary conditions and edge cases

3. **Test case construction** (If no existing tests available):
   - Create minimal reproduction case demonstrating the original bug
   - Verify test PASSES with your fix
   - Include test in your response

### Phase 5: Delivery Checklist

Before submitting response, verify ALL items:
- [ ] Code compiles without errors (warnings treated as errors for unsafe code)
- [ ] Bug is actually fixed (root cause addressed, not symptoms suppressed)
- [ ] No regressions introduced (adjacent logic still works)
- [ ] Test case provided and passing (or existing tests pass)
- [ ] Root cause explanation included in response

## Language-Specific Guardrails

### Dart / Flutter
- **Null safety**: Verify `?`, `!`, `late` usage. Never use `!` unless proven safe by prior null check.
- **Async/await**: Ensure `await` present in async functions; verify `Future` handling.
- **Flutter specific**: Never use `BuildContext` across async gaps.

### Go
- **Error handling**: Every `err` return value MUST be checked. Never ignore errors.
- **Nil safety**: Verify pointers and interface values before dereferencing.
- **Concurrency**: Check for race conditions, proper channel closure, and context cancellation handling.

### Kotlin
- **Null safety**: Avoid `!!` operator. Use `?.let` or Elvis operator `?:` instead.
- **Coroutines**: Verify scope cancellation and structured concurrency compliance.
- **Type system**: Check reified generics and inline function usage.

### Rust
- **Ownership**: Respect borrow checker. Prefer cloning over fighting borrow checker if performance acceptable.
- **Lifetimes**: If explicit lifetimes required, verify they don't over-constrain valid uses.
- **Unsafe**: Minimize unsafe blocks. If used, manually verify all invariants.
- **Error handling**: Use `?` operator. Avoid `unwrap()` or `expect()` in production paths.

### TypeScript
- **Type narrowing**: Use `typeof`, `instanceof`, or type guards. Minimize `as` casting.
- **Null safety**: Enable strict null checking behaviors.
- **Async**: Verify `Promise` return types and proper `await` usage.

### Zig
- **Memory safety**: Verify allocator usage (GeneralPurposeAllocator vs specific allocators). Check for leaks.
- **Error unions**: Functions returning errors must be called with `try` or `catch`.
- **Comptime**: Verify compile-time evaluation doesn't cause code bloat or infinite loops.

## Testing Patterns

When no test framework configured, construct manual verification:

```typescript
// TypeScript example pattern
function testFix() {
  const input = { /* reproduction case */ };
  const expected = { /* expected output */ };
  const result = fixedFunction(input);
  console.assert(
    JSON.stringify(result) === JSON.stringify(expected),
    `Failed: expected ${JSON.stringify(expected)}, got ${JSON.stringify(result)}`
  );
  console.log("✓ Test passed");
}
testFix();
```

Include similar verification for all languages. Use existing test frameworks (Cargo test, Go test, Jest, etc.) when available.

## Forbidden Patterns (NEVER DO)

- NEVER "fix" by commenting out error-causing lines without understanding root cause
- NEVER use `panic`, `unwrap`, `expect` (Rust), `!!` (Kotlin), or `!` (Dart) to suppress errors instead of fixing them
- NEVER ignore compiler warnings about unused variables (often indicate logic errors)
- NEVER assume "it compiles" means "it works" - always run the specific failing case
- NEVER deliver a fix without verifying against original bug reproduction steps