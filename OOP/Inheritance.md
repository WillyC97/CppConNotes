# Inheritance

### What is Inheritance?

* Inheritance allows a class (derived) to **reuse and extend** the functionality of another class (base)

```c++
class Base { /* ... */ };

class Derived : public Base { /* ... */ };
```

### Access Specifiers

* Access specifiers (`public`, `protected`, `private`) in inheritance control how base members are accessible in the derived class.

* Most common: `public` inheritance (**"is-a"** relationship)

### Constructors and Destructors

* Constructor order: **Base** class runs **before** the **derived** constructor
* Destructor order: **Derived** class destructor runs **before** the **base**
* For Polymorphism, always declare a virtual destructor in the base class
  * Otherwise deleting a derived object via a base pointer can cause UB

### Overriding vs Hiding

* Overriding: Derived class provides a new implementation for a **virtual** function in the base:

```c++
class Base
{
public:
    virtual void foo() { std::cout << "Base\n"; }
};

class Derived : public Base
{
public:
    void foo() override { std::cout << "Derived\n"; }
};

```

### Final and Override
* `override` ensures you are actually overriding a base virtual function
* `final` prevents further overriding

```c++
class Derived final : public Base {};
void foo() override final;
```
### Virtual functions an Polymorphism

* Key to runtime Polymorphism
* Polymorphism allows treating objects of different derived types uniformly via a base class interface.
* Base pointer/reference can call **derived implemention** via a virtual function

```c++
class Base {
public:
    virtual void speak() { std::cout << "Base speaks\n"; }
};

class Derived : public Base {
public:
    void speak() override { std::cout << "Derived speaks\n"; }
};

int main() {
    std::unique_ptr<Base> b = std::make_unique<Derived>();
    b->speak(); // Derived speaks
}
```

### Object Slicing

* Object slicing occurs when a **derived** object is assigned to a **base** object by **value** (copy assignment)
* The derived part of the object is "sliced off" leaving only the base
* Pointers or references avoid slicing

```c++
class Base {
public:
    int baseValue;
    virtual void info() { std::cout << "Base: " << baseValue << "\n"; }
};

class Derived : public Base {
public:
    int derivedValue;
    void info() override { std::cout << "Derived: " << derivedValue << "\n"; }
};

int main() {
    Derived d;
    d.baseValue = 1;
    d.derivedValue = 42;

    Base b = d; // Slicing happens here
    b.info();   // Calls Base::info(), derivedValue lost

    Base* bptr = &d; // No slicing
    bptr->info();    // Calls Derived::info()
}
```

### Multiple Inheritance

* C++ allows a class to inherit from **more than one base class**

```c++
class A { };
class B { };
class C : public A, public B { };
```

#### The diamond problem

* Occurs when two base classes share a common ancestor:

```c++
class A
{
public:
    void foo() { std::cout << "A::foo\n"; }
};

class B : public A { };
class C : public A { };

class D : public B, public C { };

```

* `D` now has two copies of `A`, one via `B` and one via `C`
* Calling `d.foo()` is ambiguous - the compiler doesn't know which `A` to call

#### The solution: virtual inheritance

```c++
class A { };
class B : virtual public A { };
class C : virtual public A { };
class D : public B, public C { };

```

* Virtual inheritance ensures only one shared instance of `A` in `D`.
* Eliminates ambiguity and multiple copies of the common ancestor.

### Key points

* Multiple inheritance is powerful but dangerous:
  * Can introduce ambiguity (diamond problem).
  * Can complicate memory layout.
* Use **virtual inheritance** to resolve diamond issues.
* Often, composition ("has-a") is safer than multiple inheritance unless modeling true “is-a” relationships.

### What is Composition?

* Composition is a design principle where a class is built by containing other objects as members, rather than inheriting from them.
