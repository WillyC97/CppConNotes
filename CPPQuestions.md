# Smart pointer questions

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>What is a resource in C++?</strong></summary>

In C++, a resource is anything your program acquires that must be explicitly released — most commonly heap memory, but also things like file handles, sockets, or database connections. Smart pointers and RAII classes manage these resources automatically to prevent leaks and errors.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Difference between std::unique_ptr and std::shared_ptr: When would you use one over the other?</strong></summary>

`std::unique_ptr` expresses exclusive ownership of a resource — there can only be one owner at a time, and it’s lightweight because it doesn’t require reference counting. It’s the best choice when ownership is clear and you want strict lifetime management.

`std::shared_ptr` allows multiple owners of the same resource, managed through a reference count in a control block. It’s useful when different parts of a program need shared access to the same resource. However, it comes with extra overhead due to reference counting and the risk of circular references, though these can be avoided with `std::weak_ptr`. If a shared_ptr is the only owner of a resource, and the shared_ptr is assigned a new resource, the old resource is deleted automatically.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Pitfalls of using std::shared_ptr and how to avoid them: For example, circular references.</strong></summary>

A common issue with `shared_ptr` is circular dependencies. For example, if class A holds a `shared_ptr` to `B`, and `B` holds a `shared_ptr` back to `A`, then neither reference count will ever reach zero — so neither object is destroyed, causing a memory leak.

The fix is to make one of those references a `weak_ptr`. A `weak_ptr` doesn’t contribute to the reference count, so when the last `shared_ptr` goes out of scope, both objects are properly destroyed. You can still safely access the object by calling `lock()` on the `weak_ptr` to get a temporary `shared_ptr`, but you avoid the ownership cycle.

A `weak_ptr` doesn’t own the resource, so you can’t directly dereference it. Instead, you call `lock()`, which tries to promote it back to a `shared_ptr`.

- If the object is still alive, `lock()` returns a valid `shared_ptr` that you can safely use.  
- If the object has already been destroyed, `lock()` returns an empty `shared_ptr`, which you can check with `.expired()` or by testing it in a boolean context.

This makes `lock()` the safe way to temporarily access an object without preventing it from being freed.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Usage of std::weak_ptr: When and why would you use a weak reference?</strong></summary>

You use a `weak_ptr` when you need to observe or temporarily access an object managed by a `shared_ptr` without taking ownership. This is especially useful to avoid circular dependencies, such as when a child object needs to reference its parent without preventing the parent from being destroyed.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Difference between std::unique_ptr with a custom deleter and the default deleter: When would you need a custom deleter?</strong></summary>

By default, a `unique_ptr` calls `delete` on the object it owns when it goes out of scope. This in turn calls the destructor of the resource. If the resource requires a different cleanup mechanism — for example, closing a file handle, freeing memory allocated with `malloc`, or calling a custom library function — you can supply a custom deleter. This lets you use `unique_ptr` for safe RAII management of resources that aren’t managed by plain `delete`.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Concept of RAII and how smart pointers fit into it: Why is RAII considered a good practice?</strong></summary>

RAII, or Resource Acquisition Is Initialisation, is a C++ idiom where resource allocation is tied to object lifetime: a resource is acquired in a constructor and automatically released in the destructor. Smart pointers like unique_ptr and shared_ptr are perfect examples of RAII because they automatically manage memory — when the pointer goes out of scope, the resource is safely freed. RAII is considered good practice because it ensures deterministic cleanup, prevents resource leaks, and provides strong exception safety.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Exception safety between std::unique_ptr and std::shared_ptr: How do they handle exceptions differently?</strong></summary>

Both `unique_ptr` and `shared_ptr` automatically free resources when they go out of scope, so they prevent leaks if an exception is thrown. `unique_ptr` is simpler and lightweight, while `shared_ptr` uses reference counting but still ensures safe cleanup even in exception scenarios.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->

# Move semantics questions

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>What’s the difference between copy semantics and move semantics in C++11 and later?</strong></summary>

Copy semantics duplicate the underlying resource via the copy constructor or assignment, which can be expensive for large or non-trivial resources. Move semantics, introduced in C++11, transfer ownership of the resource via the move constructor or assignment, leaving the source object in a valid but unspecified state. This avoids unnecessary deep copies and improves performance.

In a copy, the destination object allocates its own memory and duplicates the contents, so both objects own independent resources. In a move, the destination simply takes over the source’s pointer without allocating new memory, and the source is reset to avoid double deletion.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>What’s a dynamic resource?</strong></summary>

Dynamic resources are things your program acquires at runtime that need manual management — they don’t automatically clean themselves up when a variable goes out of scope.

Examples:
* Heap memory (allocated with `new` or `malloc`)
* File handles (opened files that need `fclose`)
* Network sockets
* Database connections
* GPU buffers / audio buffers

In C++, RAII wrappers (like smart pointers) and move semantics make managing these resources safe and efficient.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Why are move constructors and move assignment operators useful for classes managing dynamic resources?</strong></summary>

For classes that manage dynamic resources like heap memory, file handles, or sockets, copy semantics can be expensive because they require allocating and duplicating the resource. Move constructors and move assignment operators avoid this cost by transferring ownership of the underlying resource, rather than copying it. This is both faster and safer, because it ensures only one object is responsible for releasing the resource at destruction.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>What happens to the source object after it has been moved from?</strong></summary>

After a move, the source object is left in a valid but unspecified “moved-from” state. It still satisfies its invariants and can be safely destroyed or reassigned, but you shouldn’t make assumptions about its contents. For example, a moved-from std::string is guaranteed to be valid, but it might be empty or hold some implementation-defined placeholder.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>In what situations is a move constructor preferred over a copy constructor automatically by the compiler?
</strong></summary>

The compiler prefers a move constructor over a copy constructor when it detects an rvalue — a temporary object or an object that is about to go out of scope. This allows it to transfer ownership of resources instead of copying, which is much more efficient for large objects like vectors, strings, or any class managing dynamic memory or other resources. Lvalues will still call the copy constructor unless `std::move` is used.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>How do rvalue references (T&&) enable move semantics in C++?
</strong></summary>

Rvalue references, written as `T&&`, allow a function or constructor to distinguish temporary objects (rvalues) from regular lvalues. They enable move semantics by letting you implement move constructors and move assignment operators, which can transfer ownership of resources from temporaries, instead of copying them. Essentially, `T&&` signals that the object is safe to “move from,” so ownership of dynamic resources can be transferred efficiently.

</details>
<br>

```c++
class Buffer
{
public:
    Buffer(size_t n)
    : data(new int[n]),
      size(n) {}

    ~Buffer() { delete[] data; }
private:
  int* data;
  size_t size;
};

```
<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Given the class above, what’s missing and why could it be a problem?
</strong></summary>

This class manually manages a dynamic array (`int* data`) and defines a destructor. Because of that, the compiler-generated copy constructor, copy assignment, move constructor, and move assignment may not behave correctly. For example, a default copy constructor would shallow-copy the pointer, causing two objects to delete the same memory, and a move constructor is missing, so moving the object could leave the moved-from object in an unsafe state. To fix this, you should implement the copy constructor/assignment to deep-copy the array, and the move constructor/assignment to transfer ownership of the array while nulling out the source pointer.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Can you write fix the code above?
</strong></summary>

```c++
class Buffer
{
public:
  Buffer(size_t n)
  : data(new int[n]),
    size(n) {};

  Buffer(const Buffer& other)
  : data(new int[other.size])
  , size(other.size())
  {
    std::copy(other.data, other.data + size, data);
  }

  Buffer& operator=(const Buffer& other)
  {
    if (this == &other) return *this;
    delete[] data; //free old memory
    data = new int[size];
    std::copy(other.data, other.data + size, data);
    return *this;
  }

  // Move constructor (transfer ownership)
  Buffer(Buffer&& other) noexcept
  : data(other.data)
  , size(other.size)
  {
      other.data = nullptr;
      other.size = 0;
      std::cout << "Moved\n";
  }

  // Move assignment operator (transfer ownership)
  Buffer& operator=(Buffer&& other) noexcept
  {
      if (this == &other) return *this; // self-assignment check
      delete[] data;                     // free old memory
      data = other.data;
      size = other.size;
      other.data = nullptr;
      other.size = 0;
      std::cout << "Move assigned\n";
      return *this;
  }

  ~Buffer() override { delete[] data; }
private:
  int* data;
  size_t size;
};

```

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->

```c++
Buffer(Buffer&& other)
{
  data = other.data;
  size = other.size;
}
```
<details>
<summary><strong>What’s wrong with the above move constructor?
</strong></summary>

It copies the pointer from `other` to `this`, but doesn't null out `other.data`
when other is destroyed, it will call `delete[]` on the same pointer. Later when `this` is destroyed, `delete[]` is called again. This is a double delete and is undefined behaviour.

This is the fixed code:

```c++
Buffer(Buffer&& other) noexcept
    : data(other.data)
    , size(other.size)
{
    other.data = nullptr; // leave source in safe "moved-from" state
    other.size = 0;
}
```

</details>
<br>


```c++
std::string foo() {
    std::string s = "Foo";
    return s;
}
```



<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Will the return use a copy or a move on the above, and why?
</strong></summary>

Returning s here will typically use move semantics, because s is a local variable (an lvalue) but the compiler treats it as an xvalue in the return statement, allowing the move constructor of `std::string` to be called. Modern compilers often also apply Return Value Optimization (RVO), so the string may be constructed directly in the caller’s space, eliminating copies and even moves.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>Why should you often = delete the copy constructor when defining a move-only type (e.g., std::unique_ptr)?
</strong></summary>

A move-only type, like std::unique_ptr, manages a resource that cannot be safely duplicated. Even if you define a move constructor and move assignment operator, the compiler would still generate default copy operations unless you explicitly delete them. By writing = delete for the copy constructor and copy assignment operator, you enforce non-copyability, ensuring that the resource can only be transferred, not duplicated.

</details>
<br>

<!-- ---------------------------------------------------------------------------------------------------------------------------- -->
<details>
<summary><strong>How does std::move differ from a move constructor? Is it guaranteed to move an object?
</strong></summary>

std::move does not move anything by itself — it simply casts its argument to an rvalue reference (T&&), signaling that the object can be moved from. Whether a move actually happens depends on whether the resulting rvalue reference is used by a move constructor or move assignment operator. After a move, the source object is left in a valid but unspecified state, but std::move alone doesn’t perform the transfer — it’s just a hint to enable a move.

`std::move `is just a cast to an rvalue reference — it does not guarantee a move. If the type is const, not movable, or the move constructor is unavailable, the compiler may fall back to a copy.

</details>
<br>
