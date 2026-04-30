# Operating Systems - 4/9/26

## 1\. Introduction to Synchronization

**Synchronization** is the coordination of processes or threads to ensure they work together correctly when accessing shared resources. In a **time-shared system**, the exact order of instruction execution is non-deterministic (cannot be predicted).

### The Need for Serialization

To prevent data inconsistency, we must **serialize** specific execution scenarios.

- **Performance Trade-off:** We cannot serialize every statement because it significantly degrades performance.

- **Granularity:** Serialization requires system resources, so it must be applied strategically to "critical" code sections.

### Race Conditions: The Balance Example

Consider a shared variable `balance = 100`. Two threads attempt to update it:

1. **Thread A:** `balance += 200`

2. **Thread B:** `balance -= 100`

If not synchronized, the final `balance` might be `300`, `0`, or the correct `200`, depending on which thread "wins" the final write to memory. This is a **Race Condition**.

> **Teacher's Probing Question:** Why does the final value depend on the order of register writes rather than just the high-level code?

### Atomicity

To solve synchronization issues, high-level code sections must be executed **atomically**.

- **Atomic Operation:** An operation that completes in its entirety without interruption.

- **Single Processor:** Often achieved by disabling interrupts or using time-slicing algorithms like **Round Robin (RR)** or **First-Come, First-Served (FCFS)**.

- **Parallel/Distributed Systems:** Requires hardware support for truly simultaneous atomic operations (e.g., Test-and-Set).

---

## 2\. The Producer-Consumer Problem

A classic example of synchronization involving a **Producer** (generates data) and a **Consumer** (uses data) sharing a **Circular Buffer**.

### Shared Variables

```c
item buffer[n];
int in = 0;      // Next free position
int out = 0;     // Next full position
int counter = 0; // Number of items in buffer
```

### Logic

| Producer Loop                    | Consumer Loop                    |
| -------------------------------- | -------------------------------- |
| `while (counter == n) ; // Wait` | `while (counter == 0) ; // Wait` |
| `buffer[in] = next_produced;`    | `next_consumed = buffer[out];`   |
| `in = (in + 1) % n;`             | `out = (out + 1) % n;`           |
| `counter++;`                     | `counter--;`                     |

### The "Hidden" Race Condition

The statements `counter++` and `counter--` are not atomic at the machine level. They are translated into multiple register operations:

**Producer (`counter++`):**

1. `reg1 = counter`

2. `reg1 = reg1 + 1`

3. `counter = reg1`

**Consumer (`counter--`):**

1. `reg2 = counter`

2. `reg2 = reg2 - 1`

3. `counter = reg2`

If these interleave (e.g., the producer is interrupted after step 2), the final value of `counter` will be incorrect, potentially leading to buffer overflows or underflows.

---

## 3\. The Critical Section Problem

A **Critical Section (CS)** is a segment of code where a process accesses and modifies shared data.

### The General Process Protocol

In synchronization, each process must follow a specific **protocol** to access its critical section. This sequence is typically part of an infinite loop where the process cycles through the following states:

```text
+-------------------------------------------------------------+
  |  [ ENTRY SECTION ]  --> Requests permission to enter.       |
  |                         Must wait here if others are in CS. |
  +-------------------------+-----------------------------------+
                            |
                            v
  +-------------------------------------------------------------+
  | [ CRITICAL SECTION ] --> Modifies shared data/variables.    |
  |                         Executed Mutually Exclusively.      |
  +-------------------------+-----------------------------------+
                            |
                            v
  +-------------------------------------------------------------+
  |   [ EXIT SECTION ]  --> Releases permission/unlocks.        |
  |                         Signals waiting processes to enter. |
  +-------------------------+-----------------------------------+
                            |
                            v
  +-------------------------------------------------------------+
  | [ REMAINDER SECTION ] -> Non-critical code.                 |
  |                         No impact on other processes' CS.   |
  +-------------------------+-----------------------------------+
                            |
                  +--- (Repeat Loop) ---+
```

**Why the Sequence is Mandatory:**

- **Security of Data:** Skipping the **Entry Section** leads to Race Conditions.

- **System Liveness:** Skipping the **Exit Section** causes **Deadlock**, as other processes will wait forever for a lock that is never released.

- **Isolation:** The **Remainder Section** is the only place where a process can "take its time" without hindering the progress of others.

---

## 4\. Requirements for a Solution

To ensure a durable and correct solution to the CS problem, three requirements must be met:

1. **Mutual Exclusion:** If process P\_i is executing in its CS, no other processes can be executing in their CSs.

2. **Progress:** If no process is in its CS and some processes wish to enter, only those processes not in their **Remainder Section** can participate in the decision of who enters next. This decision cannot be postponed indefinitely.

3. **Bounded Waiting:** There must be a limit on the number of times other processes are allowed to enter their CSs after a process has made a request to enter its own CS and before that request is granted. (Prevents **Starvation**).

---

## 5\. Peterson's Solution

A classic software-based solution for two processes (P\_0 and P\_1). It assumes that `load` and `store` instructions are atomic.

### Shared Variables

```c
int turn;          // Whose turn it is to enter
boolean flag[2];   // flag[i] = true means Pi is ready to enter
```

### Algorithm for Process P\_i

```c
while (true) {
    flag[i] = true;           // I am ready
    turn = j;                 // Give the other process a chance
    while (flag[j] && turn == j) ; // Busy wait if it's j's turn and j is ready

    // CRITICAL SECTION
    
    flag[i] = false;          // I am done
    
    // REMAINDER SECTION
}
```

### Critical Analysis

- **Why it works:** It satisfies all three requirements. If both try to enter, `turn` will be set to either 0 or 1 last, determining who enters first.

- **Ordering Sensitivity:** If you swap `flag[i] = true;` and `turn = j;`, mutual exclusion is invalidated because both processes could set their flags *after* checking the `while` condition, potentially allowing both into the CS.

- **Limitations:**

    - It is a **Software-only** solution.

    - It does not scale easily beyond 2 processes.

    - Modern compilers and CPUs may reorder instructions (Out-of-Order Execution), which can break the logic unless **Memory Barriers** are used.

### Reference

- *Silberschatz, Galvin, and Gagne (SGG)*, **Operating System Concepts**, Chapter 6.



