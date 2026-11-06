# chapter 8: synchronization fundamentals and mechanisms

> **disclaimer:** i don't claim authorship of any of this. these are my study notes from the neso academy operating systems playlist, meant to complement the lectures, not replace them. watch the videos.

**videos covered (playlist order):**

| # | video | link | transcript status |
|---|---|---|---|
| 1 | Process Synchronization | https://www.youtube.com/watch?v=ph2awKa8r5Y | available |
| 2 | The Critical-Section Problem | https://www.youtube.com/watch?v=UtEORPakw5Y | **no transcript; partial notes from 2 screenshots** |
| 3 | Peterson's Solution | https://www.youtube.com/watch?v=gYCiTtgGR5Q | available |
| 4 | Test and Set Lock | https://www.youtube.com/watch?v=5oZYS5dTrmk | **no transcript; partial notes from 3 screenshots** |
| 5 | Semaphores | https://www.youtube.com/watch?v=XDIOC2EY5JE | available |
| 6 | Disadvantages of Semaphores | https://www.youtube.com/watch?v=2cGo2HdA0dM | available |

> **partial source material, stated plainly:** the transcript file has no transcript for videos 2 and 4 above. those two sections are built only from screenshots of the slides (The Critical-Section Problem at 4:40 and 13:10 of 13:49; Test and Set Lock at 6:54, 16:37 and 18:47 of 19:00). whatever the speaker said between those timestamps is **not** covered. slide content is marked as such, and my own explanations are marked as "why" notes so you can tell them apart. watch the videos for the rest.

## table of contents
- [process synchronization](#process-synchronization)
- [the critical section problem](#the-critical-section-problem)
- [peterson's solution](#petersons-solution)
- [test and set lock](#test-and-set-lock)
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

## the critical section problem

> **source: 2 screenshots (4:40 and 13:10 of 13:49), no transcript.** the slide content is marked "(slide)". the "why" notes are my reasoning, not the speaker's words.

**the setup (slide).** a system of n processes {P0, P1, ..., Pn}. each process has a segment of code called a **critical section**, in which it may be changing common variables, updating a table, writing a file, and so on.

**the rule (slide).** when one process is executing in its critical section, no other process is allowed to execute in its critical section. that is, no two processes are executing in their critical sections at the same time.

**the problem (slide).** design a **protocol** that the processes can use to cooperate.

**the shape every process takes** (from the code in the Peterson's Solution and Test and Set Lock lectures, not from these two slides):

```
do {
    entry section        // ask permission
    critical section     // touch the shared data
    exit section         // give permission back
    remainder section    // everything else the process does
} while (true);
```

the protocol is the entry and exit sections. the critical section is just your normal code.

**why it exists (my note).** back in Process Synchronization, `counter++` was three machine steps: load, add, store. two processes interleaving those steps lost an update, so the counter ended at 4 or 6 instead of 5. if `counter++` sits inside a critical section, the three steps behave like one indivisible step, and the race disappears.

**the three requirements (slide).** a solution must satisfy all three:

| requirement | slide wording, shortened | plain meaning (my note) | what breaks without it |
|---|---|---|---|
| **mutual exclusion** | if Pi is executing in its critical section, no other process can be executing in theirs | one inside at a time | lost updates, like the counter example |
| **progress** | if nobody is in a critical section and some processes want in, only those **not in their remainder sections** take part in deciding who goes next, and the decision can't be postponed indefinitely | a process busy with unrelated work must not block the ones who want in | the strict alternation trick: P1 wants in again but P2 is off doing other work and never takes its turn, so P1 waits forever (the first GATE 2010 problem in chapter 9 fails exactly here) |
| **bounded waiting** | there is a bound on how many times other processes may enter their critical sections after a process has requested entry and before that request is granted | your wait is finite | starvation: others keep winning and you never get in |

*Q: which requirement does "nobody ever enters" satisfy trivially? A: mutual exclusion, which is why progress exists.*

**the mental model (why there are three, not one).** mutual exclusion is a **safety** rule: nothing bad happens. progress and bounded waiting are **liveness** rules: something good eventually happens. a lock that never lets anyone in is perfectly safe and perfectly useless, so you need both kinds.

*Q: a solution lets P1 starve while P2 enters over and over. which requirement fails? A: bounded waiting.*

**two checks:**
1. **a protocol makes P2 wait for P1 even when P1 is in its remainder section and never wants in again. what fails, and why is that a problem in practice?** progress fails: a process outside its critical section is deciding who enters. in practice P2 hangs forever behind a process that has gone quiet.
2. **a design guarantees every waiting process enters within at most 3 turns of others. which requirement is that, and which one does it not give you?** bounded waiting. it still needs mutual exclusion on top, because a bound on waiting says nothing about two processes being inside together.

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

## test and set lock

> **source: 3 screenshots (6:54, 16:37 and 18:47 of 19:00), no transcript.** slide content is marked "(slide)". the "why" notes are mine. the lecture after this one (Semaphores) recaps hardware based solutions as hard for application programmers to implement, and that is what motivates semaphores.

**what it is (slide).** a hardware solution to the synchronization problem.
- there is a **shared lock variable** that can take one of two values, 0 or 1.
- before entering the critical section, a process **inquires about the lock**.
- if it is locked, it keeps on waiting until it becomes free.
- if it is not locked, it takes the lock and executes the critical section.

**the instruction (slide), labeled "atomic operation":**

```c
boolean TestAndSet(boolean *target) {
    boolean rv = *target;   // remember the old value
    *target = TRUE;         // lock it, no matter what it was
    return rv;              // answer: what was it before?
}
```

**how each process uses it (slide).** P1 and P2 run the same code:

```c
do {
    while (TestAndSet(&lock));   // do nothing: spin while the old value was TRUE
    // critical section
    lock = FALSE;                // release the lock
    // remainder section
} while (TRUE);
```

**breaking the name down (my note).** "test" means read the old value, "set" means write TRUE, and the function returns the old value. so the return value answers one question: "was the lock already taken when i grabbed it?" FALSE means no, you own it now. TRUE means someone else has it, and your write of TRUE changed nothing.

**why it has to be atomic (my note).** suppose read and write were two separate steps. P1 reads FALSE, P2 reads FALSE before P1 writes, both write TRUE, both see "free", and both enter. that is the same lost update race as `counter++`. the hardware executes the read and the write as one uninterruptible step, so exactly one caller can ever see FALSE.

**trace with two processes** (the slides don't show the initial value of `lock`; FALSE, meaning free, is the usual starting point):

| step | who | lock before | TestAndSet returns | lock after | result |
|---|---|---|---|---|---|
| 1 | P1 | FALSE | FALSE | TRUE | loop exits, P1 enters |
| 2 | P2 | TRUE | TRUE | TRUE | loop continues, P2 spins |
| 3 | P2 | TRUE | TRUE | TRUE | spins again |
| 4 | P1 | TRUE | none | FALSE | P1 runs `lock = FALSE` on exit |
| 5 | P2 | FALSE | FALSE | TRUE | P2 enters |

*Q: why is it harmless that TestAndSet writes TRUE even when the lock was already TRUE? A: it just rewrites the same value.*

**the verdict (slide).** this solution **satisfies mutual exclusion** and **does not satisfy bounded waiting**.

**why bounded waiting fails (my note).** there is no queue. after P1 sets `lock = FALSE`, whoever's next `TestAndSet` call lands first wins. P1 loops straight back to the top of its `do` loop and calls it again immediately. if that call lands before P2's next spin attempt, P1 retakes the lock, and nothing limits how many times this repeats. P2 can starve even though the lock was free many times.

*Q: which requirement does the slide say fails? A: bounded waiting.*

**the cost (my note).** the `while (TestAndSet(&lock));` loop is busy waiting, the same spin lock cost described in the Disadvantages of Semaphores lecture below: a waiting process burns cpu doing nothing useful.

**two checks:**
1. **three processes call TestAndSet at nearly the same moment on a free lock. how many see FALSE, and why?** exactly one, because the instruction is atomic: the first one to execute it flips the lock to TRUE, and the other two get TRUE back.
2. **someone replaces TestAndSet with `if (lock == FALSE) lock = TRUE;` written as two normal instructions. what goes wrong?** another process can slip in between the check and the set, so two processes can both see FALSE and both enter. that gap is the race the atomic instruction closes.

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
| safety vs liveness | mutual exclusion is safety; progress and bounded waiting are liveness | why one rule is never enough |
| test and set lock | atomic instruction: returns the old lock value and sets it to TRUE | hardware spin lock; mutual exclusion yes, bounded waiting no |
| atomic read modify write | read and write executed as one uninterruptible step | closes check then act races |
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
- (ch1) why can't an interrupt land in the middle of a TestAndSet? (it is a single indivisible instruction)
- (ch7) how is test and set's missing bounded waiting like starvation in priority scheduling, and what fixed it there? (aging)

## security angle

- **race conditions as vulnerabilities:** time of check to time of use bugs let attackers swap a file or value between the check and the use. defend with atomic operations and locks, not check then act.
- **deadlock as denial of service:** an attacker who can acquire one lock and stall can freeze others. use lock timeouts and a fixed lock order.
- **spin locks and cpu exhaustion:** a malicious process can hold a lock and force everyone else to burn cpu.
- **peterson on modern cpus:** memory reordering breaks the algorithm without barriers; synchronization primitives must be hardware aware.
- **test and set closes the check then act gap:** reading a flag and then setting it in two steps is the same hole attackers use in time of check time of use bugs. an atomic read modify write removes the gap. defense: use atomic primitives, never hand rolled check then set.
- **no bounded waiting is a starvation lever:** an attacker looping on the lock can keep winning it and starve legitimate processes. defense: fair locks with a queue, and per user limits.

## omitted chuff
- the longer apples and bus stop analogies were shortened.
- repeated reminders of earlier lectures (shared memory, producer consumer).
- the lecture's detail on why dijkstra used dutch words beyond the two names.
- the critical section problem and test and set lock lectures are covered **only from screenshots**; whatever was said between the captured slides is missing, see the note at the top.

*Q: one line summary? A: shared data plus concurrency needs a lock, and a lock needs care.*
