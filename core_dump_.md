# Core Dump — Complete Notes

> Covers: what a core dump is · setup · generating · filename decoding · GDB analysis

---

## Table of Contents

1. [What is a Core Dump?](#1-what-is-a-core-dump)
2. [Setup](#2-setup)
3. [Generating a Core Dump](#3-generating-a-core-dump)
4. [Core Filename Decoded](#4-core-filename-decoded)
5. [Analyzing with GDB](#5-analyzing-with-gdb)
6. [Key GDB Commands Reference](#6-key-gdb-commands-reference)
7. [Common Crash Types](#7-common-crash-types)
8. [Quick Reference](#8-quick-reference)

---

## 1. What is a Core Dump?

A **core dump** is a snapshot of a process's memory at the exact moment it crashed. It contains:

- The call stack (what functions were executing)
- All local and global variables
- Register values
- Memory mappings
- The signal that caused the crash

> **Analogy:** Like a black box recorder for your program — it captures everything at the moment of crash so you can replay and investigate it later.

**When is a core dump created?**

The OS automatically generates one when a process receives a fatal signal:

| Signal | Number | Cause |
|---|---|---|
| `SIGSEGV` | 11 | Segmentation fault (invalid memory access) |
| `SIGABRT` | 6 | `abort()` called, assertion failed |
| `SIGBUS` | 7 | Bus error (misaligned memory access) |
| `SIGFPE` | 8 | Floating point / division by zero |
| `SIGILL` | 4 | Illegal instruction |

---

## 2. Setup

Three things must be in place before a core dump can be generated.

### Step 1 — Compile with Debug Symbols

```bash
gcc -g -o main main.c
```

The `-g` flag embeds debug information into the binary:
- Source file names and line numbers
- Variable names and types
- Function signatures

Without `-g`, GDB can still load the core but will only show raw addresses — no function names, no line numbers, no variable values.

### Step 2 — Set the Core Pattern

```bash
sudo sysctl -w kernel.core_pattern=core.%p.%s.%c.%d.%P
```

This tells the kernel **where and how to name** the core file when it is written.

> On Ubuntu/Debian, Apport intercepts crashes by default. This command overrides Apport so the core file is written **locally in the current directory** instead.

Running without `sudo` will fail:
```
sysctl: permission denied on key "kernel.core_pattern", ignoring
```

**Core pattern tokens:**

| Token | Meaning |
|---|---|
| `%p` | PID of the crashed process |
| `%P` | Global PID (same as `%p` in most cases) |
| `%s` | Signal number that caused the crash |
| `%c` | Core file size soft limit at time of crash |
| `%d` | Dump mode |
| `%e` | Executable filename |
| `%t` | Unix timestamp of crash |
| `%u` | UID of the process |
| `%h` | Hostname |

**Make the pattern permanent** (survives reboot):
```bash
echo "kernel.core_pattern=core.%p.%s.%c.%d.%P" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Step 3 — Enable Core Dumps

```bash
ulimit -c unlimited
```

By default the core file size limit is `0` — meaning core dumps are silently suppressed. This command removes that limit for the current shell session.

```bash
ulimit -c           # check current limit
ulimit -c unlimited # allow any size
ulimit -c 524288    # limit to 512 MB (in KB)
```

> `ulimit` only applies to the current terminal session. To make it permanent, add it to `~/.bashrc` or `/etc/security/limits.conf`.

---

## 3. Generating a Core Dump

### Full Session Example

```bash
userdev@device:~/tut$ gcc -g -o main main.c
userdev@device:~/tut$ sudo sysctl -w kernel.core_pattern=core.%p.%s.%c.%d.%P
kernel.core_pattern = core.%p.%s.%c.%d.%P
userdev@device:~/tut$ ulimit -c unlimited
userdev@device:~/tut$ ./main
Segmentation fault (core dumped)
userdev@device:~/tut$ ls
core.181642.11.18446744073709551615.1.181642  main  main.c
```

### What Happened

1. Program crashed with a segmentation fault
2. Kernel received `SIGSEGV` (signal 11)
3. Kernel wrote a snapshot of the process memory to disk
4. Core file appeared in the current directory

### Manually Triggering a Core Dump on a Running Process

```bash
# Send SIGABRT to a running process (forces core dump):
kill -SIGABRT <pid>

# Or use gcore to dump without killing the process:
gcore -o mycore <pid>
# Creates: mycore.<pid>  (process keeps running)
```

---

## 4. Core Filename Decoded

Pattern used: `core.%p.%s.%c.%d.%P`

Resulting filename: `core.181642.11.18446744073709551615.1.181642`

| Token | Value | Meaning |
|---|---|---|
| `%p` | `181642` | Process ID |
| `%s` | `11` | Signal number — `SIGSEGV` (segfault) ✅ |
| `%c` | `18446744073709551615` | Core size limit — `unlimited` (max uint64) |
| `%d` | `1` | Dump mode |
| `%P` | `181642` | Global PID |

---

## 5. Analyzing with GDB

### Load the Core Dump

```bash
gdb ./main core.181642.11.18446744073709551615.1.181642
```

Format: `gdb <executable> <core-file>`

The executable is needed because the core file does not contain the program code — only the memory state. GDB combines both to reconstruct the crash.

### Disable debuginfod (Recommended First Step)

When GDB starts, it may prompt:

```
This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n])
```

Disable it to avoid GDB hanging on network requests for symbol files:

```
(gdb) set debuginfod enabled off
```

To disable it permanently so the prompt never appears, add this to `~/.gdbinit`:

```
set debuginfod enabled off
```

---

### What You See on Load

```
GNU gdb (Ubuntu 12.1) ...
Reading symbols from ./main...
[New LWP 181642]
Core was generated by `./main'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x000055a1b3e0119e in crash_function (ptr=0x0) at main.c:7
7           *ptr = 42;
(gdb)
```

GDB immediately tells you:
- Which signal killed the process (`SIGSEGV`)
- Which function was executing (`crash_function`)
- Which exact line (`main.c:7`)
- What the bad pointer value was (`ptr=0x0` — a NULL dereference)

---

## 6. Key GDB Commands Reference

### Navigation

| Command | Short | What It Does |
|---|---|---|
| `backtrace` | `bt` | Show full call stack at time of crash |
| `backtrace full` | `bt full` | Call stack + all local variables |
| `frame <n>` | `f <n>` | Switch to stack frame N |
| `up` | | Move up one frame (toward caller) |
| `down` | | Move down one frame (toward crash) |
| `info frame` | | Details about current frame |

### Variables & Memory

| Command | What It Does |
|---|---|
| `print <var>` | Print value of a variable |
| `print *<ptr>` | Dereference and print a pointer |
| `print &<var>` | Print address of a variable |
| `display <var>` | Print variable on every step |
| `info locals` | All local variables in current frame |
| `info args` | All arguments of current function |
| `info registers` | CPU register values at crash |
| `x/10x <addr>` | Examine 10 hex words at address |
| `x/s <addr>` | Examine memory as a string |
| `x/i <addr>` | Examine memory as instruction |

### Threads (for multi-threaded crashes)

| Command | What It Does |
|---|---|
| `info threads` | List all threads |
| `thread <n>` | Switch to thread N |
| `thread apply all bt` | Show backtrace for every thread |

### Source & Symbols

| Command | What It Does |
|---|---|
| `list` | Show source code around current line |
| `list <function>` | Show source of a specific function |
| `info symbol <addr>` | Find symbol name at an address |
| `info address <sym>` | Find address of a symbol |
| `disassemble` | Show assembly of current function |

---

## 7. Common Crash Types

### NULL Pointer Dereference

```
#0  0x... in my_func (ptr=0x0) at main.c:7
7       *ptr = 42;
```

`ptr=0x0` — writing to address 0 (NULL). Fix: add a null check before dereferencing.

### Buffer Overflow / Stack Smash

```
Program terminated with signal SIGSEGV
#0  0x4141414141414141 in ?? ()
```

Return address overwritten with `0x41414141` (`AAAA`). Classic stack buffer overflow.

### Use After Free

```
#0  0x... in my_func (obj=0x614000000080) at main.c:15
(gdb) print *obj
Cannot access memory at address 0x614000000080
```

Address looks valid but memory was already freed. Use AddressSanitizer (`-fsanitize=address`) to catch these.

### Assertion Failure

```
Program terminated with signal SIGABRT
#0  0x... in raise ()
#1  0x... in abort ()
#2  0x... in __assert_fail ()
#3  0x... in my_func () at main.c:22
22      assert(count > 0);
```

`assert()` fired. Scroll up in the backtrace to frame 3 to see which assertion and what the actual value was.

### Typical Analysis Session

```bash
gdb ./main core.181642.11.18446744073709551615.1.181642
```

```
(gdb) bt
#0  crash_function (ptr=0x0) at main.c:7
#1  0x... in setup () at main.c:15
#2  0x... in main () at main.c:22

(gdb) frame 0
(gdb) info locals
ptr = 0x0
value = 42

(gdb) frame 1
(gdb) info locals
result = 0x0
config = 0x55a1b3e04260

(gdb) print config->name
$1 = 0x55a1b3e04280 "test"

(gdb) info registers
rip    0x55a1b3e0119e  (instruction pointer at crash)
rsp    0x7ffd5e3a1b20  (stack pointer)
```

Reading the backtrace from bottom to top: `main` called `setup`, `setup` called `crash_function`, `crash_function` crashed at line 7 because `ptr` was NULL.

---

## 8. Quick Reference

```
SETUP (one time per session):
  gcc -g -o main main.c
  sudo sysctl -w kernel.core_pattern=core.%p.%s.%c.%d.%P
  ulimit -c unlimited

CRASH → CORE FILE:
  ./main
  Segmentation fault (core dumped)
  ls → core.<pid>.<signal>.<limit>.<mode>.<gpid>

ANALYZE:
  gdb ./main <corefile>
  (gdb) set debuginfod enabled off   ← disable network symbol lookups

FIRST COMMANDS IN GDB:
  bt            ← where did it crash?
  bt full       ← crash location + all variables
  frame N       ← jump to specific frame
  info locals   ← variables in current frame
  info args     ← function arguments
  print <var>   ← inspect any variable
  thread apply all bt  ← all threads (multi-threaded)
```

**Signal number cheat sheet:**

| Number | Signal | Meaning |
|---|---|---|
| 4 | `SIGILL` | Illegal instruction |
| 6 | `SIGABRT` | Abort / assertion failed |
| 7 | `SIGBUS` | Bus error |
| 8 | `SIGFPE` | Arithmetic error |
| 11 | `SIGSEGV` | Segmentation fault |
| 13 | `SIGPIPE` | Broken pipe |
