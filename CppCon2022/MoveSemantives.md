## Move Semantics

* Speaker: Andreas Fertig
* Talk: [C++ Move Semantics](https://www.youtube.com/watch?v=knEaMpytRMA&t=229s&ab_channel=CppCon)

### Overview of Move Semantics:

```
void Fun(std::vector<int>& byRef)
{
    std::cout << "byRef\n";
}

void Fun(const std::vector<int>& byConstRef)
{
    std::cout << "byConstRef\n";
}
```

```
void use()
{
    std::vector v {2, 3, 4};
    const std::vector cv {5, 6, 7};

    Fun(v);             // We pass an lvalue
    Fun(cv);            // We pass a const lvalue
    Fun({3, 5, 6});     // We pass a temporary
}
```

Output of above:
```
$ ./a.out
byRef
byConstRef
byConstRef
```

Add an additional overload for **Move**

```
void Fun(std::vector<int>&& byRvalueRef)
{
    std::cout << "byMoveRef\n";
}
```

Output with the additional overload:
```
$ ./a.out
byRef
byConstRef
byMoveRef
```

Move semantics allows the dynamic memory of temporaries to be reused.

### std::move:

`std::move` is a utility function that casts types to the correct signature to allow them to be passed to the correct overload.

```
void use()
{
    std::vector v{1, 2, 3};

    Fun(static_cast<std::vector<int>&&>(v));
}
```

can be replaced by 

```
void use()
{
    std::vector v{1, 2, 3};

    Fun(std::move(v));
}
```

* `std::move` is only a cast. The overloads can move if they exist.
* Temporary objects ar picked up by default using move operations.
* Only if we have an object we no longer want to use, can say say `std::move`.
* Move semantics is nothing else than an additional overload that is allowed and expected to **steal data from a source object** 

### Implementation of move constructor & operator:

Using an example string class

```
class String 
{
    public:
        string(const char* data);

        string(const string& rhs);              // Copy constructor
        string& operator=(const string& rhs);   // Copy assignment operator

        string(string&& rhs);                   //Move constructor
        string& operator=(string&& rhs);        //Move assignment operator

        char* c_str() const { return mData.get(); }

    private:
        size_t mLen{};
        std::unique_ptr<char[]> mDAta{};  // unique_ptr means lifetime management 
                                          // is automatically handled
};
```

### Moved from object:

```
string src{"hello"};

string other{std::move(src)};

std::cout << src.c_str();
```

* After applying a `std::move`, `src` becomes a *moved-from* object.
* This object is in a **valid, yet uknown** state.
* A moved-from object should be at least destroyable and assignable.
* C++ implements the non-destructive move, so after the move the move-from object is not destroyed.

* **Simple Rule**: Never touch a moved-from object. Treat the std::move as a destructive move.
* **Know what you're doing rule**: You can reuse the moved-from object once you brought the object back in a valid and known state.
jj