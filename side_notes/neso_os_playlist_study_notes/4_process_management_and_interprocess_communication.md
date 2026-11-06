# chapter 4: process management and interprocess communication

> **disclaimer:** i don't claim authorship of any of this. these are my study notes from the neso academy operating systems playlist. they complement the lectures, they don't replace them. watch the videos.

**videos covered (playlist order):**

| # | video | link |
|---|---|---|
| 1 | Process Management (Processes and Threads) | https://www.youtube.com/watch?v=OrM7nZcxXZU |
| 2 | Process State | https://www.youtube.com/watch?v=jZ_6PXoaoxo |
| 3 | Process Control Block | https://www.youtube.com/watch?v=4s2MKuVYKV8 |
| 4 | Process Scheduling | https://www.youtube.com/watch?v=2h3eWaPx8SA |
| 5 | Context Switch | https://www.youtube.com/watch?v=vTgccrbYHYs |
| 6 | Operation on Processes: Process Creation | https://www.youtube.com/watch?v=pSW9d3Oaie8 |
| 7 | Operation on Processes: Process Termination | https://www.youtube.com/watch?v=SFc3jt8t5rU |
| 8 | Interprocess Communication | https://www.youtube.com/watch?v=dJuYKfR8vec |
| 9 | Shared Memory Systems | https://www.youtube.com/watch?v=uHtzOFwgD74 |
| 10 | Message Passing Systems (Part 1) | https://www.youtube.com/watch?v=LuuSXWkDJOo |
| 11 | Message Passing Systems (Part 2) | https://www.youtube.com/watch?v=S3mS8MR7bUY |
| 12 | Message Passing Systems (Part 3) | https://www.youtube.com/watch?v=fViAyFLOYxU |
| 13 | Sockets in Operating System | https://www.youtube.com/watch?v=uagKTbohimU |
| 14 | Remote Procedure Calls (RPC) | https://www.youtube.com/watch?v=QmhTjsOOrlw |
| 15 | Issues in RPC & How They're Resolved | https://www.youtube.com/watch?v=jH3RezOHROU |

> **source note on video 12 (Message Passing Systems, Part 3):** the transcript for this lecture was missing from the transcript file. the section below is written only from the short excerpt you supplied (blocking vs nonblocking, and the three queue capacities). nothing else from that lecture is included, so it may be thinner than the video. use the video for anything beyond this.

## table of contents
- [process management (processes and threads)](#process-management-processes-and-threads)
- [process state](#process-state)
- [process control block](#process-control-block)
- [process scheduling](#process-scheduling)
- [context switch](#context-switch)
- [process creation](#operation-on-processes--process-creation)
- [process termination](#operation-on-processes--process-termination)
- [interprocess communication](#interprocess-communication)
- [shared memory systems](#shared-memory-systems)
- [message passing systems part 1](#message-passing-systems-part-1)
- [message passing systems part 2](#message-passing-systems-part-2)
- [message passing systems part 3](#message-passing-systems-part-3)
- [sockets in operating system](#sockets-in-operating-system)
- [remote procedure calls](#remote-procedure-calls-rpc)
- [issues in rpc](#issues-in-rpc--how-theyre-resolved)
- [cheat sheet](#cheat-sheet)
- [interleaving](#interleaving)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## process management (processes and threads)

**from source to running.** you write a program in a high level language (c, c++, java), a compiler converts it to binary machine code, the os loads it into memory and allocates resources. until it runs it's a **passive entity**. the moment it executes it becomes a **process**: a program in execution.

old computers ran one process at a time; today many run at once, and one program can have several processes (a browser has many).

**thread.** the unit of execution inside a process. a process has one or more threads. in task manager you see processes; a tool like process explorer shows the threads of each (chrome.exe has many).

*Q: when does a program become a process? A: the moment it begins executing.*

---

## process state

the state is defined by the process's current activity.

| state | meaning |
|---|---|
| new | being created |
| ready | created, waiting to be assigned a processor |
| running | instructions are executing |
| waiting | waiting for an event (i/o completion, a signal) |
| terminated | finished execution |

**transitions** *(see diagram, search anchor: "process States")*:

```
new --admitted--> ready --scheduler dispatch--> running --exit--> terminated
running --interrupt--> ready
running --i/o or event wait--> waiting --i/o or event completion--> ready
```

example: you open a pdf (new, ready, running); it needs the disk (waiting); the read finishes (ready); the scheduler gives it the cpu again (running).

*Q: what state follows "waiting" when the i/o finishes? A: ready, not running.*

---

## process control block

each process is represented in the os by a **pcb** (also called a task control block). contents:

| field | what it records |
|---|---|
| process id (number) | unique identifier |
| process state | new, ready, running... |
| program counter | address of the next instruction to execute |
| cpu registers | index registers, stack pointers, general purpose registers |
| cpu scheduling information | priority, scheduling queue pointers, other parameters |
| memory management information | memory the process uses |
| accounting information | cpu time, resources used |
| i/o status information | i/o devices assigned, open files |

the pcb is the bookmark that lets the os stop a process and resume it later.

*Q: which pcb field says where to resume? A: the program counter.*

---

## process scheduling

**goals.** multiprogramming aims to keep some process running all the time to maximize cpu use; time sharing switches fast so users can interact. the **process scheduler** picks an available process for the cpu. on a single processor only one process runs at a time; the rest wait.

**queues:**
- **job queue:** all processes in the system.
- **ready queue:** processes in main memory, ready and waiting for the cpu.

**what can happen to a running process** *(see diagram, search anchor: "ready queue into the running state")*:
1. it finishes and terminates.
2. a higher priority process arrives, it's swapped out to the partially executed swapped out list, then back to the ready queue.
3. it needs i/o: it joins an **i/o waiting queue**, and when the i/o completes it returns to the ready queue.

the lecture is explicitly a brief preview; the algorithms come in chapters 6 and 7.

*Q: ready queue vs job queue? A: ready are in memory and waiting for the cpu; the job queue holds all processes.*

---

## context switch

an interrupt makes the os take the cpu from its current task and run a kernel routine. before doing so the system must **save the context** of the running process so it can **restore** it later. the context lives in the **pcb**.

bookmark analogy: your mother calls while you're reading. you put a bookmark in (save state), go help her (the interrupt handler), then return to the bookmark (restore state).

- **context switch:** state save of the current process plus state restore of another.
- **pure overhead:** the system does no useful work while switching. speed depends on memory speed, number of registers to copy, and special instructions; typically a few milliseconds.

*Q: why is a context switch called pure overhead? A: no user work happens during it.*

---

## operation on processes: process creation

a process may create new processes with a **create process** system call. the creator is the **parent**, the new ones are **children**, and children can create their own, forming a **tree**. each has a unique **pid**.

example (solaris tree): `sched` (pid 0) creates `init` (pid 1), `pageout` (pid 2), `fsflush` (pid 3). `init` is the parent of all user processes: it starts `inetd` (networking, telnet, ftp) and `dtlogin` (login screen), then shells, `ls`, `cat`, a browser, an editor.

**three kinds of choices at creation:**

| question | options |
|---|---|
| execution | parent runs **concurrently** with children, or parent **waits** for some or all children to terminate |
| resources | child gets **all** or **a subset** of the parent's resources |
| address space | child is a **duplicate** of the parent (same program and data), or child has a **new program loaded** into it |

*Q: what does the parent call "the process tree" mean? A: parents create children who create their own.*

---

## operation on processes: process termination

a process terminates when it finishes its final statement and asks the os to delete it with the **exit** system call. it may return an integer status to its parent through **wait**. the os **deallocates** its resources: memory (physical and virtual), open files, i/o buffers.

**one process killing another** via a system call: normally only the **parent** may do this, otherwise users could kill each other's jobs.

**why a parent kills a child:**
1. the child exceeded its allotted resources (the parent needs a way to inspect the child's state).
2. the task given to the child is no longer needed.
3. the parent is exiting and the os doesn't let children outlive their parent (**cascading termination** in some systems).

*Q: who is normally allowed to kill a process? A: its parent.*

---

## interprocess communication

processes are **independent** (can't affect or be affected by others) or **cooperating** (can). cooperating means sharing data, so cooperating processes need **ipc**.

**why cooperate:**

| reason | meaning |
|---|---|
| information sharing | several users want the same file or data |
| computational speedup | split a task into subtasks that run concurrently |
| modularity | build the system from separate modules |
| convenience | one user does music, typing and printing at once |

**two models:**
- **shared memory:** a region of memory is shared; processes read and write it.
- **message passing:** processes exchange messages through the kernel.

in shared memory process a writes to the region, process b reads it. in message passing, a sends a message to the kernel and the kernel delivers it to b *(see diagram, search anchor: "two figures a and B")*.

*Q: which model needs the kernel to carry data? A: message passing.*

---

## shared memory systems

- processes must **establish a region of shared memory**.
- the region lives in the **address space of the process that creates it**; other processes **attach** it to their own address space.
- normally the os prevents one process from touching another's memory, so processes must **agree to remove that restriction**. the os doesn't police what they put there; they do.

**producer consumer problem.** a producer produces items that a consumer consumes. examples: a compiler produces assembly consumed by an assembler, which produces object modules consumed by the loader; a web server produces pages consumed by a browser.

solution with shared memory: a **buffer** in the shared region, filled by the producer and emptied by the consumer. they must be synchronized so the consumer never consumes an item not yet produced.

| buffer type | rule |
|---|---|
| **unbounded** | no practical size limit; consumer waits if empty; producer never waits |
| **bounded** | fixed size; consumer waits if empty; producer waits if full |

*Q: when does the producer wait? A: bounded buffer full.*

---

## message passing systems part 1

message passing lets processes **communicate and synchronize without sharing an address space**. it's especially useful in a **distributed environment** where processes sit on different computers connected by a network (a chat between two machines can't share ram).

**two operations:** `send(message)` and `receive(message)`.

**message size:**

| | fixed size | variable size |
|---|---|---|
| system implementation | simple | complex |
| programming | harder | easier |

for p and q to talk they need a **communication link**. it's implemented **logically** (the physical side comes later) with choices: direct or indirect communication, synchronous or asynchronous, automatic or explicit buffering. related issues: naming, synchronization, buffering.

---

## message passing systems part 2

this lecture covers **naming**: direct vs indirect communication.

**direct communication.** each process names the other.
- symmetric: `send(P, message)` and `receive(Q, message)`.
- asymmetric: `send(P, message)` and `receive(id, message)`, where `id` is set to the sender's name.
- link properties: set up automatically between each pair; a link belongs to **exactly two processes**; exactly **one** link per pair.
- **disadvantage (limited modularity):** changing a process identifier means checking every definition that references it.

**indirect communication.** messages go to and from **mailboxes (ports)**, each with a unique id.
- `send(A, message)` and `receive(A, message)`.
- two processes communicate only if they **share a mailbox**.
- a link can involve **more than two** processes, and a pair can have **many links** (one per mailbox).
- question: p1 sends to a mailbox shared with p2 and p3, both receive. who gets it? three answers: allow a link with at most two processes; allow only one process at a time to receive; let the system pick (e.g. round robin).
- **mailbox owner:** a process (mailbox disappears when it terminates) or the os (independent existence; the os creates and deletes).

---

## message passing systems part 3

> **source: your supplied excerpt for lecture 27, not a transcript of the video.**

this part covers the remaining two choices from part 1: **synchronization** and **buffering**.

**synchronization (blocking vs nonblocking).** message passing may be **blocking** (synchronous) or **nonblocking** (asynchronous):

| operation | behavior |
|---|---|
| blocking send | the sender is blocked until the message is received by the receiving process or by the mailbox |
| nonblocking send | the sender sends the message and resumes operation |
| blocking receive | the receiver blocks until a message is available |
| nonblocking receive | the receiver retrieves either a valid message or a null |

**buffering (queue capacity).** messages waiting in the link live in a queue. its capacity is implemented in one of three ways:

| capacity | behavior |
|---|---|
| zero capacity | max length zero, no messages can wait in the link. the sender must block until the recipient receives the message |
| bounded capacity | finite length n, at most n messages. if the queue isn't full the message goes in and the sender continues. if full, the sender blocks until space is available |
| unbounded capacity | potentially infinite length, any number can wait, the sender never blocks |

link to shared memory: bounded and unbounded capacity mirror bounded and unbounded buffers from the producer consumer problem.

*Q: with zero capacity, when does the sender continue? A: after the recipient receives the message.*

---

## sockets in operating system

a **socket** is an endpoint for communication. a pair of processes communicating over a network uses a pair of sockets. a socket is identified by an **ip address concatenated with a port number**.

**how a connection forms:** the server listens on a specified port; when a client request arrives, the server accepts the connection from the client socket to complete it.

**well known ports (below 1024):** telnet 23, ftp 21, http 80. all ports below 1024 are reserved for standard services.

**example:** the client host 146.86.5.20 gets an arbitrary port above 1024 (1625), so its socket is `146.86.5.20:1625`. the web server 161.25.19.8 listens on port 80. packets are delivered to the right process by destination port number. a second process on the client gets a different port above 1024.

sockets are used specifically in **client server** systems.

*Q: why does the client get a port above 1024? A: below that is reserved for standard services.*

---

## remote procedure calls (rpc)

rpc is a protocol one program uses to request a service from a program on another computer on a network without understanding the network's details. earlier ipc handled processes on the same system; rpc handles processes on **different systems**, so it uses **message based communication** (shared memory isn't possible).

- rpc messages are **well structured**, not raw packets. each is addressed to an **rpc daemon** listening on a port of the remote system and contains the **function identifier** and **parameters**.
- the function runs and the output returns in a separate message.

**stubs.** the client side has a **stub** per remote procedure that hides the network details. flow:
1. client invokes the remote procedure; the rpc system calls the stub.
2. the stub **locates the port** on the server and **marshals** the parameters (packs them into a form that can travel over the network).
3. the stub sends a message to the server.
4. a server side stub receives it, unpacks, and invokes the procedure.
5. results go back to the client the same way.

---

## issues in rpc and how they're resolved

| issue | resolution |
|---|---|
| **data representation differs** between machines (big endian vs little endian 32 bit integers) | convert to a machine independent form, **xdr (external data representation)**, on the client during marshaling; unmarshal back to the local form on the server |
| **network errors:** calls can fail or run more than once, unlike local calls | the os ensures **exactly once** semantics, not at most once. the server **acknowledges** execution; the client keeps resending until it gets the acknowledgment |
| **binding:** how do client and server learn each other's port numbers, given they share no memory? | (a) fixed port numbers at compile time (inflexible), or (b) **dynamic binding with a rendezvous (matchmaker) daemon** on a fixed rpc port: the client asks it for the port of a named rpc, receives the port, then sends calls to it |

**the full execution (the lecture's diagram):**
1. user asks the kernel to send an rpc to procedure x.
2. the kernel messages the matchmaker port: "address for rpc x?"
3. the matchmaker replies "port p".
4. the kernel puts p in the user's rpc message and sends the rpc to port p.
5. the daemon on port p processes it and sends the output back.
6. the kernel passes the reply to the user.

*Q: why xdr? A: so machines with different data layouts can understand each other.*

---

## cheat sheet

| concept | explanation | real use |
|---|---|---|
| program vs process | passive file vs program in execution | each app you launch |
| process states | new, ready, running, waiting, terminated | explaining why a program "hangs" |
| pcb | record of a process's state | saved at every context switch |
| job vs ready queue | all processes vs those waiting for the cpu | scheduler input |
| context switch | save one pcb, load another; pure overhead | multitasking |
| fork style creation choices | concurrent or wait; shared or subset resources; duplicate or new program | shell launching commands |
| termination | exit plus wait, resources freed, parent may kill | cleaning up zombies |
| independent vs cooperating | no effect vs affects each other | decides if ipc is needed |
| shared memory | agreed memory region | fast ipc on one machine |
| message passing | send and receive via kernel | networks, safer sharing |
| direct vs indirect | name the process vs use a mailbox | chat vs queue style apps |
| blocking vs nonblocking | send or receive waits vs returns immediately | synchronous apis vs async apis |
| queue capacity | zero, bounded, unbounded | buffering strategy |
| socket | ip plus port endpoint | all tcp/udp networking |
| rpc | call a function on another machine | distributed services |
| xdr, ack, matchmaker | fix the three rpc problems | design of rpc systems |

## interleaving

| pair | difference |
|---|---|
| shared memory vs message passing | agreed region without kernel per message vs kernel mediated messages; message passing works across machines |
| direct vs indirect communication | name the peer vs use a mailbox; indirect allows many processes |
| blocking vs nonblocking | waits vs returns, a pair for send and a pair for receive |
| bounded vs unbounded (buffer or queue) | producer or sender may wait vs never waits |
| sockets vs rpc | raw endpoint for any byte stream vs a function call abstraction |
| process vs thread | program in execution vs a unit of execution inside it |

## feynman checks

1. **two processes on different machines must exchange data. why can't you use shared memory, and what would you use?** a shared region can't be mapped across machines; use message passing, via sockets or rpc.
2. **a client sends an rpc and the reply is lost. what could go wrong, and how does the design handle it?** the client doesn't know if it ran, so it resends; the exactly once design uses acknowledgments (and the server must not rerun a call it already completed) so the effect happens once.
3. **a producer faster than the consumer feeds a bounded buffer. what happens, and how would zero capacity differ?** the producer blocks when the buffer fills; with zero capacity it would block on every send until the receiver takes the message.

## review questions

- (ch1) which hardware event forces a context switch, and which structure holds the saved state?
- (ch2) is exit or wait a system call, and into which of the five categories does it fall? (process control)
- (ch3) why is a microkernel's reliance on message passing relevant to this chapter's ipc models?

## security angle

- **shared memory:** both sides must trust each other; unsynchronized or unvalidated shared data leads to races and corruption. validate on read.
- **sockets and ports:** every listening port is attack surface; close unused ones, firewall, authenticate.
- **rpc:** unmarshaling untrusted data is a classic bug source; the matchmaker is a target (spoof it and you redirect clients). defend with authentication and strict input validation.
- **process tree:** malware often hides by spawning from a trusted parent; watch parent child relationships (a word processor launching a shell).

## omitted chuff
- the detailed solaris pid example, kept as one paragraph.
- repeated "i hope this was clear" style recaps.
- the task manager and process explorer screenshots walkthrough.
- the full text of every analogy about lawyers and buses (kept the useful ones).

*Q: the single most useful picture from this chapter? A: pcb in, pcb out: the context switch.*
