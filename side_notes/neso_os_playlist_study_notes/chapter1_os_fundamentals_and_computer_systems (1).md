# chapter 1: os fundamentals and computer systems

> **disclaimer:** i don't claim authorship of any of this. these are my study notes, built from the neso academy operating systems playlist. they are not meant to replace the lectures, only to complement that amazing material. watch the videos.

**videos covered in this chapter (in playlist order):**

| # | video | link |
|---|---|---|
| 1 | Introduction to Operating Systems | https://www.youtube.com/watch?v=vBURTt97EkA |
| 2 | Basics of OS (Computer System Operation) | https://www.youtube.com/watch?v=VjPgYcQqqN0 |
| 3 | Basics of OS (Storage Structure) | https://www.youtube.com/watch?v=YcRd3WMbXnE |
| 4 | Basics of OS (I/O Structure) | https://www.youtube.com/watch?v=F18RiREDkwE |
| 5 | Computer System Architecture | https://www.youtube.com/watch?v=So9SR3qpWsM |

## table of contents
- [introduction to operating systems](#introduction-to-operating-systems)
- [basics of os (computer system operation)](#basics-of-os-computer-system-operation)
- [basics of os (storage structure)](#basics-of-os-storage-structure)
- [basics of os (i/o structure)](#basics-of-os-io-structure)
- [computer system architecture](#computer-system-architecture)
- [cheat sheet](#cheat-sheet)
- [interleaving: compare the look alikes](#interleaving-compare-the-look-alikes)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## introduction to operating systems

**what it is.** an operating system (os) is a program that manages the computer hardware, gives application programs a base to run on, and sits as an intermediary between the user and the hardware. examples: windows, linux, ubuntu, macos, ios, android.

**the layers, bottom to top:**

```
users
  application programs (word processor, spreadsheet, compiler, text editor, browser)
    operating system
      computer hardware (cpu, memory, i/o devices)
```

**why we need it (the thought experiment).** remove the os. to type a document in a word processor, the user would have to write code telling the hardware: load the program into main memory, display each keystroke on the monitor, then save the file under a name to the hard disk. every tiny task needs explicit code. nobody but programmers could use a computer. the os does all of that behind a double click.

*Q: what would a user do without an os? A: hand write code for every hardware action.*

**the three jobs the lecture lists:**

| function | meaning |
|---|---|
| interface between user and hardware | user double clicks, os talks to hardware |
| allocation of resources | cpu, memory, i/o are limited, so the os decides who gets what |
| management of memory and security | where things are stored, and how safely |

**two goals:**
- **convenience:** make the computer easy to use.
- **efficiency:** use limited resources well.
many systems favor one, most try for both.

**types of os (named only, explained later in the series):** batch, time sharing, distributed, network, real time, multi programming, multi processing, multitasking.

*Q: why is resource allocation needed at all? A: resources are finite and processes compete.*

---

## basics of os (computer system operation)

the lecture builds the hardware picture the os will manage.

**the structure.** a modern general purpose computer has one or more cpus and a number of device controllers connected through a common bus that gives access to shared memory *(see diagram, search anchor: "common bus that provides access")*.

- **cpu:** the chip on the motherboard that computes. not the whole box (cabinet) people call "the cpu".
- **device controller:** one per device type (disk controller, usb controller, video adapter). it runs the device.
- **common bus:** the shared wire all of them use to reach memory.
- **memory controller:** the cpu and controllers run **concurrently** and compete for memory cycles, so a memory controller synchronizes access. that is why you can watch a video and type at once without lag.

*Q: who stops devices clashing on memory? A: the memory controller, by synchronizing access.*

**bootstrap program.** the first program that runs at power up or reboot.
- stored in **rom** (read only memory).
- must know how to locate the os **kernel** on secondary storage and load it into main memory.
- why rom: ram is in an unknown state at startup, and rom needs no initialization and can't be modified by a virus.

**interrupt.** a signal that an event happened.
- **hardware interrupt:** a device signals the cpu through the system bus.
- **system call (monitor call):** software triggers an interrupt by running a special operation.

**what the cpu does on an interrupt:**
1. stop the current work.
2. jump to a fixed location holding the starting address of the **interrupt service routine (isr)**.
3. run the isr to completion.
4. resume the interrupted computation.

analogy: someone taps your shoulder mid sentence. you note where you were, handle them, come back.

*Q: hardware signal vs software signal, names? A: interrupt vs system call.*

---

## basics of os (storage structure)

**the hierarchy (top to bottom):** registers, cache, main memory, electronic disk, magnetic disk, optical disk, magnetic tape.

| as you go up | as you go down |
|---|---|
| faster | slower |
| smaller | bigger |
| costlier per bit | cheaper per bit |

**main memory (ram).** anything you run gets loaded here. it's fast but limited and **volatile**.
**secondary storage** holds everything permanently and is **non volatile**.

| | volatile (loses data at power off) | non volatile (keeps data) |
|---|---|---|
| devices | registers, cache, main memory | electronic disk*, magnetic disk, optical disk, tape |

*electronic disk can be built either way; nvram is a battery backed ram, a special case.

**the bookshelf analogy (the why).** secondary memory is a big bookshelf, ram is a small table. you can only read books that are on the table. a tiny table means few open books at once, so more ram means more things loaded together and the machine feels faster.

example: ms word sits installed on disk; double click and it is copied into ram to execute.

*Q: why does more ram feel faster? A: more programs fit loaded at once.*

---

## basics of os (i/o structure)

storage is just one kind of i/o device. a large part of os code manages i/o because of its effect on reliability and performance, and because devices vary so much.

**device controller keeps:** a **local buffer** and a set of **special purpose registers**.
**device driver:** the os has one per controller. it understands the controller and gives the rest of the os one uniform interface.

**interrupt driven i/o, step by step:**
1. driver loads the right registers in the controller.
2. controller reads those registers to decide what to do.
3. controller moves data from device to its local buffer.
4. controller interrupts the driver: "done".
5. driver hands control back to the os.

problem: this interrupts the cpu per byte, fine for small data, high overhead for bulk data.

**dma (direct memory access).** set up buffers, pointers and counters. then the controller moves an **entire block** straight between its buffer and memory with no cpu involvement. only **one interrupt per block** says it finished. the cpu is free meanwhile.

example: copying a 1 gb file. interrupt per byte means about a billion interrupts. dma means one per block.

*Q: what does dma remove from the cpu's load? A: per byte data copying and interrupts.*

---

## computer system architecture

classification is by the number of **general purpose** processors.

| type | what it is | notes |
|---|---|---|
| single processor | one main cpu | may still have special purpose processors (e.g. keyboard chip converting keystrokes to codes); still called single because only one general purpose cpu |
| multiprocessor (parallel, tightly coupled) | two or more cpus sharing bus, sometimes clock, memory, peripherals | advantages below |
| clustered | two or more complete systems coupled together | high availability |

**multiprocessor advantages:**
1. **increased throughput** (work done per unit time).
2. **economy of scale** (share resources instead of buying three whole machines).
3. **increased reliability** (one cpu fails, others carry on; a graceful slowdown, not total failure).

**symmetric vs asymmetric multiprocessing:**
- **symmetric (smp):** all cpus are peers and all take part in all tasks.
- **asymmetric:** master slave. the master monitors and assigns tasks; each slave handles its own processes.

**clustered systems.** several whole computers. one failing, others take over, so availability is high. same split:
- **asymmetric clustering:** one machine sits in **hot standby** monitoring; the others run applications. if one dies, the standby takes its place.
- **symmetric clustering:** all hosts run applications **and monitor each other**. better, since no machine idles.

*Q: why is symmetric clustering more efficient? A: no host sits idle just watching.*

---

## cheat sheet

| concept | explanation | real use |
|---|---|---|
| os | manager and intermediary between user and hardware | double click instead of writing hardware code |
| convenience vs efficiency | the two os goals | desktop os leans convenience, servers lean efficiency |
| bootstrap program | rom resident first program that loads the kernel | what runs when you press power |
| interrupt / system call | hardware signal / software signal that diverts the cpu | keyboard press, program asking os for a file |
| isr | code run in response to an interrupt | handler for a disk "done" signal |
| memory hierarchy | registers > cache > ram > disks > tape | cache hot data, store cold data on disk |
| volatile | loses content without power | ram contents gone after shutdown |
| device driver / controller | os side translator / hardware side manager | plugging in a new printer needs a driver |
| dma | block transfer with one interrupt | disk and network transfers |
| smp / asymmetric | peers / master slave | multicore pcs / older designs |
| cluster | whole systems coupled | web service staying up when one server dies |

## interleaving: compare the look alikes

| pair | difference |
|---|---|
| interrupt vs system call | both divert the cpu; source is hardware vs software |
| interrupt driven i/o vs dma | per byte cpu involvement vs one interrupt per block |
| multiprocessor vs cluster | shared bus and memory inside one box vs complete separate computers coupled |
| symmetric vs asymmetric (both in multiprocessing and clustering) | peers vs one supervisor |
| rom vs ram | permanent, unwritable boot code vs volatile working memory |

## feynman checks

1. **a server must keep running even if a whole machine dies, and you can't afford idle hardware. which design and why?** symmetric clustering: every host does real work and watches the others, so when one fails another takes over, and nothing sits idle like a hot standby would.
2. **you copy a huge file and the cpu usage stays low. what mechanism explains this?** dma. the controller moves blocks to memory itself and raises one interrupt per block, so the cpu is free for other processes.
3. **why does the bootstrap program live in rom and not on the disk with the os?** at power up ram is in an unknown state and the cpu must find something to run; rom holds fixed code immediately, needs no initialization, and can't be altered by a virus.

## review questions

this is the first chapter, so no earlier prerequisites. later chapters build on these:
- where will the interrupt mechanism reappear? (context switching, chapter 4)
- where will "cpu competes for resources" reappear? (scheduling, chapters 6 and 7)

## security angle

- **boot trust:** the bootstrap in rom is the root of trust. if a bootkit replaces it, everything above is compromised. defense: secure boot, signed firmware.
- **interrupt and system call boundary:** this is how untrusted programs ask the kernel to do things. every system call is an attack surface; validate arguments.
- **dma attacks:** a malicious device with dma can read or write memory with no cpu check. defense: iommu, disabling unused ports.

## omitted chuff
- the list of example operating systems and application programs, beyond one example each.
- the "box vs chip" explanation of what a cpu physically is (kept one line).
- repeated reminders that other os types come in later lectures.

*Q: what's the single biggest idea to keep from this chapter? A: the os exists to hide and share limited hardware.*
