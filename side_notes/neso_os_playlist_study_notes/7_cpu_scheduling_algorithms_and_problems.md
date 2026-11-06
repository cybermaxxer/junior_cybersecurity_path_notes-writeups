# chapter 7: cpu scheduling algorithms and problems

> **disclaimer:** i don't claim authorship of any of this. these are my study notes from the neso academy operating systems playlist, meant to complement those lectures, not replace them. watch the videos.

**videos covered (playlist order):**

| # | video | link |
|---|---|---|
| 1 | Scheduling Algorithms: First Come First Served (FCFS) | https://www.youtube.com/watch?v=7DoP1L9nAAs |
| 2 | First Come First Served Scheduling (Solved Problem 1) | https://www.youtube.com/watch?v=VSMAjMfJ6KQ |
| 3 | First Come First Served Scheduling (Solved Problem 2) | https://www.youtube.com/watch?v=8-BUGte27sk |
| 4 | Scheduling Algorithms: Shortest Job First (SJF) | https://www.youtube.com/watch?v=t0g9b3SJECg |
| 5 | Shortest Job First Scheduling (Solved Problem 1) | https://www.youtube.com/watch?v=lpM14aWgl3Q |
| 6 | Shortest Job First Scheduling (Solved Problem 2) | https://www.youtube.com/watch?v=ypOnf9mnFYg |
| 7 | Scheduling Algorithms: Priority Scheduling | https://www.youtube.com/watch?v=yKD3pcFvGmY |
| 8 | Priority Scheduling (Solved Problem 1) | https://www.youtube.com/watch?v=Z2KsfhEJOFA |
| 9 | Priority Scheduling (Solved Problem 2) | https://www.youtube.com/watch?v=gHKSmz-W0x0 |
| 10 | Scheduling Algorithms: Round Robin Scheduling | https://www.youtube.com/watch?v=YzBBJYfwdi8 |
| 11 | Round Robin Scheduling (Turnaround Time & Waiting Time) | https://www.youtube.com/watch?v=7TpxxTNrcTg |
| 12 | Round Robin Scheduling: Solved Problem (Part 1) | https://www.youtube.com/watch?v=QlCmgBOMjlI |
| 13 | Round Robin Scheduling: Solved Problem (Part 2) | https://www.youtube.com/watch?v=wioTortHb_g |
| 14 | Multilevel Queue Scheduling Algorithm | https://www.youtube.com/watch?v=fvkSXMZaBNY |
| 15 | Multilevel Feedback Queue Scheduling Algorithm | https://www.youtube.com/watch?v=1KLuC0knvs8 |
| 16 | Scheduling Algorithms: Solved Problems | https://www.youtube.com/watch?v=g6QjVDyGgSE |

**how to solve any scheduling problem (the routine every lecture follows):**
1. draw the **gantt chart** first.
2. completion time = where the process's *last* block ends.
3. turnaround = completion minus arrival.
4. waiting = turnaround minus burst.
5. average.
for preemptive problems you can also use: waiting = total waiting (start of last block) minus time already executed minus arrival.

## table of contents
- [fcfs](#first-come-first-served-fcfs)
- [fcfs solved problem 1](#fcfs-solved-problem-1)
- [fcfs solved problem 2](#fcfs-solved-problem-2)
- [sjf](#shortest-job-first-sjf)
- [sjf solved problem 1](#sjf-solved-problem-1-gate-2014)
- [sjf solved problem 2](#sjf-solved-problem-2-gate-2016)
- [priority scheduling](#priority-scheduling)
- [priority solved problem 1](#priority-solved-problem-1-gate-2017)
- [priority solved problem 2](#priority-solved-problem-2)
- [round robin](#round-robin-scheduling)
- [round robin turnaround and waiting](#round-robin-turnaround-time-and-waiting-time)
- [round robin solved problem parts 1 and 2](#round-robin-solved-problem-parts-1-and-2)
- [multilevel queue](#multilevel-queue-scheduling)
- [multilevel feedback queue](#multilevel-feedback-queue-scheduling)
- [scheduling algorithms: solved problems](#scheduling-algorithms-solved-problems-gate-questions)
- [cheat sheet](#cheat-sheet)
- [interleaving](#interleaving-compare-all-algorithms)
- [feynman checks](#feynman-checks)
- [review questions](#review-questions)
- [security angle](#security-angle)
- [omitted chuff](#omitted-chuff)

---

## first come first served (fcfs)

the simplest algorithm: the process that requests the cpu first gets it first. implemented with a **fifo queue**: a process's pcb joins the tail of the ready queue, the cpu goes to the head. it is **non preemptive**: once a process has the cpu it keeps it until it terminates or requests i/o.

**average waiting time can be long and varies a lot with arrival order.** processes arrive at 0 with bursts p1 = 24, p2 = 3, p3 = 3 ms:

| order | gantt | waits (p1, p2, p3) | average |
|---|---|---|---|
| p1, p2, p3 | p1 0 to 24, p2 24 to 27, p3 27 to 30 | 0, 24, 27 | (0+24+27)/3 = **17 ms** |
| p2, p3, p1 | p2 0 to 3, p3 3 to 6, p1 6 to 30 | 6, 0, 3 | (6+0+3)/3 = **3 ms** |

**convoy effect:** short processes stuck behind a long one wait a long time. fcfs is troublesome for time sharing systems because one process could hold the cpu far too long.

*Q: which property makes fcfs bad for time sharing? A: non preemptive, so a long job blocks everyone.*

---

## fcfs solved problem 1

| process | p1 | p2 | p3 | p4 | p5 |
|---|---|---|---|---|---|
| arrival | 4 | 6 | 0 | 6 | 5 |
| burst | 5 | 4 | 3 | 2 | 4 |

**tie rule:** p2 and p4 both arrive at 6, so the smaller process id goes first.

**gantt:** p3 0 to 3, **idle 3 to 4** (nobody has arrived), p1 4 to 9, p5 9 to 13, p2 13 to 17, p4 17 to 19.

| process | completion | turnaround (ct minus at) | waiting (tat minus bt) |
|---|---|---|---|
| p1 | 9 | 5 | 0 |
| p2 | 17 | 11 | 7 |
| p3 | 3 | 3 | 0 |
| p4 | 19 | 13 | 11 |
| p5 | 13 | 8 | 4 |

average turnaround = 40/5 = **8**, average waiting = 22/5 = **4.4**.

---

## fcfs solved problem 2

six processes arrive at 0,1,2,3,4,5 with bursts 3,2,1,4,5,2. **every scheduling decision costs 1 unit of overhead.** find the efficiency.

**gantt:** d 0 to 1, p1 1 to 4, d 4 to 5, p2 5 to 7, d 7 to 8, p3 8 to 9, d 9 to 10, p4 10 to 14, d 14 to 15, p5 15 to 20, d 20 to 21, p6 21 to 23 (d = delay).

- useless time = 6 delays = 6
- total time = 23
- useful time = 23 minus 6 = 17

$$\text{efficiency} = \frac{17}{23} = 0.7391 = 73.91\%$$

---

## shortest job first (sjf)

give the cpu to the process with the **smallest next cpu burst**. tie? break it with fcfs. it can be **preemptive** or **non preemptive**. a more accurate name: *shortest next cpu burst*, because it looks at the next burst, not the total length.

**non preemptive example** (all arrive at 0): p1 = 6, p2 = 8, p3 = 7, p4 = 3.
order p4, p1, p3, p2. gantt: p4 0 to 3, p1 3 to 9, p3 9 to 16, p2 16 to 24.
waits: p1 3, p2 16, p3 9, p4 0, average = 28/4 = **7 ms**. the same set under fcfs averages 10.25 ms, so sjf is better. it gives the **minimum average waiting time** among these algorithms.

**preemptive example (shortest remaining time first, srtf):** p1 arrives 0 burst 8, p2 arrives 1 burst 4, p3 arrives 2 burst 9, p4 arrives 3 burst 5.
- p1 runs 0 to 1; p2 arrives with burst 4 less than p1's remaining 7, so it preempts.
- p2 runs 1 to 5. while it runs, p3 (9) and p4 (5) arrive but neither beats p2's remaining time.
- at 5 the remaining times are p1 = 7, p3 = 9, p4 = 5, so p4 runs 5 to 10.
- p1 runs 10 to 17, then p3 17 to 26.

**waiting time formula for preemptive runs:** waiting = start of last run minus time already executed minus arrival.
- p1: 10 minus 1 minus 0 = **9**
- p2: 1 minus 0 minus 1 = **0**
- p3: 17 minus 0 minus 2 = **15**
- p4: 5 minus 0 minus 3 = **2**
average = 26/4 = **6.5 ms**.

**the catch:** you can't truly know the next burst length. solution: **predict** it, assuming it will be similar to the previous ones, and pick the shortest predicted burst. so sjf is optimal but is only approximated in real short term scheduling.

*Q: why can't sjf be implemented exactly? A: the next cpu burst length is unknown.*

---

## sjf solved problem 1 (gate 2014)

srtf, preemptive. p1 (arrival 0, burst 12), p2 (2, 4), p3 (3, 6), p4 (8, 5). average waiting time?

**gantt:** p1 0 to 2, p2 2 to 6, p3 6 to 12, p4 12 to 17, p1 17 to 27.

key checks:
- at 3, p3 (6) arrives but p2's remaining is 3, so p2 keeps going.
- at 6, p1 remaining 10 and p3 6, so p3 runs.
- at 8, p4 (5) arrives but p3's remaining is 4, so p3 keeps going.
- at 12, p1 10 and p4 5, so p4, then p1.

| process | formula | waiting |
|---|---|---|
| p1 | 17 minus 2 minus 0 | 15 |
| p2 | 2 minus 0 minus 2 | 0 |
| p3 | 6 minus 0 minus 3 | 3 |
| p4 | 12 minus 0 minus 8 | 4 |

average = 22/4 = **5.5 ms**. answer: option c.

---

## sjf solved problem 2 (gate 2016)

srtf. p1 (0, 10), p2 (3, 6), p3 (7, 1), p4 (8, 3). average **turnaround** time?

**gantt:** p1 0 to 3, p2 3 to 7, p3 7 to 8, p2 8 to 10, p4 10 to 13, p1 13 to 20.

- at 3, p2 (6) is less than p1's remaining 7, so p2 preempts.
- at 7, p2 has 2 left, p3 has 1, so p3 preempts.
- at 8, p2 (2), p4 (3), p1 (7): p2 first, then p4, then p1.

| process | completion | turnaround |
|---|---|---|
| p1 | 20 | 20 |
| p2 | 10 | 7 |
| p3 | 8 | 1 |
| p4 | 13 | 5 |

average = 33/4 = **8.25 ms**. answer: option a.

---

## priority scheduling

each process has a priority; the cpu goes to the highest priority. equal priority: fcfs order. **sjf is just priority scheduling where priority is the inverse of the predicted next burst** (longer burst, lower priority).

- **preemptive:** a newly arrived process with higher priority takes the cpu from the running one.
- **non preemptive:** the new process just goes to the head of the ready queue (position by priority); the running process isn't disturbed.

**example** (all arrive at 0; lower number means higher priority): p1 (burst 10, pri 3), p2 (1, 1), p3 (2, 4), p4 (1, 5), p5 (5, 2).
order p2, p5, p1, p3, p4. gantt: p2 0 to 1, p5 1 to 6, p1 6 to 16, p3 16 to 18, p4 18 to 19.
waits: p1 6, p2 0, p3 16, p4 18, p5 1; average = 41/5 = **8.2 ms**.

**problem: starvation (indefinite blocking).** a low priority process can wait forever behind a steady stream of higher priority ones.
**fix: aging.** gradually raise the priority of processes that wait a long time. example: priorities from 127 (lowest) to 0 (highest); add one priority step every 15 minutes, so even priority 127 eventually reaches 0.

*Q: what does aging fix? A: starvation of low priority processes.*

---

## priority solved problem 1 (gate 2017)

preemptive priority, **0 is the highest priority**. p1 (arrival 0, burst 11, pri 2), p2 (5, 28, 0), p3 (12, 2, 3), p4 (2, 10, 1), p5 (9, 16, 4).

**gantt:** p1 0 to 2, p4 2 to 5 (preempts p1), p2 5 to 33 (priority 0 can't be preempted), p4 33 to 40, p1 40 to 49, p3 49 to 51, p5 51 to 67.

| process | formula (last start minus executed before minus arrival) | waiting |
|---|---|---|
| p1 | 40 minus 2 minus 0 | 38 |
| p2 | 5 minus 0 minus 5 | 0 |
| p3 | 49 minus 0 minus 12 | 37 |
| p4 | 33 minus 3 minus 2 | 28 |
| p5 | 51 minus 0 minus 9 | 42 |

average = 145/5 = **29 ms**. answer: option a.

---

## priority solved problem 2

**non preemptive**, and **higher number means higher priority** (read the question, this flips between problems!). p1 (arrival 0, burst 4, pri 2), p2 (1, 3, 3), p3 (2, 1, 4), p4 (3, 5, 5), p5 (4, 2, 5).

**gantt:** p1 0 to 4 (nobody preempts), then among p2 (3), p3 (4), p4 (5), p5 (5): p4 and p5 tie on priority 5, so the earlier arrival (p4) goes first: p4 4 to 9, p5 9 to 11, p3 11 to 12, p2 12 to 15.

| process | completion | turnaround | waiting |
|---|---|---|---|
| p1 | 4 | 4 | 0 |
| p2 | 15 | 14 | 11 |
| p3 | 12 | 10 | 9 |
| p4 | 9 | 6 | 1 |
| p5 | 11 | 7 | 5 |

average turnaround = 41/5 = **8.2**, average waiting = 26/5 = **5.2**.

---

## round robin scheduling

designed for **time sharing**. like fcfs but **preemptive**. a small **time quantum (time slice)**, typically 10 to 100 ms, is defined. the ready queue is circular: the scheduler gives the cpu to each process for up to one quantum.

**implementation:** ready queue is a fifo queue; the scheduler takes the head, sets a **timer** for one quantum, and dispatches it. two cases:
1. the burst is **shorter than the quantum**: the process releases the cpu voluntarily; the next one runs.
2. the burst is **longer**: the timer fires an interrupt, a context switch happens, and the process goes to the **tail** of the queue (its context saved in its pcb).

**choosing the quantum:**
- too **large**: behaves like fcfs (e.g. quantum 50 ms, a 40 ms job blocks short jobs behind it).
- too **small**: too many context switches (a 5 ms process with 1 ms quantum is switched 4 times). context switches are pure overhead.
strike a balance.

---

## round robin: turnaround time and waiting time

example: p1 = 24, p2 = 3, p3 = 3, all arrive at 0, **quantum 4**.

**gantt:** p1 0 to 4, p2 4 to 7 (3 is less than 4 so it releases early), p3 7 to 10, then p1 only: 10 to 14, 14 to 18, 18 to 22, 22 to 26, 26 to 30 (it's preempted each time but is alone in the queue so it gets the cpu again).

| process | completion | turnaround (ct minus at) | waiting (tat minus bt) |
|---|---|---|---|
| p1 | 30 | 30 | 6 |
| p2 | 7 | 7 | 4 |
| p3 | 10 | 10 | 7 |

average turnaround = 47/3 = **15.67**, average waiting = 17/3 = **5.67**.

**method 2 (waiting time only):**

$$\text{waiting} = \text{last start} - \text{arrival} - (\text{preemptions} \times \text{quantum})$$

for p1: last start 26, arrival 0, preempted 5 times: 26 minus 0 minus 5 times 4 = **6**. p2 and p3 never preempted: 4 minus 0 = 4, and 7 minus 0 = 7. same answers. use method 1 for both times, method 2 for waiting only.

---

## round robin solved problem (parts 1 and 2)

**quantum 2.** p1 (arrival 0, burst 5), p2 (1, 3), p3 (2, 1), p4 (3, 2), p5 (4, 3). find average waiting and turnaround.

**the rules that matter when arrivals differ:**
- the alarm rings after each quantum: preempt the running process, give the cpu to the head of the ready queue.
- a process that arrives **at the same instant** another is preempted enters the queue **first**; the preempted process goes **behind** it.
- a process shorter than the quantum releases early (no alarm).

**ready queue trace:**

| time | event | queue after (head on left) |
|---|---|---|
| 0 | p1 arrives and runs | empty |
| 1 | p2 arrives | p2 |
| 2 | quantum over; p3 arrives; p1 (3 left) preempted; p2 runs | p3, p1 |
| 3 | p4 arrives | p3, p1, p4 |
| 4 | p2 (1 left) preempted; p5 arrives; p3 runs | p1, p4, p5, p2 |
| 5 | p3 done after 1 unit; p1 runs | p4, p5, p2 |
| 7 | p1 (1 left) preempted; p4 runs | p5, p2, p1 |
| 9 | p4 done; p5 runs | p2, p1 |
| 11 | p5 (1 left) preempted; p2 runs | p1, p5 |
| 12 | p2 done; p1 runs | p5 |
| 13 | p1 done; p5 runs | empty |
| 14 | p5 done | |

**gantt:** p1 0 to 2, p2 2 to 4, p3 4 to 5, p1 5 to 7, p4 7 to 9, p5 9 to 11, p2 11 to 12, p1 12 to 13, p5 13 to 14.

| process | completion | turnaround | waiting |
|---|---|---|---|
| p1 | 13 | 13 | 8 |
| p2 | 12 | 11 | 8 |
| p3 | 5 | 3 | 2 |
| p4 | 9 | 6 | 4 |
| p5 | 14 | 10 | 7 |

average turnaround = 43/5 = **8.6**, average waiting = 29/5 = **5.8**.

---

## multilevel queue scheduling

use when processes **classify into groups**. example: **foreground (interactive)** processes need fast response and may have priority over **background (batch)** processes.

- the ready queue is **partitioned into several separate queues**.
- each process is **permanently assigned** to one queue (by memory size, priority or type).
- **each queue has its own scheduling algorithm:** foreground gets **round robin** (good for time sharing), background gets **fcfs**.
- **between the queues**: usually **fixed priority preemptive scheduling**. the foreground queue can have absolute priority over background.

**example with five queues in priority order:** system processes, interactive processes, interactive editing processes, batch processes, student processes. a student process runs only when every queue above is empty. if a batch process is running and an interactive editing process arrives, the batch process is **preempted**.

so there are two levels of scheduling: within each queue, and among the queues.

---

## multilevel feedback queue scheduling

like multilevel queue but **processes can move between queues**. the idea: separate processes by the **characteristics of their cpu bursts**.

- a process that uses too much cpu time moves to a **lower priority** queue.
- i/o bound and interactive processes stay in the higher queues.
- a process waiting too long in a low queue can move up: **aging**, which prevents starvation.

**example:** three queues:
- **q0:** round robin style, quantum **8 ms** (highest priority).
- **q1:** quantum **16 ms**.
- **q2:** **fcfs** (lowest).

a process in q0 that doesn't finish in 8 ms moves to the **tail of q1**; one that doesn't finish in 16 ms there moves to the tail of q2. q1 runs only when q0 is empty, q2 only when both are empty.

**parameters that define an mlfq scheduler:**
1. the number of queues,
2. the scheduling algorithm for each queue,
3. the method to decide when to **upgrade** a process,
4. the method to decide when to **demote** a process,
5. the method to decide **which queue** a process enters when it needs service.

more complex to implement, but flexible.

---

## scheduling algorithms: solved problems (gate questions)

**q1 (gate 2001): which algorithm gives maximum throughput on a uniprocessor?** options: round robin, sjf, highest response ratio next, fcfs. **answer: shortest job first.** it finishes the most processes per unit time because short jobs go first. fcfs is out (long job first), rr depends on the quantum.

**q2 (gate 2002): which is non preemptive?** round robin, fifo, multilevel queue, multilevel queue with feedback. **answer: fifo** (that is fcfs). the others can preempt.

**q3 (gate 2010): which statements are true?** (1) srtf may cause starvation, (2) preemptive scheduling may cause starvation, (3) round robin is better than fcfs in response time. **answer: all three, option d.** a long process can keep losing to shorter arrivals; preemption can repeatedly push a process aside; rr gives each process the cpu quickly while fcfs makes them queue behind long jobs.

**q4: three processes** p1 (arrival 0, burst 5), p2 (1, 7), p3 (3, 4). completion order under fcfs and rr (quantum 2)?
- fcfs: **p1, p2, p3**.
- rr (quantum 2), my check: p1 0 to 2, p2 2 to 4, p1 4 to 6, p3 6 to 8, p2 8 to 10, p1 10 to 11 (done), p3 11 to 13 (done), p2 13 to 15, then p2 15 to 16 (done). order **p1, p3, p2**.
- answer: the option listing fcfs = p1 p2 p3 and rr = p1 p3 p2 (option c).

**q5 (gate 2013): priority proportional to waiting time, reevaluated every t units, all arrive at 0, no i/o. which algorithm is it equivalent to?** options: fcfs, round robin, sjf, srtf. **answer: round robin (b).** the periodic reevaluation every t works exactly like a time quantum.

---

## cheat sheet

| algorithm | rule | preemptive? | strength | weakness |
|---|---|---|---|---|
| fcfs | arrival order | no | simplest | convoy effect, long average wait |
| sjf | smallest next burst | optional (srtf is preemptive) | minimum average wait | burst unknowable, starvation of long jobs |
| priority | highest priority | optional | models importance | starvation, fix with aging |
| round robin | quantum rotation | yes | fair, good response | quantum choice, more switches |
| multilevel queue | classes of queues, fixed priority across | yes | separate treatment for classes | permanent assignment, starvation of low queues |
| multilevel feedback | queues plus movement | yes | adapts to behavior | complex to tune |

| formula | meaning |
|---|---|
| tat = ct minus at | turnaround |
| wt = tat minus bt | waiting |
| wt (preemptive) = last start minus executed before minus arrival | from gantt |
| wt (rr) = last start minus arrival minus preemptions times quantum | round robin only |
| efficiency = useful time / total time | with scheduling overhead |

## interleaving: compare all algorithms

| question | answer |
|---|---|
| which minimizes average waiting? | sjf / srtf |
| which is best for time sharing? | round robin |
| which is simplest to implement? | fcfs |
| which causes starvation? | sjf, priority, multilevel queue (fix: aging, feedback) |
| which suffers the convoy effect? | fcfs |
| is sjf just a priority scheduler? | yes, with priority as the inverse of the next burst |
| rr with an enormous quantum becomes? | fcfs |
| rr with a tiny quantum costs? | lots of context switch overhead |

## feynman checks

1. **a system runs 5 short interactive jobs and 1 huge batch job. which scheduler fits and why?** a multilevel queue or feedback queue: round robin for the interactive queue, fcfs for batch, with the interactive queue above batch, so interactive jobs stay responsive.
2. **a priority scheduler never runs your low priority backup job. diagnose and fix.** starvation from a stream of higher priority work; add aging (raise priority over time) or move to feedback queues.
3. **in the rr example (quantum 4, bursts 24, 3, 3) p1 finishes last at 30, yet its waiting time is only 6. how?** waiting = turnaround minus burst = 30 minus 24 = 6. the other two jobs only need 3 + 3 = 6 ms of cpu in total, so that's all the time p1 spends off the cpu; the long job finishes last because of its size, not because it waits a lot.

## review questions

- (ch6) which criterion does each algorithm try to minimize or maximize?
- (ch6) which of the four preemption circumstances produces the extra preemptions in rr? (running to ready on a timer interrupt)
- (ch4) how does a context switch show up in the round robin cost?
- (ch4) which pcb field holds the priority used by priority scheduling?

## security angle

- **starvation as denial of service:** an attacker flooding high priority work starves legitimate jobs. defense: aging, per user quotas.
- **priority inversion and escalation:** letting unprivileged users raise priority is a privilege escalation. restrict nice and priority changes.
- **scheduler timing leaks:** predictable scheduling helps side channel and timing attacks; randomization and isolation reduce this.
- **defending with cgroups and quotas** bounds what any one process or user can consume.

## omitted chuff
- the longer motivational intros on each problem, and repeated lines like "form the gantt chart first".
- the full second walkthrough of the same gantt charts (kept the final charts and traces).
- the detail of the two round robin turnaround methods' commentary beyond the formulas.
- the full prose recap of every aging example, kept the 127 to 0 case.
- gate 2001 notes on "highest response ratio next": the lecture only gestured at it, so it isn't expanded here.

*Q: one picture to remember? A: gantt chart first, formulas second, always.*
