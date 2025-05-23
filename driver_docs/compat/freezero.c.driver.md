# Purpose
This C source code file defines a utility function named [`freezero`](#freezero), which is designed to securely free allocated memory. The function takes a pointer and a size as arguments, and if the pointer is not NULL, it first zeroes out the memory block using `memset` to prevent any sensitive data from remaining in memory before calling `free` to deallocate the memory. This is particularly useful in security-sensitive applications where ensuring that data is not left in memory after it is no longer needed is crucial. The file also includes a permissive license notice, allowing for broad use and modification of the code. The inclusion of the "compat.h" header suggests that this function might be part of a larger codebase that aims to provide compatibility across different systems or environments.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `compat.h`


# Functions

---
### freezero<!-- {{#callable:freezero}} -->
The `freezero` function securely frees a memory block by first zeroing out its contents before deallocating it.
- **Inputs**:
    - `ptr`: A pointer to the memory block that needs to be zeroed and freed.
    - `size`: The size of the memory block pointed to by `ptr` that needs to be zeroed.
- **Control Flow**:
    - Check if the `ptr` is not NULL to ensure there is a valid memory block to process.
    - Use `memset` to fill the memory block pointed to by `ptr` with zeros, using the specified `size`.
    - Call `free` to deallocate the memory block pointed to by `ptr`.
- **Output**:
    - The function does not return any value; it performs an operation on the memory block pointed to by `ptr`.


