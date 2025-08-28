## What ?

I wasn't paying the greatest bit of attention when the networking parts of the OS came up in my college OS courses. So my understanding of networking has always been a bit dodgy. This became more clear to me when I became interested in building a distributed key value storage module from scratch in c++.

For building such a system different nodes and clients need to be able to communicate with each other by sending and receiving messages. Not only that for a system to scale to many nodes and many clients it needs to be able to handle this connections efficiently. 

I found networking to complex, rich and confusing. This blog documents my attemp to build networking primitives that scales.

## But why a blog ? Isn't writing code enough ?

It is most of time. But often times when I write code to do something, but then don't use the techniques that I mastered for that exercise regularly, I find gaps develop in my understanding that requires a ton of effort to remaster. This is a experiment in seeing whether documenting my work with not just technical details but my thoughts and intuitions would help with future recollection and retention.

I have another motivation. Many people have said writing helps in refining your thinking. I have found that to be true intuitively but never practiced it explicitly. Even in college, I hated making notes. This is my attempt to change that. 

And if this helps someone in any way, that is a bonus. This isn't meant to be completely self contained, nor complete. The primary audience I am writing this for is myself so I apologise if it seems incoherent in pieces. I welcome any feedback both on the technical and the writing side. 

## Setup
I have managed to break my current linux setup, so this is developed on windows, which is not ideal but I haven't been able to give time to fix linux so here we go. Please not that the OS level syscall and library support is not same, so this will not work in linux systems as is. Okay here we go.

For simplicity we are dealing with independent processes in the same physical system. These processes can be thought of nodes. You can assume some of these nodes are servers and some of these are clients. Clients need to communicate with the server for GET, PUT and UPDATE like operations. The servers needs to receive these PUT, UPDATE messages and update the KV state and for GET request it needs to send back the response of the current query to the client. The server also needs to communicate with each other so that all the server agree on the state of KV store. 
At a high level each node should be able to do the following things with respect to the networking aspect:
1. Establish a connection with another node
2. Send a message 
3. Receive a message 

A message is simply a structure you can define and serialize as byte stream.

### Sockets


I will be using sockets to interface between our processes. Every OS provides support for sockets with pretty uniform APIs. While it is true if your processes are in the same system, alternative mechanisms like shared memory / named pipes will be faster and hence more scalable, but it requires manual synchronization by the developer(mutexes/ semaphores) to prevent race conditions, while sockets have built in support for that. Another reason for using shared memory is not possible if your processes are not in the same device, and eventually we would like to evolve our system to be physically distributed without completely rewriting the code. 

Another option is message queues where you can define producers who send messages and consumers who consume messages and a central broker who stores these messages in queue. The consumer and producer don't directly interact with each but with the broker. This has built in asynchronosity as server and client need not be online at the same but it is slower as it is not point to point communication (rather broker acts as a middleman) plus the queuing mechanism can add added latency as your message may be waiting in queue. 

I don't wanna pretend I thought through all the pros/cons before deciding on sockets. I had somewhat more of an familiarity with sockets and hence started with it. It is also simplest to intuitively internalise as its a simple point to point communication between two parties. 

### Asynchrony 

read/writes to a file, or making a network calls are what one calls I/O calls. CPU makes I/O requests but doesn't handle the transfering of data from source to destination. The DMA(Direct Memory Access) Controller is a hardware component which is responsible for the transfer. In a non async system While the transfer happens, this thread / process execution is blocked, that is thread/process stops executing. The CPU is orders of magnitude faster than DMA and hence when developing low latency systems it is essential that we do not waste these cpu cycles. (Technically the CPUs cycles are not wasted but rather utilised by other processes that the kernel may schedule). But we are selfish and want to use these CPU cyles for our process. So we want to able to do other tasks while these network calls happens. Essentially we want to be able to multitask. What kind of multitask framework are we looking at ?

Consider the server. The server is connected to multiple clients and server. When a message is received, it performs certain actions conditioned on the message(Write to memory/disk if PUT / UPDATE, Read from memory if GET). It still needs to be able to listen to other messages while it is working on some task/or receiving or sending another message. A single threaded application using blocking I/O can handle one connection at a time. This is obviously not gonna work when you have so many clients and server. So there are two different approaches to solve this problem 

#### Multithreading :
The server creates a new thread for a client connection. When a new message arrives, a new thread is spawned to handle it. This allows multiple rec() calls to block on different threads without affecting each other. This is still suboptimal as you are limited by the number of threads your cpus can run parallely(it can allow spawning of many threads but can run only a $N$ number of threads parallely where $N$ is the number of cores in your system. Each threads take memory and cpu resources. 

#### Asynchronous I/O 
Modern OS support something called asynchronous I/O (epoll in linux, Input / Output Completion Port (IOCP)). We will be dealing with IOCP as we are in a windows system. It is a efficient threading model which can used to perform async I/O. Basically you can associate a port with list of file handles(your sockets, everything is a file in linux). This creates I/O completion port. You can allocate a thread pool which handles all communication from this IOCP. All the threads is waiting on a event happening on one of the the file handles. Essentially your threads registers your sockets to the OS and it tells the OS if any event happens on any of the sockets associated with me let me know. You can do this with **WSARecv** or **AcceptEx**. The function return immediately and DMA does the data transfer in the background. The thread goes to sleep waiting for I/O related to any of the socket to complete and is woken up from the OS from a getCompletionRequest(). 

#### Coroutines 
Now that we know we can handle async I/O we need a system to handle these I/O efficiently. Traditionally callbacks have been popular for these tasks. Callbacks are essentially function that the OS will run when the async I/O complete. I have not used callbacks outside of very simple scenarios but apparently callbacks can become very complicated very soon. My understanding is it is not necessarily the performance but rather code complexity. Coroutines allows you to write async I/O code almost exactly synchronous code and hence makes it easier to manage the code complexity. So what are coroutines and how do they work ? 

During the course of this exercise coroutine was the most difficult part for me to understand. I am still not sure I understand them well even now. I will not go too deeply into mechanism and the underlying details. I recommend these two blogs for understanding coroutines 
* https://theshoemaker.de/posts/yet-another-cpp-coroutine-tutorial
* https://lewissbaker.github.io/2022/08/27/understanding-the-compiler-transform

When one uses thread for managing I/O in a traditional manner, the thread cannot be used when waiting on I/O. Coroutines allow one to use the thread for other useful task while waiting on I/O. There is a lot of syntax related details which allows your compiler to transform your function to a coroutine. Basically any piece of function which contains any of the following keyword **co_await**, **co-return** or **co_yield** is coroutine. Now your co_await needs a awaitable structure which is essentially the thing the coroutine is waiting on. For example a readAwaitable is a awaitable structure that the coroutine can "await" on. The readAwaitable is handling the network call. Let me give an example of how control flow of how coroutine would work such a 




## Glossary 

## References 