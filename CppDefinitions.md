# C++ Interview Cheat Sheet

## 1. Pointers and References
- **Pointer:** Variable storing the memory address of another variable; can be null or reassigned.  
- **Reference:** Alias for another variable; must be initialized at creation, cannot be null or reseated.  
- **Dangling Pointer/Reference:** Points to memory that has been freed or gone out of scope.

## 2. Smart Pointers
- **`std::unique_ptr`:** Exclusive ownership; cannot be copied, only moved; deletes resource on destruction.  
- **`std::shared_ptr`:** Shared ownership; reference-counted; deletes resource when last owner is gone.  
- **`std::weak_ptr`:** Non-owning reference to `shared_ptr`; prevents circular dependencies; can be “locked”.

## 3. Memory Management
- **RAII (Resource Acquisition Is Initialization):** Tie resource lifetime to object lifetime; constructor acquires, destructor releases.  
- **Dynamic Allocation:** `new` / `delete` or smart pointers; allocate memory on heap.  
- **Stack vs Heap:** Stack is fast, automatic, limited; heap is flexible, manually managed, slower.

## 4. Move and Copy Semantics
- **Copy Constructor / Assignment:** Creates a copy of an object.  
- **Move Constructor / Assignment:** Transfers ownership; source object becomes valid but “moved-from”.  
- **Rule of Five:** If any of destructor, copy/move constructor, copy/move assignment is defined, define all five.

## 5. Classes and Objects
- **Classes and structs:** Are user-defined types that group data members (variables) and member functions (methods) together.
  - **Classes** are private by default, **Structs** are public


- **Encapsulation:** Wrapping data and methods into a class, restricting access to protect state.
- **Abstraction:** Hiding implementation details and exposing only essential functionality.
- **Inheritance:** A mechanism to reuse and extend code by deriving new classes from existing ones.
- **Polymorphism:** The ability for different types to be treated uniformly through a common interface, with behaviour varying by type.


- **Constructor / Destructor:** Initialise and clean up objects.  
- **`const` member functions:** Promise not to modify object.  
- **`[[nodiscard]]`:** Compiler hint to not ignore return value.

## 6. Sizes
- **1 Byte** == **8 Bits**
- **bool** - 1 byte
- **char** - 1 byte
- **int** - 4 bytes
- **long** - 4 or 8 bytes
- **float** - 4 bytes
- **double** - 8 bytes
- **Pointer (T*)** - 8 Bytes

## 7. Threads and Concurrency
- **Thread:** Lightweight unit of execution in a process.  
- **Race Condition:** Multiple threads access shared data; at least one modifies it; causes UB.  
- **Mutex (`std::mutex`):** Ensures mutual exclusion for shared resources.  
- **Lock (`std::lock_guard`, `std::unique_lock`, `std::scoped_lock`):** RAII wrappers for mutexes.  
- **Condition Variable (`std::condition_variable`):** Wait until notified by another thread.  
- **Deadlock:** Threads waiting on locks in a circular manner; stops progress.  
- **Atomic (`std::atomic<T>`):** Lock-free, thread-safe variable operations.

## 8. STL and Containers
- **`std::vector`:** Dynamic array; resizable; random access; not thread-safe for concurrent writes.  
- **`std::map` / `std::unordered_map`:** Key-value associative containers.  
- **Iterators:** Objects pointing to container elements; support pointer-like operations.

## 9. Other Keywords / Concepts
- **`constexpr`:** Compile-time constant expression.  
- **`inline`:** Suggest function code substitution at call site.  
- **`static`:** Lifetime or linkage specifier depending on context.  
- **`mutable`:** Allows modification in a `const` method.  
- **`friend`:** Grants access to private/protected members to another class/function.  
- **Undefined Behavior (UB):** Behavior not defined by the C++ standard.
