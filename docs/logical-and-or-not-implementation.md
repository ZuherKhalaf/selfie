# Assignment: logical-and-or-not — Implementation Report

## Overview

Implemented `&&` (logical AND), `||` (logical OR), and `!` (logical NOT) Boolean operators in the C* compiler (`selfie.c`). All operators use **short-circuit (lazy) evaluation**: the right operand of `&&` is skipped when the left is false, and the right operand of `||` is skipped when the left is true.

**Grader result: Note 2 — 10/10 tests passed.**

---

## What Was Added

### 1. New Token Symbols (`selfie.c`)

```c
uint64_t SYM_LOGICALAND   = 39; // &&
uint64_t SYM_LOGICALOR    = 40; // ||
uint64_t SYM_EXCLAMATION  = 41; // !
```

### 2. Scanner Changes

- `!` alone → `SYM_EXCLAMATION`; `!=` still → `SYM_NOTEQ`
- `&&` → `SYM_LOGICALAND`; `&` alone → `SYM_BITWISEAND`
- `||` → `SYM_LOGICALOR`; `|` alone → `SYM_BITWISEOR`

### 3. New Grammar Productions

Added two new precedence levels **above** `compile_expression()` (comparison level):

```
logicalor  = logicaland { "||" logicaland }
logicaland = comparison { "&&" comparison }
comparison = bitwiseor [ ( "==" | "!=" | "<" | ">" | "<=" | ">=" ) bitwiseor ]
```

`&&` binds tighter than `||`, and both are below comparisons — matching standard C precedence.

### 4. `compile_logicaland()` — Short-Circuit AND

```
compile left → ta
if ta == 0: branch to false_label   (short-circuit: skip right)
compile right → tb
ta = (tb != 0)    via sltu(ta, zero, tb)
free(tb)
jal end_label
false_label: ta = 0
end_label:
```

Multiple `&&` are handled by a `while` loop, each iteration extending the chain.

### 5. `compile_logicalor()` — Short-Circuit OR

```
compile left → ta
if ta == 0: branch to check_right   (ta is false, must check right)
ta = 1                               (short-circuit: left was true)
jal end_label
check_right:
compile right → tb
ta = (tb != 0)    via sltu(ta, zero, tb)
free(tb)
end_label:
```

### 6. `!` (Logical NOT) in `compile_factor()`

Uses the identity `!x = (x < 1)` for unsigned 64-bit integers:

```c
talloc();
emit_addi(current_temporary(), REG_ZR, 1);          // tmp = 1
emit_sltu(previous_temporary(),                      // rd = (rd < 1) = (rd == 0)
          previous_temporary(), current_temporary());
tfree(1);
```

Result: 1 if `x == 0`, 0 if `x != 0`.

### 7. Call-site Updates

All places that previously called `compile_expression()` for a "full expression" were updated to call `compile_logicalor()`:
- `compile_assignment()` (LHS address and RHS value)
- `compile_factor()` (parenthesized expressions, both call sites)
- `compile_if()` and `compile_while()` (conditions)
- `compile_return()` (return value)
- Procedure call argument parsing

---

## Grammar Change (`grammar.md`)

Before:
```
expression = bitwiseor [ ( "==" | "!=" | "<" | ">" | "<=" | ">=" ) bitwiseor ] .
```

After:
```
expression = logicalor .
logicalor  = logicaland { "||" logicaland } .
logicaland = comparison { "&&" comparison } .
comparison = bitwiseor [ ( "==" | "!=" | "<" | ">" | "<=" | ">=" ) bitwiseor ] .
factor     = [ cast ] [ "-" ] [ "~" ] [ "!" ] [ "*" ] ... .
```

Symbol count updated from 22 → 25 (added `&&`, `||`, `!`).

---

## Key Design Decisions

- **Short-circuit from day one**: Implemented lazy evaluation immediately rather than eager evaluation, since the `lazy-evaluation` assignment (which follows) tests short-circuit behavior with infinite loops.
- **`!x` via `sltu rd, rd, 1`**: Elegant single-instruction logical NOT without branches. Works because `uint64_t` is unsigned: `x < 1` ⟺ `x == 0`.
- **No new RISC-V instructions**: `&&` and `||` compile entirely to `beq`, `jal`, `addi`, and `sltu` — instructions already in the RISC-U ISA.
- **Operator precedence**: `&&` wraps `comparison`; `||` wraps `&&`. This matches C's `||` < `&&` < relational precedence.

---

## Commit

`fce277cc` — pushed to `ZuherKhalaf/selfie` branch `main`.
