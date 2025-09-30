## Pointers and References

* Speaker: Ben Saks
* Talk: [Pointers and Memory](https://www.youtube.com/watch?v=rqVWj0aVSxg&ab_channel=CppCon)

### Pointers and Addresses

#### Pointers
* A pointer object holds the **address** of another object
* A pointer type includes the type of object to which it points

```cpp
int *pi;            // a pointer to int
unsigned long *pul  // a pointer to unsigned long
```

#### Addresses
* For any object x, the expression `&x`returns the address of x.
* if x has type `T`, then &x has type "pointer to T".

```cpp
int i;
unsigned long ul;

int *pi = &i;
unsigned long *pul = &ul;
```

#### Pointer Dereferencing
* You can modify a pointers value to point to different address throughout its lifetime.
* A pointer can be null and the preferred method for this is `nullptr`

* You use the unary `*` operator to dereference a pointer
* If p is a pointer then `*p` is the object to which it points

```
int i = 13;
unsigned long ul = 42;
int *pi = &i;
unsigned long *pul = &ul;

*pi = 14  // This assigns the object pointed to, to 14 (in this case i)
*pul +=2
```

### Arrays and Pointer Arithmetic

* You can also access the elements of an array through pointers

```cpp
char x[N];
char *pc = &x[0];

*pc ='a';    // same as: x[0] = 'a'
```

* Incrementing a pointer to an array element causes it to point to the next element:

```
++pc;      // pc now points to x[1];
*pc = 'c'  // same as: x[1] = 'c';
```

* Adding an integer to a pointer yields another pointer

```cpp
int k;
t *p, *k;

q = p + k
```

* You can also treat arrays as pointers themselves

`int x[N];`

* `x` can be used as the initialiser for a pointer in place of &x[0]

```
int *pi = x;  // same as pi = &x[0]
```

* Similarly you can dereference `x` as if it were a pointer

```cpp
*x = 4;  // same as: x[0] = 4;
```

### Placing const in Pointer Declarations

* Starting with `T *p`

* you can add `const` to produce any of:


1. `const T *p`       
2. `T const *p`

These mean you can modify the pointer `p`, but cannot modify the object p points to

3. `T *const p`

This means you cannot modify the pointer, but you can modify the object that p points to

4. `const T * const p`
5. `T const *const p`

These mean neither are modifiable

### Pointer Conversions and Casts

* A pointer to a derived class object will safely convert to a pointer to a base class object:

```cpp
Derived d;
Base *pb = &d;
```

### References

* References provide an alternative to pointers as  away of associating names with objects.

```cpp
int i;
~~~
int &ri = i;
```

* The last line above:
  * defines `ri` with type "refernce to int", and
  * initialises `ri` to refer to `i`
* The Reference `ri` is an **alias** for `i`

| Reference Notation       | Equivalent Pointer Notation  |
|--------------------------|------------------------------|
| `int &ri = i;`           | `int *const cpi = &i;`       |
| `ri = 4;`                | `*cpi = 4;`                  |
| `int j = ri + 2;`        | `int j = *cpi + 2;`          |


* Once you create a reference, you cannot change it to refer to something else
