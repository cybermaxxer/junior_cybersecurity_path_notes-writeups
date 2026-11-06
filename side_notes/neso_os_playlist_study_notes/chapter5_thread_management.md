# chapter 5: thread management

> **disclaimer:** i don't claim authorship of any of this. these are my study notes from the neso academy operating systems playlist, meant to complement the lectures, not replace them. watch the videos.

**videos covered (playlist order):**

| # | video | link |
|---|---|---|
| 1 | Introduction to Threads | https://www.youtube.com/watch?v=LOfGJcVnvAk |
| 2 | Multithreading Models & Hyperthreading | https://www.youtube.com/watch?v=HW2Wcx-ktsc |
| 3 | fork() and exec() System Calls | https://www.youtube.com/watch?v=IFEFVXvjiHY |
| 4 | Threading Issues [fork() & exec() System Calls] | https://www.youtube.com/watch?v=o2Sf0GHWpmk |
| 5 | Threading Issues (Thread Cancellation) | https://www.youtube.com/watch?v=wNns0kIDC68 |

## table of contents
- [introduction to threads](#introduction-to-threads)
- [multithreading models and hyperthreading](#multithreading-models--hyperthreading)
- [fork and exec system calls](#fork-and-exec-system-calls)
- [threading issues: fork and exec](#threading-issues-fork--exec-system-calls)
- [threading issues: thread cancellation](#threading-issues-thread-cancellation)
- [cheat sheet](#cheat-sheet)
- [interleaving](#interleaving)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## introduction to threads

a **thread** is the basic unit of cpu utilization. each thread has its own:
- thread id,
- program counter,
- register set,
- stack.

threads of the same process **share** the code section, data section, and os resources such as open files and signals *(see diagram, search anchor: "multi-threaded process")*.

a traditional **heavyweight** process has a **single** thread of control and does one task at a time. a multithreaded process does several at once.

**example:** a browser process with one thread rendering the page, another downloading a file, another handling clicks. with a single thread, downloading would freeze the display.

**four benefits:**

| benefit | meaning |
|---|---|
| responsiveness | part of a program can block or run long while the rest keeps responding to the user |
| resource sharing | threads share code, data, files by default, several activities in one address space |
| economy | creating and switching threads is cheaper than creating processes (no separate resources) |
| multiprocessor use | threads can run in parallel on different cpus; a single threaded process uses one cpu no matter how many exist |

*Q: what do threads of one process NOT share? A: their stack, registers, program counter and id.*

---

## multithreading models and hyperthreading

two kinds of threads:
- **user threads:** managed above the kernel, without kernel support.
- **kernel threads:** supported and managed directly by the os.

a relationship between them must exist. three models:

### many to one
many user threads map to **one** kernel thread; thread management is in user space (efficient).
- **limits:** if one thread makes a blocking system call, the **entire process blocks**; only one thread can reach the kernel at a time, so threads **cannot run in parallel on a multiprocessor**.

### one to one
each user thread maps to its own kernel thread.
- **pros:** more concurrency (a blocking call doesn't block the others); parallel on multiprocessors.
- **cons:** creating a user thread means creating a kernel thread (costly), so implementations often **limit the number of threads**.

### many to many
many user threads multiplex over a smaller or equal number of kernel threads (the count may depend on the application or machine).
- developers create as many user threads as needed; kernel threads run in parallel on multiprocessors; a blocking call lets the kernel schedule another thread.
- the best of the three, and used in most systems.

| model | blocking call blocks whole process? | parallel on multicore? | thread limit |
|---|---|---|---|
| many to one | yes | no | none |
| one to one | no | yes | often limited |
| many to many | no | yes | flexible |

### hyperthreading (simultaneous multithreading)
intel's name for smt: the physical cores of a processor are presented as **multiple logical processors**, so it can execute two threads at the same time. two streams in parallel is almost like two processors.

**check yours on windows:**

```
wmic
cpu get NumberOfCores, NumberOfLogicalProcessors
```

the lecture's example showed 2 cores and 4 logical processors, so hyperthreading is active. if logical equals physical, it isn't.

*Q: why does many to one waste a multicore cpu? A: one kernel thread runs on one core at a time.*

---

## fork and exec system calls

> these are specific to **linux/unix** systems. the lecture demonstrates them in c.

| call | effect |
|---|---|
| `fork()` | creates a separate **duplicate** process (the child) of the caller, with a **different process id**; same contents |
| `exec()` | the program passed as parameter **replaces the entire process** (all its threads); the **process id stays the same**; no new process |

**experiment 1: one print, then fork.** the program prints `hello neso academy` and its pid. after adding one `fork()` the line printed twice, with two different pids (parent and child).

**experiment 2: three forks.** the line printed **8 times**, each with a different pid. why: each fork doubles the number of processes: 1 then 2 then 4 then 8. so n forks produce 2 to the power n processes:

$$\text{processes} = 2^n$$

**experiment 3: exec.** `ex1.c` prints its pid, then calls `execv` with the output of `ex2` as the argument. result: `ex2`'s output appeared, its pid **equal** to `ex1`'s, and the last line of `ex1` ("back to ex1") **never printed**, because exec replaced the process and never returns.

compile tip from the demo: `gcc file.c -o name` (otherwise both outputs overwrite `a.out`).

*Q: why did "back to ex1" never print? A: exec replaced the whole process.*

---

## threading issues: fork and exec system calls

**issue 1: what does fork do in a multithreaded process?** if one thread calls `fork()`, does the new process duplicate **all threads**, or only the **calling thread**?

solution: some unix systems provide **two versions of fork**: one duplicating all threads, another duplicating only the calling thread.

**exec** replaces the entire process including all threads, no matter which thread called it.

**which version when:**

| situation | right choice | why |
|---|---|---|
| `exec()` called **immediately after** fork | duplicate **only the calling thread** | exec will replace everything anyway, duplicating all threads is wasted work |
| **no exec** after fork | duplicate **all threads** | the new process is meant to continue as a full copy |

*Q: fork then exec, which fork? A: calling thread only.*

---

## threading issues: thread cancellation

**thread cancellation** is terminating a thread **before it has completed**. the thread to be cancelled is the **target thread**.

examples:
- several threads search a database; one finds the result, so the others can be cancelled.
- you press stop in a browser and all page loading threads are cancelled.

**two scenarios:**

| type | how it works |
|---|---|
| **asynchronous cancellation** | one thread immediately terminates the target |
| **deferred cancellation** | the target **periodically checks a flag** to see whether it should terminate, so it can end itself in an orderly way |

**the difficulty:** the target may hold resources or be mid update of shared data.
- the os reclaims most but **not necessarily all** system resources from a cancelled thread, so asynchronous cancellation may leave system wide resources unfreed.
- a thread cancelled **while updating shared data** leaves it inconsistent for the others.

deferred cancellation avoids both, because the thread cancels only at a **safe point**. most of these problems are solved by using it.

*Q: why is deferred cancellation safer? A: the thread picks a safe moment to die.*

---

## cheat sheet

| concept | explanation | real use |
|---|---|---|
| thread | id, pc, registers, stack; shares code, data, files | browser tabs, servers |
| 4 benefits | responsiveness, sharing, economy, multiprocessor use | why modern apps are multithreaded |
| user vs kernel threads | library managed vs os managed | thread libraries on kernel threads |
| many to one | many user, one kernel; blocks together | old green thread systems |
| one to one | one kernel per user thread; costly, limited | windows, linux style |
| many to many | multiplexed; flexible | most flexible design |
| hyperthreading | logical cores from physical cores | `wmic cpu get ...` |
| fork | duplicate process, new pid | shells, servers |
| exec | replace program, same pid | launching a new program in a child |
| fork then exec | duplicate only the calling thread | the standard unix pattern |
| cancellation | async (immediate) vs deferred (flag check) | stop buttons, search races |

## interleaving

| pair | difference |
|---|---|
| fork vs exec | new process, new pid, same contents vs same process and pid, new contents |
| process vs thread | own address space vs shares it with sibling threads |
| asynchronous vs deferred cancellation | terminated immediately vs terminates itself at a safe point |
| many to one vs many to many | one kernel thread vs several, so parallelism and non blocking |
| hyperthreading vs real multicore | logical sharing of core resources vs separate physical cores |

## feynman checks

1. **a program calls `fork()` four times in a row without any `if`. how many processes end up running, and why?** 16, because each fork doubles the count: $2^4$.
2. **a thread holding a lock is cancelled asynchronously. what could go wrong?** the lock may never be released and shared data may be half updated; deferred cancellation avoids this.
3. **why is one to one often capped on thread count while many to many isn't?** one to one creates a kernel thread for every user thread, a heavy cost; many to many reuses a smaller pool.

## review questions

- (ch4) how is a thread different from a process in what the pcb stores?
- (ch4) how does `fork` relate to the process creation choices (duplicate vs new program) from the process creation lecture?
- (ch1) which architecture feature makes multithreading's parallel speedup possible?

## security angle

- **shared memory between threads** means a bug or exploit in one thread can corrupt all others; memory safety bugs hit the whole process.
- **fork bombs:** a loop of forks creating $2^n$ processes exhausts the system. defense: per user process limits (`ulimit -u`, cgroups).
- **exec after fork** is how shells launch programs; unsanitized input into exec arguments leads to command injection.
- **cancellation races:** forcing a cancel mid update can leave shared state inconsistent, a source of time of check time of use bugs.

## omitted chuff
- the screenshot tour of process explorer and thread stacks.
- the detail on gcc terminal navigation steps in the demo.
- repeated explanations of what a system call is (chapter 2).

*Q: one sentence takeaway? A: threads share the house, but each has its own bedroom (stack and registers).*
