# Operating Systems - 4/16/26

## 1. POSIX Unnamed Semaphores

Unnamed semaphores are synchronization primitives defined in the `<semaphore.h>` header. Unlike named semaphores, which are identified by a path in the file system, unnamed semaphores are stored in memory.

### 1.1. Declaration and Initialization

A semaphore is represented by the **`sem_t`** data type. It must be initialized before use.

```c
#include <semaphore.h>

sem_t s; // Declare the semaphore

// Initialization
// pshared: 0 for thread-sharing, non-zero for process-sharing
// value: initial value of the semaphore
int sem_init(sem_t *s, int pshared, unsigned int value);
```

- **`pshared` parameter:** If set to `0`, the semaphore is shared between threads of the same process. If non-zero (e.g., `1`), it can be shared between processes (requires the semaphore to be located in a shared memory region).

### 1.2. Core Operations

| Operation | Function | Description |
| --- | --- | --- |
| **Wait (P)** | `sem_wait()` | Decrements the semaphore. Blocks if value is 0. |
| **Try Wait** | `sem_trywait()` | Non-blocking version of `sem_wait()`. Returns `EAGAIN` if locked. |
| **Post (V)** | `sem_post()` | Increments the semaphore and wakes up waiting threads. |

### 1.3. Handling Interruptions (EINTR)

When `sem_wait()` is interrupted by a signal handler, it may return `-1` with `errno` set to `EINTR`. To ensure the program correctly waits for the semaphore, this must be handled in a loop.

```c
while (sem_wait(&s) == -1 && errno == EINTR) {
    continue; // Restart the wait if interrupted by a signal
}
```

---

## 2. Pthread Library: Synchronization Structures

The Pthread library provides dedicated structures for mutual exclusion and condition-based coordination.

### 2.1. Mutex Variables (`pthread_mutex_t`)

A **Mutex** (Mutual Exclusion) variable acts as a lock to protect shared data.

- **States:** A mutex is either **Locked** (owned by a specific thread) or **Unlocked** (free).
- **Ownership:** Only the thread that locked the mutex can unlock it.
- **Queueing:** Threads waiting for a mutex are not guaranteed to be serviced in FIFO (First-In-First-Out) order.
- **Best Practice:** Hold a mutex for the shortest time possible to minimize the **Critical Section**.

### 2.2. POSIX Mutex Usage and Operations

```c
#include <pthread.h>

// 1. Definition
pthread_mutex_t lock;

// 2. Initialization (with default attributes)
pthread_mutex_init(&lock, NULL);

// 3. Lock Operation (Blocking)
pthread_mutex_lock(&lock);

// 4. Try Lock (Non-blocking)
if (pthread_mutex_trylock(&lock) == 0) {
    // Lock acquired
}

// 5. Unlock Operation
pthread_mutex_unlock(&lock);
```

#### Example: Thread-Safe Counter

```c
typedef struct {
    int value;
    pthread_mutex_t lock;
} Counter;

void increment(Counter *c) {
    pthread_mutex_lock(&c->lock); // Protect shared variable
    c->value++;
    pthread_mutex_unlock(&c->lock); // Exit Critical Section
}
```

---

## 3. Comparison: Semaphores vs. Mutex Locks

While binary semaphores can behave like mutexes, there are critical architectural differences.

| Feature | Binary Semaphore | Mutex Lock |
| --- | --- | --- |
| **Ownership** | No ownership; any thread can signal. | Owned by the thread that locked it. |
| **Initialization** | `sem_init(&s, 0, 1);` | `pthread_mutex_init(&lock, NULL);` |
| **Priority Safety** | No inherent protection. | Often includes **Priority Inversion** safety. |
| **Deletion Safety** | A thread can be deleted while holding it. | Prevents deletion of a thread owning the lock. |

> **Teacher's Probing Question:** Why does the lack of ownership in semaphores make them suitable for "event notification" but potentially dangerous for "mutual exclusion"?

---

## 4. Synchronization Goals and Logic

Effective synchronization ensures the safety and coordination of parallel computations.

### 4.1. Core Objectives
1. **Safety:** Ensure shared updates are correct and avoid **race conditions**.
2. **Coordination:** Manage the timing of threads (e.g., event notification or parallel phase alignment).
3. **Efficiency:** Impose as little overhead as possible by keeping **Critical Sections** small.

### 4.2. Challenges and Solutions

- **Spin Lock:** A thread repeatedly checks a lock in a loop (busy-waiting). Useful for very short waits but wasteful of CPU cycles.
- **Sleep/Wakeup:** Moving a waiting thread to a blocked state to save CPU cycles (used by most modern mutex and semaphore implementations).

> **Teacher's Probing Question:** If all interleavings of events must be correct for a program to be "synchronized," how does decreasing the size of the Critical Section affect the number of possible interleavings?
