# Generating and Analyzing a Core Dump

---

## Terminal Session

```bash
userdev@device:~/tut$ gcc -g -o main main.c
userdev@device:~/tut$ ls
main  main.c  steps.txt

userdev@device:~/tut$ cat steps.txt
1. sysctl -w kernel.core_pattern=core.%p.%s.%c.%d.%P
2. ulimit -c unlimited
3. gdb ./main core

userdev@device:~/tut$ sysctl -w kernel.core_pattern=core.%p.%s.%c.%d.%P
sysctl: permission denied on key "kernel.core_pattern", ignoring

userdev@device:~/tut$ sudo sysctl -w kernel.core_pattern=core.%p.%s.%c.%d.%P
[sudo] password for userdev:
kernel.core_pattern = core.%p.%s.%c.%d.%P

userdev@device:~/tut$ ulimit -c unlimited
userdev@device:~/tut$ ./main
Segmentation fault (core dumped)

userdev@device:~/tut$ ls
core.181642.11.18446744073709551615.1.181642  main  main.c  steps.txt
```

---

## Explanation of What Happened

This is a demonstration of how to generate a core dump file locally. Here is the breakdown:

1. **Compile with debug symbols** — `gcc -g -o main main.c`. The `-g` flag is critical so `gdb` can show line numbers and variable names.
2. **Change core pattern** — `sudo sysctl -w kernel.core_pattern=core.%p.%s.%c.%d.%P`. This overrides Apport so the core file is written locally with a descriptive name.
3. **Enable core dumps** — `ulimit -c unlimited`. Allows the OS to write a core file of any size.
4. **Crash** — `./main` produced a segmentation fault.
5. **Result** — Core file created: `core.181642.11.18446744073709551615.1.181642`

---

## Filename Decoded

Pattern: `core.%p.%s.%c.%d.%P`

| Token | Value | Meaning |
|---|---|---|
| `%p` | `181642` | Process ID |
| `%s` | `11` | Signal number — 11 is `SIGSEGV` (segmentation fault) ✅ |
| `%c` | `18446744073709551615` | Core file size limit — "unlimited" |
| `%d` | `1` | Dump mode |
| `%P` | `181642` | Global PID |

---

## Next Step — Analyze with GDB

```bash
gdb ./main core.181642.11.18446744073709551615.1.181642
```

Then inside GDB:

```
(gdb) bt
```
