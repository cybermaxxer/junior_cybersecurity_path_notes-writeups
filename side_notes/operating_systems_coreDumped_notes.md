# Operating Systems Notes: Core Dumped Playlist

Side notes for the Junior Cybersecurity path on Hack The Box. Source: the "Core Dumped" OS theory playlist by George, synthesized from the transcripts. Timestamps are `[MM:SS]` from each video, so you can jump straight to the spot.

**The whole playlist is one story:** how one CPU and one pile of RAM get shared by many programs without them wrecking each other. Each video fixes one problem the previous one created.

`program` → `process` (V1) → `context switch` (V2) → `kernel mode` (V3) → `IPC` (V4) → `threads` (V5, V6) → `ABI` (V7) → `hardware assist` (V8) → `scheduling` (V9) → `race conditions` (V10) → `memory protection` (V11)

## How to read this file

| Marker | Meaning |
| :--- | :--- |
| `[MM:SS]` | Where the idea shows up in the video |
| *Q: ... A: ...* | 5 second dopamine check. Cover the answer first |
| **Mental model** | The "why" behind the definition |
| **Try it** | A command or snippet with every piece explained |
| **x86 vs RISC-V** | Same idea on the architecture you are shaky on vs the one you know |
| **Correction** | The video simplified something and real systems differ |
| **Security angle** | How it breaks and how it is defended |

## Table of contents

| # | Video | Duration |
| :--- | :--- | :--- |
| 1 | [A PROGRAM is not a PROCESS](#video-1-a-program-is-not-a-process) | 8:08 |
| 2 | [The Most Successful Idea in Computer Science](#video-2-the-most-successful-idea-in-computer-science) | 16:03 |
| 3 | [How a Single Bit Inside Your Processor Shields Your Operating System's Integrity](#video-3-how-a-single-bit-inside-your-processor-shields-your-operating-systems-integrity) | 21:30 |
| 4 | [IPC: To Share Memory Or To Send Messages](#video-4-ipc-to-share-memory-or-to-send-messages) | 14:15 |
| 5 | [Why Are Threads Needed On Single Core Processors](#video-5-why-are-threads-needed-on-single-core-processors) | 16:07 |
| 6 | [Threads On Multicore Systems](#video-6-threads-on-multicore-systems) | 11:27 |
| 7 | [Why Applications Are Operating-System Specific](#video-7-why-applications-are-operating-system-specific) | 13:09 |
| 8 | [How Hardware Assist Software When Multitasking](#video-8-how-hardware-assist-software-when-multitasking) | 13:15 |
| 9 | [The Fancy Algorithms That Make Your Computer Feel Smoother](#video-9-the-fancy-algorithms-that-make-your-computer-feel-smoother) | 45:02 |
| 10 | [The Weirdest Bug in Programming: Race Conditions](#video-10-the-weirdest-bug-in-programming-race-conditions) | 18:19 |
| 11 | [Why Can't Programs Access Each Other's Memory?](#video-11-why-cant-programs-access-each-others-memory) | 14:51 |

---

## Video 1: A PROGRAM is not a PROCESS
- Duration: 8:08

### V1: 80/20 digest

**1. Two words, two different things** `[01:06]`
- The CPU does not run programs. It runs **processes**.
- Informal definition: a process is a program in execution `[01:13]`.
- A **program** is instructions plus the data the CPU needs for one task. That is just an executable file: passive, sitting on storage `[02:01]`.

*Q: Is a `.exe` sitting on disk a process? A: No, passive program.*

**2. Why loading exists** `[02:10]`
- The CPU fetches instructions and data from memory, so a program must be loaded into RAM before it runs `[02:12]`.
- **Mental model:** disk is the warehouse, RAM is the workbench. The CPU can only touch the workbench (V11 `[01:46]` proves it: no instruction takes a disk address).

*Q: Why load a program into RAM? A: CPU only fetches from memory.*

**3. The four memory sections** `[02:19]`

| Section | Holds | Changes? |
| :--- | :--- | :--- |
| **text** | Executable code | Never, size or content `[02:49]` |
| **data** | Global variables, constants | Fixed size, content can vary `[02:56]` |
| **stack** | Runtime temporaries, locals | Grows and shrinks `[02:43]` |
| **heap** | Runtime data (user input, results) | Grows and shrinks `[02:43]` |

*Q: Which section never changes? A: Text.*

**4. The layout is not the process** `[03:11]`
- Once running, the layout is the memory layout of a process, not the process itself `[03:16]`.
- The full definition of a process lands in V2.

**5. One program, many processes** `[03:42]`
- Open two files in a text editor: same program, two processes.
- Same text section content, different data, each with its own memory space `[04:10]`. A bigger file means more memory `[04:22]`.
- Program = passive. Process = active. Two processes of one program are separate execution sequences `[05:53]`.

*Q: Same program launched twice gives how many processes? A: Two.*

**6. Interpreted languages** `[06:22]`
- Running `python script.py` starts the **interpreter**. The interpreter is the program `[07:12]`.
- Your `.py` file is just text. It is not a program because source code is text the CPU cannot execute `[07:01]`.
- The new process's text section holds the interpreter's machine code. Your source is loaded into the interpreter's data, most likely the heap `[07:29]`.

*Q: What sits in the text section when running a `.py`? A: Interpreter code.*

**Try it**

```bash
python3 -c "import os,time; print(os.getpid()); time.sleep(60)" &
cat /proc/$!/maps | head
```

| Piece | What it does |
| :--- | :--- |
| `python3 -c "..."` | `-c` runs the string as code. Short for "command" |
| `&` | Run in the background so the shell stays free |
| `$!` | Shell variable holding the PID of the last background job |
| `/proc/<pid>/maps` | Virtual file listing that process's memory regions |
| `head` | First 10 lines only |

Look for the `python3` binary mapped `r-xp`: read, no write, execute, private. That is the text section of the interpreter, not of your code.

### V1: Security angle
- **Attack:** a script is just data inside a trusted interpreter. Fileless malware runs inside `python`, `powershell` or `node`. Process hollowing swaps the code of a legit process after launch.
- **Defend:** log full command lines (Sysmon Event ID 1, `auditd` on `execve`). Allowlist who may launch interpreters (AppLocker, WDAC). Alert on odd parent to child pairs.

### V1: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Program | Passive file with instructions and data | The binary in `/usr/bin` |
| Process | Program in execution plus its context | `ps`, `/proc/<pid>` |
| Text section | Code, never modified | Shows as `r-xp` in `maps` |
| Data section | Globals and constants, fixed size | Where `.data` ends up |
| Stack and heap | Runtime memory, size changes | Locals vs `malloc` |
| Interpreter process | Interpreter is the program, script is data | Hunting malicious `python -c` |

### V1: Feynman checks
1. **You run `python3 a.py` and `python3 b.py`. Why two processes with identical text but different behavior?** Same program (the interpreter), so same text. Different source loaded into each heap.
2. **Why can the CPU not run straight from disk?** It only fetches from memory and registers. No instruction carries a disk address.
3. **A teammate says "my Python file is the program". What do you tell them, and why does a defender care?** The interpreter is the program, your file is data. Defenders watch interpreter processes and their command lines, because that is where the payload lives.

### V1: Omitted chaff
- Sponsor read (Brilliant) `[00:00]`, `[04:40]`.
- 100k subscriber thanks and subscribe asks `[04:40]`, `[07:55]`.
- History of "jobs" vs "user programs" naming `[00:15]`.

*Q: Anything core left out? A: No, only sponsor and thanks.*

### V1: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Program vs process | `program vs process operating system explained simply` |
| Process memory layout | `process memory layout text data heap stack visually explained` |
| Interpreter as the real program | `how python runs your script interpreter process explained` |
| `/proc/PID/maps` | `linux proc maps file explained memory regions` |

---

## Video 2: The Most Successful Idea in Computer Science
- Duration: 16:03

### V2: 80/20 digest

**1. A process is more than its memory** `[01:18]`
- The OS assigns an **address space** `[01:08]`.
- Concurrency means processes wait in a queue. Only one uses the CPU at a time `[01:41]`.
- CPU parts that matter: general purpose registers, instruction register, program counter (address register), stack pointer, flags `[01:47]`.

*Q: How many processes use one core at once? A: One.*

**2. Why you cannot just switch** `[02:13]`
Naive switching creates two problems:
- **Security:** the next process sees the leftover registers. Part of a password being hashed could still sit there `[02:35]`.
- **Correctness:** the next process overwrites registers, so the interrupted one resumes with wrecked state `[03:00]`.

*Q: Name both problems with naive switching. A: Leaking data, corrupted state.*

**3. The bug, step by step** `[03:29]`
Each process gets 2 instructions per turn `[03:55]`.

| Step | Registers after | What went wrong |
| :--- | :--- | :--- |
| A runs 2 instructions | `r0=12`, `r1=20` | fine |
| B runs 2 instructions | `r0=100`, `r1=35` | A's values gone |
| A resumes (PC saved by luck) | `r0 = 100+35 = 135` | should be 32 `[06:39]` |
| B resumes | `r0 = 135+35 = 170` | should be 135 `[06:44]` |

- The program counter is also lost the moment the interrupt fires `[05:20]`.
- Results are nondeterministic. Add a third process or start B a few milliseconds later and the garbage changes `[06:53]`.

*Q: Why did A compute 135 instead of 32? A: B overwrote its registers.*

**4. The fix: context switch** `[08:48]`
1. A special OS routine snapshots registers, flags and program counter into memory `[09:09]`.
2. The OS keeps one saved copy per process `[09:30]`.
3. It restores the next process's saved state. A brand new process starts with zeroed registers and the PC on its first instruction `[09:45]`.
4. Each process finds registers exactly as it left them `[10:35]`.

Capturing one state and restoring another is a **context switch** `[10:55]`.

*Q: What two problems does a context switch solve? A: Leakage and corruption.*

**5. Process = context** `[11:14]`
- Address space, program counter, registers, open files, IO devices: the entire context, isolated from other processes `[11:37]`.
- **Mental model:** someone asks "in what context?" before answering. Same question, different answer. Same code in two processes gives different results because the context differs `[11:55]`.

*Q: Why do two copies of one program behave differently? A: Different contexts.*

**6. The PCB** `[12:26]`
A context is not an object you can drop in a queue. So the OS keeps a **Process Control Block** per process `[12:36]`.

| PCB field | Why it exists |
| :--- | :--- |
| Process ID `[12:46]` | Every process is unique |
| State `[12:51]` | Ready, running, waiting (V9) |
| Program counter, registers, flags `[12:56]` | The saved CPU state for context switches |
| Memory limits of the address space `[13:29]` | Catch illegal access, place new processes `[13:46]` |
| Open files, IO devices `[14:19]` | Part of the context |

- The PCB is a representation of the process, not the process. The PCB is what goes in the queue `[15:08]`.
- Linux calls it `task_struct` (`include/linux/sched.h`). One struct for processes and threads `[14:36]`. It holds a parent pointer and a list of children `[14:54]`.

*Q: What actually gets queued, process or PCB? A: The PCB.*

**7. Multicore** `[15:33]` Same rules. Each core just juggles its own context at the same time `[15:41]`.

**x86 vs RISC-V: what a context switch has to save**

| State | x86_64 | RISC-V (RV64) |
| :--- | :--- | :--- |
| Program counter | `rip` | `pc` |
| Stack pointer | `rsp` | `sp` (`x2`) |
| General registers | 16 (`rax` to `r15`) | 31 usable (`x1` to `x31`), `x0` is hardwired to zero |
| Flags | `rflags` | None. RISC-V has no flags register, so less to save |

The video's toy program, as real code:

```asm
; x86_64, intel syntax (destination is also a source)
mov eax, 12
mov ebx, 20
add eax, ebx        ; eax = 32, and r0 is overwritten like in the video
```

```asm
# riscv
li  t0, 12
li  t1, 20
add t2, t0, t1      # three operand form: result goes to a third register
```

**Mental model:** x86 `add` writes its result over an input, which is exactly why the video's second process clobbers `r0` `[06:17]`. RISC-V's three operand form does not, but the context switch still has to save every register.

**Try it**

```bash
ps -ef --forest
```

| Piece | What it does |
| :--- | :--- |
| `ps` | Process status snapshot |
| `-e` | Every process, not just yours |
| `-f` | Full format: UID, PID, PPID, command |
| `--forest` | ASCII tree showing parent and child. A GNU long option, convention to memorize |

### V2: Security angle
- **Attack:** leftover state is a leak channel. Register and cache residue across switches is the idea behind side channel bugs (Spectre family). The parent pointer in the PCB is also what attackers abuse to spoof lineage (PPID spoofing).
- **Defend:** the kernel saves, then clears, state on each switch. Mitigations such as flushing predictors on switches exist for the hardware leaks. Blue team: read the process tree. `winword.exe` spawning `cmd.exe` is the classic red flag.

### V2: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Context switch | Save current CPU state, restore the next | Why multitasking is safe |
| CPU state | PC, registers, flags, stack pointer | Saved in the PCB |
| Process | Address space plus CPU state plus files plus devices | `/proc/<pid>` shows pieces |
| PCB | OS struct representing a process | `task_struct` in Linux |
| Parent and children | Stored in the PCB | `ps --forest`, forensics |
| Nondeterministic bug | Corruption depends on when the switch hits | Why races are hard to debug |

### V2: Feynman checks
1. **Two processes both do `add r0, r1`. Without saving state, what breaks and why is it nondeterministic?** Whoever ran last leaves their values in the registers. The scheduler decides who runs when, so the wrong sums vary per run.
2. **Why does the PC need hardware help to save?** The interrupt overwrites it before any software instruction runs. That is the V8 story.
3. **You find `excel.exe` as the parent of `powershell.exe`. Which PCB field makes that visible and why is it suspicious?** The parent pointer. Excel has no business launching a shell, so it hints at a malicious macro.

### V2: Omitted chaff
- Sponsor read (Brilliant) `[00:00]`.
- Joke about the other PCB (printed circuit board) `[12:36]`.
- Linux naming history guess (Unix "tasks") `[14:40]`.

*Q: Did I drop the PCB or context switch? A: No, both kept.*

### V2: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Context switch | `context switch explained animated registers saved` |
| Process control block | `process control block PCB explained visually` |
| Why naive switching corrupts | `why CPU must save registers on interrupt explained` |
| `task_struct` | `linux task_struct explained kernel source walkthrough` |
| Process tree forensics | `process tree parent child suspicious processes explained` |

---

## Video 3: How a Single Bit Inside Your Processor Shields Your Operating System's Integrity
- Duration: 21:30

### V3: 80/20 digest

**1. The question** `[00:11]`
OS and user programs are both software. Why can a user program not overpower the OS? Software alone cannot solve it, so the hardware helps `[00:59]`.

**2. The mode bit** `[01:24]`
- Some instructions are **privileged** `[01:19]`. A 1 bit register, the **mode bit**, decides who may run them.
- `1` = privileged mode: any instruction `[01:39]`. `0` = restricted mode: no privileged instructions `[01:47]`.
- Normal instructions ignore the mode bit. Only privileged ones check it `[02:09]`.
- **Mental model:** `1` means "I'm the OS", `0` means zero power. Names: kernel mode and user mode `[04:54]`.

*Q: Which mode runs the OS kernel? A: Mode bit 1.*

**3. Who flips the bit? Interrupts** `[02:27]`
- If user code could flip it, the whole feature would be pointless `[02:36]`.
- An **interrupt** is a signal to the CPU that something happened: key press, mouse move `[03:00]`.
- The CPU pauses the current process and jumps to a fixed memory location, the handler `[03:14]`.
- Handler flow: save state, handle, restore state, run `interrupt return` `[03:25]`.
- The interrupt also flips the mode bit to `1`, so the handler runs privileged `[04:06]`.
- Whatever code sits at that address gets full power, so the OS must be what is loaded there `[04:38]`.

*Q: Why must the OS own the handler address? A: Handler code gets full privilege.*

**4. What only kernel mode can do** `[05:11]`
- Talk to IO devices: network, graphics, disk, keyboard `[05:15]`.
- Control the **MMU**, which limits which physical memory the CPU can touch. Owning the MMU means owning memory `[05:32]`.
- Redefine where interrupts jump. The MMU keeps user code away from the handler memory `[06:34]`.
- Hardware gives the tools. The OS has to use them correctly `[07:00]`.
- User mode still allows moving data, arithmetic, floating point, bitwise ops, conditions, loops `[07:18]`.

*Q: Why can user code not rewrite interrupt handlers? A: MMU blocks it, privileged only.*

**5. System calls: the sanctioned door** `[07:31]`
- Programs still need files and network. The OS exposes functions called **system calls** `[08:37]`: open, read, write, close `[08:49]`.
- Puzzle: syscall wrappers live in your user program, yet need privilege `[09:25]`.
- Answer: inside is an instruction that triggers a software interrupt (`int` in the video) `[09:41]`. CPU jumps into OS code, flips to kernel mode, does the work `[10:00]`.
- Return: `interrupt return` jumps back and sets the mode bit to `0` `[10:11]`.
- Half the function runs in user space, half in the kernel `[10:46]`.

*Q: What does a syscall wrapper execute to enter the kernel? A: A software interrupt.*

**6. Pros and cons of syscalls** `[10:56]`

| Pros | Cons |
| :--- | :--- |
| Hardware abstraction: think in code, not transistors `[11:10]` | Performance cost: interrupts, context switch, caching `[12:15]` |
| Security: a bad C program cannot fry your hardware `[11:34]` | OS may delay your request, no latency guarantee `[12:36]` |
| Portability if platforms follow the same guidelines `[12:06]` | Platform dependency, weird variations exist `[13:15]` |

The video calls syscalls traps: hardware abstraction is the bait that hands control back to the OS `[13:04]`.

*Q: Biggest drawback of syscalls? A: Interrupt overhead.*

**7. Cooperative vs preemptive** `[14:13]`
- **Cooperative (nonpreemptive):** the OS relies on programs to give the CPU back through syscalls `[14:16]`.
- Infinite loop without syscalls means the CPU never returns and all other processes starve `[14:30]`.
- Fix: a hardware **timer** that fires an interrupt. Only privileged code can program it `[15:00]`.
- The OS arms the timer before dispatching a process. When it expires, the process is forced out `[15:12]`.
- Same for rogue instructions: a privileged instruction in user mode triggers an interrupt, and the OS usually kills the process `[15:50]`.
- That is a **preemptive** OS `[16:22]`.

*Q: What breaks a cooperative OS? A: A loop with no syscalls.*

**8. Kernel mode for third parties** `[17:03]`
- **Drivers** run in kernel mode because hardware control needs privilege `[17:35]`.
- All kernel code shares one address space. A buggy driver can corrupt the OS or crash it, hence the blue screen `[17:56]`.
- Anticheat and malware detection also run in the kernel: no syscall overhead `[18:50]`, and they can watch hardware and other processes' memory `[19:19]`.
- Cost: **CrowdStrike** crashed millions of Windows machines `[19:50]`. **Vanguard** (Riot's anticheat) runs in kernel mode even when no game is open, which raises privacy questions. The video says there is no evidence it is abused `[20:19]`.

**Correction:** the video dates CrowdStrike to September and blames a file filled with zeros. The outage was 19 July 2024, and CrowdStrike's own root cause analysis points to an out of bounds memory read triggered by a content update. The lesson holds: one bug in kernel mode takes the whole OS down.

*Q: Why does one crashing kernel driver kill the OS? A: Shared kernel address space.*

**x86 vs RISC-V: privilege and syscalls**

| Idea | x86_64 | RISC-V |
| :--- | :--- | :--- |
| "Mode bit" | Current privilege level in the low 2 bits of `cs`: rings 0 to 3. Linux uses ring 0 and ring 3 | Privilege modes M, S, U. Not readable directly by software |
| Privileged examples | `hlt`, `lgdt`, `lidt`, `mov cr3, ...`, `wrmsr` | CSR writes to privileged CSRs, `sret`, `mret`, `sfence.vma` |
| Illegal use in user mode | General protection fault (#GP, vector 13) | Illegal instruction exception (`scause = 2`) |
| Enter kernel | `syscall` (old: `int 0x80`) | `ecall` |
| Return to user | `sysretq` (old: `iret`) | `sret` |
| Interrupt table | IDT, loaded with privileged `lidt` | `stvec` CSR points at the handler |
| Preemption timer | Local APIC timer | `mtimecmp` (or `stimecmp` with Sstc) |

`write(1, buf, n)` side by side (Linux):

```asm
; x86_64
mov rax, 1        ; syscall number: write
mov rdi, 1        ; arg1: fd = stdout
lea rsi, [buf]    ; arg2: buffer address
mov rdx, n        ; arg3: length
syscall           ; flip to kernel mode, jump to the kernel entry
```

```asm
# riscv
li  a7, 64        # syscall number: write
li  a0, 1         # arg1: fd = stdout
la  a1, buf       # arg2: buffer address
li  a2, n         # arg3: length
ecall             # trap into the kernel
```

Numbers differ because each architecture has its own table. That is V7.

**Try it**

```bash
strace -e trace=write echo hi
```

| Piece | What it does |
| :--- | :--- |
| `strace` | Prints every syscall a program makes |
| `-e trace=write` | `-e` is "expression". `trace=` picks the filter type. `write` is the syscall name |
| `echo hi` | The command being traced |

You will see `write(1, "hi\n", 3) = 3`: fd 1, the bytes, the length, bytes written.

### V3: Security angle
- **Attack:** the goal of kernel exploitation is flipping into ring 0 with your own code. Paths: kernel bugs, vulnerable signed drivers (BYOVD: bring your own vulnerable driver), rootkits hooking the syscall table or IDT.
- **Defend:** SMEP and SMAP stop the kernel from executing or reading user pages. Driver signing, Secure Boot, kernel lockdown. Keep kernel code small (eBPF instead of kernel modules). CrowdStrike also proved that availability is a security property: a defender in ring 0 is a single point of failure.

### V3: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Mode bit | 1 bit: privileged (1) or restricted (0) | User vs kernel mode |
| Privileged instruction | Only runs in kernel mode | IO, MMU, timer, interrupt table |
| Interrupt | Pause, jump to handler, flip to kernel mode | Syscalls, timers, IO events |
| System call | Controlled entry into the kernel | `strace` shows them |
| Timer interrupt | Hardware forces the OS back in | Preemptive multitasking |
| Kernel mode drivers | Third party code at full privilege | BSODs, EDR, anticheat risk |

### V3: Feynman checks
1. **Why does the CPU flip the mode bit during the interrupt rather than letting the program do it?** If the program could, any program would gain privilege on demand. The jump target is fixed by the OS, so privilege only goes to OS code.
2. **A process runs `while(1);` with no syscalls. Why does the system still respond?** A hardware timer fires, forcing a privileged interrupt back into the OS. Only the OS can program that timer.
3. **Why is a kernel mode EDR a bigger target than a user mode one?** A bug in it gives an attacker ring 0, and a bug that crashes it takes the machine down (CrowdStrike).

### V3: Omitted chaff
- Sponsor read (NeetCode) `[07:48]`.
- "Microsoft okay" joke `[13:26]`.
- Disclaimer that assembly shown is simplified `[00:44]`.
- LowLevel channel recommendation `[21:08]`.

*Q: Missed any mechanism? A: No, only jokes and sponsor.*

### V3: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| User mode vs kernel mode | `user mode vs kernel mode explained animated` |
| Interrupts | `how CPU interrupts work step by step explained` |
| System calls | `how system calls work user space to kernel space visually` |
| Preemptive vs cooperative | `preemptive vs cooperative multitasking timer interrupt explained` |
| Kernel drivers risk | `why kernel level anticheat and EDR are risky explained` |

---

## Video 4: IPC: To Share Memory Or To Send Messages
- Duration: 14:15

### V4: 80/20 digest

**1. Isolation has a price** `[01:17]`
- A process is its whole context, isolated from the rest `[01:13]`. Isolated processes cannot collaborate.
- **Independent** processes ignore each other. **Cooperating** processes exchange data `[01:37]`.
- Reasons to cooperate: sharing data, speedup through parallel work `[01:50]`, modularity `[02:02]`.
- **IPC** (interprocess communication) is the mechanism. Two models: **shared memory** and **message passing** `[02:24]`.

*Q: Name the two IPC models. A: Shared memory, message passing.*

**2. Shared memory** `[02:39]`
- Normally the OS enforces isolation. Touching another process's memory gets you interrupted and terminated `[03:06]`.
- To share, processes ask the OS to drop that restriction for one region `[03:15]`.
- One process creates the region with a syscall. Others **attach** it to their address space with a syscall `[03:24]`.
- After that, read and write like normal memory. **No more syscalls** `[03:41]`.

*Q: When are syscalls needed in shared memory? A: Only setup and attach.*

**3. The OS stops supervising** `[03:47]`
- Layout, offsets, types: all decided by the processes `[03:56]`.
- Producer writes 8 bit signed numbers, consumer reads unsigned: same bits, different meaning `[04:21]`. Wrong type or wrong address gives undefined behavior `[04:30]`.
- Writers touching the same spot at once cause **race conditions** (V10) `[04:56]`.
- **Mental model:** the OS gave you a shared whiteboard and left the room. Agree on the format, or fight.

*Q: Who defines the data layout in shared memory? A: The processes, not the OS.*

**4. Who uses it** `[05:08]`
- **Chrome** splits into browser, renderer (one per tab) and plugin processes `[06:58]`. A crashing tab only kills its renderer `[07:26]`.
- Also: simulators, game engines, databases, deep learning frameworks `[07:33]`.

**Correction:** Chromium's main IPC is message passing (Mojo message pipes). Shared memory is used for big buffers like image data. Both models usually appear together in real systems.

**5. Message passing** `[07:43]`
- Shared memory puts a lot of burden on the programmer `[07:47]`. With messages, address spaces stay isolated `[08:04]`.
- Mechanisms: **pipes**, **sockets**, **RPC** `[08:12]`.
- A syscall asks the OS for a link. Under the hood the kernel makes a **queue in its own address space**, a mailbox `[08:44]`.
- Mailbox variants: sync or async, buffered or not, two queues for full duplex `[09:02]`.
- Processes cannot touch the mailbox. They use **send** and **receive** syscalls and the OS copies the message `[09:25]`.

*Q: Why do you need syscalls to read the mailbox? A: It lives in kernel memory.*

**6. Mach and ports** `[10:37]`
- Mach (ancestor of iOS and macOS kernels) called mailboxes **ports** `[10:54]`.
- Messages go to ports, not to processes `[10:59]`. A **listening port** accepts connection requests and creates new private links `[11:11]`.

**7. Sockets and the network** `[11:27]`
- No shared address space means processes can sit on different machines `[11:40]`. Needs network stack and NIC drivers underneath, but the API looks identical `[11:55]`.
- In client server, clients and servers are **processes**, not machines. `localhost` proves it `[12:26]`.
- **IP** picks the machine. **Port** is the mailbox of the server process `[12:44]`. HTTP, FTP, SSH all sit on sockets `[12:55]`.

*Q: What does the port number identify? A: The server's mailbox.*

**8. Speed tradeoff** `[13:16]`
- Message passing: a syscall per send and per receive. Costly `[13:19]`.
- Shared memory: syscalls only at setup, then as fast as direct memory access `[13:33]`.
- Message passing is good enough in 99% of cases `[14:05]`.

**x86 vs RISC-V: shared memory ordering**

| Topic | x86_64 | RISC-V |
| :--- | :--- | :--- |
| Memory model | TSO (strong). Only store then load can reorder | RVWMO (weak). Loads and stores can reorder more |
| Producer: write data, then set `ready` flag | Usually works in hardware, you still need a compiler barrier | Needs `fence w,w` or an atomic with release semantics |
| Explicit full fence | `mfence` | `fence rw,rw` |

**Mental model:** two cores see shared memory through their own store buffers. x86 keeps those buffers mostly in order, RISC-V lets them leak out of order unless you say `fence`. Portable code uses C11 atomics (`memory_order_release` and `memory_order_acquire`) and lets the compiler emit the right instruction.

**Try it**

```c
int fd = shm_open("/demo_shm", O_CREAT | O_RDWR, 0600);
ftruncate(fd, 4096);
int8_t *buf = mmap(NULL, 4096, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
buf[0] = 42;   // any process mapping "/demo_shm" now sees 42
```

| Piece | What it does |
| :--- | :--- |
| `shm_open` | Create or open a named shared memory object. Returns a file descriptor |
| `O_CREAT \| O_RDWR` | Create if missing, open for read and write. Flags are OR'ed bits |
| `0600` | Permissions, octal: owner read and write, nobody else |
| `ftruncate(fd, 4096)` | Set the size. A new object is 0 bytes |
| `mmap(...)` | Map the object into your address space and return a pointer |
| `MAP_SHARED` | Writes are visible to others. `MAP_PRIVATE` would copy on write instead |

```bash
ls -l /dev/shm
ss -xlp
```

| Piece | What it does |
| :--- | :--- |
| `/dev/shm` | tmpfs where POSIX shared memory shows up as files |
| `ss` | Socket statistics |
| `-x` | Unix domain sockets only |
| `-l` | Listening sockets |
| `-p` | Show the owning process (needs root for other users' processes) |

A shell pipe like `ls | grep txt` is message passing: a kernel buffer between two processes.

### V4: Security angle
- **Attack:** double fetch bugs. A process validates data in shared memory, the other side changes it before use (TOCTOU). Wide permissions on `/dev/shm` or a unix socket let any local user inject. A compromised renderer talks to the browser process over IPC, so bugs in that IPC are sandbox escapes. Services trusting "only localhost" fall to SSRF and port forwarding.
- **Defend:** copy shared data into private memory, then validate the copy. Use `0600` and tight ownership. Authenticate unix socket peers with `SO_PEERCRED`. Keep the IPC surface tiny and fuzz it.

### V4: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| IPC | How isolated processes exchange data | Almost every modern app |
| Shared memory | Region mapped into several address spaces | `shm_open` plus `mmap` |
| Message passing | Kernel queue, send and receive syscalls | Pipes, sockets, RPC |
| Mailbox or port | Kernel queue that carries messages | Mach ports, TCP ports |
| Socket | Message passing that also works across machines | HTTP, SSH servers |
| Speed tradeoff | Shared memory is faster, messages are safer | Pick per workload |

### V4: Feynman checks
1. **Why is shared memory faster than a pipe?** Once mapped, access is plain loads and stores with no syscall. A pipe needs a syscall and a kernel copy per message.
2. **Why can't a process read its own mailbox directly?** The mailbox is in the kernel's address space. The MMU and privilege rules block user code, so it must ask via `receive`.
3. **A privileged daemon reads a config struct from shared memory, checks a length field, then uses it. What is the bug?** Another process can change the length between the check and the use. Copy first, then check the copy.

### V4: Omitted chaff
- Fractal demo with Rust and TypeScript `[00:00]`.
- JetBrains sponsor `[05:14]`.
- CodeCrafters plug `[13:03]`.
- Complaint about the client server diagram `[12:23]`, kept as the "processes not machines" point.

*Q: Did I skip any IPC model? A: No, both covered.*

### V4: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Shared memory vs message passing | `shared memory vs message passing IPC explained visually` |
| POSIX shared memory | `shm_open mmap shared memory between processes C tutorial` |
| Pipes and sockets | `unix pipes and sockets how they work kernel buffer explained` |
| Chrome process model | `chrome multi process architecture sandbox explained` |
| Memory ordering | `x86 TSO vs RISC-V weak memory model fence explained simply` |

---

## Video 5: Why Are Threads Needed On Single Core Processors
- Duration: 16:07

### V5: 80/20 digest

**1. Concurrency has two jobs** `[00:31]`
- Alternate the CPU between processes fast enough to create the illusion of simultaneity `[00:45]`.
- Fill the gaps. A process waiting on IO cannot use the CPU, so someone else should `[01:30]`.

*Q: What is the second purpose of concurrency? A: Use CPU time idle processes waste.*

**2. The wall: one program counter per process** `[02:01]`
- Processes were the basic unit of execution. A process has one program counter, so two functions inside one process cannot take turns `[02:30]`.
- One must finish before the other starts `[02:45]`.

**3. The server that blocks** `[03:22]`
- Server on port 3000: accept request, load an image from disk (slow IO), send it back `[03:22]`.
- Sequential handling means request 2 waits for request 1. The CPU sits idle during the disk read `[04:41]`. That is the **blocking effect** `[04:58]`.

*Q: Why is the CPU idle while one request loads from disk? A: It waits on IO.*

**4. Old fix: a process per client** `[05:07]`
- A listener process spawns a new process for every client `[05:16]`.
- Costs: memory (each process has its own address space) `[05:44]`, creation time `[06:04]`, and IPC for any shared state `[06:18]`.

**5. The real fix: threads** `[07:41]`
- Stop tying the program counter to the process. Give **each thread** its own `[08:20]`.
- New thread for a client means no new process needed. If one thread waits on IO, others keep running `[08:44]`.

*Q: What did we separate from the process to get threads? A: The program counter.*

**6. What is shared, what is private** `[08:56]`

| Shared by all threads of a process | Private to each thread |
| :--- | :--- |
| Address space (text, data, heap) | Program counter |
| Open files, IO devices | Registers, flags |
| Process ID | **Stack** and stack pointer |

- Each thread needs its own CPU state, otherwise a switch would corrupt it (V2) `[09:00]`.
- Each needs its own stack, or functions would overwrite each other's locals `[09:30]`.
- But stacks sit inside the shared address space, so one thread **can** read another's stack. Avoid it `[09:50]`.
- Share data through the heap, with care `[10:19]`. Synchronization is covered later `[10:40]`.
- Threads from different processes still need IPC `[10:53]`.

*Q: Can thread A read thread B's stack? A: Yes, same address space.*

**7. How the OS tracks threads** `[11:09]`
- Linux uses one struct, the `task`, for both processes and threads `[11:26]`. One scheduler handles both `[11:52]`.
- Every process has at least one thread, the **main thread**, created automatically `[12:02]`.
- Most OSes start with one thread and spawn more at runtime, because nobody knows the count at compile time `[12:29]`.
- In many implementations, if the main thread ends, all the other threads die `[13:04]`.

*Q: Who creates a process's first thread? A: The OS, automatically.*

**8. Servers with threads** `[13:22]`
- A thread per client gives a similar effect to process per client with far less memory `[13:25]`. Creating threads is also faster `[13:35]`.

**9. What a thread is not** `[13:52]`
- Not a function. It holds no code. It **points** at code through its program counter `[14:04]`.
- Many threads can point at the same function `[14:09]`. That is why `pthread_create` takes a function pointer `[14:52]`.
- Not an asynchronous function `[15:07]`.
- Two definitions: for the OS, the most basic unit of execution `[15:15]`. For you, a way to tell the OS some code can run concurrently `[15:23]`. Also "lightweight process" `[15:33]`.

**Correction:** the video says text and data sections are never written, so threads can read them safely `[14:21]`. Code and constants are read only. Global variables live in the data section and **are** writable (V1 `[02:56]`), so threads sharing a global can still race (V10).

*Q: Is a thread a function? A: No, it points at one.*

**x86 vs RISC-V: where a thread keeps its identity**

| Piece | x86_64 | RISC-V |
| :--- | :--- | :--- |
| Per thread stack pointer | `rsp` | `sp` |
| Thread local storage base | `fs` segment base (`%fs:0`) | `tp` register (`x4`) |
| OS entry that makes a thread | `clone` syscall (number 56) | `clone` syscall (number 220) |

On Linux, `pthread_create` ends in `clone` with flags like `CLONE_VM` (share the address space), `CLONE_FILES` (share the file descriptor table) and `CLONE_THREAD` (same process). A "process" is the same call with fewer flags.

**Try it**

```c
#include <pthread.h>
void *handle_client(void *arg) { return NULL; }   // each thread starts here
int main(void) {
    pthread_t tid;
    pthread_create(&tid, NULL, handle_client, NULL);
    pthread_join(tid, NULL);   // wait, so main does not exit first
}
```

```bash
gcc -pthread demo.c -o demo
ps -eLf
```

| Piece | What it does |
| :--- | :--- |
| `pthread_create(&tid, NULL, fn, arg)` | Writes the thread id to `tid`, `NULL` means default attributes, `fn` is the code address to run `[14:52]` |
| `pthread_join` | Blocks until that thread finishes |
| `-pthread` | Tell gcc to link the thread library and set thread safe defines |
| `ps -eLf` | `-e` all processes, `-L` one row per thread (`LWP` column is the thread id), `-f` full format |

### V5: Security angle
- **Attack:** all threads share one address space, so a single memory corruption bug anywhere gives an attacker every key and secret in the heap. Thread per client servers fall to connection floods (Slowloris holds threads open).
- **Defend:** use processes for privilege separation (OpenSSH privsep, Chrome). Use thread pools, timeouts, and limits (`ulimit -u`, cgroup `pids.max`).

### V5: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Blocking effect | Independent tasks wait on each other | Slow servers |
| Thread | Own PC, registers, stack. Shares the process address space | `pthread_create` |
| Main thread | Created automatically with the process | Dies, and the rest may die |
| Shared heap | Memory threads can use to talk | Needs synchronization (V10) |
| `task` | One kernel struct for threads and processes | `task_struct` |
| Thread vs process cost | Threads: less memory, faster to create | Thread per client servers |

### V5: Feynman checks
1. **Why does each thread need its own stack but not its own heap?** Stacks hold per thread call frames and locals that would be overwritten otherwise. The heap is shared on purpose so threads can exchange data.
2. **Ten threads run the same function. Where is the code, and is that safe?** One copy in the text section. Reading code is safe because it is not written. Their data lives in separate stacks or on the heap.
3. **A web server crashes under a flood of slow connections. Why do threads make that worse, and what is the fix?** Each connection holds a thread and its stack memory. Cap thread count, use timeouts, and use a pool or event loop.

### V5: Omitted chaff
- Brilliant sponsor `[00:00]`, `[06:37]`.
- "Four concepts" opener `[00:25]`.
- Listener process diagrams with 1000 clients `[04:34]`.

*Q: Anything essential cut? A: No, only sponsor and pacing.*

### V5: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Why threads exist | `why threads exist operating system single core explained` |
| Thread vs process | `thread vs process memory stack heap shared explained visually` |
| Blocking IO in servers | `blocking io thread per request server explained` |
| `clone` and threads | `linux clone syscall threads vs processes explained` |
| Thread local storage | `thread local storage how it works fs register tp explained` |

---

## Video 6: Threads On Multicore Systems
- Duration: 11:27

> The transcript file contains this video twice. Notes below cover it once.

### V6: 80/20 digest

**1. Why threads matter (recap)** `[00:51]`
- A short task should not wait behind a long one `[00:56]`.
- A task waiting on IO should not hold the CPU `[01:05]`.
- Example: an email client draws the UI, reads keystrokes, uploads an attachment, checks grammar, and watches for new mail, none waiting on the others `[01:33]`.

**2. When concurrency stops feeling smooth** `[01:54]`
Too many tasks means each waits too long for its turn. Three fixes:
1. **Faster CPU.** Gets harder every decade and only delays the problem `[02:16]`.
2. **Smarter scheduling.** Needs its own video, that is V9 `[02:41]`.
3. **More processors.** Brute force `[02:50]`.

*Q: Which fix is "just add more"? A: More processors.*

**3. Terminology** `[03:25]`
- "CPU" often means the whole chip. Each **core** inside is an independent processing unit that shares some pieces, such as cache, with other cores `[03:29]`.
- The OS sees each core as a separate processor `[03:43]`.
- Ways to scale: more sockets, multicore chips, or both `[03:06]`.

**4. Concurrency vs parallelism** `[05:00]`
- 8 threads on 1 core: execution is interleaved over time.
- 8 threads on several cores: some truly run at the same time `[05:13]`.
- **Concurrent:** all tasks make progress. **Parallel:** more than one task runs at literally the same instant `[05:35]`. Concurrency is possible without parallelism.

*Q: Can you have concurrency without parallelism? A: Yes, single core.*

**5. What it means for your code** `[06:01]`
- Modern OSes schedule **threads**, not processes `[06:01]`.
- You declare tasks concurrent with threads. The OS interleaves on one core or spreads across many `[06:21]`. No recompile per core count `[06:38]`.
- Caveats: cores are fixed, so with $n$ cores up to $n$ threads run in parallel, not 1000 `[06:44]`. And other processes also want CPU, so "up to $n$" is an upper bound `[06:57]`.

*Q: 1000 threads on 4 cores run how many in parallel? A: At most four.*

**6. Two kinds of parallelism** `[07:42]`

| Type | Idea | Video example |
| :--- | :--- | :--- |
| **Data** parallelism | Same operation on different slices of the data | Find primes: split the array into 4 parts, one per core `[09:10]` |
| **Task** parallelism | Different operations, often on the same data | Min, max, mean, "contains 101" on the same array, one per core `[10:02]` |

- Prime checks are independent, so slicing works `[08:55]`.
- 4 cores do **not** give 4 times the speed `[09:25]`. The video leaves why out of scope.

**Extension (not in the video): Amdahl's law.** If a fraction $p$ of the work is parallel, speedup on $n$ cores is

$$S(n) = \frac{1}{(1-p) + \frac{p}{n}}$$

With $p = 0.9$ and $n = 4$: $S = \frac{1}{0.1 + 0.225} \approx 3.08$. The serial 10% caps you, and more cores cannot beat $\frac{1}{1-p} = 10$.

*Q: Why is 4 cores less than 4 times faster? A: Serial part and overhead.*

**x86 vs RISC-V: what counts as a "core"**

| Term | x86_64 | RISC-V |
| :--- | :--- | :--- |
| One hardware execution context | Logical processor (SMT or "hyperthread"), usually 2 per core | **hart** (hardware thread), read from `mhartid` |
| Id of the current context | APIC id via `cpuid` | `mhartid` CSR (M mode) |
| Memory model across cores | TSO | RVWMO, needs fences (V4) |

The OS treats each logical processor or hart as one schedulable CPU.

**Try it**

```bash
nproc
lscpu
taskset -c 0 ./demo
```

| Piece | What it does |
| :--- | :--- |
| `nproc` | Number of processing units available to you |
| `lscpu` | Sockets, cores per socket, threads per core, cache sizes |
| `taskset` | Set or read CPU affinity, which cores a process may use |
| `-c 0` | `-c` takes a **c**PU list. `0` means core 0 only |

### V6: Security angle
- **Attack:** cores share caches, so another process on a neighbor core can time your memory accesses (Flush+Reload, Spectre, SMT based leaks). Parallelism is also an attacker tool: cracking is data parallel, which is why `hashcat` on a GPU and `john --fork=N` scale so well.
- **Defend:** disable SMT or use core scheduling on shared hosts, pin sensitive workloads, write constant time crypto, and use slow hashes (bcrypt, scrypt, Argon2) so parallel cracking stays expensive.

### V6: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Concurrency | Tasks make progress by interleaving | Works on 1 core |
| Parallelism | Tasks run at the same instant | Needs more than 1 core |
| Core | Independent processing unit in the chip | `nproc`, `lscpu` |
| Data parallelism | Same op, split data | Cracking, image processing |
| Task parallelism | Different ops, own thread each | UI plus network plus disk |
| Amdahl's law | Serial fraction caps speedup | Predict gains before coding |

### V6: Feynman checks
1. **Your program spawns 16 threads on a 4 core machine. How many run in parallel, and what happens to the rest?** At most 4, minus whatever other processes use. The OS interleaves the rest.
2. **Is cracking 1 million hashes data or task parallelism, and why does it scale well?** Data parallelism. Each hash is independent, so you split the list across cores.
3. **Why does a shared cache between cores matter to a security architect?** It is a shared channel. One process can infer another's memory access patterns by timing cache hits.

### V6: Omitted chaff
- CodeCrafters giveaway and comment contest `[00:25]`, `[09:45]`, `[10:53]`.
- Brilliant sponsor `[03:54]`.
- "I messed up my computer" apology `[08:22]`.
- Benchmark of 10 million runs `[09:30]`.

*Q: Missed parallelism types? A: No, both kept.*

### V6: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Concurrency vs parallelism | `concurrency vs parallelism explained with animation` |
| Data vs task parallelism | `data parallelism vs task parallelism explained examples` |
| Amdahl's law | `amdahl's law explained intuition with examples` |
| CPU affinity | `taskset cpu affinity linux explained` |
| Hyperthreading and side channels | `hyperthreading smt security side channel explained` |

---

## Video 7: Why Applications Are Operating-System Specific
- Duration: 13:09

### V7: 80/20 digest

**1. The puzzle** `[00:04]`
- A Windows executable will not run on a Mac. "Macs use Arm, Windows uses x86" is only half the answer.
- Before 2019 Macs were Intel and it still failed `[00:22]`. Linux and Windows on the same CPU cannot run each other's programs either `[00:33]`.
- Real answer: apps are architecture specific **and** OS specific `[01:04]`. The OS part comes from **system calls** `[01:19]`.

*Q: Same CPU, different OS. Does the binary run? A: No.*

**2. Quick recap** `[01:35]`
- User mode cannot touch hardware. Kernel mode can (IO, timers, MMU) `[02:14]`.
- So user programs ask the OS through system calls `[03:34]`.

**3. Each OS has its own syscall set** `[03:53]`
- Syscalls show up as C or C++ functions, grouped in six categories `[03:58]`. Each OS ships its own set `[04:02]`.
- Programs make thousands of syscalls per minute `[04:09]`. If another OS does not implement them the same way, the program breaks `[04:46]`.
- Same goal, different design `[05:06]`:
  - Windows `CreateProcess` takes a path and starts that program.
  - Unix `fork` **clones the caller** `[05:27]`. Then the child calls `exec` to replace its text section with a new executable `[05:41]`.
- One platform may need 1 call, another several. Compiled machine code differs, extra instructions mean nothing on the other OS, so you get undefined behavior and crashes `[06:21]`.

*Q: What does Unix `fork` create? A: A clone of the caller.*

**4. Even with the same syscalls, more must match** `[06:29]`
- **Which syscall?** Each has a unique number written to a register before the trap (`syscall` on x86_64 `[08:05]`). The kernel reads the register and looks it up in a **syscall table** `[08:36]`. Different tables mean the same machine code requests different services `[08:59]`.
- **Which register holds the number?** Wrong register, wrong syscall `[09:17]`.
- **How are parameters passed?** Registers, memory, or both `[09:39]`. If memory: stack or fixed location? Pointer width matters `[10:26]`.

*Q: What if two OSes use the same table but different registers? A: Wrong syscall runs.*

**Correction:** the video says Linux passes fewer than six parameters in registers and spills the rest to memory `[09:51]`. Linux x86_64 syscalls take **up to six** arguments, all in registers. The "extra args go on the stack" rule belongs to normal function calls under the System V ABI, not to syscalls.

**5. The ABI** `[10:38]`
Together, these conventions are the **Application Binary Interface**. An API says what functions exist at source level. An **ABI** says how binary code interacts on a given OS and architecture `[10:49]`.

**6. Executable file format** `[10:59]`
- A binary is not just instructions. It also carries data and metadata the loader needs `[11:14]`. Where each part lives in the bytes is fixed by the format `[11:27]`.
- **ELF** on Linux, **PE** on Windows `[11:48]`. (macOS uses Mach-O, not in the video.)

**7. Runtimes** `[11:59]`
- VMs and interpreters sidestep many of these problems `[12:05]`. But one module needing a missing runtime can break the whole app `[12:27]`.

*Q: Why does the same Python run on Windows and Linux? A: Interpreter hides syscalls.*

**x86 vs RISC-V: syscall ABI on Linux**

| Item | x86_64 | RISC-V (RV64) |
| :--- | :--- | :--- |
| Trap instruction | `syscall` | `ecall` |
| Syscall number | `rax` | `a7` |
| Arguments | `rdi`, `rsi`, `rdx`, `r10`, `r8`, `r9` | `a0` to `a5` |
| Return value | `rax` | `a0` |
| `write` number | 1 | 64 |
| `execve` number | 59 | 221 |
| Table lives in | `arch/x86/entry/syscalls/syscall_64.tbl` | Generic table: `include/uapi/asm-generic/unistd.h` |

**Why `r10` and not `rcx`?** Ordinary x86_64 function calls pass arg 4 in `rcx`. But the `syscall` instruction overwrites `rcx` with the return address and `r11` with the saved flags, so the kernel ABI moved arg 4 to `r10`. RISC-V's `ecall` stores the return address in `sepc` instead, so `a0` to `a5` work for both functions and syscalls.

Windows x64 is a third world: function args in `rcx`, `rdx`, `r8`, `r9`, and syscall numbers change between Windows builds. Linux numbers are stable per architecture.

**Try it**

```bash
file /bin/ls
readelf -h /bin/ls | grep -E 'Class|Machine|OS/ABI'
xxd -l 4 /bin/ls
strace -c ls > /dev/null
grep ctxt /proc/$$/status
```

| Piece | What it does |
| :--- | :--- |
| `file` | Guesses format from magic bytes: ELF, architecture, linking |
| `readelf -h` | `-h` = file **h**eader. Shows class (32 or 64 bit), machine (CPU), OS/ABI |
| `grep -E 'A\|B'` | `-E` = extended regex. `\|` inside means "A or B" |
| `xxd -l 4` | Hex dump, `-l` = length in bytes. You will see `7f45 4c46` = `\x7fELF` |
| `strace -c` | Count syscalls and print a summary table instead of each call |
| `> /dev/null` | Throw away `ls` output so only the strace table remains |
| `grep ctxt /proc/$$/status` | `$$` is your shell PID. Prints voluntary and nonvoluntary context switches |

**Correction:** the video reads "voluntary context switches" as the number of syscalls `[04:30]`. It counts the times a process **blocked and gave up the CPU** (waiting on IO, a lock, sleep). Not every syscall blocks. Nonvoluntary means the timer preempted it (V9). Use `strace -c` to count actual syscalls.

### V7: Security angle
- **Attack:** shellcode is OS and architecture specific. The same `execve` is 59 on x86_64 Linux and 221 on RISC-V, so exploit payloads must target both. On Windows, malware resolves syscall numbers per build to call the kernel directly and dodge user mode API hooks.
- **Defend:** `seccomp-bpf` allowlists syscalls per process. Always check the `arch` field in the filter, or 32 bit compatibility calls (`int 0x80`) bypass 64 bit rules. Monitor syscalls with `strace` for triage, `auditd` or Falco at scale.

### V7: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Why binaries are OS specific | Syscall set, numbers, calling convention, file format all differ | Why cross OS execution fails |
| Syscall number | Unique id in a register, looked up in the kernel table | Writing shellcode, reading `strace` |
| ABI | Binary level conventions for one OS plus architecture | Linking, FFI, exploit dev |
| `fork` plus `exec` vs `CreateProcess` | Clone then replace vs start fresh | Reading process creation logs |
| ELF and PE | Executable container formats | `readelf`, `file`, malware analysis |
| Runtimes | VM or interpreter hides OS differences | Python, Java, Node portability |

### V7: Feynman checks
1. **Two Linux boxes, one x86_64 and one RISC-V. Why can't you copy the binary across?** Different instruction set, and different syscall numbers and registers. Same OS is not enough.
2. **Why does the x86_64 kernel use `r10` for argument 4?** `syscall` clobbers `rcx` with the return address, so the kernel ABI picked another register.
3. **A sandbox allows only `read`, `write`, `exit` by number but ignores `arch`. How can it be bypassed?** Call the same numbers through the 32 bit compatibility path, where numbers mean different syscalls.

### V7: Omitted chaff
- Brilliant sponsor `[00:00]`, `[06:42]`.
- Dinosaur book joke and "JavaScript on the server" jokes `[03:42]`, `[12:47]`.
- Ask for a `fork` plus `exec` episode `[05:55]`.

*Q: Any piece of the ABI missing? A: No, number, register, params, format.*

### V7: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Why apps are OS specific | `why windows programs don't run on linux syscalls ABI explained` |
| Syscall numbers and registers | `linux x86_64 syscall calling convention registers explained` |
| `fork` and `exec` | `fork and exec explained how unix creates processes` |
| ELF vs PE | `ELF vs PE executable format explained visually` |
| seccomp | `seccomp bpf syscall filtering explained sandbox` |

---

## Video 8: How Hardware Assist Software When Multitasking
- Duration: 13:15

### V8: 80/20 digest

**1. The question from the comments** `[01:11]`
- A context switch needs the OS to run code, and that code uses registers. How does it save state without disturbing it? `[01:02]`
- Setup: simple CPU with interrupts. Without an interrupt mechanism, every program must save its own state, and such CPUs are rare and unsafe `[01:31]`. Interrupts exist even on the MOS 6502 and microcontrollers `[02:00]`.

**2. What an interrupt does** `[02:23]`
- Triggered by software (syscall) or hardware (timer, IO) `[02:26]`.
- The CPU finishes the current instruction, **overwrites the program counter**, and jumps to the handler `[02:45]`.
- The old PC is already gone before the first handler instruction runs `[03:08]`.

*Q: What does an interrupt overwrite before the handler starts? A: The program counter.*

**3. Why software alone fails** `[03:18]`
Try writing the handler:
1. Push general registers on the stack `[03:28]`. Push flags and stack pointer too `[03:46]`.
2. Problem: `sp` points at the **user's** stack. The OS wants its own `[03:55]`.
3. Switch `sp` first? Then you lost the user `sp` `[04:16]`.
4. Stash `sp` in a fixed memory spot, then switch. Works, but you are inventing workarounds `[04:28]`.
5. The killer: the PC is overwritten **before any software runs**. No instruction can save it `[04:44]`.

**Mental model:** the handler needs one free register or one free memory slot to start. The hardware must hand it that, because software has nothing to bootstrap with.

*Q: Why can software never save the PC itself? A: It's overwritten before code runs.*

**4. Solution 1: multiple register sets** `[06:45]`
- Give the CPU two sets: a **common** set for user code and a **privileged** set for the kernel `[07:29]`.
- On interrupt: switch to kernel mode, deactivate the common set, use the privileged set. The user's registers stay untouched `[07:56]`.
- Privileged instructions copy values out of the disabled set. Normal instructions then store them in the PCB `[08:14]`.
- Load the next process the same way, set the timer (privileged), then run `sysret`, which flips the register set and returns to user mode `[09:04]`.
- Not popular: duplicating the register file on the die is costly `[09:32]`. A separate set for only the critical registers (PC, SP) would be enough `[09:55]`.

**5. Solution 2: hardware does the critical saves** `[10:10]`
- Hardwired steps run just before the interrupt takes effect `[10:30]`.
- The CPU can always reach the **OS stack pointer**, through a register or a fixed location `[10:53]`.
- On interrupt it automatically pushes the PC and SP onto the OS stack, then jumps `[11:02]`.
- Software saves the rest, being careful not to touch a register before storing it `[11:15]`.

**6. Solution 3: all in hardware** `[11:35]`
- A segment register points at an OS managed structure, the **TSS** (task state segment), found through the **task register** `[11:46]`.
- On interrupt the CPU copies every register into the TSS. From software it "just happens" `[12:03]`.
- Major OSes avoid using it `[12:25]`.

*Q: Which solution saves only PC and SP in hardware? A: Solution 2.*

**x86 vs RISC-V: what the hardware actually does on a trap**

| Step | x86_64 | RISC-V (S mode) |
| :--- | :--- | :--- |
| Find the handler | IDT entry for the vector (table set by `lidt`) | `stvec` CSR |
| Save the PC | Hardware pushes `rip` on the kernel stack | `sepc` ← pc |
| Save flags and mode | Hardware pushes `rflags`, `cs`, `ss`, `rsp` | `sstatus.SPP` (previous mode), `sstatus.SPIE` (previous interrupt enable). No flags register |
| Switch to the kernel stack | Automatic: loads `rsp` from the TSS (`RSP0`) | **Not done.** Software must find a kernel `sp` itself |
| Save general registers | Software `push` | Software `sd` stores |
| Bootstrap trick | `swapgs`: swaps `gs` base with a kernel owned value, giving the handler a pointer to per CPU data | `csrrw sp, sscratch, sp`: atomically swaps `sp` with `sscratch` |
| Return | `iretq` (interrupts) or `sysretq` (syscalls) | `sret` |

How this maps to the video:
- **x86_64** is **solution 2**. Hardware pushes critical state onto an OS stack found through the TSS. In 64 bit mode the TSS only supplies stack pointers. The full hardware task switch (**solution 3**) exists in 32 bit protected mode and Linux and Windows never use it `[12:25]`.
- **RISC-V** is the minimal version the video asks for `[06:31]`. Hardware preserves only the PC (in `sepc`) and gives the kernel one scratch CSR, `sscratch`, so software can bootstrap itself. That is the "separate critical registers" idea from solution 1 `[09:55]`.

A RISC-V trap entry sketch (simplified, RV64):

```asm
trap_entry:
    csrrw sp, sscratch, sp     # swap: sp = kernel stack, sscratch = user sp
    addi  sp, sp, -256         # make room for the saved registers
    sd    ra, 0(sp)            # save regs BEFORE using them
    sd    t0, 8(sp)
    # ... save the rest ...
    csrr  t0, sepc             # the hardware saved the user pc here
    sd    t0, 248(sp)
```

| Piece | What it does |
| :--- | :--- |
| `csrrw rd, csr, rs` | Atomic read and write of a CSR. Old CSR value goes to `rd`, `rs` goes into the CSR. Here it is the one step that fixes the video's stack pointer dilemma |
| `sd` | Store doubleword (8 bytes) |
| `csrr t0, sepc` | Read a CSR. Convention: it is `csrrs t0, sepc, x0` underneath |

And the x86_64 side, after the hardware already pushed `ss`, `rsp`, `rflags`, `cs`, `rip`:

```asm
entry:
    push rax        ; software saves the general registers
    push rbx
    ; ... rest ...
    ; ... handle ...
    pop  rbx
    pop  rax
    iretq           ; hardware pops rip, cs, rflags, rsp, ss and drops to user mode
```

**Try it** (read the real thing)

```bash
grep -n swapgs /usr/src/linux/arch/x86/entry/entry_64.S | head
```

| Piece | What it does |
| :--- | :--- |
| `grep -n` | Search, `-n` prints line numbers |
| `swapgs` | The pattern to find |
| `entry_64.S` | Linux x86_64 interrupt and syscall entry assembly |
| `head` | First 10 matches |

Adjust the path to wherever your kernel source lives.

### V8: Security angle
- **Attack:** this saved state is attacker controlled territory. Sigreturn oriented programming (SROP) forges a fake saved register frame on the stack and calls `sigreturn`, so the kernel loads attacker chosen registers. Bugs in the entry code itself (such as the SWAPGS speculation issue, CVE-2019-1125) turn into kernel leaks.
- **Defend:** the kernel never trusts a user `rsp`. It always switches stacks first (`RSP0`, IST, `swapgs`). Use seccomp to block `sigreturn` where unused, plus ASLR and shadow stacks (CET).

### V8: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Interrupt overwrites PC | Old PC is lost before software runs | Why hardware help is mandatory |
| Register sets | Separate user and kernel registers | Early and some embedded designs |
| Hardware pushes PC and SP | CPU stores critical state on an OS stack | x86_64 interrupts |
| TSS and task register | Hardware task switch structure | Legacy 32 bit x86. Linux uses it only for kernel stack pointers |
| `sepc`, `sscratch` | RISC-V minimal trap support | Writing a RISC-V kernel entry |
| `swapgs` | x86_64 way to reach per CPU data safely | Reading `entry_64.S` |

### V8: Feynman checks
1. **Why can't the handler's first instruction be `push rax` with no hardware help?** If `rsp` still points to user memory, that push lands on a user controlled stack. Also, the PC was already overwritten, so hardware must have saved it.
2. **What does `csrrw sp, sscratch, sp` solve that a plain `mv` cannot?** It swaps in one atomic step, so the user `sp` is never lost and no free register is needed.
3. **Why is it dangerous if the kernel trusts the user stack pointer on entry?** The attacker can aim it at kernel memory. The very first push then becomes a controlled write.

### V8: Omitted chaff
- Skillshare sponsor `[05:13]`.
- Author's book aside `[05:17]`.
- CodeCrafters interpreter course `[12:45]`.
- "Makes this whole video pointless" aside, covered in solution 3 `[12:25]`.

*Q: All three hardware solutions covered? A: Yes.*

### V8: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Interrupt handling in hardware | `how cpu interrupt saves program counter stack explained` |
| x86_64 interrupt entry | `x86_64 interrupt descriptor table TSS rsp0 explained` |
| RISC-V traps | `risc-v trap handling sepc stvec sscratch explained` |
| `swapgs` | `swapgs instruction explained linux kernel entry` |
| Context switch code | `linux context switch code walkthrough switch_to` |

---

## Video 9: The Fancy Algorithms That Make Your Computer Feel Smoother
- Duration: 45:02

### V9: 80/20 digest

**1. What scheduling is** `[00:59]`
- The CPU stays at 100% yet the desktop stays usable, because general purpose OSes optimize for **response time** `[00:11]`.
- **CPU scheduling** decides which task gets the CPU next, based on criteria `[01:02]`:

| Criterion | Meaning |
| :--- | :--- |
| **CPU utilization** | How busy the CPU is kept with useful work |
| **Throughput** | Processes completed per time unit `[31:23]` |
| **Turnaround time** | Creation to completion: $T_{ready\ wait} + T_{cpu} + T_{io}$ `[31:06]` |
| **Waiting time** | Time spent ready but not running |
| **Response time** | Request to first response, how fast it starts answering `[32:17]` |

*Q: Which criterion makes a desktop feel smooth? A: Response time.*

**2. First come first served (FCFS)** `[01:27]`
- "Allocating the CPU to a process" means the program counter points into its text section `[02:05]`. Switching means a context switch `[02:30]`.
- The OS queues **PCBs**, not processes: a FIFO queue. New PCB goes to the tail. When the CPU frees up, the head runs `[03:28]`.
- Naive version breaks: not all processes terminate (servers) `[04:33]`, and processes do not use the CPU continuously `[04:52]`.

*Q: What goes into the ready queue? A: PCBs.*

**3. CPU bursts and IO bursts** `[06:28]`
- A copy loop: read 8 bytes, write 8 bytes. Each read or write is a **syscall** that needs the disk `[07:19]`. The CPU sits idle while the IO finishes `[07:40]`.
- Alternating phases: **CPU burst** then **IO burst**, repeat. The last CPU burst ends with an exit request `[08:34]`.
- Measured across systems: **many short CPU bursts, few long ones** `[09:18]`. So when a process starts an IO burst, hand the CPU to someone else.
- Scheduler consequence: allocate the CPU only until the **current burst** ends, not until the process terminates `[10:12]`.

*Q: What does the CPU do during an IO burst? A: Runs another process.*

**4. Process states** `[10:34]`

| State | Meaning |
| :--- | :--- |
| **new** | Being loaded into memory `[10:36]` |
| **ready** | Waiting for CPU time `[10:45]` |
| **running** | Has the CPU `[10:55]` |
| **waiting** | Blocked on IO or an event (keystroke, network) `[11:01]` |
| **terminated** | Finished, resources being freed `[12:05]` |

- After IO completes the process goes back to **ready**, not running `[11:26]`.
- Normal life: ready, running, waiting, repeat `[11:52]`. State names are arbitrary and vary by OS `[12:09]`.
- Only one process runs per core, so there is no queue for "running" `[13:01]`.

*Q: Where does a finished IO process go? A: Ready.*

**5. The dispatcher** `[13:12]`
- **Scheduler** decides who is next by managing the ready queue with a policy. **Dispatcher** carries it out `[14:25]`.
- Dispatcher steps: save the CPU state into the PCB, place the PCB in the waiting queue, pop the head of the ready queue, restore its state, switch to user mode, resume `[13:21]`.
- Time spent doing this is **dispatch latency**, pure overhead. Hence the dispatcher is one of the most optimized parts of an OS `[14:37]`.
- A **Gantt chart** shows who ran when `[15:09]`.

*Q: Who picks the next process, scheduler or dispatcher? A: Scheduler picks, dispatcher executes.*

**6. FCFS and the convoy effect** `[15:20]`
- Real name would be "first CPU burst come first served" `[15:38]`.
- **IO bound** processes: many short bursts. **CPU bound** processes: few long bursts `[16:19]`.
- A CPU bound process grabs the CPU. All IO bound processes finish their IO and queue up, IO devices idle. Then the CPU bound one goes to IO, the IO bound ones blitz through, the CPU idles. Repeat `[17:03]`.
- That is the **convoy effect**: everyone waits behind one big process, so CPU and device utilization drop `[18:02]`.

*Q: What causes the convoy effect? A: One long CPU burst hogging.*

**7. Shortest job first (SJF)** `[18:15]`
- Give the CPU to the process with the **shortest next CPU burst**. Ties go FCFS. Real name: shortest next CPU burst first `[18:38]`.
- Implementation: a **priority queue** keyed on predicted burst length `[18:54]`.
- Provably optimal for **average waiting time** `[19:09]`.
- **Why:** moving a short job ahead of a long one cuts the short job's wait by more than it adds to the long one's `[19:50]`.
- Illustration (my numbers, ms): bursts A=24, B=3, C=3.
  - FCFS order A, B, C: waits 0, 24, 27, average 17.
  - SJF order B, C, A: waits 0, 3, 6, average 3.

*Q: Why does SJF lower average waiting time? A: Short jobs stop waiting behind long.*

**8. SJF problem: predicting the future** `[20:17]`
- You cannot know the next burst. Reading the PC and counting instructions fails because of loops, calls and user input `[20:36]`.
- So **estimate from history** with an **exponential average** `[21:14]`:

$$\tau_{n+1} = \alpha\, t_n + (1-\alpha)\,\tau_n$$

| Symbol | Meaning `[21:51]` |
| :--- | :--- |
| $t_n$ | Actual length of the most recent burst |
| $\tau_n$ | Previous prediction |
| $\tau_{n+1}$ | Prediction for the next burst |
| $\alpha$ | Weight between recent and old, from 0 to 1 |

- Expanding the formula shows every old burst is weighted by $\alpha(1-\alpha)^j$, so **older bursts fade exponentially** `[23:41]`:

$$\tau_{n+1} = \sum_{j=0}^{n} \alpha(1-\alpha)^j\, t_{n-j} + (1-\alpha)^{n+1}\tau_0$$

- Only the last prediction needs storing, not the whole history `[22:45]`.
- $\alpha = 0$ ignores the latest burst `[23:01]`. $\alpha = 1$ ignores history `[23:15]`. Common choice: $\alpha = \tfrac{1}{2}$ `[23:29]`.
- New process has no history, so start with a constant `[24:03]`.
- Illustration (my numbers): $\tau_0 = 100$ ms, $\alpha = \tfrac12$. First burst 20 ms gives $\tau_1 = 0.5 \cdot 20 + 0.5 \cdot 100 = 60$. Next burst 40 ms gives $\tau_2 = 0.5 \cdot 40 + 0.5 \cdot 60 = 50$.

*Q: What does $\alpha = 1$ mean? A: Only the last burst counts.*

**9. Preemptive vs nonpreemptive** `[24:40]`
- Process A runs. Process B arrives with a shorter burst than A's remainder. Interrupt A?
- **Preemptive:** yes, the OS may take the CPU back `[25:08]`. **Nonpreemptive:** never, B waits `[25:20]`. The schedule order can differ `[25:33]`.
- Modern default is preemptive. Reason: a nonpreemptive system freezes on an infinite loop with no syscalls, and malware could monopolize the CPU on purpose `[25:50]`.
- **Starvation:** a process waits in the ready queue indefinitely `[26:56]`. SJF starves long jobs when short ones keep arriving `[27:07]`. It is a consequence of the **policy**, not just burst length `[27:37]`.

*Q: What is starvation? A: Waiting in the queue forever.*

**10. Round robin (RR)** `[27:58]`
- FCFS plus preemption. Define a **time quantum** `q` `[28:12]`.
- Pick the head, arm a timer for `q`, dispatch. Two outcomes `[28:33]`:
  - Burst shorter than `q`: the process gives up the CPU itself.
  - Burst longer: timer interrupt, context switch, process goes to the **tail** of the ready queue.
- A preempted process goes to **ready**, not waiting, because it did not request IO `[29:35]`.
- Fair: with $n$ processes each gets $\frac{1}{n}$ of the CPU, and a process waits at most $(n-1)\,q$ `[29:50]`.
- Quantum too large: behaves like FCFS `[30:12]`. Quantum too small: too many context switches, which are pure overhead `[30:34]`.
- Preemption needs a hardware timer (V3) `[29:19]`.

*Q: Too small a quantum causes what? A: Switching overhead.*

**11. Response time vs the speed limit** `[32:17]`
- RR is the simplest way to get concurrency `[32:53]`. As process count grows, each waits longer and the illusion breaks `[33:12]`.
- Shrinking the quantum just adds context switches `[33:35]`.
- Three ways out: faster CPU, more cores, or a **smarter scheduler** `[33:53]`. The third is software only.

**12. Priority scheduling** `[34:17]`
- Not every process needs the same CPU share. A game and a mail sync are not equal `[34:21]`. Higher priority for the right ones improves response without new hardware `[35:09]`.
- Implementation: a **priority queue** with an integer per process. Ranges vary (0 to 255, 0 to 32000), and which direction is "high" varies by OS `[35:35]`.
- Can combine with RR so equal priorities share fairly `[36:00]`.
- **Starvation** appears again `[36:15]`. SJF is just priority scheduling where priority is the inverse of the predicted burst `[36:29]`.
- Trivia: the IBM 7094 was reportedly shut down in 1973 with a low priority job from 1967 never run `[36:41]`.
- Fix: **aging**. Slowly raise the priority of waiting processes, for example +1 every second until a priority 127 job reaches 0 `[37:03]`.

*Q: What does aging fix? A: Starvation of low priority.*

**13. Multilevel queue** `[37:34]`
- One queue per priority. Searching one big queue costs more `[37:41]`.
- Scheduling **between** queues: strict priority (high queue always first) or time slicing across queues so low ones still get turns `[37:57]`.
- Each queue can run a different policy: foreground RR, background FCFS `[38:35]`. That is the shape of a general purpose scheduler `[39:00]`.

**14. Multilevel feedback queue (MLFQ)** `[39:08]`
- Static queue assignment is inflexible. Your mail client idles in the background, then needs instant response when you open it `[39:25]`.
- Rules `[39:54]`:
  - Everyone starts in the highest priority queue.
  - Finish the burst within the quantum: stay (or move up).
  - Need more than the quantum: move **down** to a lower queue, which has a **larger quantum**.
- Interactive and short burst processes stay high. CPU hogs **sink** `[40:28]`.
- **Mnemonic:** short bursts rise, long bursts sink.
- "Doesn't that punish CPU bound work?" `[41:21]` Two facts: IO bound and interactive both have short bursts, and the CPU is mostly idle (apps combined use under 10%) `[42:00]`. So the high queues are mostly empty and low queues still get served `[42:15]`.
- That is why a 100% CPU benchmark leaves the desktop smooth `[42:36]`.

*Q: Where do CPU hogs end up in MLFQ? A: The lowest queue.*

**15. Modern reality** `[43:13]`
- Real OSes schedule **threads**, not processes `[43:38]`. The UI thread keeps high priority while a worker thread grinds `[43:55]`.
- The IO queue shown is simplified. The IO scheduler handles devices `[44:10]`.
- Note beyond the video: Linux replaced CFS with EEVDF in kernel 6.6.

**x86 vs RISC-V: the hardware parts of a scheduler**

| Scheduler need | x86_64 | RISC-V |
| :--- | :--- | :--- |
| Arm the quantum timer | Local APIC timer (TSC deadline via `wrmsr`) | `mtimecmp`, or `stimecmp` with Sstc. Often via the SBI call `sbi_set_timer` |
| Nothing to run (idle task) | `hlt` (privileged) | `wfi` (wait for interrupt) |
| Hand the CPU back to a process | `iretq` or `sysretq` | `sret` |
| Context saved in the PCB | `rip`, `rsp`, `rflags`, 16 GPRs and more | `pc` (as `sepc`), `sp`, 31 GPRs |

Linux's switch between kernel threads only needs the **callee saved** registers, because the caller already treats the others as clobbered across a call. On RISC-V that is `ra`, `sp` and `s0` to `s11`.

**Try it**

```bash
ps -eo pid,ni,pri,cls,comm --sort=-pcpu | head
nice -n 10 sha256sum bigfile
renice -n 5 -p 1234
chrt -p $$
grep ctxt /proc/$$/status
```

| Piece | What it does |
| :--- | :--- |
| `ps -eo ...` | `-e` all processes, `-o` choose columns |
| `ni`, `pri`, `cls` | Nice value, priority, scheduling class (`TS` normal, `FF` realtime FIFO, `RR` realtime round robin) |
| `--sort=-pcpu` | Sort by CPU percent. The leading `-` means descending, a convention to memorize |
| `nice -n 10 cmd` | Start `cmd` with niceness +10. **Mental model:** a higher value means nicer to others, so lower priority. Range is -20 to 19 |
| `renice -n 5 -p 1234` | Change the niceness of a running PID. `-p` marks the value as a process id |
| `chrt -p $$` | Show the scheduling policy and realtime priority of your shell |
| `grep ctxt /proc/$$/status` | Voluntary switches (it blocked) vs nonvoluntary (the timer preempted it) |

### V9: Security angle
- **Attack:** denial of service through CPU and process exhaustion (fork bombs, busy loops). Crypto miners show up as steady 100% CPU. Realtime priorities can lock up a box if someone gains `CAP_SYS_NICE`. Scheduling patterns can also leak timing information.
- **Defend:** cgroup limits (`cpu.max`, `pids.max`), `ulimit -u`, quotas per user, and alerts on sustained CPU or an unexpected `nice -n -20`. On the blue team, sorting by CPU is the first triage move.

### V9: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| CPU burst vs IO burst | Process alternates between computing and waiting | Why other processes can use gaps |
| Process states | new, ready, running, waiting, terminated | Reading `ps` and `top` states |
| Scheduler vs dispatcher | Decide vs execute the switch | Dispatch latency is overhead |
| FCFS and convoy effect | FIFO ready queue, long burst blocks everyone | Why FCFS is not general purpose |
| SJF and burst prediction | Shortest next burst first, estimated by $\tau_{n+1} = \alpha t_n + (1-\alpha)\tau_n$ | Optimal average waiting time |
| Preemption and starvation | OS can take the CPU, policies can starve | Timer driven, fairness |
| Round robin | FIFO plus quantum, waits at most $(n-1)q$ | Fair response, tune `q` |
| Priority, aging | Integer priority, raise over time | Starvation fix, `nice` |
| MLFQ | Multiple queues, move by burst behavior | How desktops stay smooth |

### V9: Feynman checks
1. **The CPU is at 100% from a benchmark but your terminal still responds. Which mechanism explains it?** The terminal has short CPU bursts, so MLFQ keeps it in a high priority queue. The benchmark has long bursts and sinks to a low queue, and it only gets leftover time.
2. **Why can SJF not be implemented exactly, and what do real schedulers do instead?** The next burst length is unknowable (loops, input). They predict it with an exponentially weighted average of past bursts.
3. **Someone proposes a 1 microsecond quantum for maximum responsiveness. Why does it backfire?** Context switches cost time with no useful work. Overhead dominates and throughput and turnaround time get worse.

### V9: Omitted chaff
- Brilliant sponsor `[05:13]`.
- Intro about benchmark satisfaction `[00:00]`.
- Table of contents note and like request `[00:43]`.
- Chart and animation commentary, IBM 7094 trivia (kept as one line) `[36:37]`.

*Q: Any algorithm missing? A: No: FCFS, SJF, RR, priority, MLQ, MLFQ.*

### V9: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| CPU scheduling overview | `cpu scheduling algorithms explained visually FCFS SJF round robin` |
| Convoy effect | `convoy effect fcfs scheduling explained simply` |
| SJF burst prediction | `shortest job first exponential averaging prediction explained` |
| Round robin and time quantum | `round robin scheduling time quantum tradeoff explained` |
| MLFQ | `multilevel feedback queue scheduler explained intuition` |

---

## Video 10: The Weirdest Bug in Programming: Race Conditions
- Duration: 18:19

### V10: 80/20 digest

**1. Most operations are not atomic** `[00:48]`
- One line of source is not one CPU instruction. Adding two numbers needs: load each operand into a register, add, store the result back `[01:08]`.
- First concept to burn in: **most operations are not atomic** `[01:34]`. (Atomic means it finishes as one indivisible step.)

*Q: Is `a = b + c` one CPU instruction? A: No, several.*

**2. Threads recap** `[01:42]`
- `pthread_create` asks the OS to run a function as a separate executable entity. The caller keeps going without waiting `[01:58]`.
- Concurrency is not parallelism. On one core the threads take turns `[02:21]`.
- Typical use: an HTTP server where the main thread listens and spawns a handler thread per request `[03:01]`. Independent handlers are fine. Trouble starts when threads share data `[03:36]`.

**3. The experiment** `[04:14]`
- Shared global string. A writer thread cycles through hardcoded sentences. Handler threads copy it into each response `[04:14]`. Everything looks fine.
- A search program over the logs found one response with a wrong last letter: **1 in over 9 million** `[04:42]`.
- After about 85 million requests, more responses were **mixes of different sentences** `[05:19]`.

*Q: How often did the first bad response appear? A: About 1 in 9 million.*

**4. Why: strings are multi step** `[07:23]`
- C strings are characters ending in a null byte `[07:38]`. Printing loops over them `[08:00]`. Copying loops too `[08:09]`. Even chunked copies need several steps `[08:27]`.
- If the OS interrupts the writer midcopy, the string is in an **invalid state**: a value we never meant to exist `[08:44]`.
- A reader dispatched before the writer resumes copies the half updated string `[09:04]`. That is why errors are fragments of valid sentences `[09:25]`.

*Q: Why did responses contain mixed sentences? A: Reader caught writer midcopy.*

**5. Definition** `[09:44]`
A **race condition** is a bug where the outcome depends on the unpredictable order in which threads or processes access and modify shared resources.
- Side effect of **preemptive** systems `[09:56]`: the timer interrupt can hit anywhere `[10:38]`.
- Operation longer than one time quantum: guaranteed interruption `[10:49]`.
- Shorter does not mean safe. The operation may start near the end of a quantum `[11:08]`, and other interrupts (network packet, key press) can preempt at any time `[11:15]`. You cannot disable them without making the system less responsive `[11:39]`.

*Q: Does a short operation avoid races? A: No, interrupts can land anywhere.*

**6. A naive fix fails: a flag variable** `[11:50]`
- Idea: `writing = 1` while modifying, readers check it `[12:03]`.
- The flag is now **shared data** too `[12:25]`. Reading and comparing is not atomic: load into a register, compare, act `[12:35]`.
- Failure: the reader loads the flag (0), gets interrupted, the writer sets it to 1 and starts writing, the reader resumes with the **stale** 0 in its register and reads anyway `[13:01]`.
- Avoid plain variables as locks `[13:46]`. Classic tricks like **Peterson's solution** rely on subtle assumptions about compilers and hardware `[13:56]`. Real tools come in the thread synchronization video.

*Q: Why does a plain flag not work as a lock? A: Check and set aren't atomic.*

**7. Multiple writers: the lost update** `[14:18]`
- Shared counter, each thread that finds a prime does `count++` `[14:28]`.
- Increment is load, add, store `[14:55]`.
- **Parallel case:** two cores load the same value, both add 1, both store. Count rises by 1 instead of 2 `[15:09]`. No interrupt needed `[15:25]`.
- **Interrupt case:** count is 10, five threads find primes, expected 15 `[15:35]`. Thread 1 loads and adds, then is preempted before the store. Threads 2 to 5 finish and the count hits 14. Thread 1 resumes and stores its stale 11 `[15:50]`. Four updates vanish `[16:28]`.

| Step | Count in memory | Thread 1 register |
| :--- | :--- | :--- |
| Thread 1 loads and adds | 10 | 11 |
| Threads 2 to 5 finish | 14 | 11 |
| Thread 1 stores | **11** | 11 |

*Q: Why did the counter drop from 14 to 11? A: Stale register stored.*

**8. Why it is the weirdest bug** `[16:35]`
- Computers follow instructions and do not notice the problem `[16:38]`.
- Re-running gives a different result almost every time, sometimes even the right one. It depends on uncontrollable timing `[16:52]`.
- Named a **race** because operations race to the shared data and the result depends on who finishes first `[17:10]`.
- Worst feature: **rarity**. 14 bad responses in 85 million `[17:26]`. Every thread you spawn can be a time bomb `[17:44]`.

**x86 vs RISC-V: `counter++`**

| | x86_64 | RISC-V (RV64) |
| :--- | :--- | :--- |
| Plain C `counter++` | Often one instruction: `add DWORD PTR [counter], 1`, or three: load, `inc`, store | Always three: `lw`, `addi`, `sw` (load/store architecture) |
| Safe across interrupts on one core? | A single read modify write instruction, yes | No, it can split between `lw` and `sw` |
| Safe across cores? | **No**, still racy | No |
| Atomic version | `lock add DWORD PTR [counter], 1` (or `lock xadd`) | `amoadd.w zero, t1, (a0)` (atomic memory operation) |
| Optional loop style | `lock cmpxchg` | `lr.w` and `sc.w` (load reserved, store conditional) |

```asm
# riscv: the 3 steps are always visible, which is the race window
lw   t0, 0(a0)      # load count
addi t0, t0, 1      # add 1 in a register
sw   t0, 0(a0)      # store back, overwrites whatever others wrote since the load
```

```asm
; x86_64: one instruction hides the window, but only from interrupts
add  DWORD PTR [counter], 1      ; racy on multicore
lock add DWORD PTR [counter], 1  ; atomic: the lock prefix locks the cache line
```

**Mental model:** on RISC-V the load, add and store are separate instructions, so the gap is in plain sight. x86 can pack them into one instruction, which gives a false sense of safety on multicore. The `lock` prefix says "no one else touches this cache line until I finish".

**Try it**

```c
#include <pthread.h>
#include <stdatomic.h>
long plain_count = 0;            // racy
atomic_long safe_count = 0;      // atomic
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
long locked_count = 0;

void *work(void *arg) {
    for (int i = 0; i < 1000000; i++) {
        plain_count++;                         // bug: load, add, store
        atomic_fetch_add(&safe_count, 1);      // fix 1: one atomic step
        pthread_mutex_lock(&lock);             // fix 2: one thread at a time
        locked_count++;
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}
```

```bash
gcc -fsanitize=thread -g -O1 race.c -o race -pthread
./race
```

| Piece | What it does |
| :--- | :--- |
| `atomic_fetch_add(&c, 1)` | Adds 1 as a single indivisible read modify write. Compiles to `lock xadd` or `amoadd` |
| `pthread_mutex_lock` and `unlock` | Only one thread holds the lock, others wait. The lock is the critical section guard |
| `-fsanitize=thread` | Compile in ThreadSanitizer: instruments memory accesses and reports data races |
| `-g` | Keep debug info so reports show line numbers |
| `-O1` | Light optimization. TSan is slow at `-O0` |
| `-pthread` | Link the thread library |

Run it with two threads and print `plain_count`. It will be below 2,000,000 and change every run. The other two will hit the exact value.

### V10: Security angle
- **Attack:** **TOCTOU** (time of check, time of use): a program checks a file with `access()`, then opens it, and the attacker swaps in a symlink between the two. Dirty COW (CVE-2016-5195) was a kernel race that let an unprivileged user write to read only memory and get root. On the web, **limit overrun races** (redeem a coupon twice, withdraw twice) fire many requests at once so they all pass the check before any updates the balance. Attackers just loop until they win, because a rare race becomes certain with enough tries.
- **Defend:** make check and use one atomic step. Use `O_EXCL`, `O_NOFOLLOW`, `openat`, and operate on file descriptors, not paths. Use `mkstemp` for temp files. Use atomic DB updates (`UPDATE ... WHERE balance >= x`) and row locks. Run TSan in CI.

### V10: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Atomic operation | Finishes as one indivisible step | `lock add`, `amoadd`, `atomic_fetch_add` |
| Invalid state | Shared data caught midupdate | Mixed strings, half written structs |
| Race condition | Outcome depends on unpredictable access order | Rare, timing dependent bugs |
| Lost update | Two read modify writes overwrite each other | Counters, balances |
| Preemption | Timer and other interrupts hit anywhere | Why short code is not automatically safe |
| Flag variable as lock | Check and set not atomic | Why you need real synchronization |
| TOCTOU | Gap between check and use | Privesc, symlink attacks |

### V10: Feynman checks
1. **Why can `counter++` lose updates even with no interrupts at all?** On a multicore machine two cores can load the same old value, each add 1, and store the same result.
2. **Why does a `volatile int locked` flag not fix the string bug?** Check and set are separate steps. A thread can read the old value, get preempted, and act on it after the writer already started.
3. **Your coupon endpoint checks `used == false`, then sets it. How do you break it, and how do you fix it?** Send many parallel requests so they all pass the check. Fix with one atomic statement or a lock: `UPDATE coupons SET used = 1 WHERE id = ? AND used = 0`, and check the affected row count.

### V10: Omitted chaff
- Flexispot sponsor `[05:50]`.
- Log file hunting details and the "9 million" rerun narrative (kept as the numbers) `[04:42]`.
- "Computers are dumb" rant (kept as one idea) `[16:38]`.
- Next episode teaser on synchronization `[18:00]`.

*Q: Anything about fixes left out? A: Only a teaser, synchronization is next.*

### V10: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Race condition basics | `race condition explained animated threads counter` |
| Lost update | `lost update problem counter++ not atomic explained` |
| Atomic instructions | `atomic operations lock prefix compare and swap explained` |
| ThreadSanitizer | `threadsanitizer tsan data race detection tutorial` |
| TOCTOU and web races | `toctou race condition vulnerability explained limit overrun` |

---

## Video 11: Why Can't Programs Access Each Other's Memory?
- Duration: 14:51

### V11: 80/20 digest

**1. Memory as the CPU sees it** `[00:00]`
- Memory is a grid of tiny cells. Each cell has a unique position, so a binary **address** pinpoints it `[00:30]`.
- One bit holds only two states. So an address maps to a **byte** (8 bits, 256 states) `[01:00]`.
- Developers see memory as a huge array of bytes with unique addresses `[01:27]`.

**2. Only RAM and registers are directly reachable** `[01:44]`
- `load` and `store` take memory addresses. **No instruction takes a disk address** `[02:08]`.
- Everything the CPU runs, instructions and data, must be in memory first `[02:09]`.
- Even incrementing a counter: the instruction fetch is a memory read, `load` is a read, `store` is a write `[02:32]`.

*Q: Can the CPU read straight from disk? A: No, only RAM and registers.*

**3. Concurrency affects memory too** `[03:03]`
- A launched program is loaded into memory and becomes a process `[03:46]`. Its total memory is the **address space** `[04:02]`.
- **Rule:** by default, a process must never reach another process's address space `[04:12]`.
- **Read** attack: a payment form's buffer holds the user's input. Any process running `load` instructions could copy it and send it out `[04:24]`.
- **Write** attack: change the amount or recipient, or even the code of another process or the OS `[05:12]`. The address space is sacred `[05:39]`.

*Q: What can a process do with write access to another's memory? A: Alter its data or code.*

**4. Why hardware, not software** `[05:47]`
- Software would need extra instructions around **every** memory access of the user process, a massive overhead `[05:55]`. So the hardware must do it `[06:27]`.

**5. Base and limit registers** `[08:27]`
- Memory is untouched. The CPU puts addresses on the **address bus**. Memory does not know where they come from `[08:09]`.
- Two special registers define the process's legal range `[08:36]`:
  - **base**: smallest legal physical address.
  - **limit**: size of the range.
- Check on every access:

$$base \le address < base + limit$$

- The upper bound is **exclusive**, hence `<` not `<=` `[09:12]`.
- Example: `base = 0x4000`, `limit = 0x1000`. Legal range is `0x4000` to `0x4FFF`. `0x4FFF` passes. `0x5000` fails.
- Hardware parts: one **adder** for `base + limit` `[09:25]`, two **comparators** (`address >= base`, `address < upper`) `[09:47]`, one **AND gate** so both must be true `[10:30]`.

*Q: Is `base + limit` itself a legal address? A: No, the bound is exclusive.*

**6. How one wire blocks access** `[10:47]`
- Memory acts only when **read enable** or **write enable** is on. Read puts data on the data bus. Write overwrites the cell `[11:02]`.
- The validator's output **gates** those enable signals: if the address is out of range, the signal never reaches memory `[11:35]`.
- Extra gates also raise an **interrupt** so the OS can deal with it, usually by killing the process `[12:04]`. On Unix like systems you see **segmentation fault** `[12:26]`.

*Q: What do you see when a process breaks the bounds? A: Segmentation fault.*

**7. The OS has almost nothing to do** `[12:36]`
- No extra OS instructions run per access. The circuit validates automatically `[12:40]`.
- OS job: set `base` and `limit` before dispatching `[12:58]`.
- These registers must be **privileged**, writable only in kernel mode (V3) `[13:14]`.
- On every **context switch** (V2) the OS also swaps `base` and `limit` for the incoming process, then drops to user mode `[13:30]`.
- Next episode topic: paging, using more memory than physically exists `[14:21]`.

**8. Mental model**
Think of base and limit as a fence around your yard that the building itself enforces, not a guard checking each step. The OS only moves the fence when a new tenant arrives.

**x86 vs RISC-V: the same idea in real hardware**

| Mechanism | x86 | RISC-V |
| :--- | :--- | :--- |
| Literal base plus limit | **Segmentation** in protected mode: each segment descriptor has base and limit, checked on every access. Loaded with privileged `lgdt`. In 64 bit mode most bases are 0 and limits ignored | **PMP** (physical memory protection): `pmpcfg` and `pmpaddr` CSRs define ranges and R, W, X rights. Range checks like base and limit |
| What replaced it for processes | **Paging**: `cr3` points at page tables, bits `U/S`, `R/W`, `NX` | **Paging**: `satp` CSR points at page tables, bits `U`, `R`, `W`, `X` |
| Violation signal | `#GP` (13) or `#PF` (14), kernel sends `SIGSEGV` | Access fault or page fault (`scause` 12, 13, 15 for page faults), kernel sends `SIGSEGV` |
| Privileged to change it | Yes, ring 0 | Yes, M and S mode |

The video's design is the simple ancestor. Modern CPUs check **page by page** instead of one range per process, which is what makes paging (next topic) possible.

**Try it**

```bash
cat /proc/self/maps
dmesg | grep -i segfault
```

| Piece | What it does |
| :--- | :--- |
| `/proc/self` | Symlink to the PID of whoever is reading it. Here, `cat` itself |
| `maps` columns | Address range, permissions `rwxp`, offset, device, inode, path |
| `dmesg` | Kernel log buffer |
| `grep -i segfault` | `-i` ignores case. Shows lines like `... segfault at 0 ip ... sp ...` with the instruction and stack pointer |

Make one on purpose:

```c
int main(void) { int *p = 0; *p = 1; }   // store to address 0: outside any legal range
```

The store triggers the check, the hardware raises a fault, the OS kills the process: `Segmentation fault`. With `ulimit -c unlimited` you also get a **core dump**, which is where the channel gets its name.

### V11: Security angle
- **Attack:** memory is the prize. Credential dumpers read another process's memory (Mimikatz reading `lsass.exe`), RAM scrapers lift card data, and exploits try to break the boundary itself. Meltdown (CVE-2017-5754) let user code speculatively read kernel memory before the permission check took effect.
- **Defend:** the page permission bits do most of the work: `NX` (no execute) stops running shellcode from the stack, `SMEP` and `SMAP` keep the kernel off user pages, **ASLR** randomizes where things sit, and KPTI mitigates Meltdown. On Linux, `kernel.yama.ptrace_scope=1` limits one process from reading another with `ptrace` or `/proc/<pid>/mem`. On Windows, Credential Guard and protected processes guard `lsass`.

### V11: Cheat sheet

| Concept | Explanation | Practical use |
| :--- | :--- | :--- |
| Address space | Memory a process may use | `/proc/<pid>/maps` |
| Byte | 8 bits, smallest addressable unit | Pointers, buffers |
| Base and limit | Range check on every access: $base \le a < base + limit$ | Ancestor of segmentation and PMP |
| Read and write enable gating | Block the signal to memory when out of range | Why illegal access never hits RAM |
| Segmentation fault | Hardware interrupt, OS terminates the process | Debugging crashes, core dumps |
| Privileged memory registers | Only kernel mode may set them | Isolation survives user bugs |
| Context switch tie in | OS swaps bounds for the next process | Per process isolation |

### V11: Feynman checks
1. **Why can't the OS simply check each memory access in software?** It would run extra instructions on every access of every process, destroying performance. A comparator circuit does the check for free, in parallel.
2. **Why is `limit` compared with `<` rather than `<=` against `base + limit`?** `base + limit` is the first byte past the range, so the upper bound is exclusive.
3. **A malware sample tries to read another process's password buffer. Name two layers that stop it.** The MMU or bounds check blocks direct loads from another address space, and OS policy (ptrace scope, protected processes) blocks the sanctioned routes.

### V11: Omitted chaff
- Brilliant sponsor `[06:30]`.
- Reminder to like the video `[03:18]`.
- Transistor and capacitor memory cell recap `[00:10]`.
- Gate level circuit drawing details (kept as adder, comparators, AND) `[09:47]`.

*Q: Any core memory protection piece missing? A: No, bounds check and gating covered.*

### V11: Watch it explained

| Concept | YouTube search query |
| :--- | :--- |
| Memory protection | `base and limit registers memory protection explained animated` |
| Segmentation fault | `what is a segmentation fault really explained hardware` |
| Address space isolation | `process address space isolation how OS protects memory visually` |
| Meltdown | `meltdown vulnerability explained user kernel memory isolation` |
| `/proc/PID/maps` | `linux proc maps memory layout explained` |

