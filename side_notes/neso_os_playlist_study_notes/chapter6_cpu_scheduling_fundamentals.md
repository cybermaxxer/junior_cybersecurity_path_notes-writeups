# chapter 6: cpu scheduling fundamentals

> **disclaimer:** i don't claim authorship of any of this. these are my study notes from the neso academy operating systems playlist, complementing the lectures rather than replacing them. watch the videos.

**videos covered (playlist order):**

| # | video | link |
|---|---|---|
| 1 | Introduction to CPU Scheduling | https://www.youtube.com/watch?v=EWkQl0n0w5M |
| 2 | CPU and I/O Burst Cycles | https://www.youtube.com/watch?v=pVzb3TUcDLo |
| 3 | Preemptive and Non-Preemptive Scheduling | https://www.youtube.com/watch?v=4DhFmL-6SDA |
| 4 | Scheduling Criteria | https://www.youtube.com/watch?v=bWHFY8-rL5I |

## table of contents
- [introduction to cpu scheduling](#introduction-to-cpu-scheduling)
- [cpu and i/o burst cycles](#cpu-and-io-burst-cycles)
- [preemptive and non preemptive scheduling](#preemptive-and-non-preemptive-scheduling)
- [scheduling criteria](#scheduling-criteria)
- [cheat sheet](#cheat-sheet)
- [interleaving](#interleaving)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## introduction to cpu scheduling

cpu scheduling is the basis of multiprogrammed operating systems. by switching the cpu among processes, the os makes the computer more productive.

**the problem.** on a single processor only one process runs at a time. a process runs until it must wait, typically for an i/o request. in a simple system the cpu then **sits idle** and all that waiting time is wasted.

**the fix.** keep several processes in memory. when one must wait, the os takes the cpu away and gives it to another. this pattern repeats, so the cpu is never idle if any process is ready. *scheduling* itself means deciding which process gets the cpu and when, using one of several algorithms (chapter 7).

example: a process starts a disk read. instead of idling for ten milliseconds, the cpu runs another ready process.

*Q: what are we trying to eliminate? A: cpu idle time.*

---

## cpu and i/o burst cycles

process execution is a cycle of **cpu execution** and **i/o wait**.

| term | meaning |
|---|---|
| cpu burst | a period when the process is using the cpu |
| i/o burst | a period when the process waits for i/o |

execution **begins with a cpu burst**, then i/o burst, then cpu burst, and so on, and **ends with a final cpu burst** that issues the request to terminate *(see diagram, search anchor: "alternating sequence of CPU and I/O bursts")*.

example sequence: `load, store, add, store, read from file` (cpu burst), wait for i/o (i/o burst), `store, increment index, write to file` (cpu burst), wait for i/o, and so on.

*Q: how does every process's life start and end? A: starts and ends with a cpu burst.*

---

## preemptive and non preemptive scheduling

**cpu scheduler (short term scheduler).** when the cpu becomes idle the os must pick a process from the ready queue; the scheduler selects one and allocates the cpu to it.

**dispatcher.** the module that **gives control of the cpu** to the process the scheduler selected. **dispatch latency** is the time the dispatcher takes to stop one process and start another; it should be as small as possible.

**when scheduling decisions happen (4 circumstances):**

| # | transition | choice? |
|---|---|---|
| 1 | running to waiting (e.g. i/o request) | **no choice**: a new process must be picked |
| 2 | running to ready (e.g. interrupt) | **choice** |
| 3 | waiting to ready (e.g. i/o completes) | **choice** |
| 4 | process terminates | **no choice**: a new process must be picked |

- **non preemptive (cooperative):** scheduling happens **only** under circumstances 1 and 4. a process keeps the cpu until it terminates or waits.
- **preemptive:** otherwise. the cpu can be taken from a process **before** it finishes or waits.

| | non preemptive | preemptive |
|---|---|---|
| cpu taken away mid run? | never | yes |
| strength | simple, no surprise interruptions | responds to higher priority work |
| risk | long jobs delay everyone | inconsistent shared data if preempted mid write |

shared memory example: a writer is preempted halfway through an update, a reader then reads inconsistent data. we need both styles depending on the system.

*Q: which two circumstances give no choice? A: running to waiting, and termination.*

---

## scheduling criteria

five criteria for comparing algorithms:

| criterion | meaning | want |
|---|---|---|
| cpu utilization | how busy the cpu is (real systems: roughly 40% lightly loaded to 90% heavily loaded) | high |
| throughput | processes completed per unit time | high |
| turnaround time | submission to completion: waiting to get into memory, waiting in the ready queue, executing on the cpu, doing i/o | low |
| waiting time | sum of time spent **waiting in the ready queue** (the scheduler only affects this, not run or i/o time) | low |
| response time | from submission of a request until the **first response** is produced (not until output finishes) | low |

**why response time:** in an interactive system a process can start producing output early while computing further. clicking a video file, the process "starts" at once, even though the screen takes a moment to show it. response time measures when it starts to respond, not when output completes, which depends on the output device's speed.

**the formulas used in chapter 7** (derive, don't just memorize):

$$\text{turnaround} = \text{completion} - \text{arrival}$$
$$\text{waiting} = \text{turnaround} - \text{burst}$$

turnaround is the whole stay; subtracting the burst (time actually on the cpu) leaves the waiting.

*Q: which criterion does the scheduler affect directly? A: waiting time (in the ready queue).*

---

## cheat sheet

| concept | explanation | real use |
|---|---|---|
| cpu scheduling | pick who gets the cpu | foundation of multiprogramming |
| cpu burst / i/o burst | cpu use / waiting for i/o | classify jobs as cpu bound or i/o bound |
| scheduler vs dispatcher | chooses vs hands over | dispatch latency matters |
| 4 circumstances | run to wait, run to ready, wait to ready, terminate | decides preemptive vs not |
| non preemptive | only on wait or terminate | simple batch style |
| preemptive | cpu can be taken | interactive systems |
| 5 criteria | utilization, throughput, turnaround, waiting, response | scorecard for algorithms |
| formulas | turnaround = completion minus arrival; waiting = turnaround minus burst | every solved problem in chapter 7 |

## interleaving

| pair | difference |
|---|---|
| scheduler vs dispatcher | decision vs action |
| turnaround vs waiting | whole stay vs ready queue time only |
| waiting vs response | total ready queue time vs time until the first response |
| throughput vs utilization | jobs done per time vs how busy the cpu is |
| preemptive vs non preemptive | cpu can be taken mid run vs only when the process gives it up |

## feynman checks

1. **a process (no i/o) arrives at time 0, sits in the ready queue for 6 ms, then runs its 4 ms burst and completes at time 10. what are its turnaround and waiting times?** turnaround is 10 minus 0 = 10. waiting is turnaround minus burst = 10 minus 4 = 6, which matches the 6 ms it sat in the ready queue. turnaround includes waiting plus running.
2. **why can't a non preemptive scheduler stop a long running job when an urgent one arrives?** non preemptive scheduling only acts when the running process waits or ends.
3. **why might a response time of 0.1 s beat a faster turnaround for an interactive app?** users judge the system by how soon it reacts, not when it finishes.

## review questions

- (ch4) which process states correspond to circumstances 1, 2 and 3 above?
- (ch4) what does the dispatcher have to restore, and where is it stored? (pcb, context switch)
- (ch1) why is the cpu idle during an i/o request in a simple system?

## security angle

- **scheduler as a covert channel and dos target:** a process that never yields (in a non preemptive system) can starve others. defense: preemption with fair timeslices, per process cpu quotas.
- **priority abuse:** if users can set priorities, one can starve others. defense: restrict raising priority to privileged users.
- **timing side channels:** cpu timing and cache behavior visible across processes can leak secrets; isolation and constant time code mitigate.

## omitted chuff
- the repeated multiprocessor reminder (covered in chapter 1).
- discussion that cpu utilization "conceptually" ranges 0 to 100 percent beyond one sentence.
- the long recap at the start of each lecture.

*Q: the one line summary? A: keep the cpu busy, then judge the schedule by waiting, turnaround and response.*
