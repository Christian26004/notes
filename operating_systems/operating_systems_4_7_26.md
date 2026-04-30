# Operating Systems - 4/7/26

## 1\. Thread Lifecycle and Management

### Execution and Exit

- **Return from Function:** A thread typically terminates when its entry function returns.
- **Explicit Exit:** A thread can also exit by calling `pthread_exit()`.
- **Process Impact:** A process terminates if: - `exit()` is called directly.
    - One of its threads calls `exit()`.
    - The `main` thread returns.
    - It receives a termination signal.

- **Cleanup:** When a process terminates, **all** of its threads are immediately terminated by the kernel.

### Detached Threads

- **Detached State:** A detached thread cannot be joined by other threads.
- **Automatic Resource Release:** Unlike joinable threads, a detached thread's resources (like its stack and thread descriptor) are automatically reclaimed by the system as soon as it exits.
- **Self-Detachment:** A thread can detach itself using `pthread_detach(pthread_self())`.
- *Analogy:* Think of a detached thread like a "fire-and-forget" task—like a waiter who delivers a plate and moves on without needing to check back in with the head chef for a final report.

## 2\. Thread Cancellation

### Cancellation States

- **Enabled (Default):** `PTHREAD_CANCEL_ENABLE`. The thread can be cancelled.
- **Disabled:** `PTHREAD_CANCEL_DISABLE`. Cancellation requests are held until it is re-enabled.
- **Function:** Use `pthread_setcancelstate(int state, int *oldstate)` to toggle this.

### Cancellation Types

- **Deferred (Default):** Cancellation only occurs when the thread reaches a "cancellation point" (usually a blocking system call like `read()` or `sleep()`).
- **Asynchronous:** The thread is terminated immediately upon request, regardless of its current execution point. *Warning:* This is dangerous if the thread holds locks or is mid-update of shared data.

## 3\. Thread Attributes and Scheduling

### Thread Scope

- **Process Scope (`PTHREAD_SCOPE_PROCESS`):** Threads compete for CPU time within the same process (User-level scheduling).
- **System Scope (`PTHREAD_SCOPE_SYSTEM`):** Threads compete with all other threads on the entire system (Kernel-level scheduling). In modern Linux (NPTL), system scope is the default.

### Scheduling Policies

- **Real-time Policies:**- `SCHED_FIFO` (First-In-First-Out)
    - `SCHED_RR` (Round Robin)

- *Note:* While theoretical models like Shortest Job First (SJF) exist, standard POSIX threads primarily use FIFO and Round Robin for real-time priorities.

## 4\. The Thread Memory Model: What is Shared?

Each thread has its own **Thread Context** (ID, Stack, Registers, Program Counter), but all threads within a process share the **Process Context**.

| Shared (Process Context)          | Private (Thread Context)          |
| --------------------------------- | --------------------------------- |
| Code segments                     | Thread ID                         |
| Data segments (Global variables)  | Registers (Program Counter, etc.) |
| Heap memory                       | Stack (Local variables)           |
| Shared library segments           |                                   |
| Open files / File descriptors     |                                   |
| Signal handlers                   |                                   |

> **Teacher's Probing Question:** If threads can technically access each other's memory (since they share the same address space), why is it so important for them to have their own private stacks?

## 5\. The Necessity of Synchronization

### The Race Condition

When multiple threads access and modify shared data concurrently, the outcome depends on the specific order of execution (the "race").

**Example: The Bank Balance**

- Initial `Balance = 200`
- Thread A: `Balance += 100`
- Thread B: `Balance -= 200`

Without synchronization (like Mutexes or Semaphores), if a context switch happens in the middle of the "Read-Modify-Write" cycle, the final balance could incorrectly be `0`, or `300` instead of the expected `100`.

*Conceptual Depth:* We need synchronization to ensure **Atomicity**—making sure that a multi-step operation appears to happen all at once or not at all.

