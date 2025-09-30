## Pointers

* Speaker: Mike Shah
* Talk: [Object-Oriented Programming](https://youtu.be/0zd8eznWv4k)

### What is a pointer?

* A pointer is a variable that stores the **memory address** of a specific object type.

``` c ++
int x = 7;
int* px = &x;
```

* `px` is a pointer
* `px`'s type is `int*`
  * `px` can store the address of integers
* `&` operator retrieves the address of a variable

![til](./pointersAnimation.gif)

* **Dereferencing** a pointer is accessing the address stored in our pointer and accessing
the value pointed to by our pointer.

* When the asterisk `*` is before a pointer variable, it **retrieves** the value at this address
`*px`

``` c ++
int x = 7;
int* px = &x;

std::cout << "x value        : " << x   << std::endl;
std::cout << "x address      : " << &x  << std::endl;
std::cout << "px points to   : " << px  << std::endl;
std::cout << "px dereferenced: " << *px << std::endl;
```
Output
```
x value        : 7
x address      : 0x16fdff25c
px points to   : 0x16fdff25c
px dereferenced: 7
```

#### Pass by pointer

Pass by pointer is essentially pass by value, however the address of the variable
is being copied as opposed to the variable itself, meaning any changes will be made
to the original value stored at the address pointed to.

``` c ++
#include <iostream>

void passByValue(int x){
    x = 9999;
}

void passByPointer(int* intPointer){
    *intPointer = 9999;
}

int main()
{
    int x = 5;
    int y = 6;
    passByValue(x);
    std::cout << "x is now: " << x << std::endl;
    passByPointer(&y);
    std::cout << "y is now: " << y << std::endl;

    return 0;
}
```
Output
```
x is now: 5
y is now: 9999
```
### Pointer pit falls

#### Memory leaks

``` c ++
int main(){

  //Not the worst, but not ideal
  int* memory = new int [1000];

  while(1){
    //Very bad... lots of allocations
    int* lotsOfAllocations = new int [1];
  }

  return 0;
};
```
Once the program ends, the memory being allocated is never deleted, so you end up
with memory leaks.

#### Dangling Pointers

* Dangling pointers arise when we point to the address of a value that may not exist.
* We try to avoid pointing to data that does not have the same lifetime as out pointer.

``` c ++
char* dangerouslyReturnLocalValue(){
        char c = 'c';
        return &c;
}

int main(){

    // The internal char c of the function is out of scope here, so who knows what it will return
    char* danglingPointer1 = dangerouslyReturnLocalValue();

    std::cout << "*danglingPointer1 is: " << *danglingPointer1 << std::endl;

    return 0;
}
```

#### Double free

* A double free occurs when we are sharing data between 2 or more pointers.
* We are _trying_ to be good and free our memory
  * The issue is we end up freeing the same memory twice

``` c ++
int main(){

    float* f1 = new float[100];
    float* f2 = f1;

    delete[] f2;
    f2 = nullptr;
    delete[] f1;
    // Be good and set f1 to nullptr
    f1 = nullptr;
    // Did I delete f2? I'll try again
    delete[] f2;    

    return 0;
}
```
