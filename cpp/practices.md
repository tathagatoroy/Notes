## Good Practices

* when some objects are created and deleted periodically and each of these objects need to be associated or own a certain resource, it is better to create a pool of resources and carefully assign them to the object (ensuring no two object own the same resource) and then carefully return them to the resource pool to be reissued and reinitialised.Ensure data from first object use is not kept when assigned to a new object.
* when dealing with system resources like sockets, it is better to implement RAII wrapper to ensure proper memory cleanup.
*  **const vs constexpr** : const and constexpr both provides immutability. but for constexpr the initialisation value needs to be known at compile time. const can be initialised at runtime

## Concepts 
### explicit 
what does explicit keyword for a constructor

An explicit constructor prevents the compiler from using that constructor for implicit conversions and copy-initialization. It forces callers to use direct-initialization (e.g., A a(1); or A a{1};) and blocks implicit conversions like A a = 1;.

Why use it:
Avoid accidental/undesired conversions.
Make intent clear and reduce surprising overload resolution.
examples 
```cpp
// Example showing difference
struct A {
    explicit A(int x) {}   // explicit prevents implicit conversions
};

struct B {
    B(int x) {}            // non-explicit allows implicit conversions
};

void takesA(A) {}
void takesB(B) {}

int main() {
    A a1(10);      // OK (direct-initialization)
    A a2{10};      // OK (direct-list-initialization)
    // A a3 = 10;  // ERROR: copy-initialization not allowed for explicit ctor

    B b1 = 10;     // OK: implicit conversion from int to B
    takesB(20);    // OK, implicit conversion creates B(20)
    // takesA(20); // ERROR: cannot implicitly convert int -> A

    return 0;
}
```
### Condition Variable
A condition variable is a synchronization primitive in C++ used to block one or more threads until a specific condition is met. It's always used in conjunction with a mutex to protect the shared data that the condition variable is monitoring.
A condition variable allows threads to "wait" efficiently instead of busy-waiting (constantly checking a condition in a loop, which wastes CPU cycles). There are two primary roles:
Waiting Thread (Consumer):
  1. Acquires a std::unique_lock on the mutex.
  2. Calls the wait() function on the condition variable.
  3. wait() atomically releases the mutex and puts the thread to sleep.
  4. When the thread is woken up (either by a notification or a "spurious wakeup"), wait() reacquires the mutex before returning, ensuring the thread can safely check the condition. A while loop or predicate check is used to handle spurious wakeups.

Notifying Thread (Producer):
  1. Acquires the same mutex.
  2. Modifies the shared data.
  3. Calls either notify_one() to wake up a single waiting thread or notify_all() to wake up all waiting threads.

Releases the mutex.

Common Use Case: Producer-Consumer

Imagine a scenario with a shared queue.
A producer thread adds items to the queue.
A consumer thread removes items from the queue.
Without a condition variable, the consumer would have to repeatedly lock the mutex, check if the queue is empty, and then unlock it—a very inefficient process. With a condition variable:
The consumer thread acquires the mutex and waits on the condition variable, releasing the mutex.
When the producer adds an item, it notifies the condition variable.
The condition variable wakes up the consumer thread, which then reacquires the mutex, sees that the queue is no longer empty, and proceeds to process the item.
Condition Variable in Modern cpp and unique lock | Introduction to Concurrency in C++ is a video that explains how to use a condition variable in C++.

### Thread vs Coroutine
#### The Thread Model 
In a traditional threaded model, a thread is an operating system entity. It's a complete execution context with its own stack, instruction pointer, and registers. The operating system's scheduler is responsible for swapping these threads in and out of the CPU. This is a heavyweight context switch. When one thread is blocked waiting for I/O, it's put to sleep by the OS, and another thread is scheduled to run.
One thread per task: A single task is typically handled by one thread until it's finished or explicitly yields. If the task involves blocking I/O, the thread is blocked.
OS manages context: The OS saves and restores the entire state of the thread's CPU registers and stack.
Task is bound to a thread: Yes, the task and its execution are tightly coupled to the thread.

#### The Coroutine Model
In a coroutine model, one thread does different tasks.  The coroutine's state, including its local variables and where it needs to resume execution, is saved to the heap (or another part of memory) when it's suspended. The crucial part is that this suspension and resumption is managed by the program's event loop, not the OS.
Multiple tasks on one thread: A single thread can run many coroutines. When a coroutine hits a suspension point (like an await for I/O), it tells the event loop, "I'm going to wait here. Here is my state. Call me back when the I/O is done."
Event loop manages context: The event loop, which runs on a single OS thread, simply picks up the next ready coroutine and starts it. The context switch is a lightweight jump between coroutines' suspended states. It avoids the overhead of a full OS thread context switch.
Task is not bound to a thread: A coroutine is an independent task. It can be paused and resumed on the same thread without the thread being blocked. The thread can work on other coroutines while one is suspended.

The key difference is the granularity and overhead of the context switch. Threads are managed by the OS and have a high-cost context switch, while coroutines are managed by a runtime (event loop) and have a very low-cost context switch, allowing for massive concurrency on a single thread.


### Pointer to a class vs class object :
When you have a class which contains other classes as an member object should you use 
object or the pointer to a object.

Most of the time you should use the the object directly as it is safer and you dont have to manage lifetime directly and deal with null or dangling pointer.

1. **Polymorphism**: If you have a base class and several derived classes, you can use a base class pointer to refer to objects of any of the derived types. This allows you to call virtual functions and have the correct derived class implementation execute at runtime.


2. **Optional Objects**: When a member object might not always exist, a pointer (like std::unique_ptr) is an excellent way to represent that optionality. An empty std::unique_ptr clearly signals that the object is not present. This is safer and more expressive than a regular member object, which must always be constructed.

3. **Large Objects**: If a class contains a very large object, storing it as a direct member can make the containing class unnecessarily large. Using a pointer to a heap-allocated object can reduce the size of the containing object, which can be beneficial for performance, especially when passing the object by value or storing it in containers.

   

5. **Dynamic Lifetime**: When an object's lifetime is not tied to the scope of its creator, you need a pointer to manage it. This is common in factories, where one part of the code creates an object and another part is responsible for destroying it. Smart pointers like std::unique_ptr and std::shared_ptr are designed for these scenarios.
