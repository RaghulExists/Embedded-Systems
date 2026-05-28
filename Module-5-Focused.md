# Module-5: Focused Answers

> Each answer covers exactly what the question asks — based on `Module-5.md` notes. Where the notes do not cover something, I say so honestly and use standard textbook knowledge.

---

## Q1. Describe Process States and State Transition Representation with diagram.

A process traverses through a series of states from creation to termination. This cycle is called the **Process Life Cycle**. The change of a process from one state to another is called **state transition**.

### The five process states

| State | Description |
|---|---|
| **Created** | The process is recognised by the OS but no resources are allocated yet. |
| **Ready** | The process has been admitted to memory and is waiting in the **Ready queue** for CPU time. |
| **Running** | The process's instructions are being executed by the CPU. |
| **Blocked / Wait** | The running process is temporarily suspended — waiting for an event (e.g., I/O, user input, or access to a shared resource). |
| **Completed** | The process has finished execution. |

### State Transition Diagram

```
           +-----------+
           |  Created  |
           +-----+-----+
                 |
                 | (admitted to memory)
                 v
           +-----+-----+
       +-->|   Ready   |<-------+
       |   +-----+-----+        |
       |         |              | (event occurs / 
       |         | (scheduled)  |  I/O complete)
(pre- |         v              |
empted)|   +-----+-----+        |
       |   |  Running  |--------+
       +---+-----+-----+    (waits for I/O
                 |          or resource)
                 |              |
                 |              v
                 |       +------+------+
                 |       |   Blocked   |
                 |       +-------------+
                 | (execution finished)
                 v
           +-----------+
           | Completed |
           +-----------+
```

### Transitions explained

- **Created → Ready**: process loaded into memory, placed in Ready queue.
- **Ready → Running**: scheduler picks this process.
- **Running → Ready**: process is preempted (e.g., time slice over, or higher priority arrives).
- **Running → Blocked**: process requests I/O or waits for a resource.
- **Blocked → Ready**: the awaited event occurs.
- **Running → Completed**: process finishes execution.

### Step-by-step

1. A process is born in the **Created** state — OS knows about it but resources are not yet allocated.
2. Once admitted, it moves to **Ready** and joins the Ready queue.
3. The scheduler picks it for execution → **Running**.
4. From Running, three things can happen: it gets preempted (→ Ready), it waits for I/O (→ Blocked), or it finishes (→ Completed).
5. From Blocked, when the event occurs, it moves back to **Ready**, not directly to Running.

---

## Q2. Discuss Task Scheduling and what factors must be considered for the selection of a scheduling algorithm.

**Task scheduling** is the mechanism of determining which task/process is to be executed at a given point of time. The kernel service that implements the scheduling algorithm is called the **scheduler**.

Scheduling is broadly classified into:
- **Non-preemptive scheduling** — the running task runs until it terminates or enters the Wait state.
- **Preemptive scheduling** — the running task can be stopped temporarily and another task from the Ready queue is selected.

### Factors to consider when selecting a scheduling algorithm

| Factor | Meaning | Goal |
|---|---|---|
| **CPU Utilisation** | Percentage of time the CPU is being utilised. | Should be **high**. |
| **Throughput** | Number of processes executed per unit time. | Should be **high**. |
| **Turnaround Time (TAT)** | Total time taken by a process for completing execution (waiting for memory + ready queue + I/O + execution). | Should be **minimal**. |
| **Waiting Time** | Time a process spends in the Ready queue waiting for the CPU. | Should be **minimal**. |
| **Response Time** | Time elapsed between submission of a process and its first response. | Should be **least**. |

A good scheduling algorithm has **high CPU utilisation, minimum TAT, maximum throughput, and least response time.**

### Step-by-step

1. Scheduling exists because multiple processes share one CPU — we need a rule for who runs next.
2. The scheduler runs as a kernel service and applies a chosen scheduling algorithm.
3. Scheduling decisions are made when a process: switches Running → Ready (preemption), Running → Blocked (waits), Blocked → Ready (event done), or Running → Completed.
4. To pick a good algorithm, evaluate it against the five factors above. There is usually a trade-off (e.g., higher throughput may increase response time).

---

## Q3. Explain the different types of message passing between processes.

**Message passing** is an (a)synchronous information exchange mechanism used for Inter Process/Thread Communication. Compared to shared memory, only a limited amount of data is passed, but message passing is faster and free from synchronisation overheads.

Message passing is classified into three types:

### 1. Message Queue

A **First-In-First-Out (FIFO) queue** that stores messages temporarily in a system-defined memory object until the destination process reads them.

- The sender posts a message using `send(destination_process, message)`.
- The receiver reads using `receive(source_process, message)`.
- Implementation is OS kernel dependent.

```
   Process 1  --->  [ msg1 | msg2 | msg3 ]  --->  Process 2
                       Message Queue (FIFO)
```

The Windows XP kernel maintains one system message queue plus a thread-specific queue.

Messaging is further classified as:
- **Asynchronous** — sender posts the message and continues; does not wait for a reply.
- **Synchronous** — sender blocks (waits) until it receives a reply from the receiver.

### 2. Mailbox

An alternate form of message queue used in RTOS, usually for **one-way messaging**.

- A task creates a mailbox and is called the **mailbox server**.
- Other tasks (called **mailbox clients**) subscribe to the mailbox.
- The server posts messages and notifies all subscribers; clients read on notification.
- Mailbox typically holds a **single message** (vs. multiple in a message queue).

```
                 +-----------+
                 |  Mailbox  |
                 +-----+-----+
                       |
       Task 1 (server) posts → broadcasts to all subscribers
                       |
        +--------------+---------------+
        v              v               v
      Task 2         Task 3          Task 4
     (clients receive notifications and read)
```

### 3. Signalling

A **primitive** form of communication used for **asynchronous notifications**.

- One process/thread fires a signal to indicate the occurrence of an event.
- Signals are **not queued** and **do not carry any data**.
- Used in RTX51 Tiny OS and VxWorks for inter-process communication.

### Step-by-step

1. Message passing is used when processes need to exchange small, structured information rather than large blocks of data.
2. **Message queue** is used when multiple messages need to be buffered FIFO-style.
3. **Mailbox** is used in RTOS for one-way messaging — typically a single message broadcast to multiple subscribers.
4. **Signalling** is the simplest form — just an event notification, no data, no queueing.

---

## Q4. Three processes with process IDs P1, P2, P3 with estimated completion time 6, 4, 2 ms respectively, enter the ready queue together in the order P1, P2, P3. Calculate the waiting time and Turn Around Time (TAT) for each process and the average WT and TAT in **RR algorithm with Time slice = 2 ms**.

### Given
- P1 = 6 ms, P2 = 4 ms, P3 = 2 ms (all arrive at t = 0)
- Time slice = 2 ms
- Order in Ready queue at t=0: P1, P2, P3

### Execution sequence (Gantt chart)

```
| P1 | P2 | P3 | P1 | P2 | P1 |
0    2    4    6    8    10   12
```

### Ready queue at each instant

| Time | Event | Ready Queue (after event) |
|---|---|---|
| 0 ms | P1 scheduled | [P2, P3] |
| 2 ms | P1 preempted (4 ms left), P2 scheduled | [P3, P1] |
| 4 ms | P2 preempted (2 ms left), P3 scheduled | [P1, P2] |
| 6 ms | P3 completed, P1 scheduled | [P2] |
| 8 ms | P1 preempted (2 ms left), P2 scheduled | [P1] |
| 10 ms | P2 completed, P1 scheduled | [ ] |
| 12 ms | P1 completed | — |

### Waiting Time (WT)

Waiting time = total time spent in Ready queue (not including execution).

- **P1**: arrived at 0, waited (0 to 0) + (2 to 6) + (8 to 10) = **0 + 4 + 2 = 6 ms**
- **P2**: arrived at 0, waited (0 to 2) + (4 to 8) = **2 + 4 = 6 ms**
- **P3**: arrived at 0, waited (0 to 4) = **4 ms**

**Average Waiting Time** = (6 + 6 + 4) / 3 = **16 / 3 = 5.33 ms**

### Turn Around Time (TAT)

TAT = Waiting Time + Execution Time

- **P1**: 6 + 6 = **12 ms**
- **P2**: 6 + 4 = **10 ms**
- **P3**: 4 + 2 = **6 ms**

**Average TAT** = (12 + 10 + 6) / 3 = **28 / 3 = 9.33 ms**

### Final answers table

| Process | Burst Time | Waiting Time | TAT |
|---|---|---|---|
| P1 | 6 | 6 | 12 |
| P2 | 4 | 6 | 10 |
| P3 | 2 | 4 | 6 |
| **Average** | | **5.33 ms** | **9.33 ms** |

### Step-by-step

1. At t = 0, all processes are in the Ready queue in order [P1, P2, P3]. P1 runs first for one slice (2 ms).
2. At t = 2, P1 is preempted (still has 4 ms left) and goes to the back of the queue. P2 runs next.
3. Continue this round-robin until each process completes. When a process finishes within or at the end of its slice, it leaves the queue (does not go back).
4. **Waiting time** for each process = sum of time spent in Ready queue between arrivals to running. Calculate by adding up the gaps between when the process is scheduled out and when it is scheduled in again.
5. **TAT** = Waiting Time + Burst (execution) time.
6. Take the arithmetic mean of WT and TAT across all processes for averages.

> Note: The question text appears to have a stray sentence "Explain Swap instruction of ARM7 with an example." appended at the end of Q4. The Module-5 notes do **not** cover the ARM7 SWP/SWAP instruction — it is a Module-3/4 ARM-architecture topic. If you want, I can answer it separately using my knowledge, but I'm not sure if it's part of Q4 or a typo, so I have left it out to stay strictly on what Module-5 covers. Let me know if you want it added.

---

## Q5. Write a multithreaded application to print "Hello I'm in main thread" from the main thread and "Hello I'm in new thread" 5 times each, using the `pthread_create()` and `pthread_join()` POSIX primitives.

> Note: The Module-5 notes describe the **signatures** of `pthread_create()` and `pthread_join()` but do **not** contain a complete worked "Hello" example. The code below is written using my standard POSIX knowledge, consistent with the primitives described in your notes.

### Code

```c
#include <stdio.h>
#include <pthread.h>

/* Function executed by the new thread */
void *new_thread_function(void *arg)
{
    int i;
    for (i = 0; i < 5; i++) {
        printf("Hello I'm in new thread\n");
    }
    return NULL;
}

int main(void)
{
    pthread_t new_thread_ID;
    int i;

    /* Create the new thread */
    pthread_create(&new_thread_ID, NULL, new_thread_function, NULL);

    /* Main thread prints its message 5 times */
    for (i = 0; i < 5; i++) {
        printf("Hello I'm in main thread\n");
    }

    /* Wait for the new thread to finish before exiting */
    pthread_join(new_thread_ID, NULL);

    return 0;
}
```

### Explanation of POSIX primitives used (from notes)

`int pthread_create(pthread_t *new_thread_ID, const pthread_attr_t *attribute, void *(*start_function)(void *), void *arguments);`

- Creates a new thread that runs `start_function`.
- `new_thread_ID` receives the handle (TCB pointer) of the created thread.
- `attribute` = NULL means default attributes.
- Returns 0 on success.

`int pthread_join(pthread_t new_thread, void **thread_status);`

- Blocks the current (main) thread until `new_thread` completes.
- Ensures the program does not exit before the new thread finishes.

### Step-by-step

1. The program starts in the **main thread**.
2. `pthread_create` is called → a new thread is spawned which runs `new_thread_function()` independently.
3. From this point, both threads run concurrently. The main thread continues into its `for` loop printing "Hello I'm in main thread" five times. The new thread prints "Hello I'm in new thread" five times.
4. The actual interleaving of output depends on the scheduler — both messages may appear in any mixed order.
5. `pthread_join` blocks the main thread until the new thread finishes, ensuring `main()` does not return (and end the program) before the new thread has printed its messages.
6. After join returns, `main` returns and the program exits cleanly.

> Compile with: `gcc program.c -lpthread -o program`

---

## Q6. Three processes with process IDs P1, P2, P3 with estimated completion time 8, 4, 6 ms respectively enter the Ready queue together. A new process P4 with estimated completion time 3 ms enters the 'Ready' queue after 2 ms. Calculate the waiting time and TAT for each process and the average WT and TAT in **SJF non-preemptive scheduling**.

### Given
- At t = 0: P1 = 8, P2 = 4, P3 = 6 ms (all in Ready queue)
- At t = 2: P4 = 3 ms arrives
- Algorithm: **SJF (Shortest Job First) non-preemptive** — once a process starts, it runs to completion.

### Execution

At **t = 0**, Ready queue = {P1=8, P2=4, P3=6}. Shortest is **P2 (4 ms)** → schedule P2.
- P2 runs from **0 → 4** ms.
- At t = 2, P4 arrives but P2 cannot be preempted (non-preemptive). P4 just waits.

At **t = 4**, P2 completes. Ready queue = {P1=8, P3=6, P4=3}. Shortest is **P4 (3 ms)** → schedule P4.
- P4 runs from **4 → 7** ms.

At **t = 7**, P4 completes. Ready queue = {P1=8, P3=6}. Shortest is **P3 (6 ms)** → schedule P3.
- P3 runs from **7 → 13** ms.

At **t = 13**, P3 completes. Ready queue = {P1=8}. Schedule P1.
- P1 runs from **13 → 21** ms.

### Gantt Chart

```
|   P2  |  P4 |    P3    |     P1     |
0       4     7          13           21
```

### Waiting Time (WT) = Start Time − Arrival Time

| Process | Arrival | Start | Waiting Time |
|---|---|---|---|
| P2 | 0 | 0 | 0 ms |
| P4 | 2 | 4 | 4 − 2 = 2 ms |
| P3 | 0 | 7 | 7 ms |
| P1 | 0 | 13 | 13 ms |

**Average Waiting Time** = (0 + 2 + 7 + 13) / 4 = **22 / 4 = 5.5 ms**

### Turn Around Time (TAT) = Waiting Time + Burst Time

| Process | WT | Burst | TAT |
|---|---|---|---|
| P2 | 0 | 4 | 4 ms |
| P4 | 2 | 3 | 5 ms |
| P3 | 7 | 6 | 13 ms |
| P1 | 13 | 8 | 21 ms |

**Average TAT** = (4 + 5 + 13 + 21) / 4 = **43 / 4 = 10.75 ms**

### Final answers

| Process | Burst | WT | TAT |
|---|---|---|---|
| P1 | 8 | 13 | 21 |
| P2 | 4 | 0 | 4 |
| P3 | 6 | 7 | 13 |
| P4 | 3 | 2 | 5 |
| **Average** | | **5.5 ms** | **10.75 ms** |

### Step-by-step

1. **Non-preemptive** means once a process is given the CPU, it runs to completion — no interruption.
2. At t = 0, all of P1, P2, P3 are in the queue. SJF picks the one with the **shortest burst time** → P2 (4 ms).
3. While P2 is running, P4 arrives at t = 2. But because we are non-preemptive, P2 keeps running until it completes at t = 4.
4. At t = 4, the Ready queue contains P1(8), P3(6), and P4(3). SJF picks the shortest → P4.
5. After P4 completes at t = 7, the next shortest is P3 (6 ms).
6. After P3 completes at t = 13, only P1 remains; it runs until t = 21.
7. **Waiting time** = (start time) − (arrival time). For P4, arrival is 2 ms and start is 4 ms, so WT = 2 ms.
8. **TAT** = WT + Burst. **Averages** are arithmetic means.
