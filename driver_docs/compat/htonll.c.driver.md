# Purpose
This C source code file provides a utility function [`htonll`](#htonll), which converts a 64-bit integer from host byte order to network byte order. The function is particularly useful in network programming where data needs to be transmitted over the network in a consistent byte order, typically big-endian. The implementation leverages the standard `htonl` function to handle the conversion of the lower and upper 32-bit segments of the 64-bit integer separately, then combines them to form the final result. The file includes necessary headers for network operations and types, and it is accompanied by a permissive license allowing for broad use and distribution.
# Imports and Dependencies

---
- `arpa/inet.h`
- `sys/types.h`
- `compat.h`


# Functions

---
### htonll<!-- {{#callable:htonll}} -->
The `htonll` function converts a 64-bit integer from host byte order to network byte order.
- **Inputs**:
    - `v`: A 64-bit unsigned integer to be converted from host byte order to network byte order.
- **Control Flow**:
    - The function takes a 64-bit unsigned integer `v` as input.
    - It splits `v` into two 32-bit parts: the lower 32 bits and the upper 32 bits.
    - The lower 32 bits are converted to network byte order using `htonl` and stored in `b`.
    - The upper 32 bits are shifted right by 32 bits, converted to network byte order using `htonl`, and stored in `t`.
    - The function combines `b` and `t` by shifting `b` left by 32 bits and performing a bitwise OR with `t`.
    - The combined result is returned as a 64-bit unsigned integer in network byte order.
- **Output**:
    - A 64-bit unsigned integer in network byte order.


