# Pointers and Addresses

## Pointers

### Definition
* A `pointer` holds the address of another object
* A pointer type includes the type of object to which it points

```c++
int *pi;            // a "pointer to int"
unsigned long *pul; // a "pointer to unsigned long"
```

* For any object `x`, the expression `&x` returns the **address** of `x`
* If `x` has type `T`, then `&x` has type "pointer to `T`"
* i.e. `&x` gets the pointer of `x`

```c++
int i;
unsigned long ul;

int *pi = &i;
unsigned long *pul = &ul;

```

### Modifying pointers

* You can modify a pointers value

```c++
int a = 1, b=2;
int *p = &a;

p = &b;
```

* They can also point to "nothing"

```c++
int *p3 = nullptr; //unique type that converts to only pointers *
```

### De-referencing

* Use the unary `*` to dereference a pointer
* is `p` is a pointer, the `*p` is the **object to which it points**

```c++
int i = 13;
unsigned long ul = 42;

int *pi = &i;
unsigned long *pul = &ul;

*pi = 14;
*pul += 2;

```

### Object Lifetimes

```c++
// Takes a pointer as an argument
void f(int *pi) //pi's lifetime begins
{
  
}               //pi's lifetime ends

int i = 10;
f(&i) = 10; //de-reference i to get it's address (pointer)

```

* Each call to `f` creates a new instance of parameter `pi`
* Each instance exists only for the duration of the call

* The object `i` exists before the call and after the call to `f`

```c++
int *g() 
{
  int i = 0;      // i's lifetime begins
  
  return &i; // return address of i (pointer)
}                // i's lifetimes ends

int *pi = g(); // pi points to a non-existent variable

```

* In the example above, `pi` is now a **dangling pointer**
* Accessing `*pi` is UB

### Const correctness

* Starting with:

```c++
T *p
```

You can have any of

```c++
const T *p
T const *p
T *const p
const T *const p
T const *const p
```
---

`const T *p`
`T const *p`

If the `const` is on the **left hand side** of the `*` then it means `p` has type "pointer to const T"

The object p points to **can** be changed, but the underlying object itself **cannot** be modified

```c++
T x, y;
p = &x; // OK: Can modify pointer p
*p = y; // NO: Can't modify T object
```
---

`T const *p` 

If the `const` is on the **right hand side** of the `*` then it means `p` has type "const pointer to T"

The object p points to **cannot** be changed, but the underlying object itself **can** be modified

```c++
T x, y;
p = &x; // NO: Can't modify pointer p
*p = y; // OK: Can modify T object
```
---

`const T *const p`
`T const *const p`

If the `const` is on the **both sides** of the `*` then it means `p` has type "const pointer to const T"

The object p points to **cannot** be changed, and the underlying object itself **cannot** be modified


```c++
T x, y;
p = &x; // NO: Can't modify pointer p
*p = y; // NO: Can't modify T object
```
---

### Pointer conversions

* A pointer to a derived class object will safely convert to a pointer to a base class object

```c++
Derived derived;

Base *ptrBase = &derived;

```

## References

### Definition

* References provide an alternative to pointers as a way of associating names with objects

```c++
int i;

int &refi = i;
```

* The last line above:
  * Defines `refi` with type `reference to int`
  * initialises `refi` to refer to `i`
* Reference `ri` is an **alias** for `i`

* Once you create a reference, you can't change it to refer to something else:

```c++
int i, j;
int &ri = i; // binds ri to i
ri = j;      // assigns j to i THROUGH ri
// same as
i = j;
```

* Since you can't change what a refernece refers to, you must give it a value at the time you define it:

```c++
int &ri; // error: initialiser required
```
