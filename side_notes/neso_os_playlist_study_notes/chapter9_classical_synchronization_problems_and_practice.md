# chapter 9: classical synchronization problems and practice

> **disclaimer:** i don't claim authorship of any of this. these are my study notes from the neso academy operating systems playlist, meant to complement the lectures, not replace them. watch the videos.

**videos covered (playlist order):**

| # | video | link | transcript status |
|---|---|---|---|
| 1 | *(title not in the transcript file; the next lecture calls it "the bounded buffer problem or the producer consumer problem")* | https://www.youtube.com/watch?v=Qx3P2wazwI0 | **corrupted** |
| 2 | The Readers Writers Problem | https://www.youtube.com/watch?v=p2XDhW5INOo | available |
| 3 | The Dining Philosophers Problem | https://www.youtube.com/watch?v=FYUi-u7UWgw | available |
| 4 | Monitors | https://www.youtube.com/watch?v=ufdQ0GR855M | available |
| 5 | Dining Philosophers Solution using Monitors | https://www.youtube.com/watch?v=K52NiClfvyE | available |
| 6 | Process Synchronization: Problem 1 (GATE 2010) | https://www.youtube.com/watch?v=WIj06NCxkWE | available |
| 7 | Process Synchronization: Problem 2 (GATE 2010) | https://www.youtube.com/watch?v=_yl0qAnsa_8 | available |
| 8 | Process Synchronization: Problem 3 (GATE 2007; title not shown in the file, labeled from content) | https://www.youtube.com/watch?v=A41_0uRnb2A | available |
| 9 | Process Synchronization: Problem 4 (GATE 2019) | https://www.youtube.com/watch?v=fWL8HAIrgMw | available |
| 10 | Process Synchronization: Problem 5 (counting semaphore, GATE 1998; title not shown in the file, labeled from content) | https://www.youtube.com/watch?v=LlnBI2yjvlg | available |

> **corrupted source, stated plainly:** the transcript for video 1 (bounded buffer) is garbled (the text comes out as mixed turkish and english nonsense). i can only recognize keywords: it uses **three semaphores**, a **mutex**, and "empty" and "full" counts, plus `wait` and `signal`. below i give the standard textbook solution, **clearly marked as my reconstruction, not a summary of that lecture**. the readers writers lecture confirms the problem is "solved using semaphores".

## table of contents
- [bounded buffer problem (video corrupted)](#bounded-buffer-problem-video-corrupted)
- [the readers writers problem](#the-readers-writers-problem)
- [the dining philosophers problem](#the-dining-philosophers-problem)
- [monitors](#monitors)
- [dining philosophers solution using monitors](#dining-philosophers-solution-using-monitors)
- [solved problem 1 (gate 2010)](#problem-1-gate-2010)
- [solved problem 2 (gate 2010)](#problem-2-gate-2010)
- [solved problem 3 (gate 2007)](#problem-3-gate-2007)
- [solved problem 4 (gate 2019)](#problem-4-gate-2019)
- [solved problem 5 (gate 1998)](#problem-5-gate-1998)
- [cheat sheet](#cheat-sheet)
- [interleaving](#interleaving)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## bounded buffer problem (video corrupted)

**the problem (from chapter 4's shared memory lecture):** a producer fills a fixed size buffer and a consumer empties it. the producer must wait when it's full, the consumer when it's empty, and they must not touch the buffer at the same instant.

> **reconstruction, not from the lecture.** standard semaphore solution, consistent with the keywords visible in the garbled transcript (a mutex, "empty", "full"):

```c
semaphore mutex = 1;   // mutual exclusion on the buffer
semaphore empty = n;   // free slots
semaphore full  = 0;   // filled slots

producer:                   consumer:
do {                        do {
    produce item;               wait(full);     // wait for an item
    wait(empty);                wait(mutex);
    wait(mutex);                remove item;
    add item to buffer;         signal(mutex);
    signal(mutex);              signal(empty);  // free a slot
    signal(full);               consume item;
} while (true);             } while (true);
```

`empty` is a **counting semaphore** (n slots), `mutex` a **binary semaphore**. the order `wait(empty)` before `wait(mutex)` matters: reversing it can deadlock.

---

## the readers writers problem

a **database is shared among concurrent processes**. **readers** only read; **writers** read and write (update).

- two readers at once: **no problem**.
- a writer plus any other process (reader or writer): **chaos**.
- so writers need **exclusive access**.

**solution with semaphores** (readers get priority in this version):

```c
semaphore mutex = 1;   // protects readcount
semaphore wrt   = 1;   // common to readers and writers
int readcount   = 0;   // plain integer: how many readers are reading

writer:                         reader:
do {                            do {
    wait(wrt);                      wait(mutex);
    /* writing */                   readcount++;
    signal(wrt);                    if (readcount == 1) wait(wrt);
} while (true);                     signal(mutex);

                                    /* reading */

                                    wait(mutex);
                                    readcount--;
                                    if (readcount == 0) signal(wrt);
                                    signal(mutex);
                                } while (true);
```

**how it works:**
- a writer holds `wrt` for the whole write, so no reader or writer can enter.
- the **first reader** takes `wrt`, locking writers out; later readers just increment `readcount` and read. `mutex` lets only one process update `readcount` at a time.
- the **last reader** to leave (`readcount == 0`) releases `wrt`, so a writer can proceed.
- readers have priority: writers can wait a long time while readers keep arriving.

*Q: why does only the first reader wait on `wrt`? A: once it holds it, all later readers ride along; the last one releases it.*

---

## the dining philosophers problem

five philosophers sit at a round table, one **chopstick (fork)** between each pair (five forks). they alternate **thinking** and **eating**; to eat they need **both** adjacent forks.

rules: pick up **one fork at a time**; can't take a fork a neighbor holds; when you have both, eat without letting go; then put both down and think. no two adjacent philosophers may eat at once.

this represents **resource allocation**: philosophers are processes, forks are shared resources.

**semaphore solution:** each fork is a semaphore, `semaphore chopstick[5]`, all initialized to 1.

```c
// philosopher i
do {
    wait(chopstick[i]);
    wait(chopstick[(i + 1) % 5]);   // % 5 wraps philosopher 5 to fork 1
    /* eat */
    signal(chopstick[i]);
    signal(chopstick[(i + 1) % 5]);
    /* think */
} while (true);
```

**guarantee:** no two neighbors eat together.
**flaw: deadlock.** if all five get hungry at once and each grabs the **left** fork, every `chopstick[i]` becomes 0. each then waits for the right fork held by a neighbor, forever.

**remedies:**
1. allow **at most four** philosophers to sit at once, so someone can always finish.
2. pick up forks **only if both are available** (done in a critical section).
3. **asymmetric solution:** odd philosophers pick up left then right; even philosophers pick up right then left.

---

## monitors

semaphores have timing errors and deadlocks, and **every process must use wait and signal correctly**: one badly written process can break the whole system. a **monitor** is a **high level abstraction** that provides **mutual exclusion within the monitor itself**, so processes just call its procedures.

```
monitor monitor_name {
    // shared variable declarations
    procedure P1(...) { ... }
    procedure P2(...) { ... }
    ...
    procedure Pn(...) { ... }
    initialization code(...) { ... }
}
```

**rules:**
- a procedure inside the monitor can access only variables declared **locally in the monitor** and its parameters.
- the monitor's local variables can be accessed **only by its own procedures**.
- **only one process at a time can be active within the monitor**: this is what gives mutual exclusion.

**condition variables.** the monitor alone isn't enough, so we add `condition x, y;`. the only operations on them:
- `x.wait()`: the invoking process is **suspended** until another process calls `x.signal()`.
- `x.signal()`: **resumes exactly one** suspended process; if none are waiting, it has **no effect**.

**the schematic:** shared data, the operations, initialization code, an entry queue, and a queue for each condition variable *(see diagram, search anchor: "schematic view of a monitor")*.

*Q: what does a monitor do for you that a semaphore doesn't? A: it builds mutual exclusion in, so processes can't misuse it.*

---

## dining philosophers solution using monitors

restriction: a philosopher may pick up his chopsticks **only if both are available**. this avoids the all left forks deadlock.

**data structures** (three philosopher states: thinking, hungry, eating):

```c
enum {THINKING, HUNGRY, EATING} state[5];
condition self[5];     // philosopher i can delay himself when hungry but can't get both forks
```

philosopher i may set `state[i] = EATING` only if both neighbors are **not** eating: `state[(i+4)%5] != EATING` and `state[(i+1)%5] != EATING`.

```c
monitor DP {
    enum {THINKING, HUNGRY, EATING} state[5];
    condition self[5];

    void pickup(int i) {
        state[i] = HUNGRY;
        test(i);
        if (state[i] != EATING)
            self[i].wait();
    }

    void putdown(int i) {
        state[i] = THINKING;
        test((i + 4) % 5);   // maybe the left neighbor can now eat
        test((i + 1) % 5);   // maybe the right neighbor can now eat
    }

    void test(int i) {
        if (state[(i + 4) % 5] != EATING &&
            state[i] == HUNGRY &&
            state[(i + 1) % 5] != EATING) {
            state[i] = EATING;
            self[i].signal();
        }
    }

    initialization_code() {
        for (int i = 0; i < 5; i++)
            state[i] = THINKING;
    }
}
```

each philosopher calls `DP.pickup(i)`, eats, then `DP.putdown(i)`.

**why no deadlock:** the test checks both neighbors before granting eating, so nobody ever holds one fork and waits for another. only one process is active in the monitor at a time. after eating, `putdown` re checks both neighbors so waiting philosophers get their chance.

*Q: what does `pickup` do if the test fails? A: the philosopher waits on `self[i]` until a neighbor's putdown signals.*

---

## problem 1 (gate 2010)

p1 and p2 share booleans `S1` and `S2` (random initial values):

```
P1: while (S1 == S2);   CS;   S1 = S2;
P2: while (S1 != S2);   CS;   S2 = !S1;
```

which property holds: mutual exclusion but not progress, progress but not mutual exclusion, neither, or both?

- **mutual exclusion holds:** p1 enters only when `S1 != S2`; p2 only when `S1 == S2`. these are opposite conditions, so never both.
- **progress fails:** after p1 runs, it sets `S1 = S2` (equal). if p1 wants in again but p2 doesn't want in at all, p1's condition `S1 == S2` stays true, so it **spins** until p2 enters its critical section and flips `S2`. p1 is held up by a process outside its critical section.

**answer: option a (mutual exclusion but not progress).**

---

## problem 2 (gate 2010)

three processes, binary semaphores `S0 = 1`, `S1 = 0`, `S2 = 0`:

```
P0: while(true){ wait(S0); print '0'; release(S1); release(S2); }
P1: wait(S1); release(S0);
P2: wait(S2); release(S0);
```

how many times does p0 print 0?

- **case 1:** p0 prints (1st), releases S1 and S2. p1 runs and sets S0 = 1. p2 runs and releases S0 (still 1). p0 runs again, prints (2nd), releases S1 and S2. p1 and p2 are finished, nobody releases S0 again, so p0 blocks. **total: 2.**
- **case 2:** after p1 releases S0, p0 runs **before** p2. p0 prints (2nd), then p2 runs and releases S0, p0 prints (3rd). then p1 and p2 are done; p0 blocks. **total: 3.**

minimum 2, maximum 3. **answer: at least twice (option a).**

---

## problem 3 (gate 2007)

```
P1: while(true){ wants1 = true;  while(wants2 == true);  CS;  wants1 = false; }
P2: while(true){ wants2 = true;  while(wants1 == true);  CS;  wants2 = false; }
```

`wants1`, `wants2` start false. which statement is true? options: no mutual exclusion; no bounded waiting; requires strict alternation; doesn't prevent deadlock but ensures mutual exclusion.

- **mutual exclusion holds:** each enters only when the other's flag is false.
- **deadlock is possible:** p1 sets `wants1 = true` and is preempted before checking; p2 sets `wants2 = true` and spins because `wants1` is true; when p1 resumes it spins because `wants2` is true. neither flag is ever cleared: each waits for the other forever.

**answer: option d** (does not prevent deadlock, but ensures mutual exclusion). found by checking the easy properties first.

---

## problem 4 (gate 2019)

shared `D = 100` on a uniprocessor time shared system. each process does read, calculate, update:

```
P1: D = D + 20     P2: D = D - 50     P3: D = D + 10
```

with x the minimum and y the maximum possible final D, find y minus x.

- **minimum:** p2 **reads 100** and is preempted. p1 runs fully (D = 120), p3 runs fully (D = 130). p2 resumes with its stale 100: 100 minus 50 = 50, writes **D = 50**. so x = 50.
- **maximum:** p1 **reads 100** and is preempted. p2 runs fully (D = 50). p1 resumes with stale 100 plus 20 = 120, writes D = 120. p3 reads 120, writes **D = 130**. so y = 130.

$$y - x = 130 - 50 = 80$$

**answer: option a (80).** (if there were no race, any serial order gives 80.)

---

## problem 5 (gate 1998)

a **counting semaphore** is initialized to 10, then **6 P (wait)** and **4 V (signal)** operations are completed. final value?

- each wait decrements: 10 minus 6 = 4.
- each signal increments: 4 plus 4 = 8.

$$10 - 6 + 4 = 8$$

**answer: option b (8).**

---

## cheat sheet

| concept | key idea | real use |
|---|---|---|
| bounded buffer | mutex plus empty and full counters | queues, pipes, thread pools |
| readers writers | `wrt`, `mutex`, `readcount`; first reader locks writers out, last releases | databases, caches |
| dining philosophers | resource allocation with limited shared resources | deadlock teaching case |
| fork semaphores | `chopstick[i]` all 1 | naive solution, deadlocks |
| deadlock remedies | max 4 seated, pick up both together, odd/even order | general deadlock prevention |
| monitor | built in mutual exclusion, shared data plus procedures | java `synchronized`, language level locks |
| condition variables | `x.wait()` suspends, `x.signal()` resumes one | waiting for a state |
| monitor dining solution | states plus `test` before eating | deadlock free |

| problem | answer | method |
|---|---|---|
| gate 2010 (S1, S2) | mutual exclusion but not progress | check conditions, then trace one process alone |
| gate 2010 (S0, S1, S2) | at least twice | trace interleavings, min and max |
| gate 2007 (wants flags) | deadlock possible, mutual exclusion holds | preempt after setting flag |
| gate 2019 (D) | 80 | stale read gives min 50, max 130 |
| gate 1998 | 8 | 10 minus 6 plus 4 |

## interleaving

| pair | difference |
|---|---|
| semaphore vs monitor | caller must use wait and signal correctly vs mutual exclusion built in |
| semaphore signal vs monitor `x.signal()` | always increments vs only resumes a waiting process, no effect if none |
| dining philosophers semaphores vs monitors | can deadlock if all grab left vs checks both neighbors first, deadlock free |
| readers vs writers | many at once allowed vs exclusive access required |
| `mutex` vs `wrt` in readers writers | guards `readcount` vs guards the data |

## feynman checks

1. **a writer arrives while three readers are reading. what happens and for how long?** the writer blocks on `wrt` because the first reader holds it; it continues to wait until the last reader leaves and `readcount` reaches 0. with readers constantly arriving it could wait indefinitely (reader priority).
2. **seven philosophers instead of five. does the left fork deadlock still happen? which remedy scales best?** yes, all seven grabbing left still deadlocks. allowing at most six seated (n minus 1) or the monitor approach (check both neighbors) scales.
3. **a teammate says "just use semaphores everywhere, monitors are redundant". what's the weak point of that?** every process must follow the wait and signal protocol exactly; one mistake causes deadlock or lost mutual exclusion, whereas a monitor enforces it in one place.

## review questions

- (ch8) which of the three critical section requirements does the gate 2010 first problem fail, and why?
- (ch8) why does problem 3's deadlock look like the p0 and p1 semaphore example in the disadvantages lecture?
- (ch4) how does the bounded buffer here relate to the shared memory producer consumer lecture?
- (ch7) how is the readers priority solution's starvation similar to priority scheduling's starvation, and what is the shared fix idea (aging)?

## security angle

- **readers writers and race conditions:** a time of check time of use bug appears when permissions are checked but data changes before use; hold the lock across both.
- **deadlock as denial of service:** philosophers waiting forever is what an attacker triggers by holding one lock and stalling. fix: timeouts, global lock ordering.
- **monitors reduce misuse:** by centralizing locking, they cut the chance of a buggy or malicious process violating the protocol.
- **stale reads (problem 4)** are the root of lost update bugs, including balance and counter manipulation; use atomic operations.

## omitted chuff
- the bounded buffer lecture has **no notes from the lecture**, only a flagged reconstruction (corrupted transcript).
- the long intro analogies for the dining table and bus door.
- repeated reminders about what wait and signal mean.
- the mid lecture detours on how to read `while(...);` with a semicolon (covered in chapter 8).
- the full line by line narration of each gate trace; kept the key interleavings.

*Q: one line summary? A: classical problems all ask the same thing: share fairly without deadlock or corruption.*
