# Basic Multithreading 

When you are creating applications that either have long-running jobs, large datasets, deploy a GUI with background tasks, or need services that provide some functionality-on-request, it may be wise to use the Java concurrency API. The mother classes serving this API, `Thread` and `Runnable`, can be found in package `java.lang`, while the extended API classes are located in package `java.util.concurrent`.

Any single thread needs two things: a `Thread` instance and an object defining the job to be run, a `Runnable` implementation.

Below is an example of the most basic use of both. First, a job blueprint is defined (a simple message print):

```java
package nl.bioinf.multithreading;

public class SimpleWorker implements Runnable {
    private String name;

    public SimpleWorker(String name) {
        this.name = name;
    }

    @Override
    public void run() {
         System.out.println("SimpleWorker '" + name + "' doing its thing");
    }
}
```

The Runnable interface defines only a single method, `run()`. This is the method that will be called by its owner Thread instance when it deems hte Runnable's moment has come to be executed.

This is how the job can be executed:

```java
package nl.bioinf.multithreading;

public class SimpleMultithreadingDemo {
    public static void main(String[] args) {
        SimpleWorker firstWorker = new SimpleWorker("My First Worker");
        Thread firstThread = new Thread(firstWorker);
        firstThread.start();
    }
}
```

This will output

<pre class="console_out">
SimpleWorker 'My First Worker' doing my thing
</pre>

Note that the `run()` method is a plain Java method; it can also be run without being held by a Thread; however, it will then be executed in main the process stack. 

```{admonition} Thread.run() does not start a Thread
:class: warning

The Thread class has two methods that will run the given Runnable job.  

If the run() method is called directly instead of start() the method, run() will be treated as a normal overridden method of the Thread class. This run method will be executed within the context of the current thread, not in a new thread.

Why then, you may ask, is there such a confusing method? Because it implements the Runnable interface itself.
```

## Concepts and key terminology

Multithreading is a fundamental concept in computer science and software development that involves the simultaneous execution of multiple threads within a single process. To understand multithreading, let's break down some key terms:

1. **Thread**: A thread is the smallest unit of execution within a process. _Threads share the same memory space and resources as the process to which they belong but have their own program counter and stack_, which allows them to run independently. Threads within a process can perform different tasks concurrently.

2. **Process**: A process is an independent program that runs on a computer. Each process has its own memory space, system resources, and at least one thread. Processes are isolated from each other and do not share memory or resources by default. 

3. **Concurrency**: Concurrency is the concept of making progress on multiple tasks at the same time. In the context of multithreading, concurrency means that multiple threads are executing their tasks simultaneously. This can lead to more efficient resource utilization and improved responsiveness in applications, as tasks can overlap rather than running sequentially.

4. **CPU Core**: A CPU core is a physical processing unit within a computer's central processing unit (CPU). Modern CPUs often have multiple cores, allowing them to execute multiple threads simultaneously. Multithreading can take advantage of multiple CPU cores to achieve true parallelism and further improve performance.

Multithreading is widely used in software development to enhance the performance and responsiveness of applications, particularly in tasks where certain operations can be executed concurrently without waiting for others to complete. It allows developers to leverage the power of multiple CPU cores and achieve efficient use of system resources, ultimately leading to faster and more responsive software. However, managing threads can be complex, and care must be taken to avoid issues like race conditions and deadlocks.

## User Threads and Daemon threads
In Java, there are two types of threads: user threads and daemon threads. These threads have different characteristics and behaviors, which are important to understand when working with multithreaded Java applications:

**User Threads**
User threads are the most common type of threads in Java. They are created by the application programmer and are typically responsible for carrying out application-specific tasks. User threads are not terminated when the main method of the program completes. They continue running until they complete their tasks or are explicitly terminated by the programmer. The JVM (Java Virtual Machine) will not exit until all user threads have finished executing. This ensures that the program keeps running as long as user threads are active.

**Daemon Threads**  
Daemon threads are a special type of threads in Java. They are typically used for supporting tasks or services in the background, and they are meant to be non-essential for the functioning of the application. Daemon threads are automatically terminated when all user threads have finished execution and the main method of the program exits. They do not prevent the JVM from exiting. A common use case for daemon threads is for tasks like garbage collection, monitoring, and clean-up operations that can run in the background but are not critical for the application to function.  

Below is an example to illustrate the difference (note the use of lambdas to implement the Functional interface Runnable).
A thread is marked daemon by calling `setDaemon(true)`.

```java
public class UserAndDaemonThread {
    public static void main(String[] args) {
        Thread userThread = new Thread(() -> {
            for (int i = 1; i <= 5; i++) {
                System.out.println("User Thread - Iteration " + i);
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread daemonThread = new Thread(() -> {
            while (true) {
                System.out.println("Daemon Thread - Running in the background");
                try {
                    Thread.sleep(400);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        daemonThread.setDaemon(true); // Set the thread as a daemon.

        userThread.start();
        daemonThread.start();

        System.out.println("Main Method - Exiting");
    }
}
```

Here is the output:

<pre class="console_out">
Main Method - Exiting
Daemon Thread - Running in the background
User Thread - Iteration 1
User Thread - Iteration 2
User Thread - Iteration 3
User Thread - Iteration 4
Daemon Thread - Running in the background
User Thread - Iteration 5
</pre>

As you can see, the Main method exits first. Because User Thread is still running, the process does not finish. However, when the User Thread finishes, so does the daemon Thread, even though it did not finish.

It's important to be cautious when working with daemon threads because they can be abruptly terminated, which might not be suitable for certain tasks that require proper cleanup or finalization.

## The threat of Threads

There are three major issues to consider when creating multithreaded applications:  

- You never know which thread will be selected to run, and you do not know the order in which they will finish - **UNCERTAINTY**. This requires another way of designing your algorithms, usually through the use of callback function (the observer pattern discussed later).
- Concurrent modification of shared objects – **DATA CORRUPTION**. Threads run independently on their own call stack. On the other hand, they share main memory. This opens the door to data corruption, when two threads try to modify a shared collection or other object. More on this topic later.
- Deadlocks – threads endless waiting on each other – **HANGING APP**. A deadlock is usually a very rare phenomenon, and very hard to debug. Using synchronized blocks in a sensible manner helps a lot. It is however not discussed here.

## The Uncertainty principle

Below is an extension of the first example of this chapter, with the same SimpleWorker class:

```java
public class SimpleMultithreadingDemo {
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            SimpleWorker worker = new SimpleWorker("_" + i + "_");
            Thread t = new Thread(worker);
            t.start();
        }
    }
}
```

which outputs (on my machine)

<pre class="console_out">
SimpleWorker '_1_' doing its thing
SimpleWorker '_3_' doing its thing
SimpleWorker '_4_' doing its thing
SimpleWorker '_2_' doing its thing
SimpleWorker '_0_' doing its thing
</pre>

As you can see, the output is not the same order in which the threads were created. 

```{admonition} No guarantees on thread execution
:class: warning

There is absolutely no guarantee on the order in which threads will execute. Moreover, threads can be suspended mid-operation to give way to another thread, to be restarted at a later point. Even though you may get reproducible results on your machine (JVM), this is not transferable to other machines (JVMs/OS-es).
```

## The Data Corruption danger

Suppose we have some kind of banking app in its most rudimentary form:

```java
    static class Account {
        int balance;

        public Account() {
            this.balance = 100;
        }

        boolean canWithdraw(int amount) {
            return balance - amount >= 0;
        }

        void withdraw(int amount) {
            this.balance -= amount;
            if (this.balance < 0) throw new IllegalStateException("Balance is negative: " + this.balance + '!');
        }
    }
```

Here is the Worker acting on the shared (static) account:

```java
public class SimpleWorker implements Runnable {
    private String name;
    private static Account account = new Account();

    public SimpleWorker(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println("SimpleWorker '" + name + "' doing its thing");
        final int toWithdraw = 15;
        if (account.canWithdraw(toWithdraw)) {
            try {
                Thread.sleep(10); // Simulating another Thread gets priority
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            account.withdraw(toWithdraw);
        }
        System.out.println("balance = " + account.balance);
    }
}
```

When I run this code 

```java
    for (int i = 0; i < 10; i++) {
        SimpleWorker worker = new SimpleWorker("_" + i + "_");
        Thread t = new Thread(worker);
        t.start();
    }
```

I get 

<pre class="console_out">
SimpleWorker '_4_' doing its thing
SimpleWorker '_7_' doing its thing
SimpleWorker '_1_' doing its thing
SimpleWorker '_3_' doing its thing
SimpleWorker '_5_' doing its thing
SimpleWorker '_0_' doing its thing
SimpleWorker '_8_' doing its thing
SimpleWorker '_9_' doing its thing
SimpleWorker '_2_' doing its thing
SimpleWorker '_6_' doing its thing
balance = 70
balance = 85
balance = 40
balance = 55
balance = 10
balance = 25
balance = 40

Exception in thread "Thread-4" Exception in thread "Thread-1" Exception in thread "Thread-7" java.lang.IllegalStateException: Balance is negative: -5!
	at nl.bioinf.multithreading.SimpleWorker$Account.withdraw(SimpleWorker.java:41)
	at nl.bioinf.multithreading.SimpleWorker.run(SimpleWorker.java:22)
	at java.base/java.lang.Thread.run(Thread.java:833)
java.lang.IllegalStateException: Balance is negative: -20!
	at nl.bioinf.multithreading.SimpleWorker$Account.withdraw(SimpleWorker.java:41)
	at nl.bioinf.multithreading.SimpleWorker.run(SimpleWorker.java:22)
	at java.base/java.lang.Thread.run(Thread.java:833)
java.lang.IllegalStateException: Balance is negative: -20!
	at nl.bioinf.multithreading.SimpleWorker$Account.withdraw(SimpleWorker.java:41)
	at nl.bioinf.multithreading.SimpleWorker.run(SimpleWorker.java:22)
	at java.base/java.lang.Thread.run(Thread.java:833)
</pre>

As you can see, there are not one, but two overdrawing operations, leaving the bank account at -20!  
This is because `account.canWithdraw(toWithdraw)` and `account.withdraw(toWithdraw)` are not coupled into a single transaction (atomic operation) making it possible for other Threads to come in between when the permission has been "granted" but before the actual withdraw has been executed.

To prevent this, we need to put all statements that need to be executed "as one" into a `synchronized` block. 
Here is the safe way to do this.

```java
    @Override
    public void run() {
        /*put a lock on it!*/
        synchronized (SimpleWorker.class) {
            System.out.println("SimpleWorker '" + name + "' doing its thing");
            final int toWithdraw = 15;
            if (account.canWithdraw(toWithdraw)) {
                try {
                    //simulating a forced change of thread execution
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                account.withdraw(toWithdraw);
            }
            System.out.println("balance = " + account.balance);
        }
    }
```

This gives the following output:

<pre class="console_out">
SimpleWorker '_0_' doing its thing
balance = 85
SimpleWorker '_9_' doing its thing
balance = 70
SimpleWorker '_8_' doing its thing
balance = 55
SimpleWorker '_6_' doing its thing
balance = 40
SimpleWorker '_7_' doing its thing
balance = 25
SimpleWorker '_5_' doing its thing
balance = 10
SimpleWorker '_4_' doing its thing
balance = 10
SimpleWorker '_3_' doing its thing
balance = 10
SimpleWorker '_2_' doing its thing
balance = 10
SimpleWorker '_1_' doing its thing
balance = 10
</pre>

There are multiple ways to to make use of the synchronized keyword; this is a very common one.
The block `synchronized (SimpleWorker.class) {}` is marked synchronized, or locked, meaning that only a thread holding the key will be able to enter. The key can be any object, but usually this will be the class object of the current executing class, which is guaranteed to be present in single copy during runtime (singleton).

