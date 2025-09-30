## RAII

* Speaker: Rainer Grimm
* Talk: [Object-Oriented Programming](https://youtu.be/Ou5WsJzn7Ws)

### Key Ideas

#### Class

A class must be within a programming language to be considered an OOP language.

C++ supports:

* Class
* Struct
* Union (ignored)

Class types **encapsulate** its members and member functions from the outside world
(information hiding)

Classes provide the separation of interface from implementation. The **interface**
is the **public members** of the class, the **implementation** is **"everything else"**.

* Encapsulation -
The process of binding together data and functions into a class. It prevents direct access to data.

* Abstraction -
Abstraction is the process of hiding internal implementations and giving access to only what is necessary.

#### Inheritance

An inheriting class:

* Gets all members and member function from the inherited Class
* Uses the members and the member functions of the inherited class and adds new ones
* A publicly derived class make use of the "__is-a__" relationship. The derived class is-a base class


* The access specifier of the inherited class and the access specifier of the inheritance must be considered.

``` c ++
#include <iostream>
#include <memory>

class Account
{
public:
    int pub{1};
protected:
    int prot{2};
private:
    int priv{3};
};
//---------------------------------------------------
class PubAccount
    : public Account
{
public:
    PubAccount()
    {
        // Access stays the same
        pub + prot; //public + protected
    }
};
//---------------------------------------------------
class ProtAccount
    : protected Account
{
public:
    ProtAccount()
    {
        //Access changes
        pub + prot; //protected + protected
    }
};
//---------------------------------------------------
class PrivAccount
    : private Account
{
public:
    PrivAccount()
    {
        //Access changes
        pub + prot; //private + private
    }
};
//---------------------------------------------------
int main()
{
    PubAccount  pubAccount;
    ProtAccount protAccount;
    PrivAccount privAccount;

    return 0;
};
```

* A classes members are by default **private**
* A struct's members are by default **public**


* When **constructing** a derived class, the **base class** is **constructed first**
  * Construction is Base -> Derived (top to bottom)


* When **destructing** a derived class, the **derived class** is **destructed first**
  * Destruction is Derived -> Base (bottom to top)

#### Polymorphism

Polymorphism is the characteristic of an object to behave **differently** at run time.

Polymorphism:
* Inheritance is the base of Polymorphism
* Enables the separation of interfaces and implementation
* Involves a small overhead (pointer indirection)

Allows derived classes to create different implementations of methods with the same signatures.

There are two types of polymorphism; compile time and runtime.

**Compile time**
* Is achieved through function overloading and operator overloading
* Is fast to execute as it is known at compile time

**Runtime**
* Achieved through virtual functions and pointers
* Slower than compile time

A **virtual** function is a function from a base class that can be overridden and redefined by a derived class. C++ determines which function is to be invoked at runtime based on the type of object pointed to by the base class pointer

``` c ++
struct Account {
    virtual void deposit() {
        std::cout << "Deposit in Account" << std::endl;
    }
};

struct BankAccount: Account
{
    void deposit() override {
        std::cout << "Deposit in BankAccount" << std::endl;
    }
};

int main()
{
    BankAccount bankAccount;

    // Here Account is a static type and bankAccount is a dynamic type
    Account* aPtr = &bankAccount;
    aPtr->deposit();

    Account& aRef = bankAccount;
    aRef.deposit();

    return 0;
};
```
Output
```
Deposit in BankAccount
Deposit in BankAccount
```

The overriding member function must be identical to the overridden virtual
function including the parameters, the return type and the `const` qualifiers

Pure virtual functions suppress the instantiation of a class and can have default
implementations.

``` c ++
// Here a Window class cannot be instantiated
struct Window {
  virtual void show() = 0;
};

void Window::show() { //implementation };
```

### Template Method - Behavioural pattern

Purpose:
* An algorithm consists of a typical sequence of steps.
* Subclasses can adapt the steps, but not the sequence

``` c ++
#include <iostream>
#include <memory>

class Sort
{
public:
    void ProcessData()
    {
        readData();
        sortData();
        writeData();
    }
private:
    virtual void readData() { std::cout << "default read" << std::endl; };
    virtual void sortData() = 0;
    virtual void writeData() { std::cout << "default write" << std::endl; };
};

//------------------------------------------
class QuickSort: public Sort
{
    void readData() override{
        std::cout << "Quick sort: readData" << std::endl;
    }

    void sortData() override{
        std::cout << "Quick Sort: sort" << std::endl;
    }

    void writeData() override{
        std::cout << "Quick Sort: writeData" << std::endl;
    }
};

//------------------------------------------
class BubbleSort: public Sort
{
    void sortData() override{
        std::cout << "Bubble Sort: sort" << std::endl;
    }
};
//------------------------------------------
int main()
{
    std::unique_ptr<Sort> sorter = std::make_unique<QuickSort>();
    sorter->ProcessData();

    std::cout << "" << std::endl;
    sorter = std::make_unique<BubbleSort>();
    sorter->ProcessData();

    return 0;
};
```

Output
```
Quick sort: readData
Quick Sort: sort
Quick Sort: writeData

default read
Bubble Sort: sort
default write
```

Don't call a virtual function from a constructor or destructor as the object is
not available at that time.

When a derived class is copied to a base class, the derived class becomes a base class.
