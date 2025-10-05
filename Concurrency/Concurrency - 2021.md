# Concurrency


* One mechanism for achieving concurrency is a `thread`
* A `thread` allows us to execute two **control flows** at the same time


* The main thread is where our program starts
  * We may then have one or more additional threads:
    * Executing a block of code
    * Executing other functions
    * Sharing the same code and the same data

### High-level view of threads

* One process (e.g. an application) can have many threads:
  * Each thread shares the same code, data and kernel context
  * A thread has its own thread id
  * A thread has its own logical flow
  * A thread has its on stack for local variables (its own storage that is unique to them)


### Thread Libraries

`std::thread()` - Creation of a thread, arguments passed are the callable and the callables args


`.join` - Join back to the main thread when the thread process has finished i.e "Main thread wait for me to finish my process before continuing."

`std::jthread` - Launches a thread and will attempt to join as soon as the destructor of the thread has been called. The thread is destroyed when it leaves scope.

```c++
#include <iostream>
#include <thread>

// Test function
void test(int x)
{
  std::cout << "Hello from our thread" << std::endl;
  std::cout << "Argument passed: " << x << std::endl;
}

int main(int argc, char const *argv[])
{
  // Create new thread, pass address of test and its parameter
  // This will start executing as soon as construction finishes
  std::thread myThread(&test, 100);

  // Join with the main thread, which is the same as
  //  saying "Hey main thread, wait until myThread finishes before
  // Executing any further"
  myThread.join();

  // Continue executing the main thread
  std::cout << "Hello from the main thread!" << std::endl;

  return 0;
}
```

### Thread pitfalls

Data races (or race conditions)
  * Because threads have access to shared data, one or more threads may try and write to the same piece of memory at the same time
    * One thread may have read a 'stale' value right before the new 'write' to the value
    * The thread that then writes will have updated a stale value, overwriting the other threads value.
  * This leads to non-deterministic operations

#### Locks
We can fix data races using a "lock". A lock protects data by only allowing one thread at a time to access memory.

In C++ we call these `mutexes` - They allow mutual exclusion on a block of code

*Analogy*:
* Think about having exactly one key to your home, and you always carry that key with you.
* Only the person who has the key can access the house
* When the person enters, they lock the door
* When the person leaves, they can pass on the key to someone else to enter, who will also lock the door when they enter

```c++
#include <mutex>

static int sharedValue {};
std::mutex gLock {};

void incrementSharedValue()
{
  gLock.lock();
    sharedValue++;
  glock.unlock();
}
```

The code between the locks is known as teh **critical secion**. All other threads are blocked from accessing this code whilst the mutex is locked.

The issue with locks is you can get into a **deadlock** state. Is this where a mutex is locked, but never unlocked.

This can be avoided by using a `std::lock_guard`. This uses RAII techniques to automatically unlock the mutex when the lock goes out of scope.

```c++
void incrementSharedValue()
{
  std::lock_guard<std::mutex> lg{ gLock };

  gLock.lock();
    sharedValue++;
  glock.unlock();
}
```

#### atomic
To avoid giving access to shared variables, we can use `std::atomic`. Any type that can be trivially copied can be wrapped as an atomic. If you are using just this trivial type, you can avoid using mutexes.

#### condition_variables
Given two threads (a worker and a reporter), we want one thread to wait on the result of the other.
`condition_variable` allows us to in a sense, take one thread off the queue, until it is notified that it should start working again.

```c++
std::mutex gLock {};
std::condition_variable gConditionVariable {};

int main(int argc, char const *argv[])
{
  AudioBuffer buffer {};
  bool notified {};

  std::thread consumer([&]
  {
    std::unique_lock<std::mutex> lock(gLock);

    gConditionVariable.wait(lock, [&]{ return notified; });

    doTHingWithBuffer(buffer);
  });

  std::thread producer([&]
  {
    buffer.fill(); // can be done outside lock if safe
    {
      std::lock_guard<std::mutex> lock(gLock);
      notified = true;
    }
    gConditionVariable.notify_one();
  });

  consumer.join();
  producer.join();

  return 0;
}

```

### Asynchronicity

std::async

std::promise

std::future
