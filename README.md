<div align="center">

# CDNEE Programming Language

### *Control Dynamic Native Execution Engine*

**Version 1.0 · Powered by Pythonaibrain-NASM / DNEE v1.0**

</div>

---

## What is CDNEE?

CDNEE (Control Dynamic Native Execution Engine) is a high-level assembly-oriented programming language designed for direct access to native machine-code execution without the complexity of traditional assemblers or low-level compilers. It sits at a unique intersection: **readable, structured syntax** on the surface, **bare-metal native performance** underneath.

Programs are written in `.cdnee` source files using a clean, instruction-like syntax and run through the `CDNEE` runtime, which compiles and executes x86-64 machine code on the fly using a built-in JIT engine backed by NASM.

CDNEE is not a scripting language. It is not a systems language with manual memory management. It is an **execution engine language** — one where you can drop into raw NASM assembly, benchmark it, patch it at runtime, inspect its disassembly, snapshot its memory, and call foreign libraries, all from a single coherent source file.

---

## Design Philosophy

CDNEE is built around four principles:

**1. Native First**
Every math operation, every sort, every hash function goes through the DNEE engine's NASM-compiled native library. There is no slow path and no interpreted fallback by default. The language exposes what the CPU does, not what is convenient.

**2. JIT as a First-Class Citizen**
Inline NASM assembly is not an escape hatch — it is a core language feature. You write assembly snippets directly in source using triple-quoted blocks, compile them to executable memory regions with a single instruction, and call them like any other function. Regions have names (tags), can be benchmarked, patched, cloned, snapshotted, exported to disk, and reloaded in future runs.

**3. Introspection and Control**
A CDNEE program can disassemble its own compiled regions, compute their checksums, walk their patch history, inspect CPU cache topology, and read cycle-accurate timing via RDTSC. You always know what is in memory and how it is performing.

**4. Structured Simplicity**
Despite its low-level power, CDNEE code reads linearly. There are no pointers in source syntax, no manual stack management, no header files. Labels, functions, error handling, and I/O all follow a clean, consistent grammar that a programmer can learn in an afternoon.

---

## Language Tiers

CDNEE is organized into two execution tiers.

### Tier 1 — Native Math Library

Tier-1 operations call into DNEE's built-in NASM math library — a collection of highly optimized x86-64 routines covering:

- **Scalar arithmetic** — ADD, SUB, MUL, DIV, MOD, POW, ABS, NEG, SIGN
- **Integer math** — ISQRT, GCD, LCM, FACTORIAL, FIBONACCI, MULDIV, CLAMP
- **Number theory** — IS_PRIME, NEXT_PRIME, PREV_PRIME, TOTIENT, COLLATZ, IS_PERFECT, IS_ABUNDANT, IS_DEFICIENT
- **Bit operations** — POPCOUNT, NLZ, NTZ, BIT_REV, ROL64, ROR64, BSWAP64, PARITY64, IS_POW2, NEXT_POW2, LOG2, LOG10
- **Hashing** — HASH_FNV64 (FNV-1a 64-bit), HASH_MURMUR64
- **Modular arithmetic** — MULMOD64, ADDMOD64, SUBMOD64, POWMOD64, MODINV64
- **Safe arithmetic** — SAFE_ADD, SAFE_SUB, SAFE_MUL (with overflow flags), DIVMOD
- **Sorting & searching** — SORT_NET4, BIN_SEARCH, LIN_SEARCH, LOWER_BOUND
- **Timing** — RDTSC_START, RDTSC_STOP, CPUID

Tier-1 calls require no compilation step. They are available immediately in any CDNEE program.

### Tier 2 — JIT / MEX Engine

Tier-2 gives you the full power of the DNEE Machine Execution (MEX) engine:

- **JIT compilation** — compile inline NASM or `.asm` files into executable memory regions
- **Binary I/O** — export compiled regions to `.bin` files; reload them in future runs
- **Memory control** — freeze/thaw regions, take snapshots, restore, clone, copy-on-write
- **Hot patching** — overwrite bytes, NOP ranges, replace hex patterns, revert patches step-by-step
- **Inspection** — disassemble regions, hex-dump, compute CRC-32, walk patch history
- **FFI** — load shared libraries, bind exported symbols with type signatures, call them as native functions
- **Profiling** — RDTSC-accurate benchmarking, profile blocks, per-region call counters, cache statistics

---

## Program Structure

A CDNEE source file is plain text with the `.cdnee` extension. Execution begins at the label declared by `@entry`, or at the top of the file if no entry is declared.

```cdnee
; program.cdnee
@entry main

main:
    PRINTLN "Hello, CDNEE!"
    EXIT
```

- Comments start with `;` or `#` and run to the end of the line.
- `@entry <label>` names the entry point (optional).
- Labels are identifiers followed by `:` and act as jump targets or function markers.
- Dot-prefixed labels (`.loop:`, `.done:`) are local labels following NASM/GAS convention.
- Instructions are written one per line; no semicolons are needed to terminate statements.
- String literals use double quotes and support `\n`, `\t`, `\r`, `\\`, `\"`, `\0`.
- Integer literals may be decimal or hexadecimal (`0xFF`). Floats use a decimal point.
- Booleans are `TRUE` and `FALSE`.

---

## Variables

CDNEE variables are dynamically typed and untyped in source — you simply name them. They hold integers, floats, strings, or booleans.

```cdnee
SET x, 42
SET name, "Alice"
SET pi, 3.14159
SET flag, TRUE
LET y, 100          ; LET is an alias for SET
```

Several variables are automatically set by the runtime after certain operations:

| Variable | Set by |
|---|---|
| `_last_offset` | FIND |
| `_found_offset` | BIN_SEARCH, LIN_SEARCH |
| `_checksum` | CHECKSUM |
| `_disasm` | DISASM (when captured) |
| `_dump` | DUMP (when captured) |
| `_bench_mean` | BENCHMARK |
| `_bench_min` | BENCHMARK |
| `_asm_size` | ASM_SIZEOF |
| `_reverted` | PATCH_REVERT |
| `_profile_ms` | PROFILE_END |

---

## Functions

User-defined functions are declared with `FUNC` and closed with `END`. Parameters are named in the declaration and accessed as local variables inside the body.

```cdnee
FUNC square, n:
    MUL r, n, n
    RETURN r
END

main:
    SET result, CALL square, 9
    PRINTLN result          ; 81
    EXIT
```

Functions can call themselves recursively. The return value is captured by prefixing `CALL` with `SET result,`.

---

## Control Flow

CDNEE uses explicit compare-and-jump for all conditional logic, following an assembly-style discipline.

```cdnee
CMP a, b       ; compare a and b (sets internal state)
JE  label      ; jump if equal
JNE label      ; jump if not equal
JLT label      ; jump if less than
JGT label      ; jump if greater than
JLE label      ; jump if less than or equal
JGE label      ; jump if greater than or equal
JMP label      ; unconditional jump
```

Loops are built from labels and jumps:

```cdnee
SET i, 1
loop:
    CMP i, 10
    JGT done
    PRINTLN i
    ADD i, i, 1
    JMP loop
done:
    EXIT
```

---

## JIT Compilation

Inline NASM assembly is written inside triple-quoted blocks and compiled to a named memory region:

```cdnee
JIT_COMPILE_INLINE """
BITS 64
mov rax, rdi
imul rax, rdi
ret
""", tag=sq

SET result, CALL sq, 9     ; result = 81
```

Assembly can also be loaded from a `.asm` file:

```cdnee
JIT_COMPILE "square.asm", tag=sq
```

Multiple regions can be compiled in parallel:

```cdnee
JIT_COMPILE_MANY tag=sq "square.asm", tag=cube "cube.asm"
```

Regions are freed explicitly or all at once:

```cdnee
JIT_RELEASE sq
JIT_RELEASE_ALL
```

---

## Error Handling

```cdnee
TRY:
    SET r, CALL risky_fn, 42
CATCH err:
    PRINT "Error: "
    PRINTLN err
END
```

Any runtime error inside `TRY` is caught; the error message is stored in the named catch variable.

---

## Foreign Function Interface

CDNEE can load shared libraries and bind exported symbols at runtime:

```cdnee
; Load and bind a symbol
EXTERN_FUNC "libm.so.6", func=sqrt, alias=c_sqrt, argtypes=f64, rettype=f64

SET r, CALL c_sqrt, 2.0
PRINTLN r                  ; 1.4142...

; Pre-load a library
EXTERN_LIB "libssl.so.1.1", alias=ssl

; Import a .cdneef function definition file
EXTERN "mylib.cdneef"
```

---

## File Extensions

| Extension | Purpose |
|---|---|
| `.cdnee` | CDNEE source program |
| `.cdneef` | CDNEE function definition file (importable via EXTERN) |
| `.asm` | NASM assembly source for JIT_COMPILE |
| `.bin` | Exported compiled binary region |

---

## CLI Reference

```
DNEE_CLI [OPTIONS] [file.cdnee]

  -h, --help             Show help and exit
  -v, --verbose          Verbose output during execution
  -V, --version          Print runtime version
  --no-dnee              Disable JIT; use software math fallback only
  --lex                  Tokenise source and print token stream
  --ast                  Parse source and print AST
  --fmt                  Pretty-format source
  --disasm TAG           Disassemble region TAG after run
  --dump TAG             Hex-dump region TAG after run
  --stats                Print JIT and cache statistics after run
  --bench TAG [args...]  Benchmark region TAG after run
  --bench-iters N        Benchmark iteration count (default 100000)
  --strict-wx            Enforce W^X memory protection
  --workers N            JIT thread-pool size (default 4)
  --math PATH            Path to custom math.asm
  --out FILE             Redirect PRINT/PRINTLN output to FILE
  --repl                 Force REPL mode
```

---

## REPL

CDNEE includes an interactive REPL for exploratory use:

```
$ CDNEE

DNEE> RUN program.cdnee
DNEE> EXEC SET x, 42
DNEE> EXEC PRINTLN x
DNEE> DISASM sq
DNEE> DUMP sq
DNEE> JIT_STATS
DNEE> CACHE_STATS
DNEE> REGION_INFO sq
DNEE> BENCHMARK sq 9 iterations=500000
DNEE> VARS
DNEE> HELP
DNEE> EXIT
```

---

## Version History

| Version | Notes |
|---|---|
| 1.0 | ExternFileNode, ExternSymNode, ExternLibNode; CDNEE_CTYPES_MAP; `_source_dir` resolution |
| 1.0 | Dot-label support (`.name:`), lexer fixes FIX-1 through FIX-5 |
| 1.0 | JitCompileNode argtypes/rettype fields; DneeCallNode |
| 1.0 | Full Tier-2 JIT/MEX, binary export/import, patching, profiling |
| 1.0 | Initial release — Tier-1 math, control flow, functions |

---

## Author & Credits

**Language Design & Runtime:** Divyanshu Sinha / Pythonaibrain  
**JIT Backend:** Pythonaibrain-NASM · DNEE v1.0  
**Disassembler:** Integrated via DNEE engine  

---

<div align="center">

*CDNEE v1.0 — Write high. Run native.*

</div>
