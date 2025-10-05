# C++ Interview Cheat Sheet II

## 1. STL and Containers
- **`std::vector`:** Dynamic array; resizable; random access; not thread-safe for concurrent writes.  
- **`std::map` / `std::unordered_map`:** Key-value associative containers.  
- **Iterators:** Objects pointing to container elements; support pointer-like operations.
  - Behaves like a generalised pointer, letting you traverse and access elements without knowing the container’s internal implementation.
  - Support `*`, `++`, `--`, and `==`operations.
  - Abstraction: Iterators provide a uniform interface to traverse any container, hiding implementation details.
  - Non-const version: Can be used to read and **modify** the element it points to
  - Const version: Can be used only to **read** the element it points to.
  - Reverse iterator: traverses through a container from the **last** element to the **first**
  - `rbegin()` points to the **last element**
  - `rend()` points to the one before the first element

## 2. Exceptions
- A runtime error that occurs during the execution of a program.
- The normal flow of the program is interrupted.
- Searches for a matching catch block in the current function.
- The stack unwinds, destructors for all local objects are called, moving up the call stack to find a handler.
- If no handler is found at all, std::terminate() is called and the program ends.

- `try`
  - Wraps a block of code that might throw an exception.

- `throw`
  - Used to signal that an error occurred.
  - Transfers control to the nearest matching catch block.

- `catch`
  - Defines a block of code to handle a thrown exception.
  - The program does not terminate if the exception is caught.

- `noexcept` - promises not to throw any exceptions
  - if one is thrown program `std::terminates()`

- Basic guarantee: In a valid state, but the exact state may have changed after an exception.
- Strong guarantee: Operation completes successfully or has no effect (rollback).
- No-throw (nothrow) guarantee: Guaranteed not to throw any exceptions.

- Don't throw an exception in a destructor: stack unwinding automatically calls destructors
- If an exception is already propagating, the program will std::terminate() which could lead to inproper clean up
