## Smart Pointers

* Speaker: Rainer Grimm
* Talk: [Smart Pointers](https://www.youtube.com/watch?v=sQCSX7vmmKY&list=PLHTh1InhhwT5o3GwbFYy3sR7HDNRA353e&index=11&ab_channel=CppCon)

### Overview of Smart Pointers:

* Smart pointers automatically manage the lifetime of their resource
* They allocate and deallocate their resource according to the `RAII` idiom (**R**esource **A**cquisition **I**s **I**nitilisation)
  * RAII - You acquire a resource in the `constructor` and release it in the `destructor`.
* They support automatic memory management with reference counting
* Can be thought of as a kind of garbage collection in C++

| Name              | Description                                                                                                                                              |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| `std::unique_ptr` | • Owns the resource exclusively<br>• Cannot be copied<br>• Deals with non-copyable objects                                                               |
| `std::shared_ptr` | • Shares a resource<br>• supports a reference counter to the shared resource and manages it<br>• Deletes the resource if the reference counter becomes 0 |
| `std::weak_ptr`   | • Borrows the resource<br>• Helps break cyclic references<br>• Doesn't change the reference counter                                                      |


### std::unique_ptr

The `std::unique_ptr` exclusively manages the lifetime of it's resource.

* `std::unique_ptr`:
  * Doesn't support the copy semantic
  * Can be used in STL containers and algorithms

#### API

* `.release()` - Returns a pointer to the resource and releases it
* `.get()` - Returns a pointer to the resource
* `.reset(ptr)` - Resets the resource to a new one. If called without an argument, deletes the old resource
* `.get_deleter()` - Returns the deleter
* `std::make_unique(...)` - Creates the resource and wraps it in a `std::unique_ptr`

#### Example:
```c ++
#include <iostream>
#include <memory>
#include <utility>

struct MyInt {
  MyInt(int i) : i_(i) {}
  ~MyInt() {
    std::cout << "Good bye from " << i_ << std::endl;
  }
  int i_;
};


int main() {

  std::cout << std::endl;

  std::unique_ptr<MyInt> uniquePtr1 = std::make_unique<MyInt>(1998);

  std::cout << "uniquePtr1.get(): " << uniquePtr1.get() << std::endl;

  std::unique_ptr<MyInt> uniquePtr2{std::move(uniquePtr1)};
  std::cout << "uniquePtr1.get(): " << uniquePtr1.get() << std::endl;
  std::cout << "uniquePtr2.get(): " << uniquePtr2.get() << std::endl;

  std::cout << std::endl;

  {
    std::unique_ptr<MyInt> localPtr{new MyInt(2003)};
  } //localPtr goes out of scope here, so prints Goodbye from 2003 (as seen in output)   

  std::cout << std::endl;

  uniquePtr2.reset(new MyInt(2011));
  MyInt* myInt = uniquePtr2.release();
  delete myInt;

  std::cout << std::endl;
}
```

#### Output:
```
uniquePtr1.get(): 0x2014280
uniquePtr1.get(): 0         //uniquePtr1 has been changed to nullptr
uniquePtr2.get(): 0x2014280

Good bye from 2003

Good bye from 1998         //caused by reset of uniquePtr2
Good bye from 2011
```
***

### std::shared_ptr

The `std::shared_ptr` shares a resource and manages it's lifetime.

* `std::shared_ptr`:
  * Has a reference to the resource and the reference counter
  * Can be thought of as C++ answer to garbage collection
  * Deletes the resource
  * The access to the control block of the `std::shared_ptr` is thread-safe

#### API

* `.unique()` - Checks if the `std::shared_ptr` is the unique owner of the resource
* `use_count()` - Returns the value of the reference counter
* `.get()` - Returns a pointer to the resource
* `.reset(ptr)` - Resets the resource to a new one. If called without an argument, deletes the old resource
* `.get_deleter()` - Returns the deleter
* `std::make_shared(...)` - Creates the resource and wraps it in a `std::shared_ptr`

#### Example:
```c ++
#include <iostream>
#include <memory>

class MyInt {
public:
  MyInt(int v) : val(v) {
   std::cout << "  Hello: " << val << std::endl;
  }
  ~MyInt() {
   std::cout << "  Good Bye: " << val << std::endl;
  }
private:
  int val;
};

int main() {

  std::cout << std::endl;

  std::shared_ptr<MyInt> sharPtr(new MyInt(1998));

  std::cout << "sharedPtr.use_count(): " << sharPtr.use_count() << std::endl;
  {
    std::shared_ptr<MyInt> locSharPtr(sharPtr); //use count becomes two because of additional owner
    std::cout << "locSharPtr.use_count(): " << locSharPtr.use_count() << std::endl;
  }
  std::cout << "sharPtr.use_count(): "<<  sharPtr.use_count() << std::endl;

  std::shared_ptr<MyInt> globSharPtr = sharPtr;
  std::cout << "sharPtr.use_count(): "<<  sharPtr.use_count() << std::endl;

  globSharPtr.reset();
  std::cout << "sharPtr.use_count(): "<<  sharPtr.use_count() << std::endl;

  sharPtr = std::shared_ptr<MyInt>(new MyInt(2011));

  std::cout << std::endl;

}
```

#### Output:
```
Hello: 1998

sharedPtr.use_count(): 1
locSharPtr.use_count(): 2

sharPtr.use_count(): 1
sharPtr.use_count(): 2
sharPtr.use_count(): 1
Hello: 2011
Good Bye: 1998

Good Bye: 2011
```

***

### std::weak_ptr

The `std::weak_ptr` is not a classic smart pointer.

* `std::weak_ptr`:
  * Owns no resource
  * Borrows the resource from a `std::shared_ptr`
  * Cannot access the resource
  * Can create a `std::shared_ptr` to the resource

**Note**: The `std::weak_ptr` doesn't change the reference counter.
  * It helps break cycles of `std::shared_ptr`

#### API

* `.expired()` - Checks if the resource exists
* `use_count()` - Returns the value of the reference counter
* `.lock()` - Creates a `std::shared_ptr` to the resource if available
* `.reset(ptr)` - Releases the resource

#### Example:
```c ++
#include <iostream>
#include <memory>

int main() {

  std::cout << std::boolalpha << std::endl;

  auto sharedPtr = std::make_shared<int>(2011);
  std::weak_ptr<int> weakPtr(sharedPtr);

  std::cout << "weakPtr.use_count(): " << weakPtr.use_count() << std::endl;
  std::cout << "sharedPtr.use_count(): " << sharedPtr.use_count() << std::endl;
  std::cout << "weakPtr.expired(): " << weakPtr.expired() << std::endl;

  if( std::shared_ptr<int> sharedPtr1 = weakPtr.lock() )
  {
    std::cout << "*sharedPtr: " << *sharedPtr << std::endl;
  }
  else
  {
    std::cout << "Don't get the resource!" << std::endl;
  }

  weakPtr.reset();
  if( std::shared_ptr<int> sharedPtr1 = weakPtr.lock() ) //check fails because weakPtr has been reset.
  {
    std::cout << "*sharedPtr: " << *sharedPtr << std::endl;
  }
  else
  {
    std::cout << "Don't get the resource!" << std::endl;
  }

  std::cout << std::endl;

}
```

#### Output:

```
weakPtr.use_count(): 1
sharedPtr.use_count(): 1
weakPtr.expired(): false
*sharedPtr: 2011
Don't get the resource!
```

### Functions

Ownership semantic for function parameters

| Name              | Description                                                                                             |
|-------------------|---------------------------------------------------------------------------------------------------------|
| `func(value)`     | • Is an independent owner of the resource<br>• Deletes the resource automatically at the end of `func`  |
| `func(ptr*)`      | • Borrows the resource<br>• The resource could be empty<br> • Must not delete the resource              |
| `func(ref&)`      | • Borrows the resource<br>• The resource could not be empty<br> • Must not delete the resource          |
| `func(unique_ptr)`| • Is an independent owner of the resource<br>• Deletes the resource automatically at the end of `func`  |
| `func(shared_ptr)`| • Is a shared owner of the resource<br>• May delete the resource at the end of `func`                   |
