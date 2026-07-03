# Debugging

When a program crashes or gives wrong results, a debugger lets you stop
execution, inspect variables, and step through code. On PARAM Rudra the standard
command-line debugger is **`gdb`** (the GNU Debugger).

## Compile with debug symbols

Add `-g` so the binary carries source information (and lower the optimisation
level while debugging so line numbers map cleanly):

```bash
gcc -g -O0 program.c -o program        # -g adds debug symbols
```

## Start gdb

```bash
gdb ./program                           # load the executable
```

Run heavy debugging **inside an [interactive job](batch.md#interactive-jobs)**,
not on the login node:

```bash
salloc -A myproject -p cpu -N 1 -t 01:00:00
srun --pty bash
gdb ./program
```

## Essential gdb commands

| Command | What it does |
| --- | --- |
| `run [args]` | Start the program (stops on error/breakpoint) |
| `start` | Run and stop at the first line of `main` |
| `break <file:line>` | Set a breakpoint (`break foo.c:42`) |
| `break <line> if <cond>` | Conditional breakpoint |
| `watch <expr>` | Break when a variable/expression changes |
| `info breakpoints` / `info watchpoints` | List them |
| `continue` (`c`) | Resume to next breakpoint |
| `next` (`n`) | Step one source line, **over** function calls |
| `step` (`s`) | Step one source line, **into** function calls |
| `print <expr>` (`p`) | Show a variable/expression value |
| `list [line]` | Show nearby source |
| `backtrace` (`bt`) | Print the call stack |
| `delete <n>` | Remove a breakpoint/watchpoint |
| `help [cmd]` | Built-in help |

Typical session:

```gdb
(gdb) break main
(gdb) run
(gdb) next
(gdb) print myvar
(gdb) watch total          # stop when 'total' changes
(gdb) continue
(gdb) backtrace            # where am I when it crashed?
```

`gdb` also debugs multi-threaded programs (`info threads`, `thread <n>`).

## Investigating job failures

Jobs stop for two common reasons: **exceeding resource limits** or
**software errors**.

1. **Always capture output.** Keep `--output`/`--error` files — don't redirect to
   `/dev/null`. They are the first place to look.
   ```bash
   #SBATCH --output=%x-%j.out
   #SBATCH --error=%x-%j.err
   ```
2. **Resource limits.** If a job exceeds its walltime or memory, SLURM kills it.
   Right-size `--time`, `--mem`, node/task counts. Check what it used:
   ```bash
   sacct -j <jobid> --format=JobID,State,Elapsed,MaxRSS,ReqMem,ExitCode
   ```
3. **Exit codes.** Any non-zero exit code marks the job `FAILED`. If a signal
   killed it, the signal number is shown after the exit code (e.g. `9:0`).
4. **Why is it pending?**
   ```bash
   scontrol show job <jobid>      # read the 'Reason' field
   ```

## Common pitfalls (from real cases)

- **Integer overflow / implicit conversions** — a value too large for its type
  (e.g. a negative `int` passed to an `unsigned`) can silently produce huge
  numbers or crashes. Watch the suspect variables and check types.
- **Uncontrolled recursion** — a bad argument can cause deep recursion and a
  **stack overflow**. `ulimit -s unlimited` in job scripts helps large legit
  stacks, but fix the root cause.
- **Endianness** — a binary data file written on a different-endian machine may
  need byte reordering; most compilers offer a conversion option.
- **Mixed compilers** — compiling different modules with different compilers
  (e.g. `pgcc` + `icc`) can cause link/runtime errors. Use one toolchain.
- **Version mismatch** — compile and run with the **same** compiler/library
  versions (load identical Spack packages in build and job scripts).

Next: [Data Management](data.md) and [Policies](policies.md).
