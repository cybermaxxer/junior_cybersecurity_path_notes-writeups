# chapter 8: synchronization fundamentals and mechanisms

> **disclaimer:** i don't claim authorship of any of this. these are my study notes from the neso academy operating systems playlist, meant to complement the lectures, not replace them. watch the videos.

**videos covered (playlist order):**

| # | video | link | transcript status |
|---|---|---|---|
| 1 | Process Synchronization | https://www.youtube.com/watch?v=ph2awKa8r5Y | available |
| 2 | *(title not in the transcript file; per the lecture order and the next lecture's opening, this is the critical section problem)* | https://www.youtube.com/watch?v=UtEORPakw5Y | **missing** |
| 3 | Peterson's Solution | https://www.youtube.com/watch?v=gYCiTtgGR5Q | available |
| 4 | *(title not in the transcript file; the semaphore lecture opens by recapping "various hardware based solutions", so this one is inferred to be hardware synchronization)* | https://www.youtube.com/watch?v=5oZYS5dTrmk | **missing** |
| 5 | Semaphores | https://www.youtube.com/watch?v=XDIOC2EY5JE | available |
| 6 | Disadvantages of Semaphores | https://www.youtube.com/watch?v=2cGo2HdA0dM | available |

> **missing source material, stated plainly:** no transcripts exist in my source file for videos 2 and 4 above (the transcript file marks them "no transcript"). i did **not** write notes for what those lectures taught. where i mention the critical section problem's requirements below, they come from how the *other* lectures in this chapter and chapter 9 define them. for the critical section problem lecture itself and the hardware solutions lecture, watch the videos.

## table of contents
- [process synchronization](#process-synchronization)
- [critical section problem (video missing)](#critical-section-problem-video-missing)
- [peterson's solution](#petersons-solution)
- [hardware solutions (video missing)](#hardware-solutions-video-missing)
- [semaphores](#semaphores)
- [disadvantages of semaphores](#disadvantages-of-semaphores)
- [cheat sheet](#cheat-sheet)
- [interleaving](#interleaving)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## process synchronization

**cooperating processes** can affect or be affected by other processes. they either share a logical address space (code and data) or share data only through files or messages (chapter 4).

**the problem:** concurrent access to shared data may result in **data inconsistency**. the chapter's goal is mechanisms for the **orderly execution** of cooperating processes so data stays consistent.

**the example: the producer consumer counter.** reuse the bounded buffer from chapter 4. add an integer `counter` that counts items in the buffer. producer does `counter++` after adding, consumer does `counter--` after removing. start with `counter = 5`.

at machine level each statement is three steps:

```
counter++ :   register1 = counter
              register1 = register1 + 1
              counter   = register1

counter-- :   register2 = counter
              register2 = register2 - 1
              counter   = register2
```

interleave them (producer and consumer both run at the same time):

| time | who | action | register / counter |
|---|---|---|---|
| t0 | producer | register1 = counter | register1 = 5 |
| t1 | producer | register1 = register1 + 1 | register1 = 6 |
| t2 | consumer | register2 = counter | register2 = 5 |
| t3 | consumer | register2 = register2 minus 1 | register2 = 4 |
| t4 | producer | counter = register1 | counter = **6** |
| t5 | consumer | counter = register2 | counter = **4** |

the correct answer is **5** (one added, one removed). depending on who writes last you get 4, 5 or 6. if the consumer writes before the producer, the final value is 6 instead.

apple analogy: you hold 5 apples, get 1 (6), eat 1 (5). but if two people both start from "5" and overwrite each other, you end with 4 or 6.

**race condition:** several processes access and manipulate the same data concurrently and the outcome depends on the **particular order** of access. we want the changes not to interfere, so we need synchronization.

*Q: what makes this a race condition? A: the final value depends on who writes last.*

---

## critical section problem (video missing)

> **no transcript was available for this lecture.** below is only what the other lectures in this chapter say, not a summary of the missing video.

from the code in the next lecture, each process has this structure: **entry code**, then the **critical section** (where it touches shared data), then an **exit** step, then the **remainder section**.

from the solved problems and Peterson's lecture, a solution must satisfy three requirements:

| requirement | meaning (as defined in the later lectures) |
|---|---|
| **mutual exclusion** | if process p is executing in its critical section, no other process can be executing in theirs |
| **progress** | if no process is in its critical section and some wish to enter, only the processes **not executing in their remainder sections** take part in deciding who enters next, and the decision can't be postponed indefinitely |
| **bounded waiting** | there is a bound on how many times others may enter their critical sections after a process has requested entry and before that request is granted |

---

## peterson's solution

a classic **software** solution to the critical section problem.

- works for **two processes** only (call them pi and pj), alternating between critical and remainder sections.
- may not work correctly on modern computer architectures (the lecture's caveat), but it is a clear algorithmic description of how to meet mutual exclusion, progress and bounded waiting.
- two shared variables:
  - `int turn`: whose turn it is to enter.
  - `boolean flag[2]`: `flag[i] = true` means pi is **ready** to enter.

**code for process pi** (pj is the same with i and j swapped):

```c
do {
    flag[i] = true;                 // i wants in
    turn = j;                       // but politely gives j the turn
    while (flag[j] && turn == j);   // wait while j wants in AND it is j's turn

    /* critical section */

    flag[i] = false;                // done

    /* remainder section */
} while (true);
```

the semicolon after the `while` means an empty body: the process **spins** there until the condition becomes false. the condition is true only when **both** `flag[j]` and `turn == j` hold.

**the "humble" idea:** each process wants in but sets `turn` to the *other* process. it's like two friends at a bus door each saying "you first". the last write to `turn` decides the winner.

**trace when both want in at once:**
1. pi sets `flag[i] = true`, pj sets `flag[j] = true`.
2. pi sets `turn = j`; then pj sets `turn = i`. the final value is **i**.
3. pi checks: `flag[j]` is true but `turn == j` is **false**, so pi leaves the loop and enters.
4. pj checks: `flag[i]` is true and `turn == i` is true, so pj spins.
5. pi finishes, sets `flag[i] = false`; now pj's condition is false and pj enters.

**why the three requirements hold:**
- **mutual exclusion:** the `while` loop lets only one through at a time.
- **progress:** only the two processes that want in decide, via `turn`; it's decided at once.
- **bounded waiting:** pj waits only until pi finishes one critical section, then gets in.

*Q: when both set their flags, who waits? A: the process that wrote `turn` last (it handed the turn to the other one).*

---

## hardware solutions (video missing)

> **no transcript was available for this lecture.** the title above is inferred from the semaphore lecture's opening, which recaps "various hardware based solutions" that application programmers find hard to use. i'm not writing the content of that lecture here.

what the semaphore lecture does say: hardware based solutions are **difficult for application programmers to implement**, which motivates the software based semaphore.

---

## semaphores

proposed by **edsger dijkstra**. a **semaphore** `S` is a **non negative integer variable shared between processes**, used to solve the critical section problem and achieve synchronization. apart from initialization it's accessed only through two **atomic** operations: **wait** and **signal**.

**names:** wait is `P` (dutch *proberen*, "to test"), signal is `V` (dutch *verhogen*, "to increment").

```c
wait(S) {            // P
    while (S <= 0);  // spin while the resource is taken
    S--;             // claim it
}

signal(S) {          // V
    S++;             // release it
}
```

**atomic (indivisible):** while one process modifies the semaphore, no other may modify it at the same time.

**binary semaphore (mutex lock).** value only **0 or 1**; initialized to **1**.
- 1 means free; 0 means held.
- scenario: p1 calls wait, `S` is 1, so it passes and sets `S = 0`, and enters. p2 calls wait, sees `S = 0`, and **spins** in the loop. when p1 calls signal, `S = 1`; p2 breaks out, decrements to 0, and enters.

**counting semaphore.** unrestricted range; controls access to a resource with **multiple instances**. initialize `S` to the number of instances.
- scenario: 2 instances, `S = 2`. p1 enters (`S = 1`), p2 enters (`S = 0`), p3 spins. when p1 signals (`S = 1`), p3 gets in.

*Q: why must wait and signal be atomic? A: two processes changing S at once is the same race as the counter example.*

---

## disadvantages of semaphores

**1. busy waiting.** while one process is in its critical section, any other that tries to enter must **loop continuously** in the entry code. that wastes cpu cycles another process could use. this kind of semaphore is called a **spin lock** (the process spins while waiting).

**fix: block instead of spin.** modify wait and signal so a process that can't proceed **blocks itself**:
- the block operation puts the process on a **waiting queue** associated with the semaphore and switches its state to **waiting**.
- control goes to the **cpu scheduler**, which picks another process, so no cpu is wasted spinning.
- (signal then wakes a blocked process; the lecture describes this in words rather than code.)

**2. deadlock and starvation, even with the waiting queue.** a semaphore with a waiting queue can leave two or more processes waiting **indefinitely** for a signal that only one of the waiting processes can send.

**deadlock example:** two semaphores `S` and `Q`, both initialized to 1.

| step | p0 | p1 |
|---|---|---|
| 1 | wait(S) | wait(Q) |
| 2 | wait(Q): **blocked** (p1 holds Q) | wait(S): **blocked** (p0 holds S) |
| 3 | signal(S) never reached | signal(Q) never reached |

p0 waits for p1 and p1 waits for p0: **deadlock**.

*Q: what's the cause in the example? A: opposite lock order, each holds what the other needs.*

---

## cheat sheet

| concept | explanation | real use |
|---|---|---|
| cooperating processes | share data, can affect each other | any shared resource |
| race condition | outcome depends on access order | counter++ and counter-- |
| critical section | code touching shared data | needs mutual exclusion |
| three requirements | mutual exclusion, progress, bounded waiting | judging any solution |
| peterson | `flag[]` plus `turn`, two processes, software | teaching algorithm |
| wait (P) / signal (V) | test and decrement / increment | all semaphore code |
| binary semaphore / mutex | 0 or 1 | locks |
| counting semaphore | n instances | connection or buffer slot pools |
| spin lock | busy waiting | short critical sections only |
| blocking semaphore | waiting queue instead of spinning | long waits |
| deadlock | circular wait on semaphores | inconsistent lock order |

## interleaving

| pair | difference |
|---|---|
| binary vs counting semaphore | 0 or 1 vs any non negative count |
| spin lock vs blocking semaphore | wastes cpu while waiting vs sleeps on a queue |
| peterson vs semaphore | two processes, software, `flag` and `turn` vs general, shared integer with wait and signal |
| race condition vs deadlock | wrong result from unsynchronized access vs stuck forever from too much locking |
| wait vs signal | test and take vs release |

## feynman checks

1. **two threads each run `balance = balance + 100` on a shared balance of 0. what final values are possible and why?** 100 or 200: both read 0, add, and the later write overwrites the earlier, so one update is lost. serialize it with a mutex.
2. **in peterson's solution both processes set `turn` to the other. why doesn't that deadlock?** the writes happen one after another; the last write decides `turn`, so exactly one process finds `turn == j` false and enters.
3. **your spin lock burns 100% cpu on a busy server. what do you change and what new risk appears?** use a blocking semaphore (waiting queue); the new risk is deadlock or starvation if lock ordering is inconsistent.

## review questions

- (ch4) shared memory systems need the producer and consumer synchronized: which mechanism from this chapter fills that gap?
- (ch5) threads share data by default: why are race conditions especially common there?
- (ch7) why does a preempted process make race conditions worse in a time sharing system? (it can be stopped mid update)

## security angle

- **race conditions as vulnerabilities:** time of check to time of use bugs let attackers swap a file or value between the check and the use. defend with atomic operations and locks, not check then act.
- **deadlock as denial of service:** an attacker who can acquire one lock and stall can freeze others. use lock timeouts and a fixed lock order.
- **spin locks and cpu exhaustion:** a malicious process can hold a lock and force everyone else to burn cpu.
- **peterson on modern cpus:** memory reordering breaks the algorithm without barriers; synchronization primitives must be hardware aware.

## omitted chuff
- the longer apples and bus stop analogies were shortened.
- repeated reminders of earlier lectures (shared memory, producer consumer).
- the lecture's detail on why dijkstra used dutch words beyond the two names.
- the two missing lectures (critical section problem, hardware solutions) have **no notes** here; see the table at the top.

*Q: one line summary? A: shared data plus concurrency needs a lock, and a lock needs care.*
