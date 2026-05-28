Embedded / Real Time
Operating System
Concepts
1
EMBEDDED SYSTEMS
```
(21ECE63)
```
MODULE – 5
Operating System Basics
2
Operating System Basics
3
• The operating system acts as a bridge between the user
applications/tasks and the underlying system resources
through a set of system functionalities and services.
• The OS manages the system resources and makes them
available to the user applications/tasks on a need basis.
• The primary functions of an operating system are:
• Make the system convenient to use
• Organise and manage the system resources efficiently and
correctly
Operating System Architecture
• Figure gives an insight into the
basic components of an
operating system and their
interfaces with rest of the
world.
User Applications
Underlying Hardware
Kernel
Services
Memory Management
Process Management
Time Management
File System Management
I/O System Management
Application
Programming
```
Interface (API)
```
4
Device Driver
Interface
```
Fig: The Operating System Architecture
```
Types of Operating
Systems
5
Types of Operating Systems
6
• Depending on the type of kernel and kernel services, purpose and
type of computing systems where the OS is deployed and the
responsiveness to applications, Operating Systems are classified
into different types.
```
• General Purpose Operating System (GPOS)
```
```
• Real-Time Operating System (RTOS)
```
General Purpose Operating System
```
(GPOS)
```
7
• The operating systems which are deployed in general computing systems are referred
```
as General Purpose Operating Systems (GPOS).
```
• The kernel of such a GPOS is more generalised and it contains all kinds of services
required for executing generic applications.
• General purpose operating systems are often quite non-deterministic in behaviour.
• Their services can inject random delays into application software and may cause slow
responsiveness of an application at unexpected times.
• GPOS are usually deployed in computing systems where deterministic behaviour is not
an important criterion.
• Personal Computer/Desktop system is a typical example for a system where GPOSs are
deployed.
• Windows XP/MS-DOS etc. are examples for General Purpose Operating Systems.
```
Real-Time Operating System (RTOS)
```
8
• 'Real-Time' implies deterministic timing behaviour.
• Deterministic timing behaviour in RTOS context means the OS services consumes only
known and expected amounts of time regardless the number of services.
• A Real-Time Operating System or RTOS implements policies and rules concerning
time-critical allocation of a system's resources.
• The RTOS decides which applications should run in which order and how much time needs
to be allocated for each application.
• Predictable performance is the hallmark of a well-designed RTOS.
• This is best achieved by the consistent application of policies and rules.
• Policies guide the design of an RTOS.
• Rules implement those policies and resolve policy conflicts.
• Windows CE, QNX, VxWorks, MicroC/OS-II, etc. are examples of Real-Time Operating
```
Systems (RTOS).
```
Tasks, Process and
Threads
9
Task
10
• The term 'task' refers to something that needs to be done.
• In the operating system context, a task is defined as the program in
execution and the related information maintained by the operating
system for the program.
• Task is also known as 'Job' in the operating system context.
• A program or part of it in execution is also called a 'Process’.
• The terms 'Task', 'Job' and 'Process' refer to the same entity in the
operating system context and most often they are used interchangeably.
Process
11
• A 'Process' is a program, or part of it, in execution.
• Process is also known as an instance of a program in execution.
• Multiple instances of the same program can execute
simultaneously.
• A process requires various system resources like CPU for executing
```
the process; memory for storing the code corresponding to the
```
process and associated variables, I/O devices for information
exchange, etc.
• A process is sequential in execution.
The Structure of a Process
12
```
• The concept of 'Process' leads to concurrent execution (pseudo
```
```
parallelism) of tasks and thereby the efficient utilisation of the CPU and
```
other system resources.
• Concurrent execution is achieved through the sharing of CPU among the
processes.
• A process mimics a processor in properties and holds a set of registers,
```
process status, a Program Counter (PC) to point to the next executable
```
instruction of the process, a stack for holding the local variables
associated with the process and the code corresponding to the process.
• This can be visualised as shown in the figure.
```
The Structure of a Process (continued)
```
Stack
```
(Stack Pointer)
```
Working Registers
Status Registers
```
Program Counter (PC)
```
Process
Code memory
corresponding to the
Process
```
Fig: Structure of a Process
```
13
```
The Structure of a Process (continued)
```
14
• A process which inherits all the properties of the CPU can be
considered as a virtual processor, awaiting its turn to have its
properties switched into the physical processor.
• When the process gets its turn, its registers and the program
counter register becomes mapped to the physical registers of the
CPU.
```
The Structure of a Process (continued)
```
• From a memory perspective, the
memory occupied by the process
is segregated into three regions as
shown in the figure:
• Stack memory - holds all
temporary data such as variables
local to the process
• Data memory - holds all global data
for the process
• Code memory - contains the
```
program code (instructions)
```
corresponding to the process
```
Fig: Memory Organisation of a Process
```
15
Process States and State Transition
16
• The process traverses through a series of states during its transition from
the newly created state to the terminated state.
• The cycle through which a process changes its state from 'newly created'
to 'execution completed' is known as 'Process Life Cycle’.
• The various states through which a process traverses through during a
Process Life Cycle indicates the current status of the process with respect
to time and also provides information on what it is allowed to do next.
• The transition of a process from one state to another is known as 'State
transition’.
• Figure represents the various states and state transitions associated with
a process.
Created
Ready
Blocked
Running
17
Completed
```
Fig: Process States and State Transition Representation
```
Process States and State Transition
```
(continued)
```
18
• The state at which a process is being created is referred as 'Created
State’.
• The Operating System recognises a process in the 'Created State' but no resources
are allocated to the process.
• The state, where a process is incepted into the memory and awaiting the
processor time for execution, is known as 'Ready State’.
• At this stage, the process is placed in the 'Ready list' queue maintained by the OS.
• The state where in the source code instructions corresponding to the
process is being executed is called 'Running State’.
• Running State is the state at which the process execution happens.
Process States and State Transition
```
(continued)
```
19
• 'Blocked State/Wait State' refers to a state where a running process
is temporarily suspended from execution and does not have
immediate access to resources.
• The blocked state might be invoked by various conditions like:
```
• the process enters a wait state for an event to occur (e.g. Waiting for user inputs
```
```
such as keyboard input) or
```
• waiting for getting access to a shared resource
• A state where the process completes its execution is known as
'Completed State’.
Process Management
20
• Process management deals with
• creation of a process
• setting up the memory space for the process
• loading the process's code into the memory space
• allocating system resources
```
• setting up a Process Control Block (PCB) for the process
```
• process termination/deletion
Threads
21
• A thread is the primitive that can execute code.
• A thread is a single sequential flow of control within a process.
• 'Thread' is also known as light-weight process.
• A process can have many threads of execution.
```
• Different threads, which are part of a process, share the same address space;
```
meaning they share the data memory, code memory and heap memory area.
```
• Threads maintain their own thread status (CPU register values), Program
```
```
Counter (PC) and stack.
```
• The memory model for a process and its associated threads are given in the
figure.
```
Threads (continued)
```
Stack memory for Thread 1
Stack memory for ProcessStack memory for Thread 2
Data memory for Process
Code memory for Process
```
Fig: Memory organisation of a Process and its associated Threads
```
22
The Concept of Multithreading
23
• A process/task in embedded application may be a complex or
lengthy one and it may contain various suboperations like getting
input from I/O devices connected to the processor, performing
some internal calculations/operations, updating some I/O devices
etc.
• If all the subfunctions of a task are executed in sequence, the CPU
utilisation may not be efficient.
• For example, if the process is waiting for a user input, the CPU enters
the wait state for the event, and the process execution also enters a
wait state.
The Concept of Multithreading
```
(continued)
```
24
• Instead of this single sequential execution of the whole process, if the
task/process is split into different threads carrying out the different
subfunctionalities of the process, the CPU can be effectively utilised and when
the thread corresponding to the I/O operation enters the wait state, another
threads which do not require the I/O event for their operation can be switched
into execution.
• This leads to more speedy execution of the process and the efficient utilisation of
the processor time and resources.
• If the process is split into multiple threads, which executes a portion of the
process, there will be a main thread and rest of the threads will be created
within the main thread.
• The multithreaded architecture of a process can be better visualised with the
thread-process diagram, shown in the figure.
```
Fig: Process with multithreads
```
Code Memory
Data Memory
Stack Stack Stack
Registers Registers Registers
Thread 1 Thread 2 Thread 3
```
void main (void) int ChildThread1 int ChildThread2
```
```
{ (void) (void)
```
```
//create child { {
```
thread 1 //Do something //Do something
```
CreateThread (NULL,
```
```
1000,(LPTHREAD_START
```
```
_ROUTINE)ChildThread } }
```
1, NULL, 0,
```
&dwThreadID);
```
//create child
thread 2
```
CreateThread (NULL,
```
```
1000,(LPTHREAD_START
```
```
_ROUTINE)ChildThread
```
2, NULL, 0,
```
&dwThreadID);
```
```
}
```
25
Task/Process
The Concept of Multithreading
```
(continued)
```
26
• Use of multiple threads to execute a process brings the following
```
advantages:
```
• Better memory utilisation
• Multiple threads of the same process share the address space for data memory.
• This also reduces the complexity of inter thread communication since variables can be
shared across the threads.
• Speedy execution of the process
• Since the process is split into different threads, when one thread enters a wait state, the
CPU can be utilised by other threads of the process that do not require the event, which
the other thread is waiting, for processing.
• Efficient CPU utilisation
• The CPU is engaged all time.
Thread Standards
27
• Thread standards deal with the different standards available for
thread creation and management.
• These standards are utilised by the operating systems for thread
creation and thread management.
• It is a set of thread class libraries.
• The commonly available thread class libraries are:
• POSIX Threads
• Win32 Threads
• Java Threads
POSIX Threads
28
• POSIX stands for Portable Operating System Interface.
• The POSIX.4 standard deals with the Real-Time extensions and
POSIX.4a standard deals with thread extensions.
• The POSIX standard library for thread creation and management is
'Pthreads’.
• 'Pthreads' library defines the set of POSIX thread creation and
management functions in 'C' language.
```
POSIX Threads (continued)
```
29
• This primitive creates a new thread for running the function start_function.
• Here pthread_t is the handle to the newly created thread and pthread_attr_t is
the data type for holding the thread attributes.
• 'start_function' is the function the thread is going to execute and arguments is
the arguments for 'start_function’.
```
• On successful creation of a Pthread, pthread_create() associates the Thread
```
```
Control Block (TCB) corresponding to the newly created thread to the variable
```
```
of type pthread_t (new_thread_ID in our example).
```
```
int pthread_create(pthread_t *new_thread_ID, const pthread_attr_t, *attribute,
```
```
void * (*start_function) (void *), void *arguments);
```
```
POSIX Threads (continued)
```
30
• This primitive blocks the current thread and waits until the
```
completion of the thread pointed by it (new_thread in this example).
```
• All the POSIX 'thread calls' returns an integer.
• A return value of zero indicates the success of the call.
```
int pthread_join(pthread_t new_thread, void * *thread_status);
```
Thread Pre-emption
31
• Thread pre-emption is the act of pre-empting the currently running
thread.
• It means, stopping the currently running thread temporarily.
• Thread pre-emption is performed for sharing the CPU time among
all the threads.
• The execution switching among threads is known as 'Thread context
switching’.
• Thread context switching is dependent on the Operating system's
scheduler and the type of the thread.
Types of Threads
32
• User Level Threads
• User level threads do not have kernel/Operating System support and they exist
solely in the running process.
• Even if a process contains multiple user level threads, the OS treats it as single
thread and will not switch the execution among the different threads of it.
• It is the responsibility of the process to schedule each thread as and when required.
• In summary, user level threads of a process are non-preemptive at thread level
from OS perspective.
```
• The execution switching (thread context switching) happens only when the
```
currently executing user level thread is voluntarily blocked.
• Hence, no OS intervention and system calls are involved in the context switching of user level
threads.
• This makes context switching of user level threads very fast.
```
Types of Threads (continued)
```
33
• Kernel Level Threads
• Kernel level threads are individual units of execution, which the OS treats as
separate threads.
• The OS interrupts the execution of the currently running kernel thread and
switches the execution to another kernel thread based on the scheduling
policies implemented by the OS.
• In summary, kernel level threads are pre-emptive.
• Kernel level threads involve lots of kernel overhead and involve system calls
for context switching.
• However, kernel threads maintain a clear layer of abstraction and allow
threads to use system calls independently.
Thread Binding Models
34
• There are many ways for binding user level threads with system/kernel
level threads.
• Many-to-One Model
• Here, many user level threads are mapped to a single kernel thread.
• In this model, the kernel treats all user level threads as single thread and the
execution switching among the user level threads happens when a currently
executing user level thread voluntarily blocks itself or relinquishes the CPU.
• Solaris Green threads and GNU Portable Threads are examples for this.
• The 'PThread’ example is an illustrative example for application with Many-
to-One thread model.
```
Thread Binding Models (continued)
```
35
• One-to-One Model
• Here, each user level thread is bonded to a kernel/system level thread.
• Windows XP/NT/2000 and Linux threads are examples for One-to-One
thread models.
• The modified 'PThread' example is an illustrative example for application
with One-to-One thread model.
• Many-to-Many Model
• In this model, many user level threads are allowed to be mapped to many
kernel threads.
• Windows NT/2000 with ThreadFibre package is an example for this.
Thread Process
Thread is a single unit of execution and is part of process. Process is a program in execution and contains one or
more threads.
A thread does not have its own data memory and heap
memory. It shares the data memory and heap memory
with other threads of the same process.
Process has its own code memory, data memory and stack
memory.
```
A thread cannot live independently; it lives within the
```
process.
A process contains at least one thread.
There can be multiple threads in a process. The first
```
thread (main thread) calls the main function and occupies
```
the start of the stack memory of the process.
Threads within a process share the code, data and heap
memory. Each thread holds separate memory area for
```
stack (share the total stack memory of the process).
```
Threads are very inexpensive to create. Processes are very expensive to create. Involves many OS
overhead.
Context switching is inexpensive and fast. Context switching is complex and involves lot of OS
overhead and is comparatively slower.
If a thread expires, its stack is reclaimed by the process. If a process dies, the resources allocated to it are
reclaimed by the OS and all the associated threads of the
process also die.
36
Thread vs. ProcessTypes of Multitasking
The following section describes the various types of multitasking existing in the
Operating System’s context.
Co-operative Multitasking: Co-operative multitasking is the most primitive form of
multitasking in which a task/process gets a chance to execute only when the currently
executing task/process voluntarily relinquishes the CPU. In this method, any
task/process can hold the CPU as much time as it wants. Since this type of
implementation involves the mercy of the tasks each other for getting the CPU time for
execution, it is known as co-operative multitasking. If the currently executing task is non-
cooperative, the other tasks may have to wait for a long time to get the CPU.
37
Preemptive Multitasking
Pre emptive multitasking ensures that every task/process gets a chance to execute.
When and how much time a process gets is dependent on the implementation of the
preemptive scheduling. As the name indicates, in preemptive multitasking, the currently
running task/process is preempted to give a chance to other tasks/process to execute.
The preemption of task may be based on time slots or task/process priority
38
Non-preemptive Multitasking
In non-preemptive multitasking, the process/task, which is currently given the CPU time,
```
is allowed to execute until it terminates (enters the ‘Completed’ state) or enters the
```
‘Blocked/Wait’ state, waiting for an I/O or system resource. The co-operative and non-
preemptive multitasking differs in their behaviour when they are in the ‘Blocked/Wait’
state. In co-operative multitasking, the currently executing process/task need not
relinquish the CPU when it enters the ‘Blocked/Wait’ state, waiting for an I/O, or a
shared resource access or an event to occur whereas in non-preemptive multitasking the
currently executing task relinquishes the CPU when it waits for an I/O or system
resource or an event to occur.
39
Task Scheduling
40
Task Scheduling
41
• Multitasking involves the execution switching among the different tasks.
• There should be some mechanism in place to share the CPU among the different tasks
and to decide which process/task is to be executed at a given point of time.
• Determining which task/process is to be executed at a given point of time is known as
task/process scheduling.
• Scheduling policies forms the guidelines for determining which task is to be executed
when.
• The scheduling policies are implemented in an algorithm and it is run by the kernel as
a service.
• The kernel service/application, which implements the scheduling algorithm, is known
as 'Scheduler'.
Task Scheduling
42
• Based on the scheduling algorithm used, scheduling can be
classified into:
• Non-preemptive Scheduling
• The currently executing task/process is allowed to run until it terminates or
enters the ‘Wait’ state waiting for an I/O or system resource.
• Preemptive Scheduling
```
• The currently executing task/process is preempted (stopped temporarily)
```
and another task from the Ready queue is selected for execution.
```
Task Scheduling (continued)
```
43
• The process scheduling decision may take place when a process
switches its state to
1. 'Ready' state from 'Running' state
2. 'Blocked/Wait' state from 'Running' state
3. 'Ready' state from 'Blocked/Wait' state
4. 'Completed' state
• A process switches to 'Ready' state from the 'Running' state when it
is preempted.
• Hence, the type of scheduling in scenario 1 is pre-emptive.
```
Task Scheduling (continued)
```
44
• When a high priority process in the 'Blocked/Wait' state completes its I/O and
switches to the 'Ready' state, the scheduler picks it for execution if the
scheduling policy used is priority based preemptive.
• This is indicated by scenario 3.
• In preemptive/non-preemptive multitasking, the process relinquishes the CPU
when it enters the ‘Blocked/Wait' state or the 'Completed' state and switching
of the CPU happens at this stage.
• Scheduling under scenario 2 can be either preemptive or non-preemptive.
• Scheduling under scenario 4 can be preemptive, non-preemptive or co-
operative.
```
Task Scheduling (continued)
```
45
• The selection of a scheduling criterion/algorithm should consider the
following factors:
• CPU Utilisation:
• The scheduling algorithm should always make the CPU utilisation high.
• CPU utilisation is a direct measure of how much percentage of the CPU is being utilised.
• Throughput:
• This gives an indication of the number of processes executed per unit of time.
• The throughput for a good scheduler should always be higher.
```
• Turnaround Time (TAT):
```
• It is the amount of time taken by a process for completing its execution.
• It includes the time spent by the process for waiting for the main memory, time spent in the
ready queue, time spent on completing the I/O operations, and the time spent in execution.
• The turnaround time should be minimal for a good scheduling algorithm.
```
Task Scheduling (continued)
```
46
• Waiting Time:
• It is the amount of time spent by a process in the 'Ready' queue waiting to get the CPU
time for execution.
• The waiting time should be minimal for a good scheduling algorithm.
• Response Time:
• It is the time elapsed between the submission of a process and the first response.
• For a good scheduling algorithm, the response time should be as least as possible.
To summarise, a good scheduling algorithm has high CPU utilisation, minimum Turn Around
```
Time (TAT), maximum throughput and least response time.
```
```
Task Scheduling (continued)
```
47
• The various queues maintained by OS in association with CPU
scheduling are:
• Job Queue:
• Contains all the processes in the system.
• Ready Queue:
• Contains all the processes, which are ready for execution and waiting for CPU to
get their turn for execution.
• The Ready queue is empty when there is no process ready for running.
• Device Queue:
• Contains the set of processes, which are waiting for an I/O device.
Preemptive Scheduling
48
```
• In preemptive scheduling, the scheduler can preempt (stop temporarily) the
```
currently executing task/process and select another task from the 'Ready'
queue for execution.
• Every task in the 'Ready' queue gets a chance to execute.
• When to pre-empt a task and which task is to be picked up from the 'Ready'
queue for execution after preempting the current task is purely dependent on
the scheduling algorithm.
• A task which is preempted by the scheduler is moved to the 'Ready' queue.
• The act of moving a 'Running' process/task into the 'Ready' queue by the
scheduler, without the processes requesting for it is known as ‘Preemption’
Preemptive Scheduling Techniques
49
• Preemptive scheduling can be implemented in different
approaches.
• Time-based preemption
• Priority-based preemption
• The various types of preemptive scheduling adopted in
task/process scheduling are:
```
• Preemptive Shortest Job First (SJF)/Shortest Remaining Time (SRT)
```
Scheduling
```
• Round Robin (RR) Scheduling
```
• Priority Based Scheduling
```
Preemptive Shortest Job First (SJF)/Shortest
```
```
Remaining Time (SRT) Scheduling
```
50
• In SJF, the process with the shortest estimated run time is scheduled first, followed by
the next shortest process, and so on.
• The preemptive SJF scheduling algorithm sorts the 'Ready' queue when a new process
enters the 'Ready' queue and checks whether the execution time of the new process
is shorter than the remaining of the total estimated time for the currently executing
process.
• If the execution time of the new process is less, the currently executing process is
preempted and the new process is scheduled for execution.
```
• Thus preemptive SJF scheduling always compares the execution completion time (It is
```
```
same as the remaining time for the new process) of a new process entered the 'Ready'
```
queue with the remaining time for completion of the currently executing process and
schedules the process with shortest remaining time for execution.
```
• Preemptive SJF scheduling is also known as Shortest Remaining Time (SRT) scheduling .
```
Preemptive SJF/SRT Scheduling - Example
51
• Three processes with process IDs P1, P2, P3 with estimated completion time
10, 5, 7 milliseconds respectively enter the ready queue together. A new
process P4 with estimated completion time 2 ms enters the 'Ready' queue
after 2 ms. Assume all the processes contain only CPU operation and no I/O
```
operations are involved. Calculate the waiting time and Turn Around Time (TAT)
```
for each process and the average waiting time and Turn Around Time in the
SRT scheduling.
Preemptive SJF/SRT Scheduling –
```
Example (continued)
```
P1
P2
P3
P2 is scheduled
‘Ready’ queue at 0 ms
Remaining
Time
10 ms
5 ms
7 ms
Process ID
P1
P2
P3
P4
‘Ready’ queue at 2 ms
P2 is preempted
P4 is scheduled
Remaining
Time
10 ms
Process ID
3 ms
7 ms
2 ms
P1
P2
P3
‘Ready’ queue at 4 ms
P4 is completed
P2 is scheduled
Remaining
Time
10 ms
Process ID
3 ms
7 ms
P1
P3
‘Ready’ queue at 7 ms
P2 is completed
P3 is scheduled
Remaining
Time
10 ms
Process ID
7 ms
P1
‘Ready’ queue at 14 ms
P3 is completed
P1 is scheduled
Remaining
Time
10 ms
Process ID
52
Preemptive SJF/SRT Scheduling –
```
Example (continued)
```
• The execution sequence can be written as below:
P2 P4 P2 P3 P1
```
Time (ms)
```
53
Preemptive SJF/SRT Scheduling –
```
Example (continued)
```
• The waiting time for all the processes are given as
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃2 = 0 𝑚𝑠 +
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃4 = 0 𝑚𝑠
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃3 = 7 𝑚𝑠
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃1 = 14 𝑚𝑠
4 − 2 𝑚𝑠 = 2 𝑚𝑠
•
𝐴𝑣𝑒𝑟𝑎𝑔𝑒 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 =
𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑎𝑙𝑙 𝑡ℎ𝑒 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
𝑁𝑢𝑚𝑏𝑒𝑟 𝑜𝑓 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
=
2+0+7+14
𝑚𝑠 =
23
𝑚𝑠
4 4
= 5.75 𝑚𝑠
54
Preemptive SJF/SRT Scheduling –
```
Example (continued)
```
```
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 (𝑇𝐴𝑇) = 𝑇𝑖𝑚𝑒 𝑠𝑝𝑒𝑛𝑡 𝑖𝑛 𝑟𝑒𝑎𝑑𝑦 𝑞𝑢𝑒𝑢𝑒 + 𝐸𝑥𝑒𝑐𝑢𝑡𝑖𝑜𝑛 𝑇𝑖𝑚𝑒
```
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃2 = 2 𝑚𝑠 + 5 𝑚𝑠 = 7 𝑚𝑠
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃4 = 0 𝑚𝑠 + 2 𝑚𝑠 = 2 𝑚𝑠
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃3 = 7 𝑚𝑠 + 7 𝑚𝑠 = 14 𝑚𝑠
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃1 = 14 𝑚𝑠 + 10 𝑚𝑠 = 24 𝑚𝑠
```
• 𝐴𝑣𝑒𝑟𝑎𝑔𝑒 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 (𝑇𝐴𝑇) =
```
𝑇𝐴𝑇 𝑓𝑜𝑟 𝑎𝑙𝑙 𝑡ℎ𝑒 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
𝑁𝑢𝑚𝑏𝑒𝑟 𝑜𝑓 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
=
7+2+14+24
𝑚𝑠 =
47
𝑚𝑠
4 4
= 11.75 𝑚𝑠
55
```
Round Robin (RR) Scheduling
```
56
• In Round Robin scheduling, each process in the 'Ready' queue is executed for a
pre-defined time slot.
• 'Round Robin' brings the message "Equal chance to all".
• The execution starts with picking up the first process in the 'Ready' queue.
• It is executed for a pre-defined time and when the pre-defined time elapses or
```
the process completes (before the pre-defined time slice), the next process in
```
the 'Ready' queue is selected for execution.
• This is repeated for all the processes in the 'Ready' queue.
• Once each process in the 'Ready' queue is executed for the pre-defined time
period, the scheduler comes back and picks the first process in the 'Ready'
queue again for execution.
• The sequence is repeated.
```
Round Robin (RR) Scheduling (continued)
```
• The 'Ready' queue can be
considered as a circular queue
in which the scheduler picks up
the first process for execution
and moves to the next till the
end of the queue and then
comes back to the beginning of
the queue to pick up the first
process.
Process 4
Process 1
Process 3
Process 2
Execution Switch Execution Switch
Execution Switch Execution Switch
57
```
Round Robin (RR) Scheduling (continued)
```
58
• The time slice is provided by the timer tick feature of the time
management unit of the OS kernel.
• Time slice is kernel dependent and it varies in the order of a few
microseconds to milliseconds.
• Round Robin scheduling ensures that every process gets a fixed amount
of-CPU time for execution.
• When the process gets its fixed time for execution is determined by the
```
First Come First Serve (FCFS) policy.
```
• If a process terminates before the elapse of the time slice, the process
releases the CPU voluntarily and the next process in the queue is
scheduled for execution by the scheduler.
```
Round Robin (RR) Scheduling - Example
```
59
• Three processes with process IDs P1, P2, P3 with estimated completion time 6,
4, 2 milliseconds respectively, enter the ready queue together in the order P1,
```
P2, P3. Calculate the waiting time and Turn Around Time (TAT) for each process
```
```
and the Average waiting time and Turn Around Time (Assuming there is no I/O
```
```
waiting for the processes) in RR algorithm with Time slice = 2 ms.
```
P1
P2
P3
P1 is scheduled
‘Ready’ queue at 0 ms
Remaining
Time
6 ms
4 ms
2 ms
Process ID
P2
P3
P1
‘Ready’ queue at 2 ms
P1 is preempted
P2 is scheduled
Remaining
Time
4 ms
Process ID
2 ms
4 ms
P3
P1
P2
‘Ready’ queue at 4 ms
P2 is preempted
P3 is scheduled
Remaining
Time
2 ms
Process ID
4 ms
2 ms
P1
P2
‘Ready’ queue at 6 ms
P3 is completed
P1 is scheduled
Remaining
Time
4 ms
Process ID
2 ms
‘Ready’ queue at 8 ms
P1 is preempted
P2 is scheduled
P2
P1
Remaining
Time
2 ms
Process ID
2 ms
P1
60
‘Ready’ queue at 10 ms
P2 is completed
P1 is scheduled
Remaining
Time
2 ms
Process ID
```
Round Robin (RR) Scheduling– Example
```
```
(continued)
```
• The execution sequence can be written as below:
P1 P2 P3 P1 P2 P1
```
Time (ms)
```
61
```
Round Robin (RR) Scheduling – Example
```
```
(continued)
```
• The waiting time for all the processes are given as
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃1 = 0 𝑚𝑠 + 6 − 2 𝑚𝑠 + 10 − 8 𝑚𝑠 = 6 𝑚𝑠
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃2 = 2 − 0 𝑚𝑠 + 8 − 4 𝑚𝑠 = 6 𝑚𝑠
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃3 = 4 − 0 = 4 𝑚𝑠
•
𝐴𝑣𝑒𝑟𝑎𝑔𝑒 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 =
𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑎𝑙𝑙 𝑡ℎ𝑒 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
𝑁𝑢𝑚𝑏𝑒𝑟 𝑜𝑓 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
=
6+6+4
𝑚𝑠 =
16
𝑚𝑠
3 3
= 5.33 𝑚𝑠
62
```
Round Robin (RR) Scheduling – Example
```
```
(continued)
```
```
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 (𝑇𝐴𝑇) = 𝑇𝑖𝑚𝑒 𝑠𝑝𝑒𝑛𝑡 𝑖𝑛 𝑟𝑒𝑎𝑑𝑦 𝑞𝑢𝑒𝑢𝑒 + 𝐸𝑥𝑒𝑐𝑢𝑡𝑖𝑜𝑛 𝑇𝑖𝑚𝑒
```
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃1 = 6 𝑚𝑠 + 6 𝑚𝑠 = 12 𝑚𝑠
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃2 = 6 𝑚𝑠 + 4 𝑚𝑠 = 10 𝑚𝑠
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃3 = 4 𝑚𝑠 + 2 𝑚𝑠 = 6 𝑚𝑠
```
• 𝐴𝑣𝑒𝑟𝑎𝑔𝑒 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 (𝑇𝐴𝑇) =
```
𝑇𝐴𝑇 𝑓𝑜𝑟 𝑎𝑙𝑙 𝑡ℎ𝑒 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
𝑁𝑢𝑚𝑏𝑒𝑟 𝑜𝑓 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
=
12+10+6
𝑚𝑠 =
28
𝑚𝑠
3 3
= 9.33 𝑚𝑠
63
Priority Based Scheduling
64
• The Priority Based Preemptive Scheduling ensures that a process with high priority is
serviced at the earliest compared to other low priority processes in the ‘Ready’ queue.
• Any high priority process entering the 'Ready' queue is immediately scheduled for
execution.
• The priority of a task/process can be indicated through various mechanisms.
• While creating the process/task, the priority can be assigned to it.
• The priority number associated with a task/process is the direct indication of its
priority.
• The priority number 0 indicates the highest priority.
• This convention need not be universal and it depends on the kernel level
implementation of the priority structure.
• Whenever a new process enters the ‘Ready’ queue, the scheduler sorts the 'Ready'
queue based on priority and picks the process with the highest level of priority for
execution.
Priority Based Scheduling - Example
65
• Three processes with process IDs P1, P2, P3 with estimated completion time
```
10, 5, 7 milliseconds and priorities 1, 3, 2 (0 – highest priority, 3 - lowest
```
```
priority) respectively enter the ready queue together. A new process P4 with
```
estimated completion time 6 ms and priority 0 enters the 'Ready' queue after 5
ms of start of execution of P1. Calculate the waiting time and Turn Around
```
Time (TAT) for each process and the Average waiting time and Turn Around
```
```
Time (Assuming there is no I/O waiting for the processes) in priority based
```
scheduling algorithm.
P1
P2
P3
P1 is scheduled
P1 is preempted
P4 is scheduled
P4 is completed
P1 is scheduled
‘Ready’ queue at 16 ms
P1 is completed
P3 is scheduled
‘Ready’ queue at 23 ms
P3 is completed
P2 is scheduled
‘Ready’ queue at 0 ms
66
Remaining
Time
10 ms
5 ms
7 ms
Process IDPriority
1
3
2
P1
P2
P3
‘Ready’ queue at 11 ms
Remaining
Time
5 ms
Process ID
5 ms
7 ms
Priority
1
3
2
P2
P3
Remaining
Time
5 ms
Process ID
7 ms
Priority
3
2
P2
Remaining
Time
5 ms
Process IDPriority
3
P1
P2
P3
P4
‘Ready’ queue at 5 ms
Remaining
Time
5 ms
Process ID
5 ms
7 ms
Priority
1
3
2
0 6 ms
Priority Based Scheduling– Example
```
(continued)
```
• The execution sequence can be written as below:
P1 P4 P1 P3 P2
```
Time (ms)
```
67
Priority Based Scheduling – Example
```
(continued)
```
• The waiting time for all the processes are given as
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃1 = 0 𝑚𝑠 +
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃4 = 0 𝑚𝑠
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃3 = 16 𝑚𝑠
• 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑃2 = 23 𝑚𝑠
11 − 5 𝑚𝑠 = 6 𝑚𝑠
•
𝐴𝑣𝑒𝑟𝑎𝑔𝑒 𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 =
𝑊𝑎𝑖𝑡𝑖𝑛𝑔 𝑡𝑖𝑚𝑒 𝑓𝑜𝑟 𝑎𝑙𝑙 𝑡ℎ𝑒 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
𝑁𝑢𝑚𝑏𝑒𝑟 𝑜𝑓 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
=
6+0+16+23
𝑚𝑠 =
45
𝑚𝑠
4 4
= 11.25 𝑚𝑠
68
Priority Based Scheduling – Example
```
(continued)
```
```
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 (𝑇𝐴𝑇) = 𝑇𝑖𝑚𝑒 𝑠𝑝𝑒𝑛𝑡 𝑖𝑛 𝑟𝑒𝑎𝑑𝑦 𝑞𝑢𝑒𝑢𝑒 + 𝐸𝑥𝑒𝑐𝑢𝑡𝑖𝑜𝑛 𝑇𝑖𝑚𝑒
```
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃1 = 6 𝑚𝑠 + 10 𝑚𝑠 = 16 𝑚𝑠
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃4 = 0 𝑚𝑠 + 6 𝑚𝑠 = 6 𝑚𝑠
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃3 = 16 𝑚𝑠 + 7 𝑚𝑠 = 23 𝑚𝑠
• 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 𝑇𝐴𝑇 𝑓𝑜𝑟 𝑃2 = 23 𝑚𝑠 + 5 𝑚𝑠 = 28 𝑚𝑠
```
• 𝐴𝑣𝑒𝑟𝑎𝑔𝑒 𝑇𝑢𝑟𝑛 𝐴𝑟𝑜𝑢𝑛𝑑 𝑇𝑖𝑚𝑒 (𝑇𝐴𝑇) =
```
𝑇𝐴𝑇 𝑓𝑜𝑟 𝑎𝑙𝑙 𝑡ℎ𝑒 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
𝑁𝑢𝑚𝑏𝑒𝑟 𝑜𝑓 𝑝𝑟𝑜𝑐𝑒𝑠𝑠𝑒𝑠
=
16+6+23+28
𝑚𝑠 =
73
𝑚𝑠
4 4
= 18.25 𝑚𝑠
69
Task Communication
70
Task Communication
71
• In a multitasking system, multiple tasks/processes run concurrently
```
(in pseudo parallelism) and each process may or may not interact
```
between.
• Based on the degree of interaction, the processes running on an OS
are classified as
• Co-operating Processes:
• One process requires the inputs from other processes to complete its execution.
• Competing Processes:
• The competing processes do not share anything among themselves but they share the
system resources.
• The competing processes compete for the system resources such as file, display device, etc.
```
Task Communication (continued)
```
72
• Co-operating processes exchanges information and communicate
through the following methods:
• Co-operation through Sharing:
• The co-operating process exchange data through some shared resources.
• Co-operation through Communication:
• No data is shared between the processes.
• But they communicate for synchronisation.
```
Task Communication (continued)
```
73
• The mechanism through which processes/tasks communicate each other
```
is known as Inter Process/Task Communication (IPC).
```
• Inter Process Communication is essential for process co-ordination.
```
• The various types of Inter Process Communication (IPC) mechanisms
```
```
adopted by process are kernel (Operating System) dependent.
```
• Some of the important IPC mechanisms adopted by various kernels are:
• Shared Memory
• Pipes and Memory Mapped Objects
• Message Passing
• Message Queue, Mailbox and Signalling
• Remote Procedure Call and Sockets
Shared Memory
74
• Processes share some area of the
memory to communicate among them.
• Information to be communicated by the
process is written to the shared memory
area.
• Other processes which require this
information can read the same from the
shared memory area.
• Different mechanisms are adopted by different kernels for implementing the concept
of shared memory:
• Pipes
• Memory Mapped Objects
Process 1
Shared
memory area
Process 2
Fig.: Concept of Shared Memory
Pipes
75
• 'Pipe' is a section of the shared memory used by processes for
communicating.
• Pipes follow the client-server architecture.
• A process which creates a pipe is known as a pipe server and a process
which connects to a pipe is known as pipe client.
• A pipe can be considered as a conduit for information flow and has
two conceptual ends.
• It can be unidirectional, allowing information flow in one direction
or bidirectional allowing bidirectional information flow.
```
Pipes (continued)
```
• A unidirectional pipe allows the process connecting at one end of
the pipe to write to the pipe and the process connected at the
other end of the pipe to read the data, whereas a bidirectional pipe
allows both reading and writing at one end.
• The figure shows a unidirectional pipe.
Pipe
```
(Named/unnamed)
```
Fig.: Concept of Pipe for IPC
Process 2
Read
Process 1
Write
76
```
Pipes (continued)
```
77
• The implementation of 'Pipes' is OS dependent.
• Microsoft Windows supports two types of 'Pipes' for Inter Process Communication:
• Anonymous Pipes:
• The anonymous pipes are unnamed, unidirectional pipes used for data transfer between two processes.
• Named Pipes:
• Named pipe is a named, unidirectional or bi-directional pipe for data exchange between processes.
• Like anonymous pipes, the process which creates the named pipe is known as pipe server and a process
which connects to the named pipe is known as pipe client.
• With named pipes, any process can act as both client and server allowing point-to-point communication.
• Named pipes can be used for communicating between processes running on the same machine or
between processes running on different machines connected to a network.
Memory Mapped Objects
78
• Memory mapped object is a shared memory technique adopted by certain
Real-Time Operating Systems for allocating a shared block of memory which
can be accessed by multiple process simultaneously.
• In this approach, a mapping object is created and physical storage for it is
reserved and committed.
• A process can map the entire committed physical area or a block of it to its
virtual address space.
• All read and write operation to this virtual address space by a process is
directed to its committed physical area.
• Any process which wants to share data with other processes can map the
physical memory area of the mapped object to its virtual memory space and
use it for sharing the data.
Message Passing
79
```
• Message passing is an (a)synchronous information exchange mechanism used
```
for Inter Process/Thread Communication.
• The major difference between shared memory and message passing technique
is that, through shared memory lots of data can be shared whereas only
limited amount of information/data is passed through message passing.
• Also, message passing is relatively fast and free from the synchronisation overheads
compared to shared memory.
• Based on the message passing operation between the processes, message
passing is classified into:
• Message Queue
• Mailbox
• Signalling
Message Queue
80
```
• 'Message queue’ is a First-In-First-Out (FIFO) queue which stores the messages
```
temporarily in a system defined memory object to pass it to the desired
process.
• Usually the process which wants to talk to another process posts the message
to a message queue.
• Messages are sent and received through send and receive methods.
```
• send (Name of the process to which the message is to be sent, message)
```
```
• receive (Name of the process from which the message is to be received, message)
```
• The implementation of the message queue, send and receive methods are OS
kernel dependent.
```
Message Queue (continued)
```
Message Queue
Process 1 Process 2
Fig.: Concept of Message Queue based indirect messaging for IPC
81
```
Message Queue (continued)
```
82
• The Windows XP OS kernel maintains a single system message queue and one process/thread
specific message queue.
• A thread which wants to communicate with another thread posts the message to the system
message queue.
• The kernel picks up the message from the system message queue one at a time and examines
the message for finding the destination thread and then posts the message to the message
queue of the corresponding thread.
• The messaging mechanism is classified into synchronous and asynchronous based on the
behaviour of the message posting thread.
• In asynchronous messaging, the message posting thread just posts the message to the queue
```
and it will not wait for an acceptance (return) from the thread to which the message is posted.
```
• In synchronous messaging, the thread which posts a message enters waiting state and waits for
the message result from the thread to which the message is posted.
• The thread which invoked the send message becomes blocked and the scheduler will not pick it up for scheduling.
Mailbox
83
• Mailbox is an alternate form of ‘Message queue’ and it is used in RTOS for IPC
usually for one way messaging.
• The task/thread which wants to send a message to other tasks/threads creates
a mailbox for posting the messages.
• The threads which are interested in receiving the messages posted to the
mailbox by the mailbox creator thread can subscribe to the mailbox.
• The thread which creates the mailbox is known as 'mailbox server' and the
threads which subscribe to the mailbox are known as 'mailbox clients’.
• The mailbox server posts messages to the mailbox and notifies it to the clients
which are subscribed to the mailbox.
• The clients read the message from the mailbox on receiving the notification.
```
Mailbox (continued)
```
Mailbox
Task 2 Task 3 Task 4
Task 1
Post message
Broadcast
message
Broadcast
messageBroadcast
message
84
Fig.: Concept of
Mailbox based
indirect messaging
for IPC
```
Mailbox (continued)
```
85
• The mailbox creation, subscription, message reading and writing
are achieved through OS kernel provided API calls.
• Mailbox and message queues are same in functionality.
• The only difference is in the number of messages supported by them.
```
• Both of them are used for passing data in the form of message(s) from a task
```
```
to another task(s).
```
• Mailbox is used for exchanging a single message between two tasks or
```
between an Interrupt Service Routine (ISR) and a task.
```
• Mailbox associates a pointer pointing to the mailbox and a wait list to hold
the tasks waiting for a message to appear in the mailbox.
Signalling
86
• Signalling is a primitive way of communication between
processes/threads.
• Signals are used for asynchronous notifications where one
process/thread fires a signal, indicating the occurrence of a
```
scenario which the other process(es)/thread(s) is waiting.
```
• Signals are not queued and they do not carry any data.
• E.g. Communication mechanisms used in RTX51 Tiny OS, inter
process communication in VxWorks OS Kernel are examples for
signalling.
```
Remote Procedure Call (RPC) and Sockets
```
87
```
• Remote Procedure Call (RPC) is the Inter Process Communication (IPC)
```
mechanism used by a process to call a procedure of another process running
on the same CPU or on a different CPU which is interconnected in a network.
• In the object oriented language terminology, RPC is also known as Remote
```
Invocation or Remote Method Invocation (RMI).
```
• RPC is mainly used for distributed applications like client-server applications.
```
• With RPC it is possible to communicate over a heterogeneous network (i.e.
```
Network where Client and server applications are running on different operating
```
systems).
```
• The CPU/process containing the procedure which needs to be invoked remotely is
known as server.
• The CPU/process which initiates an RPC request is known as client.
```
Remote Procedure Call (RPC) and Sockets
```
```
(continued)
```
CPU CPU
Process Process
Procedure
Network
TCP/IP or UDP
over Socket
Processes running on different CPUs
which are networked
Process 1
Procedure
CPU
Process 2
TCP/IP or UDP
over Socket
88
Processes running on the same CPU
```
Fig.: Concept of Remote Procedure Call (RPC) for IPC
```
```
Remote Procedure Call (RPC) and Sockets
```
```
(continued)
```
89
• It is possible to implement RPC communication with different invocation
interfaces.
```
• Interface Definition Language (IDL) defines the interfaces for RPC.
```
```
• Microsoft Interface Definition Language (MIDL) is the IDL implementation from
```
Microsoft for all Microsoft platforms.
```
• The RPC communication can be either Synchronous (Blocking) or Asynchronous
```
```
(Non-blocking).
```
• In the Synchronous communication, the process which calls the remote procedure
is blocked until it receives a response back from the other process.
• In asynchronous RPC calls, the calling process continues its execution while the
remote process performs the execution of the procedure.
• The result from the remote procedure is returned back to the caller through mechanisms like
callback functions.
```
Remote Procedure Call (RPC) and Sockets
```
```
(continued)
```
90
• On security front, RPC employs authentication mechanisms to
protect the systems against vulnerabilities.
```
• The client applications (processes) should authenticate themselves
```
with the server for getting access.
```
• Authentication mechanisms like IDs, public key cryptography (like
```
```
DES, 3DES), etc. are used by the client for authentication.
```
• Without authentication, any client can access the remote
procedure.
• This may lead to potential security risks.
```
Remote Procedure Call (RPC) and Sockets
```
```
(continued)
```
91
• Sockets are used for RPC communication.
• Socket is a logical endpoint in a two-way communication link between
two applications running on a network.
• A port number is associated with a socket so that the network layer of
the communication channel can deliver the data to the designated
application.
```
• Sockets are of different types, namely, Internet sockets (INET), UNIX
```
sockets, etc.
• The INET socket works on internet communication protocol.
• TCP/IP, UDP, etc. are the communication protocols used by INET sockets.
```
Remote Procedure Call (RPC) and Sockets
```
```
(continued)
```
92
• INET sockets are classified into:
• Stream sockets
• These are connection oriented and they use TCP to establish a reliable
connection.
• Datagram sockets
• These rely on UDP for establishing a connection.
• The UDP connection is unreliable when compared to TCP.
```
Remote Procedure Call (RPC) and Sockets
```
```
(continued)
```
93
• The client-server communication model uses a socket at the client side and a
socket at the server side.
• A port number is assigned to both of these sockets.
• The client and server should be aware of the port number associated with the
socket.
• In order to start the communication, the client needs to send a connection
request to the server at the specified port number.
• The client should be aware of the name of the server along with its port
number.
• The server always listens to the specified port number on the network.
• Upon receiving a connection request from the client, based on the success of
authentication, the server grants the connection request and a communication
channel is established between the client and server.
```
Remote Procedure Call (RPC) and Sockets
```
```
(continued)
```
94
• The client uses the host name and port number of server for sending
requests and server uses the client's name and port number for sending
responses.
```
• If the client and server applications (both processes) are running on the
```
same CPU, both can use the same host name and port number for
communication.
• The physical communication link between the client and server uses
network interfaces like Ethernet or Wi-Fi for data communication.
• The underlying implementation of socket is OS kernel dependent.
• Different types of OSs provide different socket interfaces.
THE END
95