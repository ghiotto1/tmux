# Purpose
This C source code file defines a utility function named [`freezero`](#freezero), which is designed to securely free allocated memory. The function takes a pointer `ptr` and a size `size` as arguments. If the pointer is not `NULL`, it first zeroes out the memory block of the specified size using `memset` to prevent sensitive data from lingering in memory, and then it frees the memory using `free`. This approach enhances security by ensuring that any sensitive information is erased before the memory is deallocated. The file also includes a permissive license notice, allowing for broad use and modification of the code.
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
    - This function does not return any value; it performs an operation on the memory block pointed to by `ptr`.


