# Section 4.8: Parallel Computing

## Core Idea
Since clock-frequency growth ended in the mid-2000s, programs must exploit multiple cores — but parallelism plus shared, mutable state breeds race conditions. Correctness comes from choosing a synchronization mechanism: synchronized data structures, locks, barriers, or avoiding sharing entirely via message passing.

## Frameworks Introduced
- **Threading vs multiprocessing (Python's two means of parallel execution)**:
  - *Threading*: multiple threads in one interpreter sharing data; CPython interprets code in only one thread at a time (illusion of parallelism), though external operations (file, network) may run in parallel.
  - *Multiprocessing*: multiple interpreters (processes) with no shared data; true parallelism on multiple cores; shared state must be communicated.
- **Race condition**: shared data mutated by one thread while another accesses it; only basic operations are atomic, and the interpreter may switch threads between the read/compute/write steps of an increment.
- **Synchronized data structures**: use a structure with synchronized operations — `queue.Queue` (`put`/`get`/`task_done`/`join`) guarantees no items are lost under any interleaving.
- **Locks**: `threading.Lock` — acquired by at most one thread; all threads must follow the rule "no access to the shared data unless holding its lock"; prefer `with lock:` so release happens on any exit. Disjoint sets of operations that never conflict should use different locks.
- **Barriers**: `threading.Barrier(n).wait()` divides a program into phases — all threads must reach it before any proceeds; code after a barrier cannot be concurrent with code before it. Put reads and writes in different phases, writes disjoint.
- **Message passing**: avoid concurrent access entirely — processes communicate via `multiprocessing.Pipe` (`send`/`recv`, recv blocks; duplex by default, `Pipe(False)` one-way), with a sentinel (`None`) to end communication.
- **Synchronization pitfalls**: under-synchronization (check and insert synchronized separately, not together), over-synchronization (serializing everything — a master lock — losing parallelism or hanging), and deadlock (a circular wait — threads stuck waiting on each other).

## Key Concepts
- **Clock frequency / context switching**: single-core speedups stalled; concurrency existed even on one core by rapidly switching tasks.
- **Referential transparency**: pure functions let expressions be replaced by values; independent expressions can run in parallel (why MapReduce needs no synchronization — but only 1 of Berkeley View's thirteen common computational patterns is MapReduce; the rest need shared state).
- **Atomic operation**: appears to occur instantly with no thread switch possible during it.
- **Synchronized data**: shared data protected from concurrent access.
- **Daemon thread**: the program will not wait for it to finish before exiting.
- **Blocking call**: `recv` waits until an object is received.
- **Circular wait**: the structural source of deadlock.
- **Instruction reordering**: compilers/hardware may reorder independent operations within a thread, breaking clever flag-based "synchronization-free" code.

## Mental Models
- Think of every read-modify-write on shared data as three separate steps the scheduler can split.
- Use the lock rule as a social contract: a lock protects data only if every thread wraps every access.
- Think of barriers as phase separators: mutate in a phase where nobody else reads.
- Prefer, in order: no shared mutable state (pure functions/messages) → synchronized structures → barriers → explicit locks.
- Deadlock prevention = alignment: send before receive, acquire multiple locks in one global order, get every thread to the right barrier.

## Anti-patterns
- **Unsynchronized increment**: two threads both read 0, both write 1 — counter incremented twice ends at 1; worse, the bad interleaving is rare, so the bug hides.
- **Flag-based signaling without synchronization**: works in most Python implementations, but reordering optimizations can move `flag.append('go')` before the loop — avoid unless certain the system won't reorder.
- **Separately synchronizing operations that must be atomic together**: the membership check and insertion must hold the lock across both.
- **Master lock over everything**: serializes the program; can hang a producer/consumer outright.
- **Both ends recv first on a duplex pipe**: classic deadlock — each blocks waiting for the other to send.
- **Returning before releasing a lock**: use `with`; exceptions make manual acquire/release error-prone.

## Code Examples
```python
import threading
from time import sleep

counter = [0]

def increment():
    count = counter[0]
    sleep(0) # try to force a switch to the other thread
    counter[0] = count + 1
```
- **What it demonstrates**: the canonical race condition — a thread switch between read and write loses an increment.

```python
seen = set()
seen_lock = threading.Lock()

def already_seen(item):
    with seen_lock:
        if item not in seen:
            seen.add(item)
            return False
        return True
```
- **What it demonstrates**: a lock making check+insert atomic, with `with` guaranteeing release on any exit.

## Reference Tables
| Mechanism | Module/class | Protects by | Best when |
|---|---|---|---|
| Synchronized structure | `queue.Queue` | built-in synchronized put/get | a standard structure fits (producer/consumer, work queues) |
| Lock | `threading.Lock` | mutual exclusion over a data set | no synchronized version of the structure exists |
| Barrier | `threading.Barrier` | phase separation (reads vs writes) | lock-step computations (simulations) |
| Message passing | `multiprocessing.Pipe` | no shared data at all | processes; true parallelism; explicit state transfer |

## Worked Example
The particle simulator. Each thread owns particles; each timestep computes new accelerations from all particles' positions, then updates velocities and positions. Threaded version: a read phase (all threads read all positions; each updates only its own particles' accelerations — disjoint writes need no locks) and a write phase (each updates its own velocities/positions), separated by `barrier.wait()` calls so no write is concurrent with reads. Multiprocess version: pipes form a circular pipeline — each process injects its particles' positions, which rotate through every stage; at each rotation step a process applies forces from the positions currently in its stage, so after a full rotation all forces are applied with minimal communication.

## Key Takeaways
1. Hardware now scales by cores, so applications must arrange work in parallel — and shared mutable state is where correctness dies.
2. Pure-functional code parallelizes freely (referential transparency); most computational patterns, however, need shared state.
3. Identify the atomic unit (e.g. check+insert) and synchronize it as one.
4. Reach for `queue.Queue` before hand-rolled locks; use `with lock:`; use separate locks for disjoint concerns.
5. Barriers turn time into phases: read in one, write disjointly in another.
6. Multiprocessing + pipes sidesteps sharing; mind blocking `recv` and send/receive ordering.
7. Watch for the three failure modes: under-synchronization, over-synchronization, deadlock (circular wait).

## Connects To
- **Section 4.7**: MapReduce is the pure-function escape hatch where none of this synchronization is needed.
- **Section 4.6**: message passing between processes mirrors message protocols between machines.
- **Chapter 2 (mutable state)**: the chapter's mutation hazards become nondeterministic under concurrency.
