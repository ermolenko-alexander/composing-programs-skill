# Section 4.8: Parallel Computing

## Core Idea
From the 1970s through the mid-2000s, individual processor cores grew exponentially faster, largely by increasing **clock frequency**. In the mid-2000s that growth ended abruptly due to power and thermal constraints, and CPU manufacturers began placing multiple cores in a single processor. Applications must now take advantage of parallelism to run faster — but parallelism introduces new correctness challenges, particularly with **shared, mutable state**. Concurrency itself is older: even single-core machines ran many programs via **context switching** (rapidly switching between tasks without waiting for completion).

Problems solvable in the functional model with no shared mutable state pose few difficulties: pure functions provide **referential transparency** (expressions can be replaced with their values and vice versa without changing behavior), so independent expressions can be evaluated in parallel — as MapReduce exploits. But not everything is functional: the Berkeley View project identified **thirteen common computational patterns** in science and engineering, only one of which is MapReduce; **the remaining patterns require shared state**. The section examines the resulting bugs and remedies in two running applications: a web **crawler** and a particle **simulator**.

## Parallelism in Python

### Threading
Multiple threads of execution exist within a single interpreter, each executing independently but **sharing the same data**. The CPython interpreter only interprets code in one thread at a time, switching between them to give the illusion of parallelism; however, operations external to the interpreter — writing a file, accessing the network — may run in parallel.
```python
>>> import threading
>>> def thread_hello():
        other = threading.Thread(target=thread_say_hello, args=())
        other.start()
        thread_say_hello()
>>> def thread_say_hello():
        print('hello from', threading.current_thread().name)
>>> thread_hello()
hello from Thread-1
hello from MainThread
```
`Thread(target, args)` creates a thread; `start` marks it ready to run; `current_thread` returns the current `Thread`. The prints can occur in any order — nothing is synchronized.

### Multiprocessing
A program spawns multiple interpreters (**processes**), each running code independently. Processes do not generally share data — any shared state must be communicated between them. They execute truly in parallel to the extent the OS and hardware allow (multiple cores ⇒ real concurrency). The `multiprocessing` module's classes are analogous to `threading`'s (`Process`, `current_process`). In the hello example, the interactive prompt appears before the other process's output — the display itself is unsynchronized shared state.

## The Problem with Shared State
```python
import threading
from time import sleep

counter = [0]

def increment():
    count = counter[0]
    sleep(0) # try to force a switch to the other thread
    counter[0] = count + 1

other = threading.Thread(target=increment, args=())
other.start()
increment()
print('count is now: ', counter[0])
```
CPython can switch threads at almost any time; **only the most basic operations are atomic** (appearing to occur instantly, with no switch possible during them). Incrementing requires three basic operations — read the old value, add one, write the new value — and the interpreter can switch between any of them:
```
Thread 0                    Thread 1
read counter[0]: 0
                            read counter[0]: 0
calculate 0 + 1: 1
write 1 -> counter[0]
                            calculate 0 + 1: 1
                            write 1 -> counter[0]
```
The counter ends at 1 though incremented twice. Worse, the wrong switch may occur only rarely, making the bug hard to debug — even with `sleep`, the program sometimes prints 2 and sometimes 1. A conflict where shared data may be mutated by one thread while another accesses it is a **race condition** — "a bug that only exists in the parallel world." Shared data is **synchronized** if it is protected from concurrent access.

## When No Synchronization Is Necessary
- **Read-only data** needs none: never mutated, all threads always read the same values.
- Rarely, mutated shared data needs none — but knowing when requires deep knowledge of interpreter, software, and hardware. The flag example:
```python
items = []
flag = []

def consume():
    while not flag:
        pass
    print('items is', items)

def produce():
    consumer = threading.Thread(target=consume, args=())
    consumer.start()
    for i in range(10):
        items.append(i)
    flag.append('go')

produce()
```
Works in most Python implementations — but compilers, interpreters, and even hardware commonly **reorder operations within a thread** that don't depend on each other for data; `flag.append('go')` could move before the loop. "You should avoid code like this unless you are certain that the underlying system won't reorder the relevant operations."

## Synchronized Data Structures
The simplest mechanism: a structure with synchronized operations. `queue.Queue` provides synchronized FIFO access: `put` adds, `get` retrieves, and the class guarantees items are not lost no matter how thread operations interleave.
```python
from queue import Queue

queue = Queue()

def synchronized_consume():
    while True:
        print('got an item:', queue.get())
        queue.task_done()

def synchronized_produce():
    consumer = threading.Thread(target=synchronized_consume, args=())
    consumer.daemon = True
    consumer.start()
    for i in range(10):
        queue.put(i)
    queue.join()

synchronized_produce()
```
The consumer is a **daemon** thread — the program will not wait for it before exiting, permitting its infinite loop. The consumer calls `task_done` per item; the main thread's `queue.join()` waits until all items are processed, so the program exits only then. Use case: the parallel web **crawler** hunting dead links keeps a `Queue` of URLs; multiple threads safely add and remove concurrently.

## Locks
When no synchronized version of a structure exists, provide your own synchronization. A **lock** can be *acquired* by at most one thread; no other may acquire it until the holder *releases* it. `threading.Lock` has `acquire` and `release`; threads attempting to acquire a held lock wait.

**The lock rule**: for a lock to protect a particular set of data, **all threads must follow the rule that no thread accesses any of the shared data unless it owns that lock** — every manipulation wrapped in acquire/release.

Crawler example — Python has no synchronized set, so protect a normal set:
```python
seen = set()
seen_lock = threading.Lock()

def already_seen(item):
    seen_lock.acquire()
    result = True
    if item not in seen:
        seen.add(item)
        result = False
    seen_lock.release()
    return result
```
The lock prevents another thread from adding the URL **between** this thread's membership check and its insertion; also, adding to a set is not atomic, so concurrent adds may corrupt its internal data. Manual release is error-prone (especially with exceptions, and you must not return before releasing), so prefer `with`:
```python
def already_seen(item):
    with seen_lock:
        if item not in seen:
            seen.add(item)
            return False
        return True
```
`with` acquires before the suite and releases when the suite exits **for any reason**. Operations that must be synchronized with each other must use the **same** lock; two disjoint sets of operations that only conflict within themselves should use **two different locks** to avoid over-synchronization.

## Barriers
Divide a program into **phases**, mutating shared data only in a phase where no other thread accesses it. A barrier requires all threads to reach it before any proceeds: code after a barrier cannot be concurrent with code before it. Python: `threading.Barrier(n).wait()`.
```python
counters = [0, 0]
barrier = threading.Barrier(2)

def count(thread_num, steps):
    for i in range(steps):
        other = counters[1 - thread_num]
        barrier.wait() # wait for reads to complete
        counters[thread_num] = other + 1
        barrier.wait() # wait for writes to complete

def threaded_count(steps):
    other = threading.Thread(target=count, args=(1, steps))
    other.start()
    count(0, steps)
    print('counters:', counters)

threaded_count(10)
```
Reads and writes occur in different phases; the writes share a phase but are **disjoint** (each thread writes only its own counter) — disjointness is required to avoid concurrent writes to the same data within a phase. Properly synchronized, both counters always end at 10.

**Particle simulator (threaded)**: each thread owns particles with position, velocity, acceleration; each timestep computes new accelerations from all particles' positions. Read phase: all threads read all positions; each updates its own particles' accelerations (disjoint writes, no locks needed). Write phase: each thread updates its own particles' velocities and positions — disjoint again, protected from the read phase by barriers.

## Message Passing
Avoid concurrent access to the same data entirely. Multiprocessing does this naturally: separate interpreters, separate data; required state is communicated as messages. `multiprocessing.Pipe` is a channel between processes — **duplex** (two-way) by default; `Pipe(False)` gives a one-way channel. `send` transmits an object; `recv` is **blocking** — the caller waits until an object is received.
```python
def process_consume(in_pipe):
    while True:
        item = in_pipe.recv()
        if item is None:
            return
        print('got an item:', item)

def process_produce():
    pipe = multiprocessing.Pipe(False)
    consumer = multiprocessing.Process(target=process_consume, args=(pipe[0],))
    consumer.start()
    for i in range(10):
        pipe[1].send(i)
    pipe[1].send(None) # done signal

process_produce()
```
A `None` message signals the end of communication; one pipe end is passed as an argument since state must be shared explicitly between processes.

**Particle simulator (multiprocess)**: pipes form an entire **circular pipeline** between processes to minimize communication. Each process injects its particles' positions into its pipeline stage; positions make a full rotation; at each rotation step a process applies forces from the positions currently in its stage onto its own particles — after a full rotation all forces have been applied. `multiprocessing` also provides synchronized queues, locks, and (since Python 3.3) barriers — e.g. to synchronize printing and avoid the garbled display seen earlier.

## Reference Table: choosing a mechanism

| Mechanism | Module/class | Protects by | Best when |
|---|---|---|---|
| Synchronized structure | `queue.Queue` (`put`/`get`/`task_done`/`join`) | built-in synchronized operations | a standard structure fits (producer/consumer, work queues, crawler URL queue) |
| Lock | `threading.Lock` + `with` | mutual exclusion; the "hold the lock to touch the data" rule | no synchronized version of the structure exists (e.g. a set) |
| Barrier | `threading.Barrier(n).wait()` | phase separation; disjoint writes within a phase | lock-step computations (simulations with timesteps) |
| Message passing | `multiprocessing.Pipe` (`send`/blocking `recv`) | no shared data at all | processes; true parallelism; explicit state transfer |

## Synchronization Pitfalls
- **Under-synchronization**: neglecting to synchronize accesses *together*. In the set example, the membership check and insertion must be synchronized as one unit; synchronizing them separately is still erroneous — another thread can insert between them.
- **Over-synchronization**: preventing non-conflicting operations from running concurrently. Trivial extreme: a master lock acquired at thread start, released at completion — the program is fully serialized. It can even hang: a consumer that takes the lock and never releases it stops the producer, which starves the consumer. In practice programmers routinely over-synchronize to a degree, forfeiting available parallelism.
- **Deadlock**: two or more threads/processes stuck waiting for each other. Its source is a **circular wait**. Even with correct lock releases, deadlock can occur:
```python
def deadlock(in_pipe, out_pipe):
    item = in_pipe.recv()
    print('got an item:', item)
    out_pipe.send(item + 1)

def create_deadlock():
    pipe = multiprocessing.Pipe()
    other = multiprocessing.Process(target=deadlock, args=(pipe[0], pipe[1]))
    other.start()
    deadlock(pipe[1], pipe[0])

create_deadlock()
```
Both processes `recv` first on a duplex pipe; `recv` blocks until something arrives; neither has sent — both wait forever. **Alignment rules to avoid deadlock**: send over a pipe before receiving, acquire multiple locks in the same order, and ensure all threads reach the right barrier at the right time.

## Key Concepts
- **Clock frequency / context switching**: why cores multiplied; how concurrency predates multicore.
- **Referential transparency**: the property of pure functions licensing parallel evaluation.
- **Race condition**: mutation by one thread while another accesses; parallel-only bug.
- **Atomic operation**: appears instantaneous; no switch during it — only the most basic operations qualify.
- **Synchronized data**: protected from concurrent access.
- **Daemon thread**: program exit does not wait for it.
- **Lock rule**: every access wrapped in acquire/release of the same lock.
- **Phase / barrier**: post-barrier code cannot be concurrent with pre-barrier code.
- **Blocking `recv` / duplex pipe / sentinel `None`**: pipe mechanics.
- **Circular wait**: structural source of deadlock.
- **Instruction reordering**: compilers/hardware reorder data-independent operations within a thread, breaking flag tricks.

## Mental Models
- Every read-modify-write on shared data is three separate steps the scheduler can split.
- A lock is a social contract: it protects data only if every thread honors it for every access.
- Barriers turn time into phases: mutate in a phase where nobody else reads; within a phase, writes must be disjoint.
- Prefer, in order: no shared mutable state (pure functions / messages) → synchronized structures → barriers → explicit locks.
- Deadlock prevention = alignment: send before receive, one global lock order, everyone at the right barrier.

## Anti-patterns / Common Errors
- **Unsynchronized increment**: both threads read 0, both write 1 — and the bad interleaving is rare, so the bug hides.
- **Flag-based signaling without synchronization**: reordering optimizations can hoist the flag write; avoid unless certain the system won't reorder.
- **Separately synchronizing operations that must be atomic together** (check + insert).
- **Master lock over everything**: serializes, and can hang producer/consumer outright.
- **Both ends `recv` first on a duplex pipe**: the canonical deadlock.
- **Returning before releasing a lock / manual acquire-release around exception-prone code**: use `with`.
- **Forgetting `task_done`/`join` discipline with a daemon consumer**: the program may exit before items are processed.

## Key Takeaways
1. Hardware now scales by cores; applications must arrange work in parallel — and shared mutable state is where correctness dies.
2. Pure-functional code parallelizes freely; but twelve of the Berkeley View's thirteen patterns need shared state.
3. Identify the atomic unit (e.g. check+insert) and synchronize it as one.
4. Reach for `queue.Queue` before hand-rolled locks; use `with lock:`; separate locks for disjoint concerns.
5. Barriers: read in one phase, write disjointly in another.
6. Multiprocessing + pipes sidesteps sharing; mind blocking `recv`, sentinels, and send/receive ordering.
7. The three failure modes: under-synchronization, over-synchronization, deadlock (circular wait).
8. Making parallelism easier and less error-prone is an active research area; this is "a basic introduction to this crucial area."

## Connects To
- **Section 4.7**: MapReduce is the pure-function escape hatch where none of this synchronization is needed.
- **Section 4.6**: message passing between processes mirrors message protocols between machines.
- **Chapter 2 (mutable state)**: mutation hazards become nondeterministic under concurrency.
- **Chapter 1 (pure functions)**: referential transparency is the formal reason purity parallelizes.
