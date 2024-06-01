file:///home/ezipe/git/scratchpad/python-3.8.19-docs-html/library/threading.html

## 4 types of synchronization:

- Fork Join
- Mutual Exclusion
- Test Under Lock
- Wait and Signal

## 4 Requirements of Deadlock:

- mutual exclusion: each thread of control must gain mutually exclusive access to a subset of the set of state variables that define the progress condition,
- hold-and-wait: each thread of control must remain inside its mutual exclusion region while it waits for the condition to become true,
- circular wait: some thread of control waits on a set of threads and conditions that ultimately depend on it to make progress,
- no preemption: each thread of control will wait indefinitely until the program logic defines the progress condition to be true.

## Preemption and Context Switching

The answer is that the OS and threading system arrange to multiplex the threads on the CPUs. Each thread is given a turn on some CPU. When it goes to do I/O, or a fixed amount of time has expired, the OS pauses the thread (saving off all of its machine state), and selects another thread to run. It then loads the saved machine state from the new thread onto the CPU and starts that thread at the spot where it last left off (or at the beginning if it was just created).
The process of pausing one thread to run another is called pre-emption and the second thread is said to pre-empt the first. The process of switching to a new thread (as a result of a pre-emption event) is called context switching and the saved machine state that is necessary to get a thread running again after a pause is often called a context.

## Critical Sections, Atomic Execution, and Mutual Exclusion

It may be obvious, but the way to ensure that the bank balance is always computed correctly is to ensure that at most one thread is allowed to execute the load-substract-store sequence at a time. As long as those three instructions are executed, in order, without any other threads interleaving themselves, the balance will be computed correctly.
A segment of code that must be executed sequentially without the potential interleaving of other threads is called a Critical Section. We will sometime refer to the notion that all instructions within a critical section will be executed without being interleaved as Atomic Execution as in the sentence
The instructions within a critical section are executed atomically.
meaning that they will not be interleaved with instructions executed by other threads executing in the same section of code.
The process of ensuring that at most one thread can be executing in a critical section is often termed mutual exclusion to distinguish it from other forms of synchronization. Those other forms will become clear as the course progresses. For now, the important concept to grasp is that we need a way to make sure, no matter what the circumstances, at most one thread can be executing in specific code segments where a race condition due to interleaving could produce an unwanted computation.
As the following example attempts to illustrate, mutual exclusion is typically implemented with some form of “lock.” A lock has the following semantics:

- a lock is initially in the state “unlocked”
- there is a primitive that allows a thread to attempt to “lock” the lock
- there is a primitive that allows a thread to “unlock” a lock that is locked
- any attempt to lock a lock that is already locked, blocks the thread and puts it on a list of threads waiting for the lock to be unlocked
- when a lock is unlocked by a thread, if there are threads waiting (because they tried to lock before), one is selected by the system, given the lock, and allowed to proceed.

This sounds a bit like Dr. Seuss, but is is pretty simple. Think of it as a lock on a door to a room. When one person enters and locks the door, any other attempts to entry will be blocked. In the cases we’ll study, the threads that try to get into the room (the critical section) while someone (another thread) is in it will wait patiently just outside the door. Then, when a thread that is in the room leaves, it will pick one of the waiting threads (and only one) and allow it to enter the room and lock the door behind it.

## In Through the Out Door

Mutual exclusion has some interesting properties. First, it is important that any thread that enters a critical section by locking it, leave it by unlocking it. In the room example, if a person enters the room, locks the door, and then climbs out a window without unlocking the door, no one else will ever be able to get in. In a program, leaving a critical section without calling the unlock primitive is like climbing out the window of the room. Worse, (and don’t think of this as being morbid) if the person in the room dies (or your thread exits due to a fault or because you have returned) the door never gets unlocked and threads waiting will wait forever.
The other thing to understand is that even when your threads correctly enter and leave critical sections, the size of the section influences the amount of concurrency your program will have. For example, if every ATM in the US had to lock the entire bank to implement a transaction, ATM response time would probably be pretty slow.
Thus, you typically try and keep the length of each critical section as small as possible, both to maximize the amount of concurrency and also to minimize the possibility of having a bug cause a thread to die and exit the section accidentally without calling unlock.
