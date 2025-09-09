## Miscellaneous

Will contain random information and thoughts I have or note that do not otherwise cleanly fall into some other subtopic.

#### 1. Semaphores vs Mutex
  We only care about binary semaphores here as semaphores can only be non-binary that is a integer which represents how many resources of a particular type exist. Both binary semaphores and mutex can be use for mutual exclusion, the primary difference is that binary semaphore 
  does not have any concept of ownership where mutex does. Only the thread which owns the mutex can unlock it. Semaphores are more prone to programming error as they do not have concept of ownership. Primary usecase of binary semaphore would be signal between 
  threads.Lets say thread A produce some data and thread B needs the data, then semaphore can be used by thread A to thread B to its availability as thread B is waiting on the semaphore.
