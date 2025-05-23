# Purpose
This C source code file provides a specific functionality related to terminal I/O settings. It defines a single function, [`cfmakeraw`](#cfmakeraw), which is used to configure a `termios` structure to set a terminal to raw mode. Raw mode is a configuration where input and output are minimally processed, meaning that special character processing (like newline translation and signal generation) is disabled. This is useful in scenarios where a program needs to handle input and output directly, such as in terminal-based applications or when implementing custom communication protocols over a terminal interface.

The file includes necessary headers such as `<sys/types.h>`, `<string.h>`, and `<termios.h>`, indicating its reliance on system types and terminal control definitions. The function [`cfmakeraw`](#cfmakeraw) manipulates the `termios` structure by clearing specific flags in the input, output, local, and control modes to achieve the raw mode configuration. This file is likely part of a larger library or application that deals with terminal control, and it provides a narrow, focused functionality rather than a broad set of features. The presence of a permissive license at the top of the file suggests that this code can be freely used and modified, making it suitable for integration into other projects that require terminal manipulation capabilities.
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
    - The function clears specific input flags in `c_iflag` using bitwise AND with the negation of a combination of flags: `IGNBRK`, `BRKINT`, `PARMRK`, `ISTRIP`, `INLCR`, `IGNCR`, `ICRNL`, and `IXON`.
    - The function clears the output flag `OPOST` in `c_oflag` using bitwise AND with its negation.
    - The function clears specific local flags in `c_lflag` using bitwise AND with the negation of a combination of flags: `ECHO`, `ECHONL`, `ICANON`, `ISIG`, and `IEXTEN`.
    - The function clears specific control flags in `c_cflag` using bitwise AND with the negation of `CSIZE` and `PARENB`.
    - The function sets the character size to 8 bits by setting the `CS8` flag in `c_cflag` using bitwise OR.
- **Output**:
    - The function does not return a value; it modifies the `struct termios` pointed to by `tio` in place.


