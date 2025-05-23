# Purpose
This C source code file provides a utility function, [`ntohll`](#ntohll), which converts a 64-bit integer from network byte order to host byte order. The function is particularly useful in network programming where data is often transmitted in network byte order (big-endian) and needs to be converted to the host's native byte order for processing. The code includes necessary headers for network operations and type definitions, ensuring compatibility across different systems. The function splits the 64-bit integer into two 32-bit parts, converts each part using `ntohl`, and then combines them back into a 64-bit integer. This file also includes a permissive license, allowing for broad use and modification of the code.
# Imports and Dependencies

---
- `arpa/inet.h`
- `sys/types.h`
- `compat.h`


# Functions

---
### ntohll<!-- {{#callable:ntohll}} -->
The `ntohll` function converts a 64-bit integer from network byte order to host byte order.
- **Inputs**:
    - `v`: A 64-bit unsigned integer representing a value in network byte order.
- **Control Flow**:
    - The function extracts the lower 32 bits of the input `v` using a bitwise AND operation with `0xffffffff` and converts it from network to host byte order using `ntohl`, storing the result in `b`.
    - The function extracts the upper 32 bits of the input `v` by right-shifting `v` by 32 bits and converts it from network to host byte order using `ntohl`, storing the result in `t`.
    - The function combines the converted lower and upper 32-bit values by left-shifting `b` by 32 bits and performing a bitwise OR with `t`, returning the resulting 64-bit value.
- **Output**:
    - A 64-bit unsigned integer in host byte order.


