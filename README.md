# CDNEE — Control Dynamic Native Execution Engine

> **v1.0.0 — Initial Release**

**CDNEE** is a high-level assembly-oriented programming language designed for direct access to native machine-code execution — without the complexity of traditional assemblers or low-level compilers. It sits at a unique intersection: **readable, structured syntax** on the surface, **bare-metal native performance** underneath.

Programs are written in `.cdnee` source files and run through the CDNEE runtime, which compiles and executes x86-64 machine code on the fly using a built-in JIT engine backed by NASM. You can drop into raw assembly, benchmark it, patch it at runtime, inspect its disassembly, snapshot its memory, and call foreign libraries — all from a single coherent source file.

**Platform:** Windows 10 / 11 (x86-64)  
**Powered by:** Pythonaibrain-NASM / DNEE v1.0

---

## Table of Contents

- [Philosophy](#philosophy)
- [Requirements](#requirements)
- [Installation](#installation)
- [Execution Tiers](#execution-tiers)
- [Language Syntax](#language-syntax)
- [Standard Library Tier 1](#standard-library-tier-1)
- [JIT and MEX Engine Tier 2](#jit-and-mex-engine-tier-2)
- [FFI Foreign Function Interface](#ffi-foreign-function-interface)
- [CLI Reference](#cli-reference)
- [OS Development Tier](#os-development-tier)
- [File Extensions](#file-extensions)
- [Version History](#version-history)
- [License](#license)

---

## Philosophy

CDNEE is built on four core principles:

**1. Native First** — Every math operation, sort, and hash function goes through the DNEE engine's NASM-compiled native library. No slow path, no interpreted fallback by default. The language exposes what the CPU does, not what is convenient.

**2. JIT as a First-Class Citizen** — Inline NASM assembly is not an escape hatch — it is a core language feature. Write assembly snippets directly in source using triple-quoted blocks, compile them into executable memory regions, and call them like any other function.

**3. Introspection and Control** — A CDNEE program can disassemble its own compiled regions, compute checksums, walk patch history, inspect CPU cache topology, and read cycle-accurate timing via RDTSC. You always know what is in memory and how it is performing.

**4. Structured Simplicity** — Despite its low-level power, CDNEE code reads linearly. No pointers in source syntax, no manual stack management, no header files. Labels, functions, error handling, and I/O all follow a clean, consistent grammar learnable in an afternoon.

---

## Requirements

Before installing CDNEE, ensure the following tools are installed and available on your system `PATH`:

| Dependency | Role | Minimum Version | Download |
|------------|------|-----------------|----------|
| **NASM** | Assembler | 2.15+ | [nasm.us](https://www.nasm.us/) |
| **GCC** (MinGW-w64 recommended) | Linker | Any recent | [winlibs.com](https://winlibs.com/) or [TDM-GCC](https://jmeubank.github.io/tdm-gcc/) |

Verify your setup before installing:

```cmd
nasm -v
gcc --version
```

Both commands must return version output. The CDNEE installer performs this check automatically and will alert you if either tool is missing or not on your `PATH`.

---

## Installation

1. Download **`CDNEE-Setup-v1.0.0.exe`** from the [Releases](../../releases) page.
2. Run the installer (no administrator rights required for the default install path).
3. The installer will **detect NASM and GCC** automatically:
   - ✅ **Detected** — installation proceeds normally, showing the detected version.
   - ❌ **Missing** — the installer pauses and tells you which tool is absent. Install the missing dependency, then re-run setup.
4. Follow the on-screen prompts to complete installation.
5. Once complete, both `cdnee` and `cdnee-cc` will be available from any terminal window.

---

## Execution Tiers

CDNEE is organized into three execution tiers:

**Tier 1 — Built-in Native Math Library**
Requires no compilation step. Available immediately in any program. All 50+ operations are backed by the DNEE engine's NASM-compiled routines, with a pure software fallback if the engine is unavailable.

**Tier 2 — JIT / MEX Engine (Machine Execution Engine)**
The full JIT engine. Compile inline NASM or `.asm` files into executable memory, benchmark them, patch them live, snapshot and restore state, export to disk, and reload across sessions.

**Tier 3 — OS Development**
Targets bare-metal OS development. Produces flat binaries (MBR/COM) or freestanding ELF64 kernels. The same source also runs in the interpreter's simulation mode (`--os-sim`) for unit-testing OS logic without QEMU.

---

## Language Syntax

Source files are plain UTF-8 text with the `.cdnee` extension. One statement per line, no semicolons. Comments start with `;` or `#`. Execution begins at the `@entry` label.

### Hello World

```cdnee
; hello.cdnee
@entry main

main:
    PRINTLN "Hello, CDNEE!"
    EXIT
```

### Variables and Arithmetic

```cdnee
SET  x,    42
LET  name, "Alice"
SET  pi,   3.14159
SET  flag, TRUE

ADD  result, x, 100    ; result = 142
MUL  sq,     x, x      ; sq = 1764
MOD  rem,    x, 5      ; rem = 2
PRINTLN result
```

### Control Flow

```cdnee
SET  i, 1
loop:
    CMP  i, 10
    JGT  done
    PRINTLN i
    ADD  i, i, 1
    JMP  loop
done:
    EXIT
```

### Functions

```cdnee
FUNC square, n:
    MUL  r, n, n
    RETURN r
END

main:
    SET  result, CALL square, 9
    PRINTLN result    ; -> 81
    EXIT
```

### Error Handling

```cdnee
TRY:
    SET  r, CALL risky_fn, 42
CATCH err:
    PRINT   "Error: "
    PRINTLN err
END
```

### Inline JIT Assembly

```cdnee
JIT_COMPILE_INLINE """
BITS 64
mov rax, rdi
imul rax, rdi
ret
""", tag=sq

SET result, CALL sq, 9    ; -> 81
PRINTLN result
```

### Conditional Jump Opcodes

| Opcode | Condition |
|--------|-----------|
| `JE label` | Jump if left == right (after CMP) |
| `JNE label` | Jump if left != right |
| `JLT label` | Jump if left < right |
| `JGT label` | Jump if left > right |
| `JLE label` | Jump if left <= right |
| `JGE label` | Jump if left >= right |
| `JMP label` | Unconditional jump |

### Auto-Set Runtime Variables

| Variable | Set By |
|----------|--------|
| `_last_offset` | `FIND` |
| `_found_offset` | `BIN_SEARCH`, `LIN_SEARCH` |
| `_checksum` | `CHECKSUM` |
| `_disasm` | `DISASM` (when captured) |
| `_dump` | `DUMP` (when captured) |
| `_bench_mean` | `BENCHMARK` |
| `_bench_min` | `BENCHMARK` |
| `_asm_size` | `ASM_SIZEOF` |
| `_reverted` | `PATCH_REVERT` |
| `_profile_ms` | `PROFILE_END` |

---

## Standard Library Tier 1

All Tier-1 operations run via the DNEE native engine when available, or fall back to pure Python. Every operation stores its result in the `dest` variable. No compilation step required.

### Scalar Math

```
ADD dest, a, b          SUB dest, a, b          MUL dest, a, b
DIV dest, a, b          MOD dest, a, b          POW dest, a, b
ABS dest, a             NEG dest, a             SIGN dest, a
ISQRT dest, a           LOG2 dest, a            LOG10 dest, a
MIN_VAL dest, a, b      MAX_VAL dest, a, b      CLAMP dest, val, lo, hi
CEIL_DIV dest, a, b     FLOOR_DIV dest, a, b    MULDIV dest, a, b, c
AVERAGE dest, a, b
```

### Number Theory

```
GCD dest, a, b          LCM dest, a, b          FACTORIAL dest, n
FIBONACCI dest, n       COLLATZ dest, n         TOTIENT dest, n
IS_PRIME dest, n        NEXT_PRIME dest, n      PREV_PRIME dest, n
IS_PERFECT dest, n      IS_ABUNDANT dest, n     IS_DEFICIENT dest, n
```

### Bit Operations

```
POPCOUNT dest, n        NLZ dest, n             NTZ dest, n
ROL64 dest, n, k        ROR64 dest, n, k        BIT_REV dest, n
BSWAP64 dest, n         PARITY64 dest, n        IS_POW2 dest, n
NEXT_POW2 dest, n
```

### Safe Arithmetic

```
SAFE_ADD dest, a, b, flag    ; flag = 1 on overflow
SAFE_SUB dest, a, b, flag
SAFE_MUL dest, a, b, flag
DIVMOD quot, rem, a, b
```

### Hashing and Modular Arithmetic

```
HASH_FNV64 dest, data, len      ; FNV-1a 64-bit hash
HASH_MURMUR64 dest, x           ; Murmur-style 64-bit mix
ADDMOD64 dest, a, b, m          SUBMOD64 dest, a, b, m
MULMOD64 dest, a, b, m          POWMOD64 dest, a, b, m
MODINV64 dest, a, m             ; modular inverse (extended GCD)
```

### Sort, Search and Timing

```
SORT_NET4 arr                        ; optimal sorting network for 4 elements
BIN_SEARCH dest, arr, n, target      ; binary search, -1 if not found
LIN_SEARCH dest, arr, n, target      ; linear search
LOWER_BOUND dest, arr, n, target     ; std::lower_bound equivalent
RDTSC_START dest    RDTSC_STOP dest  ; cycle-accurate timing
CPUID dest                           ; CPU identification
```

### Stdlib Example

```cdnee
@entry main

main:
    IS_PRIME   p, 97           ; p = 1
    NEXT_PRIME q, 100          ; q = 101
    TOTIENT    t, 12           ; t = 4

    SAFE_MUL r, 999999999, 999999999, ov
    CMP ov, 1
    JE  overflow_handler

    RDTSC_START t0
    FIBONACCI   f, 40
    RDTSC_STOP  t1
    SUB cycles, t1, t0
    PRINT "Fib(40)="
    PRINT f
    PRINT " cycles="
    PRINTLN cycles
    EXIT

overflow_handler:
    PRINTLN "Overflow detected!"
    EXIT
```

---

## JIT and MEX Engine Tier 2

### JIT Type System

JIT functions require explicit type annotations to build the correct `ctypes.CFUNCTYPE`. Multiple arguments use colon-separated types (e.g., `argtypes=i64:i64`).

| Type | ctypes Equivalent | Notes |
|------|-------------------|-------|
| `i64` | `c_int64` | Default for args and return |
| `u64` | `c_uint64` | -1 wraps to 2^64-1 |
| `i32` | `c_int32` | Truncated outside +-2 GiB |
| `u32` | `c_uint32` | 32-bit unsigned |
| `i8` / `u8` | `c_int8` / `c_uint8` | Wraps at 128 / 256 |
| `f64` | `c_double` | Python float (64-bit) |
| `f32` | `c_float` | Narrowed from float |
| `ptr` | `c_void_p` | Raw memory address |
| `bool` | `c_bool` | 0 / 1 |
| `void` | `None` | Return type only |

### Calling Convention

**x86-64 System V ABI:** Integer/pointer args in `rdi`, `rsi`, `rdx`, `rcx`, `r8`, `r9`. Float args in `xmm0`-`xmm5`. Integer return in `rax`. Float return in `xmm0`. Mixed-type functions use both register sets counted separately.

### JIT Operations

**Compile**
```cdnee
JIT_COMPILE "add.asm", tag=add_fn, argtypes=i64:i64, rettype=i64
JIT_COMPILE_INLINE """
BITS 64
mov rax, rdi
imul rax, rsi
ret
""", tag=mul, argtypes=i64:i64, rettype=i64, overwrite=true
JIT_COMPILE_MANY "a.asm", "b.asm", "c.asm"
JIT_COUNTED fn_name, tag=counted_fn
CALL_STATS counted_fn
```

**Export / Import**
```cdnee
EXPORT_BIN "./cache"
IMPORT_BIN "./cache/mul.bin", tag=mul
CALL_BIN   "./cache/mul.bin", 6, 7

BIN_CONTEXT "./cache/mul.bin", tag=mul:
    SET r, CALL mul, 6, 7
END
```

**Memory Control**
```cdnee
FREEZE  mul
THAW    mul
SNAPSHOT mul
RESTORE  mul
CLONE    mul, tag=mul2
COPY_ON_WRITE mul, offset=3, bytes=0x90, tag=mul_nop
```

**Hot Patching**
```cdnee
PATCH_NOP     mul, 3, 4
PATCH_INT64   mul, offset=0, value=42
PATCH_FILL    mul, offset=0, len=8, byte=0x90
PATCH_REPLACE mul, find=0xDEAD, replace=0xBEEF
PATCH_BYTES   mul, offset=0, bytes=0x48,0x89,0xC0
TRAMPOLINE    mul, offset=0, target=new_fn
PATCH_REVERT  mul, 1
PATCH_REVERT_ALL mul
```

**Inspection**
```cdnee
DISASM       mul, 32
DUMP         mul
FIND         mul, 0x90
CHECKSUM     mul
REGION_INFO  mul
PATCH_HISTORY mul
```

**Profiling**
```cdnee
BENCHMARK mul, 6, 7, iterations=1000000, warmup=100
PROFILE_BLOCK start:
    ; code
PROFILE_BLOCK end
JIT_STATS
CACHE_STATS
CACHE_WARM  mul
CACHE_CLEAR
CACHE_INVALIDATE mul
```

### JIT and Patching Example

```cdnee
@entry main

main:
    JIT_COMPILE_INLINE """
BITS 64
mov  rax, rdi
imul rax, rsi
ret
""", tag=mul, argtypes=i64:i64, rettype=i64

    SET r, CALL mul, 6, 7
    PRINTLN r              ; -> 42

    BENCHMARK mul, 6, 7, iterations=1000000, warmup=100

    SNAPSHOT mul
    SET r_, DISASM  mul, 32
    CHECKSUM mul
    PRINTLN  _checksum

    PATCH_NOP mul, 3, 4
    PATCH_REVERT mul, 1

    EXPORT_BIN "./cache"
    JIT_RELEASE mul
    EXIT
```

---

## FFI Foreign Function Interface

The CDNEE FFI has two complementary halves: **EXTERN file imports** (`.cdneef` header files that add pure-CDNEE functions) and **native symbol bindings** (shared libraries loaded via ctypes).

### EXTERN — .cdneef Headers

```cdnee
EXTERN "math_utils.cdneef"

EXTERN "vec2.cdneef", alias_prefix=v2_
EXTERN "vec3.cdneef", alias_prefix=v3_

SET r, CALL v2_add, 10, 20
SET r, CALL v3_add, 1, 2, 3
```

Path resolution order: (1) Absolute path. (2) Relative to the current `.cdnee` source file directory. (3) Relative to the current working directory. Circular imports are silently ignored.

### EXTERN_FUNC — Native Symbols

```cdnee
EXTERN_FUNC "m", func=sqrt, alias=fsqrt, argtypes=f64, rettype=f64
SET r, CALL fsqrt, 144.0
PRINTLN r     ; -> 12.0

EXTERN "kernel32", func=Sleep, alias=sleep_ms, argtypes=u32, rettype=void, cdecl=false
```

### Portable Short Library Names

| Short Name | Linux / macOS | Windows |
|------------|---------------|---------|
| `c` | `libc.so.6` | `msvcrt.dll` |
| `m` | `libm.so.6` | `msvcrt.dll` |
| `pthread` | `libpthread.so.0` | — |
| `dl` | `libdl.so.2` | — |
| `z` | `libz.so.1` | — |
| `kernel32` | — | `kernel32.dll` |
| `user32` | — | `user32.dll` |
| `ntdll` | — | `ntdll.dll` |
| `gdi32` | — | `gdi32.dll` |
| `ws2_32` | — | `ws2_32.dll` |
| `d3d11` | — | `d3d11.dll` |
| `d3d12` | — | `d3d12.dll` |
| `opengl` | `libGL.so.1` | `opengl32.dll` |

### FFI Example

```cdnee
@entry main

EXTERN_FUNC "c", func=abs, alias=iabs, argtypes=i32, rettype=i32
EXTERN_FUNC "m", func=pow, alias=fpow, argtypes=f64:f64, rettype=f64

main:
    SET a, CALL iabs, -123
    PRINTLN a         ; -> 123

    SET b, CALL fpow, 3.0, 8.0
    PRINTLN b         ; -> 6561.0
    EXIT
```

---

## CLI Reference

CDNEE ships two command-line tools: **CDNEE** (the interpreter / REPL) and **CDNEE-CC** (the ahead-of-time compiler).

### CDNEE — Interpreter

```
usage: CDNEE [OPTIONS] [file.cdnee]
```

| Flag | Description |
|------|-------------|
| `-h, --help` | Show help and exit |
| `-v, --verbose` | Verbose output during execution |
| `-V, --version` | Print runtime version |
| `--no-dnee` | Disable JIT; use software math fallback only |
| `--lex` | Tokenise source and print token stream |
| `--ast` | Parse source and print AST |
| `--fmt` | Pretty-format source |
| `--check` | Syntax check only — no execution |
| `--disasm TAG` | Disassemble region TAG after run |
| `--dump TAG` | Hex-dump region TAG after run |
| `--stats` | Print JIT and cache statistics after run |
| `--bench TAG` | Benchmark region TAG after run |
| `--bench-args [ARGS...]` | Arguments to pass when benchmarking |
| `--bench-iters N` | Benchmark iteration count (default 100000) |
| `--strict-wx` | Enforce W^X memory protection |
| `--workers N` | JIT thread-pool size (default 4) |
| `--math PATH` | Path to custom `math.asm` |
| `--out FILE` | Redirect PRINT/PRINTLN output to FILE |
| `--repl` | Force REPL mode |
| `--os-sim` | Run in OS simulation mode |
| `--os-state` | Dump `_os_state` dict after script execution |
| `--os-vga` | Render simulated 80x25 VGA text buffer after execution |
| `--os-image FILE` | Write accumulated OS_DB/DW/DD/DQ image bytes to FILE |

### CDNEE-CC — Compiler

```
usage: CDNEE-CC [OPTIONS] [source.cdnee]
```

| Flag | Description |
|------|-------------|
| `-o DIR, --output DIR` | Output directory for compiled files |
| `--name NAME` | Base name for output files |
| `--exe` | Produce a standalone executable |
| `--flat` | Compile to flat binary — `nasm -f bin`, no linker (MBR / COM) |
| `--os` | Compile to freestanding ELF64 kernel |
| `--abi {sysv,win64,auto}` | Select calling convention ABI |
| `--asm-only` | Stop after NASM generation — emit `.asm` only |
| `--show-asm` | Print generated NASM to stdout |
| `--no-export-all` | Export only the `@entry` symbol |
| `--keep-obj` | Keep the intermediate object file (`.o`) |
| `--load-test` | ctypes smoke-test the compiled output after build |
| `--list-exports` | List all exported symbols and exit |
| `--run` | Run the executable immediately after compiling (implies `--exe`) |
| `--run-args ...` | Arguments for the executable when `--run` is used |
| `--opt LEVEL` | Optimization: 0=none 1=peephole 2=peephole+strength (default: 0) |
| `--nasm-flag FLAG` | Pass extra flag to NASM |
| `--link-flag FLAG` | Pass extra flag to the linker |
| `--load-addr ADDR` | Kernel physical load address (default: `0x100000`) |
| `--ld-script FILE` | Custom LD linker script (auto-generated if omitted) |
| `--qemu` | After compiling, launch QEMU to run the result |
| `--qemu-args ...` | Extra arguments forwarded to QEMU |
| `--qemu-bin BIN` | QEMU binary to use |

**Usage examples:**

```cmd
:: Standard modes
CDNEE-CC program.cdnee                  :: compile to .so/.dll
CDNEE-CC program.cdnee -o dist/         :: output to dist/
CDNEE-CC program.cdnee --asm-only       :: emit NASM only
CDNEE-CC program.cdnee --show-asm       :: print NASM to stdout
CDNEE-CC program.cdnee --load-test      :: compile + ctypes smoke-test
CDNEE-CC program.cdnee --list-exports   :: show exported symbols
CDNEE-CC program.cdnee --exe            :: compile to executable
CDNEE-CC program.cdnee --exe --run      :: compile + run immediately
CDNEE-CC program.cdnee --opt 2          :: peephole + strength reduction

:: OS / flat-binary modes
CDNEE-CC boot.cdnee   --flat                    :: MBR / flat binary
CDNEE-CC kernel.cdnee --os                      :: freestanding ELF64 kernel
CDNEE-CC kernel.cdnee --os --load-addr 1M       :: load at 1 MiB
CDNEE-CC kernel.cdnee --os --ld-script k.ld     :: custom linker script
CDNEE-CC boot.cdnee   --flat --qemu             :: flat binary + QEMU
CDNEE-CC kernel.cdnee --os   --qemu             :: kernel + QEMU -kernel
```

### REPL Commands

```
$ CDNEE --repl

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

## OS Development Tier

CDNEE includes a complete OS-development tier. Write bootloaders, kernels, and bare-metal programs using high-level OS opcodes that compile to flat binaries or freestanding ELF64. The same source also runs in the interpreter's simulation mode for unit-testing OS logic without QEMU.

### Flat Binary (MBR / COM)

```cdnee
; Minimal x86 real-mode MBR — prints "Hi OS!" via BIOS int 0x10

OS_BITS 16
OS_ORG  0x7C00

OS_CLI
OS_STI

JIT_COMPILE_INLINE greet_bios """
    mov  si, msg
.loop:
    lodsb
    or   al, al
    jz   .done
    mov  ah, 0x0E
    int  0x10
    jmp  .loop
.done:
    hlt
msg: db "Hi OS!", 13, 10, 0
""" tag=greet_bios

CALL greet_bios, 0
OS_HLT
OS_BOOT_SIG    ; pad to 510 bytes + 0xAA55
```

Compile: `CDNEE-CC hello_mbr.cdnee --flat`
Run: `qemu-system-i386 -drive format=raw,file=hello_mbr.bin -nographic`

### ELF64 Kernel (Long Mode)

```cdnee
; Freestanding 64-bit kernel

OS_BITS 64
OS_MULTIBOOT 0x00000003

OS_GLOBAL kernel_start

kernel_start:
    OS_CLI
    OS_VGA_CLEAR 0x0F
    OS_VGA_PUTS 0, 0, "CDNEE Kernel v1.0", 0x0F
    OS_SERIAL_INIT 0x3F8, 115200
    OS_SERIAL_PUTS 0x3F8, "Kernel alive\r\n"
    OS_HLT
```

Compile: `CDNEE-CC kernel.cdnee --os`
Run: `qemu-system-x86_64 -kernel kernel.elf -serial stdio -nographic`

### OS Opcode Reference

#### Section and Layout

| Opcode | Syntax | Description |
|--------|--------|-------------|
| `OS_BITS` | `OS_BITS 16/32/64` | Set NASM instruction width — must be first in flat sources |
| `OS_ORG` | `OS_ORG addr` | Set origin address. `0x7C00` for MBR, `0x0100` for COM |
| `OS_SECTION` | `OS_SECTION ".text"` | Switch assembly section |
| `OS_GLOBAL` | `OS_GLOBAL sym` | Export symbol (NASM global directive) |
| `OS_ALIGN` | `OS_ALIGN n [, fill]` | Align to power-of-two boundary (default fill: `0x00`) |
| `OS_TIMES` | `OS_TIMES count, byte` | Repeat byte value count times |
| `OS_BOOT_SIG` | `OS_BOOT_SIG` | Pad to 510 bytes + emit `0xAA55`. Must be last in MBR. |
| `OS_MULTIBOOT` | `OS_MULTIBOOT [flags]` | Emit Multiboot 1 header for GRUB |

#### CPU Control and Interrupts

| Opcode | Syntax | Description |
|--------|--------|-------------|
| `OS_CLI` | `OS_CLI` | Clear interrupt flag (disable hardware IRQs) |
| `OS_STI` | `OS_STI` | Set interrupt flag (enable hardware IRQs) |
| `OS_HLT` | `OS_HLT` | Halt until next interrupt |
| `OS_NOP` | `OS_NOP` | Emit a single NOP instruction |
| `OS_IRET` | `OS_IRET [bits]` | Interrupt return — `iretq` (64-bit) or `iret` (16/32-bit) |
| `OS_RING` | `OS_RING 0/3` | Declare privilege ring |
| `OS_SET_ISR` | `OS_SET_ISR vector, handler` | Dynamically patch running IDT entry at vector to handler |

#### Hardware I/O, VGA and Serial

| Opcode | Syntax | Description |
|--------|--------|-------------|
| `OS_OUTB` | `OS_OUTB port, val` | Write byte to x86 I/O port |
| `OS_OUTW` | `OS_OUTW port, val` | Write word to x86 I/O port |
| `OS_INB` | `OS_INB dest, port` | Read byte from I/O port into variable |
| `OS_INW` | `OS_INW dest, port` | Read word from I/O port into variable |
| `OS_VGA_CLEAR` | `OS_VGA_CLEAR [attr]` | Fill 80x25 VGA text screen with attribute byte |
| `OS_VGA_PUTC` | `OS_VGA_PUTC col, row, char, attr` | Write one character at (col, row) |
| `OS_VGA_PUTS` | `OS_VGA_PUTS col, row, str, attr` | Write NUL-terminated string at (col, row) |
| `OS_SERIAL_INIT` | `OS_SERIAL_INIT port, baud` | Initialise 16550-compatible UART |
| `OS_SERIAL_PUTC` | `OS_SERIAL_PUTC port, char` | Transmit one byte |
| `OS_SERIAL_PUTS` | `OS_SERIAL_PUTS port, str` | Transmit a NUL-terminated string |

#### Descriptor Tables, Paging and Memory

| Opcode | Syntax | Description |
|--------|--------|-------------|
| `OS_GDT_ENTRY` | `OS_GDT_ENTRY base, limit, access, flags` | Emit 8-byte GDT segment descriptor |
| `OS_IDT_ENTRY` | `OS_IDT_ENTRY offset, selector, type_attr` | Emit 8-byte IDT gate descriptor |
| `OS_LGDT` | `OS_LGDT gdtr_label` | Load GDT register |
| `OS_LIDT` | `OS_LIDT idtr_label` | Load IDT register |
| `OS_MAP_PAGE` | `OS_MAP_PAGE virt, phys, flags` | Map one 4 KiB page (`0x1`=present, `0x2`=write, `0x4`=user) |
| `OS_UNMAP_PAGE` | `OS_UNMAP_PAGE virt` | Unmap page and flush TLB entry (INVLPG) |
| `OS_PHYS_ALLOC` | `OS_PHYS_ALLOC dest, pages` | Allocate consecutive 4 KiB physical frames |
| `OS_PHYS_FREE` | `OS_PHYS_FREE addr, pages` | Release physical frames |

#### Raw Data Directives

| Opcode | Syntax | Description |
|--------|--------|-------------|
| `OS_DB` | `OS_DB b1 [, b2, ...]` | Emit byte(s) — strings supported: `OS_DB "Hello", 0` |
| `OS_DW` | `OS_DW w1 [, w2, ...]` | Emit 16-bit little-endian word(s) |
| `OS_DD` | `OS_DD d1 [, d2, ...]` | Emit 32-bit dword(s) |
| `OS_DQ` | `OS_DQ q1 [, q2, ...]` | Emit 64-bit qword(s) — supports label references |

### Running in QEMU

```bash
# Flat binary (MBR bootloader)
CDNEE-CC hello_mbr.cdnee --flat
qemu-system-i386 -drive format=raw,file=hello_mbr.bin -nographic
qemu-system-i386 -drive format=raw,file=hello_mbr.bin -serial stdio

# ELF64 kernel (Multiboot)
CDNEE-CC kernel.cdnee --os --load-addr 0x100000
qemu-system-x86_64 -kernel kernel.elf -serial stdio -nographic

# Inspection
xxd hello_mbr.bin | tail -2              # check boot signature 55 AA
ndisasm -b 16 hello_mbr.bin | head -40  # disassemble MBR
readelf -S kernel.elf                    # ELF sections
objdump -d kernel.elf | head -80        # disassemble .text

# Interpreter simulation (no QEMU needed)
CDNEE --os-sim --os-vga --os-state kernel.cdnee
```

---

## File Extensions

| Extension | Description |
|-----------|-------------|
| `.cdnee` | CDNEE source program. Plain UTF-8 text. Entry point declared with `@entry` or defaults to top of file. |
| `.cdneef` | CDNEE function definition file. Contains only `FUNC` definitions and optional `EXTERN` imports. Supports `alias_prefix` for namespacing. |
| `.asm` | NASM assembly source loaded by `JIT_COMPILE`. Must begin with `BITS 64` for x86-64 code. Uses System V AMD64 calling convention. |
| `.bin` | Exported compiled binary region. Written by `EXPORT_BIN`, loaded by `IMPORT_BIN` or `CALL_BIN`. Bypasses the assembly step on subsequent runs. |

---

## Version History

| Version | Changes |
|---------|---------|
| v1.0.0 | **Initial Release.** OS development tier with `--flat` (MBR/COM) and `--os` (ELF64 kernel) compiler modes; full OS opcode set (OS_BITS, OS_ORG, OS_VGA_*, OS_SERIAL_*, OS_GDT_ENTRY, OS_IDT_ENTRY, OS_MAP_PAGE, OS_PHYS_ALLOC, OS_DB/DW/DD/DQ, OS_BOOT_SIG, OS_MULTIBOOT); QEMU integration; interpreter simulation mode (`--os-sim`, `--os-state`, `--os-vga`, `--os-image`). |
| v1.0.0 | FFI — `.cdneef` header imports with `alias_prefix` namespacing; `EXTERN_FUNC` native symbol bindings; portable short library name map. |
| v1.0.0 | Dot-label support (`.name:`) following NASM/GAS convention; lexer fixes FIX-1 through FIX-5; keyword label-peek disambiguation. |
| v1.0.0 | Full JIT type system: `i8/u8/i16/u16/i32/u32/i64/u64/f32/f64/ptr/bool/void`; `JitCompileNode` argtypes/rettype; `DneeCallNode`. |
| v1.0.0 | Tier-2 JIT/MEX engine — binary export/import (EXPORT_BIN, IMPORT_BIN, CALL_BIN, BIN_CONTEXT); memory operations (FREEZE, THAW, SNAPSHOT, RESTORE, CLONE, COPY_ON_WRITE, TRAMPOLINE); hot patching; profiling and stats. |
| v1.0.0 | Tier-1 math library (50+ native ops); control flow (CMP / Jxx / JMP); functions (FUNC/CALL/RETURN); error handling (TRY/CATCH/END); basic I/O (PRINT/PRINTLN). |

---

## License

CDNEE is released under the [LICENSE](LICENSE).

---

*CDNEE v1.0.0 — Control Dynamic Native Execution Engine*  
*Powered by Pythonaibrain-NASM / DNEE*  
*(c) 2026 Divyanshu Sinha — divyanshu.sinha631@gmail.com*
