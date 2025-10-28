### Threads
Single Thread : Stack + Free + Heap
Multiple Thread : Stack_1 + Free_1 + Stack_2 + Free_2 (address space) + Heap

Stack for each thread : thread-local storage

Free is the space where the stack can grow into.

* A critical section is a piece of code that accesses a shared resource, usually a variable or data structure.
* A race condition (or data race ) arises if multiple threads of execution enter the critical section at roughly the same time; both attempt to update the shared data structure, leading to a surprising(and perhaps undesirable) outcome.
* An indeterminate program consists of one or more race conditions;
* the output of the program varies from run to run, depending on which threads ran when. The outcome is thus not deterministic, something we usually expect from computer systems.
* To avoid these problems, threads should use some kind of mutual exclusion primitives; doing so guarantees that only a single threadever enters a critical section, thus avoiding races, and resulting indeterministic program outputs.
