# Concurrency and Multithreading

## What is multithreading?

Multitasking - doing multiple things at the same time. In terms of a CPU, this
would be done through multiple threads, as a single thread can only do single
thing at one time.

### Process-based multitasking

Running multiple processes concurrently. E.g. Discord and a game running at the
same time.

### Thread-based multitasking

Allows parts of the same programme to run concurrently – multiple tasks within a
single application.

### Thread vs Process

- Two threads share the same address space
- Context switching between threads is usually less expensive than processes
- Cost of communication is relatively low

### Why multithreading?

Engineering team of one vs ten. You can do more at once, therefore, work is done
faster.

In a single-threaded environment, only one task at a time can be performed. CPU
cycles are wasted, for example, when waiting for user input – they could be put
to better use.

Multitasking allows idle CPU time to be put to good use.

## `Thread.class`

A thread is an independent path of execution within a programme. E.g. the
`Main.class` runs on a single thread – the main thread.

Many threads can run concurrently within a programme.

At runtime, threads of a programme exist within a common memory space, therefore
can share data and code. This makes them lightweight, compared to processes.

Important concepts in Java:

1. Creating threads and providing the code to be executed
2. Accessing common data and code through synchronization
3. Transition between thread states

## The Main Thread

When an application starts, a user thread is automatically created to execute
`main`, and this thread is called the main thread.

If not other user thread are created, the program terminates when main finishes
executing.

All other threads, called child threads - child threads of the main thread, as
in, are spawned from the main thread.

`main` can finish, but the programme will keep running until all user threads
have completed.

The runtime environment distinguishes between user threads and daemon threads.
Preference is given to a user thread, and the programme will end if all user
threads complete, even if daemon threads are running.

Calling `setDaemon(true)` in the thread class marks a thread as a daemon thread,
and must be done before the thread is started.

As long as user thread is alive, the JVM does not terminate.

## Thread Creation

Threads can be created by implementing `Runnable`, or extending from `Thread`.

```java
public class Main {

    // Main Thread
    public static void main(String[] args) {
        // Extending Thread  
        Thread printer = new Printer("Printer One");
        
        /*
         Child Thread
         
         It does not start immediately, it waits for idle time, then calls 
         run.
         
         Generally, should not make assumptions in the order threads will 
         run. It depends on the JVM platform/version being used.
        */
        printer.start();
        
        /*
         Implementing Runnable
         
         Even though Thread implements Runnable, we have to provide our 
         class as the implementation of `run` is required, so the 'target' 
         is set.
        */
        Thread mailer = new Thread(new Mailer(), "Optional Name");
        mailer.start();

        // Third approach using lambdas
        Thread lambdaApproach = new Thread(() -> {
            // Override `run`
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread() + ", " + i);
            }
        });
        lambdaApproach.start();
    }

    public static class Printer extends Thread {

        public Printer(String name) {
            super(name);
        }

        @Override
        public void run() {
            for (int i = 0; i < 5; i++) {
                System.out.println("inside " + Thread
                        .currentThread()
                        .getName() + " " + i);
            }
        }
    }

    public static class Mailer implements Runnable {

        @Override
        public void run() {
            for (int i = 0; i < 5; i++) {
                System.out.println("inside " + Thread
                        .currentThread()
                        .getName() + " " + i);
            }
        }
    }
}
```

You should use runnable, as you are implementing, which allows you to extend
from another class if needed. You can have multiple implementations but only a
single super class – you can only be the subclass of one class.

## Synchronization

Threads share the same memory space, therefore they can share data. However, you
may want only one thread to have a resource at a given time. For example, in a
booking system with one seat left, there may be two threads reading the
remaining seats. When one thread books the last seat and the remaining seats are
zero, the other thread still thinks there is one seat left. This is known as a
race condition.

Logic around updating state can be put inside a synchronized block. Anything
inside it, can only be accessed/executed by a single thread at a time.

```java

public class SyncExample {
    Object lock; // Any type of object can be used - not primitives.

    public SyncExample() {
        lock = new Object();
    }

    public boolean push(int element) {
        synchronized (lock) {
            ++top;
            array[top] = element;
        }
    }

    /*
     When the entire method is synchronized, you can just add it as a 
     modifier. Now the lock is `this`, the current object.
     I.e. `synchronized(this) {}`. However, on static methods, it is the 
     clas. E.g. Singleton.class.
     
     public synchronized boolean push(int element) {
      ++top;
      array[top] = element;
     }
    */
}
```

If the synchronized keyword takes the same lock as another block with the same
lock, in another method, they are bound by the same lock object. Whichever
thread gets access to the lock, only that thread can access methods that have
those methods. When the lock objects are different, then they are not bounded
together. So one thread can call one method, another thread can call another
method on the same resource.

You can have multiple locks.

While a thread is using a synchronized method of an object, all other threads
wanting to use that method will have to wait. This restriction does not apply to
the thread that already has the lock, and it can call other synchronized methods
of the given object without being blocked. The non-synchronized methods can be
called at any time by any thread.

### Rules of Synchronization

- A thread must acquire the lock of a shared resource, before it can access it.
- The runtime ensures that no other thread can enter a shared resource if
  another thread holds its lock.
- If a thread cannot obtain the lock, it is blocked and waits till it is
  available.
- When a thread exits a shared resource, the runtime ensures that the lock is
  relinquished, allowing a blocked thread to try and acquire the lock.
- You should not make any assumptions about the order of which threads are
  granted a lock, as it is dependent on the JVM, CPU scheduling, runtime, etc.
- A subclass decides if the new definition of an inherited synchronized method
  will be synchronized or not. This is done by adding the keyword or using a
  `synchronized` block.
- If the lock passed into `synchronized()` evaluates to `null`, a
  `NullPointerException` is thrown.

### Static Synchronized Methods

- A thread acquiring the lock of a class to run a static synchronized methods
  does not block a thread acquiring the lock for any instance of the class. As
  the static methods lock will be `.class`, and an instance would use `this`.
- Thus, synchronization of static methods in a class is independent of the
  synchronization of instance methods.

### Race Condition

Occurs when two or more threads simultaneously update the same value, and as a
consequence, leave the value in an undefined or inconsistent state.

### Thread States

- New: Every thread is in this state until `.start()` is called on it.
- Active: When `.start()` is called on a thread
    - Substates: runnable and running
- Blockd: When waiting for another thread to finish.
- Terminated: When it has completed its task.

### Thread Priority

For example, if there are 10 threads in the runnable state and there is only one
CPU, how is the next thread to be run determined? This is decided by the Thread
Scheduler.

Each thread has a certain priority, and under normal circumstances the thread
with a higher priority will run. Priority values of 1-10 can be assigned to any
thread, with 1 being the lowest. By default, the priority is 5. Regading
"under normal circumstances", even the main thread has a priority of 5, but of
course, it must run first.

`MIN_PRIORITY`, `MAX_PRIORITY`, and `NORM_PRIORITY`.

Threads with the same priority are executed on a FIFO basis - the thread
scheduler is aware of the order as it stores threads in a queue.

## Thread Safety

Term used to describe the design of classes that ensure that the ste of their
objects is always consistent, even when the objects are used concurrently by
multiple threads. E.g.`StringBuffer`.

## Wait and Notify

The `wait` method pauses execution of a thread and allows other threads who
require the lock to execute. The `notify` method alerts threads that are
`wait`ing for a given lock that it is available.

See `WaitAndNotifyDemo.class`.

### Producer Consumer Problem

> The producer-consumer problem is a synchronization scenario where one or
> more producer threads generate data and put it into a shared buffer, while
> one or more consumer threads retrieve and process the data from the buffer
> concurrently"

See `ProducerConsumer.class`

## Executor Service

Google this

### Single Threaded Executor

By using this kind of thread pool, we can ensure the order in which tasks are
run due to the task queue adn there being a single thread. It is gauranteed the
tasks will run sequentially.

### Fixed Thread Executor

You can define a number of threads to be used by the executor service. As there
are n threads, order of execution can not be assumed.

If the thread is killed for whatever reason, a new thread will be created for
the remaining tasks.

### Cached Thread Pool Executor

Uses a synchronous queue. It only has space for one task, and searches for
threads that are created and not active. If no threads are available, the thread
pool creates a new thread to execute that task. If a thread is idle for more 
than 60 seconds, it will be killed, so it autoscales up/down in a way. Thus, 
the caching here is of the threads, as they are not instantly removed, they 
are idled.

### Ideal Thread Pool Size

It depends. Is the task CPU or IO intensive? A combination approach should 
be used. Also, if a CPU has 32 cores, and you build 300 threads, they'll be 
fighting for a slice of time, and the thread switching may become very 
expensive.

### Sequential Execution

## `Volatile`

