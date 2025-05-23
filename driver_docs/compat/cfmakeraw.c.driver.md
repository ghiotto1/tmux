# Purpose
This C source code file defines a function [`cfmakeraw`](#cfmakeraw) that configures a `termios` structure to set a terminal to raw mode. The function modifies the input, output, local, and control flags of the `termios` structure to disable various processing features such as canonical mode, echoing, and signal generation, while setting the character size to 8 bits. This is typically used in terminal handling to allow programs to read input byte-by-byte without any preprocessing by the terminal driver. The file includes necessary headers for terminal control and compatibility, and it is accompanied by a permissive license allowing free use and distribution.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `termios.h`
- `compat.h`


# Functions

---
### cfmakeraw<!-- {{#callable:cfmakeraw}} -->
The `cfmakeraw` function configures a terminal's termios structure to operate in raw mode by disabling various input, output, and local flags and setting character size to 8 bits.
- **Inputs**:
    - `tio`: A pointer to a `struct termios` which holds terminal I/O settings to be modified.
- **Control Flow**:
    - The function clears specific input flags (IGNBRK, BRKINT, PARMRK, ISTRIP, INLCR, IGNCR, ICRNL, IXON) by using bitwise AND with their negation.
    - It clears the output flag OPOST to disable post-processing of output data.
    - It clears local flags (ECHO, ECHONL, ICANON, ISIG, IEXTEN) to disable canonical mode, signal generation, and echoing.
    - It clears the control flags CSIZE and PARENB to reset character size and disable parity generation.
    - It sets the control flag CS8 to specify 8-bit characters.
- **Output**:
    - The function does not return a value; it modifies the `struct termios` pointed to by `tio` in place.


