## RAII

* Speaker: Arthur O'Dwyer
* Talk: [RAII and the Rule of Zero](https://www.youtube.com/watch?v=7Qgd9B1KuMQ&ab_channel=CppCon)

### Overview
**R**esource **A**cquisition **I**s **I**nitilisation

A "resource" is anything that requires special (manual) management

Examples include:

* Allocated memory
* POSIX file handles (open/close)
* Mutex locks (pthread_mutex_lock/pthread_mutex_lock_unlock)
* C++ threads (spawn/join)
* Resource counted objects (retain/release)

They all require some explicit action that needs to be taken to free the resource.

### Naive implementation of vector

``` C ++
class NaiveVector {
    int *ptr_;
    size_t size_;
public:
    NaiveVector() : ptr_(nullptr), size_(0) {}
    void push_back(int newvalue) {
        int *newptr = new int[size_ + 1];
        std::copy(ptr_, ptr_ + size_, newptr);
        delete [] ptr_;
        ptr_ = newptr;
        ptr_[size_++] = newvalue;
    }
};

{
    NaiveVector vec;  // here ptr_ is initialized with 0 elements
    vec.push_back(1); // ptr_ is correctly updated with 1 element
    vec.push_back(2); // ptr_ is correctly updated with 2 elements
}
```

The issue with the above however is that the push_back allocates memory on the heap.
When `vec` goes out of scope, the heap memory isn't deleted therefore ends in a resource leak.

#### The destructor

* When any object of class type is created, the compiler generates a call to a
constructor of that type.
* Likewise, when any object’s lifetime ends, the compiler generates a call to the
___destructor___ of that type.

``` C ++
{
   NaiveVector vec; // here the constructor is called
}  <----------      // here the destructor is called
```

add a destructor to `NaiveVector`

``` c ++
  ~NaiveVector() { delete [] ptr_; }
```
#### Copy Constructor

`NaiveVector` does however still have bugs.

``` c ++
{
    NaiveVector v;
    v.push_back(1);
    {
        NaiveVector w = v;
    }
    std::cout << v[0] << "\n";
}
```

The line `NaiveVector w = v;` invokes the implicitly generated copy constructor
of NaiveVector. This defaulted copy constructor simply copies each member.

As `w` is a copy of `v`, when it goes out of scope, it deletes the memory on the
heap it was managing. This means that the final line tries to access memory that
has already been freed. This is **undefined behaviour**.

To avoid this, a copy constructor should be written. The copy constructor
is responsible for duplicating resources to avoid double frees.

``` c ++
NaiveVector(const NaiveVector& rhs)
{
    ptr_ = new int[rhs.size_];
    size_ = rhs.size_;
    std::copy(rhs.ptr_, rhs.ptr_ + size_, ptr_);
}
```

#### Copy assignment operator

Initialization is not assignment.

* If an object is being created, it is initialized with `=`.
``` c ++
  NaiveVector w = v; //This is an initialization (construction)
                     // of a new object.
                     // It calls a copy constructor.
```

* If an object already exists, it is being assigned with `=`
``` c ++
    NaiveVector w; //This is an assignment
    w = v;         // to the exisiting object w
                   // It calls an assignment constructor.
```

Assignment has the same problem as copy, it calls the default assignment operator
which simply copy-assigns each member.

Using the `copy and swap` idom.

``` c ++
NaiveVector& operator=(const NaiveVector& rhs)
{
    NaiveVector copy = rhs;
    copy.swap(*this);
    return *this;
}
```

### The Rule of Three

If your class directly manages some kind of resource (such as a
new’ed pointer), then you almost certainly need to hand-write three
special member functions:

* A **destructor** to free the resource
* A **copy constructor** to copy the resource
* A **copy assignment** operator to free the left-hand resource
and copy the right-hand one

When a function definition has the body `= delete;` instead of a curly-braced compound
statement, the compiler will reject calls to that function at compile time.

When a special member function definition has the body `= default;` instead of a
curly-braced compound statement, the compiler will create a defaulted version of that
function, just as if it were implicitly generated.

**Explicitly defaulting** your special members can help your code to be self-documenting.

### The Rule of Zero

If your class does not directly manage any resource, but merely
uses library components such as vector and string, then you should
strive to write no special member functions.
Default them all!

* Let the compiler implicitly generate a defaulted destructor
* Let the compiler generate the copy constructor
* Let the compiler generate the copy assignment operator
(But your own swap might improve performance)

This is known as the **Rule of Zero**

### rvalue references

The terms “lvalue” and “rvalue” come from the syntax of assignment
expressions. An **lvalue** can appear on the **left-hand side** of an
assignment; an **rvalue** must appear on the **right-hand side**.

``` c ++
x = 1;
*p = 1;
a[2] = 1;

// x, *p and a[2] are all lvalues, 1 is an rvalue.
```

* `int&` is an lvalue reference to an int.
* `int&&` (two ampersands) is an rvalue reference to an int.
* As a general rule, lvalue reference parameters do not bind to
rvalues, and rvalue reference parameters do not bind to lvalues.
* Special case for backward compatibility: a `const` lvalue reference
will happily bind to an rvalue.

``` c ++
void l_value_ref(int&);           
void r_value_ref(int&&);          
void const_l_value_ref(const int&);     

l_value_ref(i);       // OK     
r_value_ref(i);       // ERROR
const_l_value_ref(i); // OK     

l_value_ref(42);       // ERROR
r_value_ref(42);       // OK
const_l_value_ref(42); // OK
```

An example with overload resolution:

``` c ++
void foo(const std::string&); // takes lvalues
void foo(std::string&&);      // takes rvalues

std::string s = "hello";
foo(s);                  // calls foo #1
foo(s + " world");       // calls foo #2
foo("hi");               // calls foo #2
foo(std::move(s));       // calls foo #2
```

#### Move Constructor

The most common application of rvalue references is the **move constructor**.
``` c ++
NaiveVector(NaiveVector&& rhs)
{
    ptr_  = std::exchange(rhs.ptr_, nullptr);
    size_ = std::exchange(rhs.size_, 0);
}
```

### Rule of Five

If your class directly manages some kind of resource (such as a
new’ed pointer), then you may need to hand-write five special
member functions for correctness and performance:

* A **destructor** to free the resource
* A **copy constructor** to copy the resource
* A **move constructor** to transfer ownership of the resource
* A **copy assignment operator** to free the left-hand resource and copy
the right-hand one
* A **move assignment operator** to free the left-hand resource and
transfer ownership of the right-hand one

### No longer NaiveVector

Putting it all together, you get the following Vector class:

``` c ++
class Vec
{
    //A copy constructor, to copy the resource (avoid double-frees)
    Vec(const Vec& rhs)
    {
        ptr_ = new int[rhs.size_];
        size_ = rhs.size_;
        std::copy(rhs.ptr_, rhs.ptr_ + size_, ptr_);
    }

    // A move constructor, to transfer ownership of
    // the resource (cheaper than copying)
    Vec(Vec&& rhs) noexcept
    {
        ptr_  = std::exchange(rhs.ptr_, nullptr);
        size_ = std::exchange(rhs.size_, 0);
    }

    // A two-argument swap, to make your type efficiently “std::swappable”
    friend void swap(Vec& a, Vec& b) noexcept
    {
        a.swap(b);
    }

    ~Vec() // A destructor, to free the resource (avoid leaks)
    {
        delete [] ptr_;
    }

    // An assignment operator, to free the left-hand resource
    // and transfer ownership of the right-hand one
    Vec& operator=(Vec copy)
    {
        copy.swap(*this);
        return *this;
    }

    // A member swap too, for simplicity
    void swap(Vec& rhs) noexcept
    {
        using std::swap;
        swap(ptr_, rhs.ptr_);
        swap(size_, rhs.size_);
    }
};
