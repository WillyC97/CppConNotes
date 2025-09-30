## Smart Pointers

* Speaker: David Olsen
* Talk: [Smart Pointers](https://www.youtube.com/watch?v=YokY6HzLkXs&ab_channel=CppCon)

### Raw pointer

* Single object vs. array
  * Single: allocate with `new`, free with `delete`
  * Array: allocate with `new[]`, free with `delete[]`
  * Single: don't use `++p`, `--p`, or `p[n]`
  * Array: `++p`, `--p`, and `p[n]` are fine


* Owning vs. non-owning
  * Owner must free the memory when done
  * Non-owner must **never** free the memory

* Nullable vs. non-nullable
  * Some pointers can never be null
  * It would be nice if the type system helped enforce that

### Smart Pointers

#### UNIQUE_PTR:

* Owns memory
* Assumes it is the only Owner
* Automatically destroys the object and deletes the memory
* Move-only type - It has no copy constructor or copy assignment operator

* To transfer ownership to a function, pass a std::unique_ptr by value
* To return ownership from a function, return std::unique_ptr by value

```
std::unique_ptr<float[]> blah(std::unique_ptr<float> x, std::unique_ptr<float[]> y)
{
  ...

  return z;
}
```

#### MAKE_UNIQUE:

```
template <typename T, typename... Args>
unique_ptr<T> make_unique(Args&&... args);
```
* Combines together:
  * Memory allocation
  * Construction of an object with given arguments
  * wraps the object in a std::unique_ptr<T> that owns the object

#### Gotchas

Make sure only one unique_ptr for a block of memory

```
T* p = ...
std::unique_ptr<T> a{p};
std::unique_ptr<T> b{p};
// crash due to double free


auto c = std::make_unique<T>();
std::unique_ptr<T> d { c.get() };
// crash due to double free
```

Don't create a unique_ptr from a raw pointer unless you know where the pointer came from and that it needs an owner


unique_ptr doesn't solve the dangling pointer problem

```
T* p = nullptr;
{
  auto u = std::make_unique<T>();
  p = u.get();
}

// p is now dangling and invalid
auto bad = *p; //undefined behaviour
```
It is up to the programmer to make sure that any raw pointer that was created by calling `unique_ptr::get()` is no longer needed when the `unique_ptr` goes out of scope.

#### SHARED_PTR

* Owns Memory
* Shared ownership
  * Many `std::shared_ptr` objects work together to jointly own and manage one object and its memory
* Automatically destroys the object and deletes the memory when the last shared_ptr object is destructed
* Copyable
* Contains a control block that keeps track of the number of other shared_ptrs that are managing the same object

#### MAKE_UNIQUE:

```
template <typename T, typename... Args>
unique_ptr<T> make_shared(Args&&... args);
```
* Combines together:
  * Memory allocation for both object and control_block
  * Construction of an object T with given arguments
  * Initialises the control_block
  * wraps the object in a std::shared_ptr<T>

For two or more shared_ptrs to manage ownership, exactly one of the shared_ptrs should have been created using make_shared or from the raw pointer. All the other shared_ptrs need to have been copied from a shared pointer.


If two shared_ptrs are created from the same raw pointer, they will have separate control blocks and will not work together

```
{
  T* p = ...
  std::shared_ptr<T> a(p);
  std::shared_ptr<T> b(p);
  // runtime error: double free
}
```
