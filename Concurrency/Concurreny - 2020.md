## Concurreny

* Speaker: Arthur O'Dwyer
* Talk: [Concurreny](https://www.youtube.com/watch?v=F6Ipn7gCOsY&ab_channel=CppCon)

### What is concurrency?

* **Concurrency** means doing two things concurrently -- "running together". Switching back and forth between things
    * Writing slides and answering email

* **Parallelism** means doing two things in parallel -- simultanesouly.
    * Writing slides and answering email

* In broad terms, **parallelism** is a hardware problem (multiple CPUs) and concurrency is a software problem (time-sharing, multithreading, hyperthreading)

#### Starting a new thread

To create a thread, you create a `std::thread` object. The constructor argument is a callable that says what you want the thread to do. It can be a function, or a lambda.

```
std::thread threadB - std::thread([]{
    std::cout << "Hello from threadB!";
});
```

Newly created threads start executing immediately. When it has finished its job, it becomes **joinable**

Call `.join()` on the `std::thread` object before destroying it. This call will **block** if necessary until theother thread's job is finished.


```
std::thread threadB - std::thread([]{
    std::cout << "Hello from threadB!";
});
std::cout << "Hello from threadA";
threadB.join();
```

We don't need any special way to return an "exit status" from a thread, because joining with a child thread is a synchronizing operation.

```
int result = 0;
std::thread threadB = std::thread([&](){
    std::cout << "Hello from threadB!";
    result = 42;
});
std::cout << "Hello from threadA";
threadB.join();
std::cout << "The result of threadB was " << result;
```

threadA will **block** (wait) until threadB has finished. 

#### std::atomic

`std::atomic` can fix physical data races as every access to an atomic implicity synchronises with every other access to it.

```
using SC = std::chrono::steady_clock;
auto deadline = SC::now() + std::chrono::seconds(10);

std::atomic<int> counter = 0;

std::thread threadB = std::thread([&]{
    while (SC::now() < deadline)
        std::cout << "B: " << ++counter;
});

while (SC::now() < deadline)
    std::cout << "A: " << ++counter;
threadB.join();
```

The two accesses to read and write the counter syncronise so a datarace cannot occur. 

### std::mutex

If we want one thread to "wait" before starting it's work, we can use a `std::mutex`. A std::mutex is a *mutual* exclusion mechanism. 

As an analogy, it can be compared to the key to a bathroom. When Alice holds the key, Bob can't enter (and vice versa).

`std::mutex::lock()` "acquires" the bathroom key (waiting for it if necessary). 

`std::mutex::unlock()` returns it so the next person can use it.

```
std::mutex mtx;

mtx.lock(); //threadA holds the mutex

// start threadB
std::thread threadB = std::thread([&]{
    mtx.lock(); //waiting to lock the mutex
    mtx.unlock();
    std::cout << "Hello from B";
});

std::cout << "Hello from A";
mtx.unlock(); // Now thread B can start;
threadB.join();
std::cout << "Hello from A again!";
```

#### std::lock_guard

`std::lock_guard` is an RAII class that takes a template argument of what to lock, will manage that object's life time and if the object is destroyed will unlock it in the destructor. 

```
Token  getToken() {
    std::lock_guard<std::mutex> lk(mtx_);
    if(tokens.empty())
        tokens.push_back(Token::create());
        Token T = std::move(tokens.back());
        tokens.pop_back();
        return t;
}
```
if `Token::create()` were to throw, the lk would unlock the mutex in its destructor when it went out of scope.

A similar approach can be taken using `std::unique_lock<mutex>` which will manage the lifetime of the mutex. A unique_lock's ownership can also be passed or return to/from functions.

std::unique_lock<mutex> has both `lock` and `unlock` methods

a `condition_variable` can be used to notfiy a when a thread can be unblocked.
* Many threads can queue up on wait
* Calling `notify_one` unblocks exactly one waiter
* Calling `notify_all` unblocks all waiters
* `wait` always blocks

a `once_flag` and `call_once` can be used in a similar manner. 
* Many threads can queue up on call_once
* Failing at the callback unblocks exactly one waiter: the new "owner"
* Succeeding at the callback unblocks all waiters and sets the "done" flag.
* `call_once` blocks only if the "done" flag isn't set.