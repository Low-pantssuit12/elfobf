# 👾 elfobf


<div align="center">

[![Platform](https://img.shields.io/badge/platform-Linux-blue?logo=linux&logoColor=white)](https://www.linux.org/)
[![Language](https://img.shields.io/badge/language-Python%203-3776AB?logo=python)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

*Advanced ELF binary protection — Compress, Encrypt, Polymorph, Fileless Execute*

</div>

---

> [!WARNING]
> This tool is for educational purposes and authorized security auditing only. The author is not responsible for any misuse.

---

## 📖 Table of Contents | Оглавление

- [English](#english)
  - [📋 Overview](#-overview)
  - [✨ Features](#-features)
  - [🔒 Complete Protection Pipeline](#-complete-protection-pipeline)
  - [🚀 Quick Start](#-quick-start)
  - [⚙️ Deep Technical Analysis](#️-deep-technical-analysis)
  - [📁 Output](#-output)
  - [⚠️ Requirements](#️-requirements)
  - [🔧 Troubleshooting](#-troubleshooting)

- [Русский](#русский)
  - [📋 Обзор](#-обзор)
  - [✨ Возможности](#-возможности)
  - [🔒 Полный конвейер защиты](#-полный-конвейер-защиты)
  - [🚀 Быстрый старт](#-быстрый-старт)
  - [⚙️ Глубокий технический анализ](#️-глубокий-технический-анализ)
  - [📁 Результат](#-результат)
  - [⚠️ Требования](#️-требования)
  - [🔧 Устранение неполадок](#-устранение-неполадок)

---

# English

## 📋 Overview

**ELF-OBF** is a **military-grade** ELF binary obfuscation tool that transforms standard Linux executables into **heavily armored, self-decrypting, fileless-executing binaries**.

### What makes it unique?

| Component | Implementation |
|-----------|----------------|
| **Single SYSCALL Macro** | One unified `SYSCALL(n, a1...a6)` for ALL system calls |
| **Obfuscated Syscall Numbers** | XORed with random key at compile-time |
| **Shifted Syscall Numbers** | `memfd_create` and `execveat` are bit-shifted by random amounts |
| **RLE Compression** | Custom byte-level run-length encoding |
| **XOR Stream Cipher** | 16-128 byte key + salt + feedback mechanism |
| **Polymorphic C Stub** | 100% unique per build (4-254 char random identifiers) |
| **Dead Code Injection** | 25+ ASM patterns + 15+ register operation patterns |
| **SSTRIP Integration** | Embedded pre-compiled SSTRIP binary (removes section headers) |
| **Polyglot Signatures** | 27 different file format headers injected |
| **Fileless Execution** | `memfd_create` + `execveat` with `AT_EMPTY_PATH` |
| **Anti-Debug** | TracerPid check + PR_SET_PTRACER + PR_SET_DUMPABLE |
| **Memory Protection** | mlockall(MCL_ALL) + F_SEAL_ALL |
| **Highly Optimized Builder** | Memoryviews, pre-compiled SSTRIP, single-pass compression |

## ✨ Features

### Core Protection

| Feature | Description |
|---------|-------------|
| 🗜️ **RLE Compression** | Byte-level RLE with 0x80 repeat flag, typical 30-60% reduction |
| 🔐 **XOR Stream Cipher** | Salt + key mutation: `i = (x ^ ((y << 1) ^ (i >> 1))) & 0xFF` |
| 🎲 **Polymorphic Stub** | Random function/variable names from 129-char alphabet (Latin + Cyrillic + Ukrainian) |
| 📝 **Dead ASM Injection** | 25+ patterns: `nop`, `pause`, `clc`, `xor %rax,%rax`, `lea (%rbx), %rcx` |
| 🔧 **Dead C Injection** | Random junk statements in SYSCALL macro, ANTIDEBUG, and loader body |

### Compilation & Linking

| Feature | Description |
|---------|-------------|
| 🏗️ **MUSL/GCC Auto-detect** | Prefers MUSL for smaller output, falls back to GCC |
| 📦 **Custom Linker Script** | Random base address (0x400000 + random*0x1000), discards all unused sections |
| 🚫 **Compiler Flags** | `-Os -no-pie -fomit-frame-pointer -fno-stack-protector -Wl,--strip-all` |
| 🧹 **SSTRIP** | Removes section headers completely — breaks `objdump`, `readelf`, `gdb` |

### Runtime Protection

| Feature | Description |
|---------|-------------|
| 💾 **Fileless Execution** | `memfd_create("", MFD_ALLOW_SEALING)` → write → `execveat(fd, "", AT_EMPTY_PATH)` |
| 🛡️ **Anti-Debug** | Parses `/proc/self/status` for `TracerPid` (obfuscated string) |
| 🔒 **PTRACE Block** | `prctl(PR_SET_PTRACER, 0)` — only children can ptrace |
| 🚫 **No New Privs** | `prctl(PR_SET_NO_NEW_PRIVS, 1)` — prevents setuid escalation |
| 📵 **No Core Dumps** | `prctl(PR_SET_DUMPABLE, 0)` — blocks gcore and coredumps |
| 🔐 **Memory Lock** | `mlockall(MCL_ALL)` — prevents swapping to disk |
| 🔏 **Sealed Memory FD** | `fcntl(F_ADD_SEALS, F_SEALS_ALL)` — prevents modification before exec |

### Polyglot Signatures (27 formats)

| Signature | Format |
|-----------|--------|
| `\x7FELF` | ELF (native) |
| `MZ` + `PE` | Windows PE |
| `\x89PNG` | PNG Image |
| `%PDF` | PDF Document |
| `PK\x03\x04` | ZIP Archive |
| `\x1F\x8B` | GZIP |
| `\xFD\x37\x7A\x58\x5A` | XZ/LZMA |
| `\x21\x3C\x61\x72\x63\x68\x3E` | AR Archive |
| `\xCA\xFE\xBA\xBE` | Mach-O (32-bit) |
| `\xCF\xFA\xED\xFE` | Mach-O (64-bit) |
| `#!/bin/bash` | Shell Script |
| `UPX!` | UPX Packer |
| `\xFF\xD8\xFF\xE0` | JPEG |
| `BM` | BMP |
| `ID3` | MP3 |
| `ftypisom` | MP4 |
| `SQLite format 3` | SQLite |
| `\xD0\xCF\x11\xE0` | MS Office |
| `\x00\x61\x73\x6D` | WebAssembly |
| `\x7F\x45\x4C\x46` | ELF (alternative) |
| `\xFE\xED\xFA\xCE` | Mach-O (32-bit, alternative) |
| `\xBE\xBA\xFE\xCA` | Java Class |
| `\x99\x01` | DOS COM |
| `\x00\x01\x00\x00` | TrueType Font |
| `<?xml` | XML |
| `<!DOCTYPE` | HTML |
| `\x25\x21` | PostScript |
| `\x3C\x3F\x78\x6D\x6C` | XML Declaration |

## 🔒 Complete Protection Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 1: ELF VALIDATION                                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│ • Checks ELF magic (\x7FELF)                                                 │
│ • Validates x86_64 architecture                                              │
│ • Memory-maps file with MADV_SEQUENTIAL                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 2: RLE COMPRESSION                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│ Algorithm:                                                                   │
│   • Flag byte format:                                                        │
│     - 0x00-0x7F: literal run (1-127 bytes follow)                           │
│     - 0x80-0xFF: repeat run (0x80 = repeat 0, 0x81 = 1, ... 0xFF = 127)     │
│   • Minimum repeat threshold: 3 identical bytes                              │
│   • Maximum literal run: 126 bytes                                           │
│   • Compression ratio: 30-60% typical                                        │
│                                                                              │
│ Output buffer pre-allocated: size = org_sz + (org_sz // 127) + 3            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 3: XOR STREAM ENCRYPTION                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│ Key generation:                                                              │
│   • Length: 16-128 bytes (rand.randbelow(113) + 16)                         │
│   • Content: rand.token_bytes(kln)                                           │
│   • Salt: rand.randbelow(256)                                                │
│                                                                              │
│ Encryption loop (in-place):                                                  │
│   i = salt & 0xFF                                                            │
│   for j in range(size):                                                      │
│       k = key[(j + i) % key_len]                                             │
│       y = (k ^ (j & 0xFF) ^ (j >> 8)) & 0xFF                                 │
│       x = (data[j] ^ y ^ i) & 0xFF                                           │
│       x = ((x << 3) | (x >> 5)) & 0xFF    // rotate left 3                   │
│       x = (x + (y ^ 0xA5)) & 0xFF                                            │
│       data[j] = x                                                            │
│       i = (x ^ ((y << 1) & 0xFF) ^ (i >> 1)) & 0xFF                          │
│                                                                              │
│ Properties:                                                                  │
│   • State i depends on all previous bytes                                    │
│   • Non-linear due to rotation and addition                                  │
│   • /proc/self/status string also encrypted with same key                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 4: POLYMORPHIC LOADER GENERATION                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│ Random Identifiers (gen_chars):                                              │
│   • Length: 4-254 characters                                                 │
│   • Alphabet: 0-9, a-z, A-Z, _, а-я, А-Я, ґєії, ҐЄІЇ (129 chars)           │
│   • First char: never a digit                                                │
│                                                                              │
│ Generated names:                                                             │
│   • _bs, _be: payload start/end symbols                                      │
│   • ex: payload pointer variable                                             │
│   • kk: key array variable                                                   │
│   • sc: syscall function name                                                │
│   • lb_main, jmp_main: entry point labels                                    │
│   • entry, section: ELF entry point and section name                         │
│                                                                              │
│ Obfuscated Constants:                                                        │
│   • SYS_WRITE      = 1 ^ xor                                                 │
│   • SYS_FCNTL      = 72 ^ salt                                               │
│   • SYS_MLOCKALL   = 151 ^ xor                                               │
│   • SYS_PRCTL      = 157 ^ salt                                              │
│   • SYS_MEMFD_CREATE = (319 ^ salt) << shlmf  (shlmf = random 0-31)         │
│   • SYS_EXECVEAT   = (322 ^ xor) << shlex   (shlex = random 0-31)           │
│   • AT_EMPTY_PATH  = 0x1000 ^ salt                                           │
│   • PR_SET_PTRACER = 0x59616D61 ^ xor                                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 5: C COMPILATION                                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│ Compiler: MUSL (preferred) or GCC                                            │
│                                                                              │
│ Flags:                                                                       │
│   -s                 Strip symbols                                           │
│   -Os                Optimize for size                                       │
│   -no-pie            No position-independent code                            │
│   -nodefaultlibs     No default libraries                                    │
│   -nostdlib          No standard library                                     │
│   -nostartfiles      No startup files                                        │
│   -fvisibility=hidden Hide all symbols                                       │
│   -fomit-frame-pointer Omit frame pointer                                    │
│   -fno-builtin       No builtin functions                                    │
│   -fno-exceptions    No C++ exceptions                                       │
│   -fno-stack-protector No stack protection                                   │
│   -fno-ident         No .ident section                                       │
│   -fno-unwind-tables No unwind tables                                        │
│   -Wl,-O2            Linker optimization                                     │
│   -Wl,--build-id=none No build ID                                            │
│   -Wl,--strip-all    Strip all symbols                                       │
│                                                                              │
│ Linker Script:                                                               │
│   • Base address: 0x400000 + (rand(0-255) * 0x1000)                         │
│   • Single PT_LOAD segment with R+E permissions                              │
│   • All other sections discarded                                             │
│   • Random polyglot signature injected in .comment equivalent                │
└─────────────────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 6: SSTRIP                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│ Embedded SSTRIP binary (pre-compiled, ~13KB):                                │
│   • Parses ELF program headers                                               │
│   • Truncates section header table                                           │
│   • Zeroes e_shoff, e_shnum, e_shentsize                                     │
│   • Result: "ghost" ELF — runs but has no visible sections                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 🚀 Quick Start

### 📥 Download

```bash
git clone https://github.com/vk-candpython/elfobf.git
cd elfobf
```

### 📦 Requirements

```bash
# Install MUSL (recommended for smaller output)
sudo apt install musl-tools    # Debian/Ubuntu
sudo pacman -S musl            # Arch

# Or use GCC (auto-detected)
sudo apt install gcc
```

### 🏃 Usage

```bash
python3 elfobf.py <elf1> [elf2 ...]
```

**Example:**
```bash
python3 elfobf.py /bin/ls
```

**Output:**
```
[  OK  ]: architecture is valid (x86_64)
[  OK  ]: using compiler(/usr/bin/musl-gcc)

[ INFO ]: Start processing -> /bin/ls

[ INFO ]: obfuscating: ELF(/bin/ls)
[ INFO ]: compressing: {####################} 100%  (DONE)
[  OK  ]: compressed:  (142144 -> 89123) bytes  |  saved: 53021 bytes (37.3%)
[ INFO ]: encrypting:  {####################} 100%  (DONE)
[ BUILD ]: file(./_mkelfobf/link.ld)
[ BUILD ]: file(./_mkelfobf/payload.bin)
[ BUILD ]: file(./_mkelfobf/loader.c)
[ BUILD ]: file(./_mkelfobf/sstrip.elf)
[  OK  ]: compiled  .................  (DONE)
[  OK  ]: striped   .................  (DONE)
[ INFO ]: time      .................  (1.23s)
[  OK  ]: outfile(./elfobf-ls, 45678 bytes)
[  OK  ]: obfuscation complete ELF(/bin/ls)

[ INFO ]: End processing -> /bin/ls

[  OK  ]: cleanup build dir(./_mkelfobf)
```

## ⚙️ Deep Technical Analysis

### The SYSCALL Macro (One for ALL)

```c
#define SYSCALL(n, a1, a2, a3, a4, a5, a6) ({         \
    /* junk */ long _r; /* junk */                    \
    /* junk */ register long rax = (long)(n);         \
    /* junk */ register long rdi = (long)(a1);        \
    /* junk */ register long rsi = (long)(a2);        \
    /* junk */ register long rdx = (long)(a3);        \
    /* junk */ register long r10 = (long)(a4);        \
    /* junk */ register long r8  = (long)(a5);        \
    /* junk */ register long r9  = (long)(a6);        \
    /* junk */                                        \
    /* junk */ __asm__ volatile (                     \
        "junk_instruction\n"                          \
        "call %b"                                     \
            : "=a"(_r)                                \
            : "0"(rax),                               \
              "r"(rdi), "r"(rsi), "r"(rdx),           \
              "r"(r10), "r"(r8),  "r"(r9)             \
            : "rcx", "r11", "memory"                  \
    );                                                \
    /* junk */ _r;                                    \
})
```

**Key insights:**
- **Every syscall** uses this single macro
- **11 random junk statements** injected per macro expansion
- Syscall is made via `call %b` to a trampoline function `sc()`
- The trampoline contains random junk before/after `syscall` instruction
- Clobbers `rcx` and `r11` (x86_64 syscall convention)

### The Syscall Trampoline

```c
static void sc(void); __asm__ (
    ".section .text\n"
    ".hidden %b\n"
    "%b:\n"
    "    junk_instruction\n"
    "    syscall\n"
    "    junk_instruction\n"
    "    ret\n"
);
```

### RLE Decompression (Runtime)

```c
while (i < x) {
    uchar c = DEC(ex[i], i, kk, y, g); i++;
    
    if (c & 0x80) {     
        // Repeat run
        uint l = c & 0x7F;
        uchar v = DEC(ex[i], i, kk, y, g); i++;
        
        while (l--) {
            if (p == CHUNK) {
                SWRITE(d, o, CHUNK);
                ZEROS(o, CHUNK);
                p = 0;
            }
            o[p++] = v;
        }
    } else {              
        // Literal run
        uint l = c;
        
        while (l--) {
            if (p == CHUNK) {
                SWRITE(d, o, CHUNK);
                ZEROS(o, CHUNK);
                p = 0;
            }
            o[p++] = DEC(ex[i], i, kk, y, g); i++;
        }
    }
}
```

### DEC Macro (Decrypt Single Byte)

```c
#define DEC(b, j, k, l, g) ({                                         \
    uchar _b = (uchar)(b);                                            \
    uchar _i = (uchar)(g);                                            \
    uchar _k = (k)[ ((j) + _i) % (l) ];                               \
    uchar _y = ( _k ^ ((j) & 0xFF) ^ ((j) >> 8) ) & 0xFF;             \
    uchar _r = _b;                                                    \
                                                                      \
    _r  = (uchar)( (_r - (_y ^ 0xA5)) & 0xFF );                       \
    _r  = (uchar)( ((_r >> 3) | (_r << 5)) & 0xFF );                  \
    _r  = (uchar)( _r ^ _y ^ _i );                                    \
    (g) = (uchar)( (_b ^ ((_y << 1) & 0xFF) ^ (_i >> 1)) & 0xFF );    \
                                                                      \
    _r;                                                               \
})
```

**Note:** This is the **inverse** of the encryption function:
1. Subtract `(y ^ 0xA5)`
2. Rotate right 3 (inverse of left 3)
3. XOR with `y` and `i`
4. Update state `g`

### ANTIDEBUG Macro

```c
#define ANTIDEBUG() ({                                                \
    /* junk */ uchar _r; /* junk */                                   \
    uchar _e[] = PST;  // encrypted "/proc/self/status"               \
    uint  _l   = sizeof(_e);                                          \
    char  *_b  = __builtin_alloca(SIZE);                              \
    char  *_p  = _b;                                                  \
    uint   _y  = sizeof(kk);                                          \
    uint   _g  = SALT;                                                \
                                                                      \
    /* Decrypt PST string */                                          \
    for (uint _j = 0; _j < _l; _j++)                                  \
        *(_p++) = DEC(_e[_j], _j, kk, _y, _g);                        \
    *_p = '\0';                                                       \
                                                                      \
    int _d = SYSCALL(SYS_OPENAT, AT_FDCWD, _b, O_RDONLY, ...);        \
    if (_d < 0) { _r = 0; goto _ret; }                                \
                                                                      \
    int _n = SYSCALL(SYS_READ, _d, _b, SIZE-1, ...);                  \
    SYSCALL(SYS_CLOSE, _d, ...);                                      \
                                                                      \
    if (_n < 0 || _n < 12) { _r = 5; goto _ret; }                     \
                                                                      \
    _b[_n] = '\0';                                                    \
    volatile uchar _x = SALT, _z = XOR;                               \
                                                                      \
    /* Search for "TracerPid:" pattern (obfuscated) */                \
    for (uint _i = 0; _i < (_n - 12); _i++) {                         \
        if (                                                          \
            ((uchar)_b[_i] == (uchar)('T'^_z)) &&                     \
            ((uchar)_b[_i+6] == (uchar)('P'^_x)) &&                   \
            ((uchar)_b[_i+9] == (uchar)(':'^_z))                      \
        ) {                                                           \
            char *_s = &_b[_i + 10];                                  \
            while (*_s == (' '^_z) || *_s == ('\t'^_x)) _s++;         \
            _r = (uchar)*_s ^ (uchar)('0'^_z);                        \
            goto _ret;                                                \
        }                                                             \
    }                                                                 \
    _r = 13;                                                          \
_ret:                                                                 \
    ZEROS(_e, _l); ZEROS(_b, SIZE);                                   \
    _r;                                                               \
})
```

**Key insights:**
- `/proc/self/status` string is **encrypted in the binary**
- Pattern searched is **obfuscated**: `'T'^xor`, `'P'^salt`, `':'^xor`
- Returns `TracerPid` value (0 = no debugger, >0 = debugger attached)
- Memory is zeroed before return

### Fileless Execution

```c
// Create anonymous in-memory file
int d = SYSCALL((SYS_MEMFD_CREATE>>_f)^q, "", MFD_ALLOW_SEALING, ...);

// Decrypt and write payload in 4KB chunks
while (i < x) {
    // ... RLE decompression ...
    if (p == CHUNK) {
        SWRITE(d, o, CHUNK);
        ZEROS(o, CHUNK);
        p = 0;
    }
    o[p++] = byte;
}
if (p > 0) SWRITE(d, o, p);

// Seal file to prevent modification
SYSCALL(SYS_FCNTL^q, d, F_ADD_SEALS, F_SEALS_ALL, ...);

// Rewind
SYSCALL(SYS_LSEEK, d, 0, SEEK_SET, ...);

// Execute directly from memory!
SYSCALL((SYS_EXECVEAT>>_q)^f, d, "", argv, envp, AT_EMPTY_PATH^q, ...);
```

### ZEROS Macro (Secure Memory Wipe)

```c
#define ZEROS(b, l) do {                     \
    void *_p = (uchar*)(b) + ((l) - 8);      \
    long  _c = (long)(l) >> 3;               \
                                             \
    __asm__ volatile (                       \
        "std\n"                              /* Set direction flag (reverse) */ \
        "rep stosq\n"                        /* Store RAX (0) _c times */       \
        "cld\n"                              /* Clear direction flag */          \
        : "=D"(_p), "=c"(_c)                 \
        : "0"(_p),  "1"(_c), "a"(0)          \
        : "memory"                           \
    );                                       \
} while (0)
```

**Why reverse?** Harder for memory forensics to recover.

### Dead Code Injection Patterns

```python
JUNK = [
    b'nop', b'fnop', b'wait', b'pause', b'cld', b'clc', b'std\n\tcld', b'stc\n\tclc',
    # Register operations (generated combinatorially)
    b'or %b, %b', b'and %b, %b', b'not %b\n\tnot %b',
    b'neg %b\n\tneg %b', b'add $0, %b', b'sub $0, %b',
    b'shl $0, %b', b'shr $0, %b', b'rol $0, %b', b'ror $0, %b',
    b'test %b, %b', b'xchg %b, %b', b'mov %b, %b',
    b'push %b\n\tpop %b', b'lea (%b), %b'
]

# Generated for all registers: rax, rbx, rcx, rdx, rsi, rdi, r8, r9, r10, r11
# Total: 10 registers × 15 patterns = 150+ combinations
```

### Builder Optimizations

| Optimization | Implementation |
|--------------|----------------|
| **Memoryviews** | Zero-copy slicing, no intermediate allocations |
| **Pre-allocated Buffers** | Exact size calculation for RLE output |
| **MADV_SEQUENTIAL** | Hint kernel for sequential access |
| **Chunked I/O** | 4KB chunks for large files |
| **Pre-compiled SSTRIP** | Embedded as bytes, written once per session |
| **Single-pass Compression** | RLE compresses in one pass |
| **In-place Encryption** | XOR modifies buffer directly |
| **GC Control** | `gc.disable()` + manual `gc.collect()` |
| **Color Output Caching** | Memoized ANSI color codes |

## 📁 Output

```
original_elf  →  elfobf-original_elf
```

**Output binary properties:**
- Stripped of all symbols and section headers
- Contains encrypted payload in `.rodata`-like section
- Executes entirely from memory
- Leaves no traces on disk (except the binary itself)
- Typically **30-60% smaller** than original

## ⚠️ Requirements

| Requirement | Version | Notes |
|-------------|---------|-------|
| **Python** | 3.6+ | Required for builder |
| **MUSL-GCC** | Any | Recommended (smaller output) |
| **GCC** | Any | Fallback |
| **Linux Kernel** | 3.17+ | For `memfd_create` |
| **Linux Kernel** | 3.19+ | For `execveat` |
| **Architecture** | x86_64 | Only 64-bit |

## 🔧 Troubleshooting

| Issue | Solution |
|-------|----------|
| `Cannot determine architecture` | Only x86_64 supported |
| `No suitable compiler found` | Install `musl-tools` or `gcc` |
| `compilation failed` | Check compiler; try GCC if MUSL fails |
| `execveat: Function not implemented` | Kernel < 3.19 |
| `memfd_create: Function not implemented` | Kernel < 3.17 |
| Obfuscated binary segfaults | Original may need dynamic libraries |
| `gdb` still attaches | Use `PR_SET_PTRACER` + TracerPid |
| Binary size increased | Normal for very small binaries |

---

# Русский

## 📋 Обзор

**ELF-OBF** — это инструмент обфускации ELF-бинарников **военного уровня**, который превращает стандартные исполняемые файлы Linux в **сильно бронированные, саморасшифровывающиеся бинарники с бесфайловым выполнением**.

### Что делает его уникальным?

| Компонент | Реализация |
|-----------|------------|
| **Единый макрос SYSCALL** | Один `SYSCALL(n, a1...a6)` для ВСЕХ системных вызовов |
| **Обфусцированные номера syscall** | XOR со случайным ключом на этапе компиляции |
| **Сдвинутые номера syscall** | `memfd_create` и `execveat` сдвинуты на случайные биты |
| **RLE-сжатие** | Кастомное побайтовое кодирование длин серий |
| **Потоковый XOR-шифр** | Ключ 16-128 байт + соль + обратная связь |
| **Полиморфный C-заглушка** | 100% уникальна для каждой сборки |
| **Впрыск мёртвого кода** | 25+ ASM-паттернов + 15+ паттернов операций с регистрами |
| **Интеграция SSTRIP** | Встроенный предскомпилированный SSTRIP |
| **Полиглот-сигнатуры** | 27 различных заголовков форматов файлов |
| **Бесфайловое выполнение** | `memfd_create` + `execveat` с `AT_EMPTY_PATH` |
| **Анти-отладка** | Проверка TracerPid + PR_SET_PTRACER + PR_SET_DUMPABLE |
| **Защита памяти** | mlockall(MCL_ALL) + F_SEAL_ALL |
| **Высокооптимизированный билдер** | Memoryviews, предскомпилированный SSTRIP, однопроходное сжатие |

## ✨ Возможности

### Основная защита

| Возможность | Описание |
|-------------|----------|
| 🗜️ **RLE-сжатие** | Побайтовое RLE с флагом 0x80, типичное сжатие 30-60% |
| 🔐 **Потоковый XOR-шифр** | Соль + мутация ключа: `i = (x ^ ((y << 1) ^ (i >> 1))) & 0xFF` |
| 🎲 **Полиморфная заглушка** | Случайные имена из 129-символьного алфавита |
| 📝 **Впрыск мёртвого ASM** | 25+ паттернов: `nop`, `pause`, `clc`, `xor %rax,%rax`, `lea (%rbx), %rcx` |
| 🔧 **Впрыск мёртвого C** | Случайный мусор в макросе SYSCALL, ANTIDEBUG и теле загрузчика |

### Компиляция и линковка

| Возможность | Описание |
|-------------|----------|
| 🏗️ **Автовыбор MUSL/GCC** | Предпочитает MUSL для меньшего размера |
| 📦 **Кастомный линкер-скрипт** | Случайный базовый адрес, сбрасывает все неиспользуемые секции |
| 🚫 **Флаги компилятора** | `-Os -no-pie -fomit-frame-pointer -fno-stack-protector -Wl,--strip-all` |
| 🧹 **SSTRIP** | Полное удаление заголовков секций — ломает `objdump`, `readelf`, `gdb` |

### Защита времени выполнения

| Возможность | Описание |
|-------------|----------|
| 💾 **Бесфайловое выполнение** | `memfd_create("", MFD_ALLOW_SEALING)` → запись → `execveat(fd, "", AT_EMPTY_PATH)` |
| 🛡️ **Анти-отладка** | Парсинг `/proc/self/status` для `TracerPid` (обфусцированная строка) |
| 🔒 **Блокировка PTRACE** | `prctl(PR_SET_PTRACER, 0)` — только потомки могут трассировать |
| 🚫 **Запрет новых привилегий** | `prctl(PR_SET_NO_NEW_PRIVS, 1)` |
| 📵 **Запрет core-дампов** | `prctl(PR_SET_DUMPABLE, 0)` |
| 🔐 **Блокировка памяти** | `mlockall(MCL_ALL)` |
| 🔏 **Запечатанный memory FD** | `fcntl(F_ADD_SEALS, F_SEALS_ALL)` |

## 🔒 Полный конвейер защиты

*(См. английскую версию для подробной диаграммы)*

## 🚀 Быстрый старт

```bash
git clone https://github.com/vk-candpython/elfobf.git
cd elfobf
python3 elfobf.py /bin/ls
./elfobf-ls
```

## ⚙️ Глубокий технический анализ

### Макрос SYSCALL (Один на ВСЕ)

*(См. английскую версию для кода)*

**Ключевые моменты:**
- **Каждый системный вызов** использует этот единый макрос
- **11 случайных мусорных инструкций** внедряется при каждом раскрытии
- Системный вызов делается через `call %b` к функции-трамплину `sc()`
- Трамплин содержит случайный мусор до и после `syscall`

### RLE-распаковка (во время выполнения)

*(См. английскую версию для кода)*

### Макрос DEC (Расшифровка одного байта)

*(См. английскую версию для кода)*

**Обратная функция шифрования:**
1. Вычитание `(y ^ 0xA5)`
2. Поворот вправо на 3
3. XOR с `y` и `i`
4. Обновление состояния `g`

### Макрос ANTIDEBUG

*(См. английскую версию для кода)*

**Ключевые моменты:**
- Строка `/proc/self/status` **зашифрована в бинарнике**
- Ищется обфусцированный паттерн: `'T'^xor`, `'P'^salt`, `':'^xor`
- Возвращает значение `TracerPid` (0 = нет отладчика, >0 = есть)

### Бесфайловое выполнение

*(См. английскую версию для кода)*

### Макрос ZEROS (Безопасное затирание памяти)

*(См. английскую версию для кода)*

**Почему в обратном направлении?** Сложнее для memory forensics.

### Оптимизации билдера

| Оптимизация | Реализация |
|-------------|------------|
| **Memoryviews** | Zero-copy слайсинг |
| **Предвыделенные буферы** | Точный расчёт размера для RLE |
| **MADV_SEQUENTIAL** | Подсказка ядру о последовательном доступе |
| **Чанковый I/O** | Блоки по 4 КБ для больших файлов |
| **Предскомпилированный SSTRIP** | Встроен как байты, записывается один раз |
| **Однопроходное сжатие** | RLE сжимает за один проход |
| **Шифрование на месте** | XOR модифицирует буфер напрямую |
| **Контроль GC** | `gc.disable()` + ручной `gc.collect()` |
| **Кеширование цветов** | Мемоизация ANSI-кодов |

## 📁 Результат

```
оригинальный_elf  →  elfobf-оригинальный_elf
```

## ⚠️ Требования

| Требование | Версия | Примечания |
|------------|--------|------------|
| **Python** | 3.6+ | Для билдера |
| **MUSL-GCC** | Любая | Рекомендуется |
| **GCC** | Любая | Запасной |
| **Ядро Linux** | 3.17+ | Для `memfd_create` |
| **Ядро Linux** | 3.19+ | Для `execveat` |
| **Архитектура** | x86_64 | Только 64-бит |

## 🔧 Устранение неполадок

| Проблема | Решение |
|----------|---------|
| `Cannot determine architecture` | Только x86_64 |
| `No suitable compiler found` | Установи `musl-tools` или `gcc` |
| `compilation failed` | Проверь компилятор |
| `execveat: Function not implemented` | Ядро < 3.19 |
| `memfd_create: Function not implemented` | Ядро < 3.17 |
| Обфусцированный бинарник падает | Оригинал может требовать динамические библиотеки |

---

<div align="center">

**[⬆ Back to Top](#-elf-obf)**

*Military-Grade ELF Protection for Linux*

</div>
