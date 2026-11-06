# chapter 3: os design, architecture and boot

> **disclaimer:** i don't claim authorship of any of this. these are study notes from the neso academy operating systems playlist, a complement to those excellent lectures, not a replacement. watch the videos.

**videos covered (playlist order):**

| # | video | link |
|---|---|---|
| 1 | Operating System Design & Implementation | https://www.youtube.com/watch?v=t_McsJ1RGQg |
| 2 | Structures of Operating System | https://www.youtube.com/watch?v=XXPBl20J22w |
| 3 | Virtual Machines | https://www.youtube.com/watch?v=daDbY2iDmU0 |
| 4 | Operating System Generation and System Boot | https://www.youtube.com/watch?v=wD0PrF3fGSY |

## table of contents
- [design and implementation](#operating-system-design--implementation)
- [structures of operating system](#structures-of-operating-system)
- [virtual machines](#virtual-machines)
- [generation and system boot](#operating-system-generation-and-system-boot)
- [cheat sheet](#cheat-sheet)
- [interleaving](#interleaving)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## operating system design & implementation

**the problem of goals.** defining goals is hard because users and builders want different things and not all can be met. you can fix the hardware choice and the type of system (multiprocessing, multitasking, real time). beyond that, requirements get vague.

| who | wants |
|---|---|
| user goals | convenient, easy to learn and use, reliable, safe, fast |
| system goals (designers) | easy to design, implement, maintain, operate; flexible, reliable, error free, efficient |

there's no formula; software engineering gives principles, and creativity does the rest.

**mechanism vs policy.** mechanism is **how** something is done; policy is **what** will be done.
- car: the engine and brakes are the mechanism; "don't exceed 50 km/h" is the policy.
- os: how a resource gets allocated is the mechanism; whether this process gets it is the policy.

**principle: separate policy from mechanism.** if the speed limit changes to 40, you change the policy only, not rebuild the car. if the two are fused, every policy change means rewriting the mechanism.

**implementation language.** early oses were assembly; modern ones are mostly c or c++.

| high level language benefit | why |
|---|---|
| written faster | fewer lines |
| more compact | less boilerplate |
| easier to understand and debug | readable |
| easier to port | not tied to one cpu |

example: ms dos was written in intel 8088 assembly so it only ran on intel cpus. linux, mostly c, runs on intel, motorola, sparc, mips and more.

*Q: why does separating policy from mechanism help? A: change what happens without rewriting how.*

---

## structures of operating system

five structures, in the order the lecture builds them:

### 1. simple structure (ms dos)
layers (rom bios and device drivers, resident system programs, application programs) exist but are **not truly separated**: applications can reach the base hardware directly *(see diagram, search anchor: "ROM BIOS device drivers")*.
- a failing user program can crash the whole system.
- the developers had little choice: the intel 8088 had **no dual mode or hardware protection**.

### 2. monolithic structure (early unix)
everything (signals, file system, scheduling, virtual memory, drivers) packed into one big **kernel** level; users go through the system call interface to it.
- hard to implement and maintain; fixing the cpu scheduler means touching the entire kernel.

### 3. layered structure
the os is divided into layers: layer 0 hardware up to layer n user interface. each layer uses only services of the layers below.
- **pros:** easy to implement and debug (debug one layer); hardware is protected from upper layers.
- **cons:** hard to design (e.g. the backing store layer must sit below memory management, which needs it); **slower** because a request passes down through every layer, parameters possibly modified at each.

### 4. microkernel
keep only core functions in the kernel; push everything else (file servers, device drivers, process servers) to user level programs. the kernel mostly provides **message passing** between clients and services.
- **pros:** most functions run in **user mode**, so a failure doesn't crash the system.
- **cons:** performance falls due to message passing overhead.

### 5. modules (modular kernel)
a core kernel plus **loadable modules** (device and bus drivers, scheduling classes, file systems, loadable system calls, executable formats) loaded at boot or at runtime. uses object oriented techniques.
- like layered: each section has a defined, protected interface, **but** any module can call any other, so it's more flexible.
- like microkernel: a small core, but modules load straight into the kernel, so **no message passing overhead**.
- by far the best current methodology; used by most modern oses.

*Q: why is the modular kernel faster than a microkernel? A: modules call directly, no message passing.*

---

## virtual machines

**idea.** abstract the hardware of one computer (cpu, memory, disks, nic) into several execution environments, creating the illusion that each is running its own private computer *(see diagram, search anchor: "virtual machine implementation")*.

- the host runs the virtual machine software in **kernel mode**.
- each virtual machine runs in **physical user mode**, with its own guest kernel and processes.
- each vm needs its own **virtual user mode and virtual kernel mode**, both of which actually run inside the physical user mode.
- disk is split into **minidisks**, one per vm, so vms are isolated.

example: windows host, with virtualbox or vmware running linux mint, ubuntu and fedora guests at once.

**benefits:** protection of resources (isolation), the ability to run several oses on one machine, and safe testing.

*Q: in what mode does a vm run on the host? A: physical user mode.*

---

## operating system generation and system boot

**two ways to build an os:**
1. design, code and implement for **one machine at one site**: simple but not reusable.
2. design to run on **any of a class of machines** with various peripheral setups (the common modern way).

for the second, the os must be configured for each machine: **system generation (sysgen)**. the sysgen program must determine:

1. which **cpu** is used,
2. how much **memory** is available,
3. which **devices** are present,
4. which **os options** are wanted.

example: the same linux mint cd installs on your pc and your friend's different pc.

**booting.** booting is starting a computer by loading the kernel.
- the hardware doesn't know where the kernel is, so a small piece of code, the **bootstrap program (bootstrap loader)**, locates it.
- it lives in **rom** because ram is in an unknown state at startup, and rom needs no initialization and can't be infected by a virus.
- **firmware** is rom style storage; small devices (phones) may keep both loader and os there. updating means replacing the chip, which led to **eeprom** (electrically erasable programmable rom, writable on explicit command).
- on desktops and laptops: the small loader lives in rom; the os lives on disk; the loader traverses the file system, finds the kernel, loads it into memory and starts it. only then is the system **running**.

*Q: why does the loader sit in rom? A: ready at power on and virus proof.*

---

## cheat sheet

| concept | explanation | real use |
|---|---|---|
| user vs system goals | what users want vs what builders want | requirements gathering |
| mechanism vs policy | how vs what | schedulers with swappable policies |
| separate policy and mechanism | change one without the other | flexibility |
| high level implementation | c or c++ instead of assembly | portability |
| simple structure | no real layers, direct hardware access | ms dos, fragile |
| monolithic | all in one kernel level | early unix, hard to maintain |
| layered | strict layers using only lower ones | easy debug, slower |
| microkernel | tiny kernel, services in user space | resilience, message passing cost |
| modular | core plus loadable modules | modern linux, solaris |
| virtual machine | many guest oses on one host | cloud servers, testing |
| sysgen | tailor an os to a machine | installers detecting hardware |
| bootstrap loader | rom code that loads the kernel | power on |

## interleaving

| pair | difference |
|---|---|
| layered vs modular | strict downward calls vs any module calls any module through the core |
| microkernel vs modular | services in user space with messages vs modules in kernel, no messages |
| simple vs layered | looks layered but hardware reachable from all vs real protection |
| monolithic vs microkernel | everything in the kernel vs minimal kernel |
| vm kernel mode vs host kernel mode | virtual, runs in physical user mode vs real privileged |

## feynman checks

1. **you must add a new file system to a running system without rebooting. which structure makes this easiest and why?** modular (loadable modules): load a file system module at runtime into the core kernel.
2. **a driver bug crashes the machine. how does a microkernel limit this?** drivers run as user mode programs, so a crash kills that service, not the kernel.
3. **why can't ms dos protect itself the way a modern os can?** the 8088 cpu had no dual mode or hardware protection, so applications could write straight to hardware.

## review questions

- (ch1) what role does the bootstrap program play, and where is it stored?
- (ch2) what's the difference between user mode and kernel mode, and how does a vm complicate it?
- (ch2) which os structure puts most services in user mode?

## security angle

- **layered and microkernel** shrink the trusted computing base: less code in kernel mode means fewer bugs that grant full control.
- **loadable modules** are an attack path: malicious kernel modules (rootkits). defense: signed modules, module loading disabled after boot.
- **vm escape:** breaking out of a guest to the host is the prize; keep hypervisors patched and isolate sensitive guests.
- **bootkits** infect the loader; secure boot verifies signatures at each stage.

## omitted chuff
- the longer story of why design goals are "vague", beyond the user and system goal lists.
- the offer of a vmware/virtualbox demo video.
- repeated explanations of what rom, ram and volatility mean (covered in chapter 1).

*Q: best one line summary of modern os structure? A: small core, loadable modules, strong interfaces.*
