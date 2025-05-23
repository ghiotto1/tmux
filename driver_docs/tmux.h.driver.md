# Purpose
This C header file, `tmux.h`, is part of the `tmux` project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file serves as a comprehensive declaration of the various data structures, constants, macros, and function prototypes used throughout the `tmux` codebase. It provides a broad range of functionality, including definitions for handling terminal input/output, managing sessions and windows, handling key bindings, and managing the user interface elements like status lines and menus.

The file defines numerous structures such as `client`, `session`, `window`, and `window_pane`, which are central to `tmux`'s operation. These structures are used to manage the state and behavior of terminal sessions and their associated windows and panes. The file also includes a variety of macros for configuration defaults, key codes, and terminal capabilities, which are essential for `tmux`'s interaction with different terminal types. Additionally, it declares a wide array of functions for managing the lifecycle of sessions, windows, and panes, as well as for handling user input and output. This header file is intended to be included in other parts of the `tmux` codebase, providing a shared interface for the various components of the application.
# Imports and Dependencies

---
- `sys/time.h`
- `sys/uio.h`
- `limits.h`
- `stdarg.h`
- `stdio.h`
- `termios.h`
- `wchar.h`
- `utempter.h`
- `compat.h`
- `tmux-protocol.h`
- `xmalloc.h`


# Global Variables

---
### environ
- **Type**: `char **`
- **Description**: The `environ` variable is a global variable that is an array of strings, where each string is an environment variable in the format 'KEY=VALUE'. It is used to access the environment variables of the current process.
- **Use**: This variable is used to retrieve and manipulate the environment variables available to the process.


---
### __packed
- **Type**: `struct grid_cell_entry`
- **Description**: The `__packed` attribute is used to define a `struct grid_cell_entry` that ensures no padding is added between the members of the structure. This structure contains a union with an unsigned integer `offset` and a nested structure with attributes `attr`, `fg`, `bg`, and `data`, all of type `u_char`. Additionally, it has a `flags` member of type `u_char`. The `__packed` attribute is crucial for memory efficiency and ensuring the structure's layout is consistent across different compilers.
- **Use**: This variable is used to define a grid cell entry with tightly packed memory layout, often for performance optimization in grid or graphical data structures.


---
### global_options
- **Type**: `struct options*`
- **Description**: The `global_options` variable is a pointer to a `struct options`, which is likely used to store global configuration settings for the application. This structure is part of a larger system that manages various options and settings, potentially affecting the behavior of the application globally.
- **Use**: This variable is used to access and modify global configuration settings throughout the application.


---
### global_s_options
- **Type**: `struct options*`
- **Description**: `global_s_options` is a global variable that is a pointer to a `struct options`. This structure is likely used to store configuration or settings related to a specific aspect of the application, possibly related to session options given the naming convention.
- **Use**: This variable is used to access and manipulate session-related options globally across the application.


---
### global_w_options
- **Type**: `struct options*`
- **Description**: `global_w_options` is a global variable that is a pointer to a `struct options`. This structure is likely used to store configuration options or settings for a specific component or module within the software.
- **Use**: This variable is used to access and manage window-specific options or settings globally across the application.


---
### global_environ
- **Type**: `struct environ*`
- **Description**: `global_environ` is a pointer to a struct of type `environ`, which is used to manage environment variables within the application. This struct likely contains entries that represent key-value pairs of environment variables.
- **Use**: This variable is used to access and manipulate the global environment variables for the application.


---
### start_time
- **Type**: `struct timeval`
- **Description**: The `start_time` variable is a global variable of type `struct timeval`, which is typically used to represent a specific point in time with precision up to microseconds. It is declared as an external variable, indicating that it is defined elsewhere, possibly in another source file.
- **Use**: This variable is used to store the start time of a particular event or process, allowing for time calculations or measurements.


---
### socket_path
- **Type**: ``const char *``
- **Description**: The `socket_path` variable is a global constant pointer to a character string, which is used to store the path to a Unix domain socket. This path is typically used by the application to establish inter-process communication (IPC) through the socket.
- **Use**: This variable is used to define the location of the socket file that the application will use for communication.


---
### shell_command
- **Type**: ``const char *``
- **Description**: The `shell_command` is a global constant pointer to a character string, which likely holds the default shell command or path used by the application. It is declared as an external variable, indicating that its definition is expected to be found in another source file.
- **Use**: This variable is used to store and provide access to the default shell command or path across different parts of the program.


---
### ptm_fd
- **Type**: `int`
- **Description**: `ptm_fd` is a global integer variable that is declared as an external variable, indicating it is defined elsewhere in the program. It is likely used to store a file descriptor related to a pseudo-terminal master (PTM).
- **Use**: This variable is used to manage or interact with a pseudo-terminal master within the program.


---
### shell_argv0
- **Type**: `function pointer`
- **Description**: `shell_argv0` is a function pointer that takes a `const char *` and an `int` as arguments and returns a `char *`. It is likely used to manipulate or retrieve the name of the shell or command being executed.
- **Use**: This function pointer is used to handle operations related to the shell's argument vector, specifically the first argument, which is typically the name of the command or shell.


---
### sig2name
- **Type**: `function`
- **Description**: The `sig2name` function is a global function that takes an integer as an argument and returns a constant character pointer. This function is likely used to convert a signal number to its corresponding signal name as a string.
- **Use**: This function is used to map signal numbers to their respective signal names in string format.


---
### find_cwd
- **Type**: `const char*`
- **Description**: The `find_cwd` function is a global function that returns a constant character pointer. It is designed to find and return the current working directory of the process.
- **Use**: This function is used to retrieve the current working directory as a string.


---
### find_home
- **Type**: `const char *`
- **Description**: The `find_home` function is a global function that returns a constant character pointer. It is likely used to determine and return the home directory path of the current user or environment.
- **Use**: This function is used to retrieve the home directory path, which can be utilized by other parts of the program to access user-specific files or settings.


---
### getversion
- **Type**: `function`
- **Description**: The `getversion` function is a global function that returns a constant character pointer. It is likely used to retrieve the version information of the software or application.
- **Use**: This function is used to obtain the version string of the application, which can be displayed or logged as needed.


---
### proc_start
- **Type**: `function`
- **Description**: `proc_start` is a function that returns a pointer to a `struct tmuxproc`. It takes a single argument, a constant character pointer, which is likely used as a name or identifier for the process being started.
- **Use**: This function is used to initialize and start a new tmux process, returning a pointer to the process structure for further manipulation or control.


---
### proc_add_peer
- **Type**: `function`
- **Description**: The `proc_add_peer` function is a global function that adds a peer to a tmux process. It takes a pointer to a `tmuxproc` structure, an integer, a function pointer for handling messages, and a void pointer for additional data.
- **Use**: This function is used to add a new peer to a tmux process, allowing for communication and message handling between processes.


---
### cfg_finished
- **Type**: `int`
- **Description**: The `cfg_finished` variable is an external integer that likely serves as a flag to indicate whether a configuration process has been completed. It is declared as an external variable, suggesting it is defined elsewhere in the program.
- **Use**: This variable is used to track the completion status of a configuration process, allowing other parts of the program to check if the configuration has been finalized.


---
### cfg_client
- **Type**: ``struct client *``
- **Description**: The `cfg_client` is a global variable that is a pointer to a `struct client`. This structure likely represents a client in the context of the software, which could be a user or a session interacting with the application.
- **Use**: This variable is used to reference a specific client structure globally, allowing various parts of the program to access and manipulate client-related data.


---
### cfg_files
- **Type**: `char **`
- **Description**: `cfg_files` is a global variable that is a pointer to an array of strings, where each string represents a configuration file path. It is declared as an external variable, indicating that it is defined elsewhere in the program.
- **Use**: This variable is used to store and access the paths of configuration files that the program may need to read or process.


---
### cfg_nfiles
- **Type**: `u_int`
- **Description**: The variable `cfg_nfiles` is a global variable of type `u_int` that is declared as an external variable, indicating it is defined elsewhere in the program. It is likely used to store a count or number of configuration files being processed or managed by the program.
- **Use**: This variable is used to keep track of the number of configuration files.


---
### cfg_quiet
- **Type**: `int`
- **Description**: The `cfg_quiet` variable is an external integer that likely serves as a flag to control the verbosity of configuration-related operations in the software. It is declared as an external variable, indicating that its definition is expected to be found in another source file.
- **Use**: This variable is used to determine whether configuration operations should suppress output or run quietly.


# Data Structures

---
### tty_code_code
- **Type**: `enum`
- **Members**:
    - `TTYC_ACSC`: Represents the ACS (Alternate Character Set) capability.
    - `TTYC_AM`: Indicates if automatic margins are supported.
    - `TTYC_AX`: Represents the AX (alternate character set) capability.
    - `TTYC_BCE`: Indicates if back color erase is supported.
    - `TTYC_BEL`: Represents the BEL (bell) capability.
    - `TTYC_BIDI`: Indicates if bidirectional text is supported.
    - `TTYC_BLINK`: Represents the blink capability.
    - `TTYC_BOLD`: Indicates if bold text is supported.
    - `TTYC_CIVIS`: Represents the capability to make the cursor invisible.
    - `TTYC_CLEAR`: Represents the capability to clear the screen.
    - `TTYC_CLMG`: Indicates the capability to clear margins.
    - `TTYC_CMG`: Represents the capability to change margins.
    - `TTYC_CNORM`: Indicates the capability to make the cursor normal.
    - `TTYC_COLORS`: Represents the number of colors supported.
    - `TTYC_CR`: Represents the carriage return capability.
    - `TTYC_CS`: Indicates the capability to change the screen.
    - `TTYC_CSR`: Represents the capability to set the cursor position.
    - `TTYC_CUB`: Indicates the capability to move the cursor left.
    - `TTYC_CUB1`: Represents the capability to move the cursor left by one.
    - `TTYC_CUD`: Indicates the capability to move the cursor down.
    - `TTYC_CUD1`: Represents the capability to move the cursor down by one.
    - `TTYC_CUF`: Indicates the capability to move the cursor forward.
    - `TTYC_CUF1`: Represents the capability to move the cursor forward by one.
    - `TTYC_CUP`: Indicates the capability to position the cursor.
    - `TTYC_CUU`: Indicates the capability to move the cursor up.
    - `TTYC_CUU1`: Represents the capability to move the cursor up by one.
    - `TTYC_CVVIS`: Indicates the capability to make the cursor visible.
    - `TTYC_DCH`: Represents the capability to delete characters.
    - `TTYC_DCH1`: Indicates the capability to delete one character.
    - `TTYC_DIM`: Represents the capability to dim text.
    - `TTYC_DL`: Indicates the capability to delete lines.
    - `TTYC_DL1`: Represents the capability to delete one line.
    - `TTYC_DSBP`: Indicates the capability to disable the scrollback.
    - `TTYC_DSEKS`: Represents the capability to disable the escape sequences.
    - `TTYC_DSFCS`: Indicates the capability to disable the function keys.
    - `TTYC_DSMG`: Represents the capability to disable the mouse.
    - `TTYC_E3`: Represents the E3 capability.
    - `TTYC_ECH`: Indicates the capability to erase characters.
    - `TTYC_ED`: Represents the capability to erase the display.
    - `TTYC_EL`: Indicates the capability to erase the line.
    - `TTYC_EL1`: Represents the capability to erase one line.
    - `TTYC_ENACS`: Indicates the capability to enable alternate character set.
    - `TTYC_ENBP`: Represents the capability to enable backspace.
    - `TTYC_ENEKS`: Indicates the capability to enable escape sequences.
    - `TTYC_ENFCS`: Represents the capability to enable function keys.
    - `TTYC_ENMG`: Indicates the capability to enable mouse.
    - `TTYC_FSL`: Represents the capability to set the foreground color.
    - `TTYC_HLS`: Indicates the capability to highlight.
    - `TTYC_HOME`: Represents the capability to move to home position.
    - `TTYC_HPA`: Indicates the capability to set horizontal position.
    - `TTYC_ICH`: Represents the capability to insert characters.
    - `TTYC_ICH1`: Indicates the capability to insert one character.
    - `TTYC_IL`: Represents the capability to insert lines.
    - `TTYC_IL1`: Indicates the capability to insert one line.
    - `TTYC_INDN`: Represents the capability to indent.
    - `TTYC_INVIS`: Indicates the capability to make text invisible.
    - `TTYC_KCBT`: Represents the capability for key callback.
    - `TTYC_KCUB1`: Indicates the capability for key cursor back.
    - `TTYC_KCUD1`: Represents the capability for key cursor down.
    - `TTYC_KCUF1`: Indicates the capability for key cursor forward.
    - `TTYC_KCUU1`: Represents the capability for key cursor up.
    - `TTYC_KDC2`: Indicates the capability for key delete character.
    - `TTYC_KDC3`: Represents the capability for key delete character.
    - `TTYC_KDC4`: Indicates the capability for key delete character.
    - `TTYC_KDC5`: Represents the capability for key delete character.
    - `TTYC_KDC6`: Indicates the capability for key delete character.
    - `TTYC_KDC7`: Represents the capability for key delete character.
    - `TTYC_KDCH1`: Indicates the capability for key delete character.
    - `TTYC_KDN2`: Represents the capability for key down next.
    - `TTYC_KDN3`: Indicates the capability for key down next.
    - `TTYC_KDN4`: Represents the capability for key down next.
    - `TTYC_KDN5`: Indicates the capability for key down next.
    - `TTYC_KDN6`: Represents the capability for key down next.
    - `TTYC_KDN7`: Indicates the capability for key down next.
    - `TTYC_KEND`: Represents the capability for key end.
    - `TTYC_KEND2`: Indicates the capability for key end.
    - `TTYC_KEND3`: Represents the capability for key end.
    - `TTYC_KEND4`: Indicates the capability for key end.
    - `TTYC_KEND5`: Represents the capability for key end.
    - `TTYC_KEND6`: Indicates the capability for key end.
    - `TTYC_KEND7`: Represents the capability for key end.
    - `TTYC_KF1`: Represents the capability for function key 1.
    - `TTYC_KF10`: Represents the capability for function key 10.
    - `TTYC_KF11`: Represents the capability for function key 11.
    - `TTYC_KF12`: Represents the capability for function key 12.
    - `TTYC_KF13`: Represents the capability for function key 13.
    - `TTYC_KF14`: Represents the capability for function key 14.
    - `TTYC_KF15`: Represents the capability for function key 15.
    - `TTYC_KF16`: Represents the capability for function key 16.
    - `TTYC_KF17`: Represents the capability for function key 17.
    - `TTYC_KF18`: Represents the capability for function key 18.
    - `TTYC_KF19`: Represents the capability for function key 19.
    - `TTYC_KF2`: Represents the capability for function key 2.
    - `TTYC_KF20`: Represents the capability for function key 20.
    - `TTYC_KF21`: Represents the capability for function key 21.
    - `TTYC_KF22`: Represents the capability for function key 22.
    - `TTYC_KF23`: Represents the capability for function key 23.
    - `TTYC_KF24`: Represents the capability for function key 24.
    - `TTYC_KF25`: Represents the capability for function key 25.
    - `TTYC_KF26`: Represents the capability for function key 26.
    - `TTYC_KF27`: Represents the capability for function key 27.
    - `TTYC_KF28`: Represents the capability for function key 28.
    - `TTYC_KF29`: Represents the capability for function key 29.
    - `TTYC_KF3`: Represents the capability for function key 3.
    - `TTYC_KF30`: Represents the capability for function key 30.
    - `TTYC_KF31`: Represents the capability for function key 31.
    - `TTYC_KF32`: Represents the capability for function key 32.
    - `TTYC_KF33`: Represents the capability for function key 33.
    - `TTYC_KF34`: Represents the capability for function key 34.
    - `TTYC_KF35`: Represents the capability for function key 35.
    - `TTYC_KF36`: Represents the capability for function key 36.
    - `TTYC_KF37`: Represents the capability for function key 37.
    - `TTYC_KF38`: Represents the capability for function key 38.
    - `TTYC_KF39`: Represents the capability for function key 39.
    - `TTYC_KF4`: Represents the capability for function key 4.
    - `TTYC_KF40`: Represents the capability for function key 40.
    - `TTYC_KF41`: Represents the capability for function key 41.
    - `TTYC_KF42`: Represents the capability for function key 42.
    - `TTYC_KF43`: Represents the capability for function key 43.
    - `TTYC_KF44`: Represents the capability for function key 44.
    - `TTYC_KF45`: Represents the capability for function key 45.
    - `TTYC_KF46`: Represents the capability for function key 46.
    - `TTYC_KF47`: Represents the capability for function key 47.
    - `TTYC_KF48`: Represents the capability for function key 48.
    - `TTYC_KF49`: Represents the capability for function key 49.
    - `TTYC_KF5`: Represents the capability for function key 5.
    - `TTYC_KF50`: Represents the capability for function key 50.
    - `TTYC_KF51`: Represents the capability for function key 51.
    - `TTYC_KF52`: Represents the capability for function key 52.
    - `TTYC_KF53`: Represents the capability for function key 53.
    - `TTYC_KF54`: Represents the capability for function key 54.
    - `TTYC_KF55`: Represents the capability for function key 55.
    - `TTYC_KF56`: Represents the capability for function key 56.
    - `TTYC_KF57`: Represents the capability for function key 57.
    - `TTYC_KF58`: Represents the capability for function key 58.
    - `TTYC_KF59`: Represents the capability for function key 59.
    - `TTYC_KF6`: Represents the capability for function key 6.
    - `TTYC_KF60`: Represents the capability for function key 60.
    - `TTYC_KF61`: Represents the capability for function key 61.
    - `TTYC_KF62`: Represents the capability for function key 62.
    - `TTYC_KF63`: Represents the capability for function key 63.
    - `TTYC_KF7`: Represents the capability for function key 7.
    - `TTYC_KF8`: Represents the capability for function key 8.
    - `TTYC_KF9`: Represents the capability for function key 9.
    - `TTYC_KHOM2`: Represents the capability for home key 2.
    - `TTYC_KHOM3`: Represents the capability for home key 3.
    - `TTYC_KHOM4`: Represents the capability for home key 4.
    - `TTYC_KHOM5`: Represents the capability for home key 5.
    - `TTYC_KHOM6`: Represents the capability for home key 6.
    - `TTYC_KHOM7`: Represents the capability for home key 7.
    - `TTYC_KHOME`: Represents the capability for home key.
    - `TTYC_KIC2`: Represents the capability for insert key 2.
    - `TTYC_KIC3`: Represents the capability for insert key 3.
    - `TTYC_KIC4`: Represents the capability for insert key 4.
    - `TTYC_KIC5`: Represents the capability for insert key 5.
    - `TTYC_KIC6`: Represents the capability for insert key 6.
    - `TTYC_KIC7`: Represents the capability for insert key 7.
    - `TTYC_KICH1`: Represents the capability for insert character 1.
    - `TTYC_KIND`: Represents the capability for kind key.
    - `TTYC_KLFT2`: Represents the capability for left key 2.
    - `TTYC_KLFT3`: Represents the capability for left key 3.
    - `TTYC_KLFT4`: Represents the capability for left key 4.
    - `TTYC_KLFT5`: Represents the capability for left key 5.
    - `TTYC_KLFT6`: Represents the capability for left key 6.
    - `TTYC_KLFT7`: Represents the capability for left key 7.
    - `TTYC_KMOUS`: Represents the capability for mouse key.
    - `TTYC_KNP`: Represents the capability for next pane.
    - `TTYC_KNXT2`: Represents the capability for next key 2.
    - `TTYC_KNXT3`: Represents the capability for next key 3.
    - `TTYC_KNXT4`: Represents the capability for next key 4.
    - `TTYC_KNXT5`: Represents the capability for next key 5.
    - `TTYC_KNXT6`: Represents the capability for next key 6.
    - `TTYC_KNXT7`: Represents the capability for next key 7.
    - `TTYC_KPP`: Represents the capability for previous pane.
    - `TTYC_KPRV2`: Represents the capability for previous key 2.
    - `TTYC_KPRV3`: Represents the capability for previous key 3.
    - `TTYC_KPRV4`: Represents the capability for previous key 4.
    - `TTYC_KPRV5`: Represents the capability for previous key 5.
    - `TTYC_KPRV6`: Represents the capability for previous key 6.
    - `TTYC_KPRV7`: Represents the capability for previous key 7.
    - `TTYC_KRI`: Represents the capability for right key.
    - `TTYC_KRIT2`: Represents the capability for right key 2.
    - `TTYC_KRIT3`: Represents the capability for right key 3.
    - `TTYC_KRIT4`: Represents the capability for right key 4.
    - `TTYC_KRIT5`: Represents the capability for right key 5.
    - `TTYC_KRIT6`: Represents the capability for right key 6.
    - `TTYC_KRIT7`: Represents the capability for right key 7.
    - `TTYC_KUP2`: Represents the capability for up key 2.
    - `TTYC_KUP3`: Represents the capability for up key 3.
    - `TTYC_KUP4`: Represents the capability for up key 4.
    - `TTYC_KUP5`: Represents the capability for up key 5.
    - `TTYC_KUP6`: Represents the capability for up key 6.
    - `TTYC_KUP7`: Represents the capability for up key 7.
    - `TTYC_MS`: Represents the mouse support capability.
    - `TTYC_NOBR`: Indicates no break capability.
    - `TTYC_OL`: Represents the overline capability.
    - `TTYC_OP`: Represents the capability to reset attributes.
    - `TTYC_RECT`: Indicates the rectangle capability.
    - `TTYC_REV`: Represents the reverse video capability.
    - `TTYC_RGB`: Indicates RGB color capability.
    - `TTYC_RI`: Represents the capability for reverse index.
    - `TTYC_RIN`: Indicates the capability for reset index.
    - `TTYC_RMACS`: Represents the capability to remove alternate character set.
    - `TTYC_RMCUP`: Indicates the capability to remove cursor up.
    - `TTYC_RMKX`: Represents the capability to remove mouse keys.
    - `TTYC_SE`: Indicates the capability to set attributes.
    - `TTYC_SETAB`: Represents the capability to set background color.
    - `TTYC_SETAF`: Represents the capability to set foreground color.
    - `TTYC_SETAL`: Indicates the capability to set alternate line.
    - `TTYC_SETRGBB`: Represents the capability to set RGB background.
    - `TTYC_SETRGBF`: Represents the capability to set RGB foreground.
    - `TTYC_SETULC`: Indicates the capability to set underline color.
    - `TTYC_SETULC1`: Represents the capability to set underline color 1.
    - `TTYC_SGR0`: Indicates the capability to reset SGR.
    - `TTYC_SITM`: Represents the capability to set italic mode.
    - `TTYC_SMACS`: Indicates the capability to set alternate character set.
    - `TTYC_SMCUP`: Represents the capability to set cursor up.
    - `TTYC_SMKX`: Indicates the capability to set mouse keys.
    - `TTYC_SMOL`: Represents the capability to set overline.
    - `TTYC_SMSO`: Indicates the capability to set standout mode.
    - `TTYC_SMUL`: Represents the capability to set underline.
    - `TTYC_SMULX`: Indicates the capability to set underline extended.
    - `TTYC_SMXX`: Represents the capability to set multiple attributes.
    - `TTYC_SXL`: Indicates the capability to set extra line.
    - `TTYC_SS`: Represents the capability to set screen size.
    - `TTYC_SWD`: Indicates the capability to set window dimensions.
    - `TTYC_SYNC`: Represents the capability to synchronize.
    - `TTYC_TC`: Indicates the capability for terminal control.
    - `TTYC_TSL`: Represents the capability for terminal size.
    - `TTYC_U8`: Indicates the capability for UTF-8.
    - `TTYC_VPA`: Represents the capability to set vertical position.
    - `TTYC_XT`: Indicates the capability for extended terminal.
- **Description**: The `tty_code_code` enum defines a set of terminal capabilities that can be used to control various aspects of terminal behavior, such as cursor movement, text attributes, and screen manipulation. Each member of the enum corresponds to a specific capability, allowing for flexible terminal control in applications.


---
### utf8_data
- **Type**: `struct`
- **Members**:
    - `data`: An array of `u_char` to hold UTF-8 encoded character data.
    - `have`: A `u_char` indicating how many bytes of the UTF-8 character are currently available.
    - `size`: A `u_char` representing the total size of the UTF-8 character.
    - `width`: A `u_char` that indicates the width of the character, with 0xff denoting an invalid character.
- **Description**: The `utf8_data` structure is designed to represent a single UTF-8 character, encapsulating its encoded data, the number of bytes currently available, the total size of the character, and its visual width. This structure is essential for handling UTF-8 characters in applications that require precise character representation, such as text editors or terminal emulators.


---
### utf8_state
- **Type**: `enum`
- **Members**:
    - `UTF8_MORE`: Indicates that more bytes are expected to complete the UTF-8 character.
    - `UTF8_DONE`: Indicates that a complete UTF-8 character has been successfully parsed.
    - `UTF8_ERROR`: Indicates that an error occurred while parsing a UTF-8 character.
- **Description**: The `utf8_state` enum defines the possible states for processing UTF-8 encoded characters, allowing the system to track whether it is still expecting more bytes, has successfully completed a character, or encountered an error during parsing.


---
### colour_palette
- **Type**: `struct`
- **Members**:
    - `fg`: An integer representing the foreground color.
    - `bg`: An integer representing the background color.
    - `palette`: A pointer to an array of integers representing a custom color palette.
    - `default_palette`: A pointer to an array of integers representing the default color palette.
- **Description**: The `colour_palette` structure is designed to manage color settings for a terminal or graphical interface, containing fields for foreground and background colors, as well as pointers to custom and default color palettes.


---
### grid_cell
- **Type**: `struct`
- **Members**:
    - `data`: Contains UTF-8 character data.
    - `attr`: Stores attributes for the grid cell.
    - `flags`: Holds various flags related to the cell.
    - `fg`: Represents the foreground color.
    - `bg`: Represents the background color.
    - `us`: Indicates underlined state.
    - `link`: Stores a link identifier.
- **Description**: The `grid_cell` structure represents a single cell in a grid, encapsulating information about the cell's content, attributes, colors, and state flags, allowing for detailed control over how each cell is rendered in a terminal interface.


---
### grid_extd_entry
- **Type**: `struct`
- **Members**:
    - `data`: A single UTF-8 character.
    - `attr`: Attributes associated with the character.
    - `flags`: Flags indicating special properties of the character.
    - `fg`: Foreground color index.
    - `bg`: Background color index.
    - `us`: Underlining style index.
    - `link`: Link index for additional data.
- **Description**: The `grid_extd_entry` structure represents an extended entry in a grid, containing a UTF-8 character along with its associated attributes, colors, and flags, allowing for detailed representation of text in a terminal or graphical interface.


---
### grid_cell_entry
- **Type**: `struct`
- **Members**:
    - `offset`: An unsigned integer representing an offset.
    - `data`: A structure containing attributes for character display.
    - `flags`: A byte representing various flags for the grid cell.
- **Description**: The `grid_cell_entry` structure represents an entry in a grid cell, which can either store an offset or a set of attributes (like foreground and background colors, character data, and additional flags) for rendering text in a terminal interface. It utilizes a union to efficiently manage memory by allowing the storage of either an offset or a detailed data structure, while the `flags` member provides additional information about the cell's state.


---
### grid_line
- **Type**: `struct`
- **Members**:
    - `celldata`: Pointer to an array of `grid_cell_entry` structures representing the cell data.
    - `cellused`: The number of cells currently used in the line.
    - `cellsize`: The total allocated size for the cell data.
    - `extddata`: Pointer to an array of `grid_extd_entry` structures for extended cell data.
    - `extdsize`: The size of the extended data.
    - `flags`: Integer flags for various attributes of the grid line.
    - `time`: Timestamp indicating the last modification time of the grid line.
- **Description**: The `grid_line` structure represents a single line in a grid, containing cell data and extended attributes. It includes pointers to arrays of cell entries and extended entries, along with metadata such as the number of used cells, total cell size, flags for line attributes, and a timestamp for tracking modifications. This structure is essential for managing the visual representation of text in terminal applications.


---
### grid
- **Type**: `struct`
- **Members**:
    - `flags`: An integer representing various flags for the grid.
    - `sx`: An unsigned integer representing the width of the grid.
    - `sy`: An unsigned integer representing the height of the grid.
    - `hscrolled`: An unsigned integer indicating the number of lines scrolled.
    - `hsize`: An unsigned integer representing the size of the history.
    - `hlimit`: An unsigned integer representing the limit of the history.
    - `linedata`: A pointer to an array of `grid_line` structures representing the lines in the grid.
- **Description**: The `grid` structure represents a two-dimensional grid used for managing a collection of cells, typically for terminal emulation. It contains metadata such as dimensions, scrolling history, and a pointer to the line data, which is an array of `grid_line` structures. Each line in the grid can hold multiple cells, allowing for complex text rendering and manipulation.


---
### grid_reader
- **Type**: `struct`
- **Members**:
    - `gd`: Pointer to a `grid` structure representing the grid being read.
    - `cx`: Unsigned integer representing the current x-coordinate of the cursor.
    - `cy`: Unsigned integer representing the current y-coordinate of the cursor.
- **Description**: The `grid_reader` structure is designed to facilitate reading from a grid, maintaining the current cursor position within the grid. It contains a pointer to a `grid` structure, as well as two unsigned integers that track the cursor's x and y coordinates, allowing for efficient navigation and manipulation of grid data.


---
### style_align
- **Type**: `enum`
- **Members**:
    - `STYLE_ALIGN_DEFAULT`: Default alignment style.
    - `STYLE_ALIGN_LEFT`: Aligns content to the left.
    - `STYLE_ALIGN_CENTRE`: Centers the content.
    - `STYLE_ALIGN_RIGHT`: Aligns content to the right.
    - `STYLE_ALIGN_ABSOLUTE_CENTRE`: Aligns content to the absolute center.
- **Description**: The `style_align` enum defines various alignment styles for content, allowing for flexible positioning such as left, center, right, and absolute center, which can be utilized in styling user interfaces.


---
### style_list
- **Type**: `enum`
- **Members**:
    - `STYLE_LIST_OFF`: Represents the off state of the style list.
    - `STYLE_LIST_ON`: Represents the on state of the style list.
    - `STYLE_LIST_FOCUS`: Indicates the focus state of the style list.
    - `STYLE_LIST_LEFT_MARKER`: Designates the left marker in the style list.
    - `STYLE_LIST_RIGHT_MARKER`: Designates the right marker in the style list.
- **Description**: The `style_list` enum defines a set of constants that represent various states and markers for styling elements in a user interface, allowing for easy management of visual styles such as on/off states and focus indicators.


---
### style_range_type
- **Type**: `enum`
- **Members**:
    - `STYLE_RANGE_NONE`: Indicates no specific style range.
    - `STYLE_RANGE_LEFT`: Indicates a style range that applies to the left side.
    - `STYLE_RANGE_RIGHT`: Indicates a style range that applies to the right side.
    - `STYLE_RANGE_PANE`: Indicates a style range that applies to a specific pane.
    - `STYLE_RANGE_WINDOW`: Indicates a style range that applies to a specific window.
    - `STYLE_RANGE_SESSION`: Indicates a style range that applies to a specific session.
    - `STYLE_RANGE_USER`: Indicates a style range that applies to a user-defined context.
- **Description**: The `style_range_type` is an enumeration that defines various types of style ranges that can be applied in a terminal multiplexer context. Each enumerator represents a different scope for styling, such as none, left, right, pane, window, session, or user, allowing for flexible styling options based on the context in which they are used.


---
### style_range
- **Type**: `struct`
- **Members**:
    - `type`: An enumeration indicating the type of style range.
    - `argument`: An unsigned integer argument associated with the style range.
    - `string`: A character array to hold a string representation of the style range.
    - `start`: An unsigned integer representing the start position of the style range.
    - `end`: An unsigned integer representing the end position of the style range, not included.
    - `entry`: A TAILQ_ENTRY structure for linking style ranges in a queue.
- **Description**: The `style_range` structure defines a range of styles that can be applied to text or UI elements, including the type of style, associated arguments, and the specific range of text it applies to. It includes an enumeration for the style type, an argument for additional parameters, a string for naming the style, and start and end positions to define the range. The structure also supports linking multiple style ranges using a TAILQ (tail queue) entry.


---
### style_default_type
- **Type**: `enum`
- **Members**:
    - `STYLE_DEFAULT_BASE`: Represents the base style default.
    - `STYLE_DEFAULT_PUSH`: Indicates a push style default.
    - `STYLE_DEFAULT_POP`: Indicates a pop style default.
- **Description**: The `style_default_type` enum defines three constants that represent different default styles used in the application, allowing for a clear distinction between base, push, and pop styles.


---
### style
- **Type**: `struct style`
- **Members**:
    - `gc`: A `grid_cell` structure representing the graphical cell attributes.
    - `ignore`: An integer flag to indicate whether to ignore certain styles.
    - `fill`: An integer value representing the fill style.
    - `align`: An enumeration indicating the alignment of the style.
    - `list`: An enumeration indicating the list style.
    - `range_type`: An enumeration indicating the type of range for the style.
    - `range_argument`: An unsigned integer representing an argument for the range.
    - `range_string`: A character array for storing a string related to the range.
    - `width`: An integer representing the width of the style.
    - `pad`: An integer representing padding for the style.
    - `default_type`: An enumeration indicating the default type of the style.
- **Description**: The `style` structure encapsulates various attributes related to the styling of grid cells in a terminal interface. It includes graphical cell attributes, alignment options, fill styles, and range specifications, allowing for flexible and customizable visual representation in terminal applications.


---
### image
- **Type**: `struct`
- **Members**:
    - `s`: Pointer to a `screen` structure representing the screen context for the image.
    - `data`: Pointer to a `sixel_image` structure containing the image data.
    - `fallback`: String representing a fallback image or data.
    - `px`: Unsigned integer representing the pixel width of the image.
    - `py`: Unsigned integer representing the pixel height of the image.
    - `sx`: Unsigned integer representing the source width of the image.
    - `sy`: Unsigned integer representing the source height of the image.
    - `all_entry`: Entry for linking this image in a global list of images.
    - `entry`: Entry for linking this image in a specific list.
- **Description**: The `image` structure is designed to represent an image in a terminal application, encapsulating details such as the screen context, image data, dimensions, and fallback options. It includes pointers to a `screen` and `sixel_image`, as well as pixel dimensions and linked list entries for managing multiple images.


---
### screen_cursor_style
- **Type**: `enum`
- **Members**:
    - `SCREEN_CURSOR_DEFAULT`: Represents the default cursor style.
    - `SCREEN_CURSOR_BLOCK`: Represents a block cursor style.
    - `SCREEN_CURSOR_UNDERLINE`: Represents an underline cursor style.
    - `SCREEN_CURSOR_BAR`: Represents a vertical bar cursor style.
- **Description**: The `screen_cursor_style` enum defines various styles for the cursor in a terminal screen, allowing for different visual representations such as a block, underline, or bar cursor, as well as a default style.


---
### screen
- **Type**: `struct`
- **Members**:
    - `title`: A pointer to a string representing the title of the screen.
    - `path`: A pointer to a string representing the path associated with the screen.
    - `titles`: A pointer to a `screen_titles` structure containing titles for the screen.
    - `grid`: A pointer to a `grid` structure that holds the grid data for the screen.
    - `cx`: An unsigned integer representing the current cursor x-coordinate.
    - `cy`: An unsigned integer representing the current cursor y-coordinate.
    - `cstyle`: An enumeration representing the current cursor style.
    - `default_cstyle`: An enumeration representing the default cursor style.
    - `ccolour`: An integer representing the current cursor color.
    - `default_ccolour`: An integer representing the default cursor color.
    - `rupper`: An unsigned integer representing the top of the scroll region.
    - `rlower`: An unsigned integer representing the bottom of the scroll region.
    - `mode`: An integer representing the current mode of the screen.
    - `default_mode`: An integer representing the default mode of the screen.
    - `saved_cx`: An unsigned integer representing the saved cursor x-coordinate.
    - `saved_cy`: An unsigned integer representing the saved cursor y-coordinate.
    - `saved_grid`: A pointer to a `grid` structure representing the saved grid.
    - `saved_cell`: A `grid_cell` structure representing the saved cell.
    - `saved_flags`: An integer representing the saved flags.
    - `tabs`: A pointer to a `bitstr_t` structure representing tab settings.
    - `sel`: A pointer to a `screen_sel` structure representing the screen selection.
    - `images`: A structure representing images, enabled if SIXEL support is compiled.
    - `write_list`: A pointer to a `screen_write_cline` structure representing the write list.
    - `hyperlinks`: A pointer to a `hyperlinks` structure representing hyperlinks.
- **Description**: The `screen` structure represents a virtual screen in a terminal multiplexer, encapsulating various attributes such as the title, path, cursor position, cursor style, grid data, scroll region, and selection state. It also maintains saved states for the cursor and grid, as well as support for hyperlinks and images, allowing for a rich user interface in terminal applications.


---
### screen_write_ctx
- **Type**: `struct`
- **Members**:
    - `wp`: Pointer to a `window_pane` structure representing the associated window pane.
    - `s`: Pointer to a `screen` structure representing the screen context.
    - `flags`: Integer flags for controlling screen write behavior, including synchronization.
    - `init_ctx_cb`: Callback function for initializing the screen write context.
    - `arg`: Pointer to user-defined argument passed to the initialization callback.
    - `item`: Pointer to a `screen_write_citem` structure representing the current item being written.
    - `scrolled`: Unsigned integer representing the amount scrolled in the screen.
    - `bg`: Unsigned integer representing the background color.
- **Description**: The `screen_write_ctx` structure is used to manage the context for writing to a screen in a terminal multiplexer environment. It holds references to the associated window pane and screen, as well as flags to control the writing behavior, a callback for initialization, and additional parameters for managing the writing process. This structure facilitates the organization and execution of screen output operations, ensuring that the correct context and settings are applied during screen updates.


---
### box_lines
- **Type**: `enum`
- **Members**:
    - `BOX_LINES_DEFAULT`: Represents the default box line style.
    - `BOX_LINES_SINGLE`: Represents a single line box style.
    - `BOX_LINES_DOUBLE`: Represents a double line box style.
    - `BOX_LINES_HEAVY`: Represents a heavy line box style.
    - `BOX_LINES_SIMPLE`: Represents a simple line box style.
    - `BOX_LINES_ROUNDED`: Represents a rounded box line style.
    - `BOX_LINES_PADDED`: Represents a padded box line style.
    - `BOX_LINES_NONE`: Indicates no box lines.
- **Description**: The `box_lines` enum defines various styles for box borders that can be used in a terminal interface, allowing for customization of how boxes are visually represented.


---
### pane_lines
- **Type**: `enum`
- **Members**:
    - `PANE_LINES_SINGLE`: Represents a single line pane border.
    - `PANE_LINES_DOUBLE`: Represents a double line pane border.
    - `PANE_LINES_HEAVY`: Represents a heavy line pane border.
    - `PANE_LINES_SIMPLE`: Represents a simple line pane border.
    - `PANE_LINES_NUMBER`: Represents the total number of pane line types.
- **Description**: The `pane_lines` enum defines various styles of borders that can be applied to panes in a terminal multiplexer, allowing for visual differentiation between panes based on the selected border style.


---
### screen_redraw_ctx
- **Type**: `struct`
- **Members**:
    - `c`: Pointer to a `client` structure representing the associated client.
    - `statuslines`: Unsigned integer representing the number of status lines.
    - `statustop`: Integer indicating the position of the status line.
    - `pane_status`: Integer representing the status of the pane.
    - `pane_lines`: Enumeration indicating the type of pane lines.
    - `pane_scrollbars`: Integer indicating the visibility of scrollbars in the pane.
    - `pane_scrollbars_pos`: Integer indicating the position of the scrollbars.
    - `no_pane_gc`: A `grid_cell` structure representing the graphical context when no pane is present.
    - `no_pane_gc_set`: Integer indicating whether the no pane graphical context is set.
    - `sx`: Unsigned integer representing the screen width.
    - `sy`: Unsigned integer representing the screen height.
    - `ox`: Unsigned integer representing the original x offset.
    - `oy`: Unsigned integer representing the original y offset.
- **Description**: The `screen_redraw_ctx` structure is used to maintain the context for redrawing the screen in a terminal multiplexer. It holds information about the associated client, the status lines, pane configurations, graphical context for when no pane is present, and the dimensions of the screen. This structure is essential for managing how the terminal interface is rendered and updated.


---
### menu_item
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the name of the menu item.
    - `key`: A `key_code` representing the key associated with the menu item.
    - `command`: A pointer to a string representing the command to be executed when the menu item is selected.
- **Description**: The `menu_item` structure represents an individual item in a menu, containing a name, a key code for keyboard shortcuts, and a command that is executed when the item is selected. This structure is useful for creating interactive menus in applications, allowing users to navigate and execute commands efficiently.


---
### menu
- **Type**: `struct`
- **Members**:
    - `title`: A pointer to a string representing the title of the menu.
    - `items`: A pointer to an array of `menu_item` structures representing the items in the menu.
    - `count`: An unsigned integer representing the number of items in the menu.
    - `width`: An unsigned integer representing the width of the menu.
- **Description**: The `menu` structure represents a menu in a user interface, containing a title, an array of menu items, and metadata such as the count of items and the width of the menu. Each menu item is defined by the `menu_item` structure, which includes the item's name, associated key, and command to execute when selected.


---
### window_mode
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the name of the window mode.
    - `default_format`: A pointer to a string representing the default format for the window mode.
    - `init`: A function pointer to initialize the window mode.
    - `free`: A function pointer to free resources associated with the window mode.
    - `resize`: A function pointer to handle resizing of the window mode.
    - `update`: A function pointer to update the window mode.
    - `key`: A function pointer to handle key events in the window mode.
    - `key_table`: A function pointer that returns the key table for the window mode.
    - `command`: A function pointer to execute commands in the window mode.
    - `formats`: A function pointer to handle format trees in the window mode.
    - `get_screen`: A function pointer to retrieve the screen associated with the window mode.
- **Description**: The `window_mode` structure defines a set of functions and properties that represent a specific mode for a window in a terminal multiplexer. It includes function pointers for initializing, freeing, resizing, updating, and handling key events, as well as properties for the mode's name and default format. This structure allows for flexible management of different window modes, enabling the terminal multiplexer to switch between various operational contexts.


---
### window_mode_entry
- **Type**: `struct`
- **Members**:
    - `wp`: Pointer to a `window_pane` structure representing the main window pane.
    - `swp`: Pointer to a `window_pane` structure representing the secondary window pane.
    - `mode`: Pointer to a `window_mode` structure defining the current mode of the window.
    - `data`: Pointer to user-defined data associated with the window mode.
    - `screen`: Pointer to a `screen` structure representing the screen context.
    - `prefix`: An unsigned integer representing a prefix value for input handling.
    - `entry`: A `TAILQ_ENTRY` structure for linking `window_mode_entry` instances in a queue.
- **Description**: The `window_mode_entry` structure is used to represent an entry in a window's mode, encapsulating information about the current state of the window pane, including pointers to the main and secondary panes, the mode being used, associated data, and the screen context. It also includes a prefix for input handling and a queue entry for managing multiple mode entries.


---
### window_pane_offset
- **Type**: `struct`
- **Members**:
    - `used`: Indicates the amount of data used in the pane.
- **Description**: The `window_pane_offset` structure is designed to track the amount of data that has been utilized within a specific pane, represented by the `used` member, which is of type `size_t`.


---
### window_pane_resize
- **Type**: `struct`
- **Members**:
    - `sx`: The current width of the window pane.
    - `sy`: The current height of the window pane.
    - `osx`: The original width of the window pane before resizing.
    - `osy`: The original height of the window pane before resizing.
    - `entry`: A linked list entry for managing multiple resize requests.
- **Description**: The `window_pane_resize` structure is used to represent the dimensions of a window pane in a terminal multiplexer, specifically tracking both the current and original sizes. It includes fields for the current size (`sx`, `sy`) and the original size (`osx`, `osy`), as well as a linked list entry (`entry`) to facilitate queuing multiple resize requests.


---
### window_pane
- **Type**: `struct`
- **Members**:
    - `id`: Unique identifier for the pane.
    - `active_point`: Indicates the active point within the pane.
    - `window`: Pointer to the associated `window` structure.
    - `options`: Pointer to the `options` structure for configuration.
    - `layout_cell`: Pointer to the current layout cell.
    - `saved_layout_cell`: Pointer to the saved layout cell.
    - `sx`: Width of the pane.
    - `sy`: Height of the pane.
    - `xoff`: X offset of the pane.
    - `yoff`: Y offset of the pane.
    - `flags`: Flags indicating the state of the pane.
    - `sb_slider_y`: Y position of the scrollbar slider.
    - `sb_slider_h`: Height of the scrollbar slider.
    - `argc`: Argument count for the pane.
    - `argv`: Array of argument values for the pane.
    - `shell`: Shell associated with the pane.
    - `cwd`: Current working directory of the pane.
    - `pid`: Process ID of the pane.
    - `tty`: Terminal type associated with the pane.
    - `status`: Status of the pane.
    - `dead_time`: Time when the pane was marked as dead.
    - `fd`: File descriptor for the pane.
    - `event`: Pointer to the event associated with the pane.
    - `offset`: Offset structure for the pane.
    - `base_offset`: Base offset for the pane.
    - `resize_queue`: Queue of resize requests for the pane.
    - `resize_timer`: Timer for handling pane resize events.
    - `ictx`: Input context for the pane.
    - `cached_gc`: Cached grid cell for rendering.
    - `cached_active_gc`: Cached active grid cell for rendering.
    - `palette`: Colour palette used in the pane.
    - `pipe_fd`: File descriptor for the pipe.
    - `pipe_event`: Event associated with the pipe.
    - `pipe_offset`: Offset structure for the pipe.
    - `screen`: Pointer to the screen structure.
    - `base`: Base screen structure.
    - `status_screen`: Screen for displaying status.
    - `status_size`: Size of the status screen.
    - `modes`: List of modes associated with the pane.
    - `searchstr`: Search string for the pane.
    - `searchregex`: Flag indicating if the search string is a regex.
    - `border_gc_set`: Flag indicating if border graphics context is set.
    - `border_gc`: Graphics context for the border.
    - `control_bg`: Background colour for control.
    - `control_fg`: Foreground colour for control.
    - `scrollbar_style`: Style for the scrollbar.
    - `entry`: Link in the list of all panes.
    - `sentry`: Link in the list of last visited panes.
    - `tree_entry`: Entry for the red-black tree of panes.
- **Description**: The `window_pane` structure represents a single pane within a window in a terminal multiplexer. It contains various attributes such as its dimensions, position, associated window, options, and state flags. The structure also manages input context, event handling, and graphical representation, allowing for complex interactions and rendering within the terminal environment.


---
### window
- **Type**: `struct`
- **Members**:
    - `id`: A unique identifier for the window.
    - `latest`: Pointer to the latest event or data associated with the window.
    - `name`: A string representing the name of the window.
    - `name_event`: An event structure associated with name changes.
    - `name_time`: A timestamp indicating the last time the name was updated.
    - `alerts_timer`: An event structure for managing alert timers.
    - `offset_timer`: An event structure for managing offset timers.
    - `activity_time`: A timestamp indicating the last activity time of the window.
    - `active`: Pointer to the currently active pane in the window.
    - `last_panes`: A list of the last panes used in the window.
    - `panes`: A list of all panes associated with the window.
    - `lastlayout`: An integer representing the last layout used.
    - `layout_root`: Pointer to the root layout cell of the window.
    - `saved_layout_root`: Pointer to the saved layout root for restoration.
    - `old_layout`: A string representing the previous layout.
    - `sx`: The current width of the window in character cells.
    - `sy`: The current height of the window in character cells.
    - `manual_sx`: The manually set width of the window.
    - `manual_sy`: The manually set height of the window.
    - `xpixel`: The width of the window in pixels.
    - `ypixel`: The height of the window in pixels.
    - `new_sx`: The new width of the window to be set.
    - `new_sy`: The new height of the window to be set.
    - `new_xpixel`: The new width of the window in pixels.
    - `new_ypixel`: The new height of the window in pixels.
    - `fill_character`: Pointer to the character used to fill the window.
    - `flags`: Flags representing various states of the window.
    - `alerts_queued`: Count of alerts that are queued for the window.
    - `alerts_entry`: Entry structure for managing alerts in a linked list.
    - `options`: Pointer to the options associated with the window.
    - `references`: Count of references to the window.
    - `winlinks`: A linked list of winlink structures associated with the window.
    - `entry`: Red-black tree entry for managing windows.
- **Description**: The `window` structure represents a window in a terminal multiplexer, encapsulating various attributes such as its dimensions, name, active pane, layout information, and event handling. It maintains a list of panes, manages alerts, and tracks the window's state through various flags. The structure is designed to facilitate the management of multiple terminal panes within a single window, allowing for complex layouts and user interactions.


---
### winlink
- **Type**: `struct`
- **Members**:
    - `idx`: An integer representing the index of the winlink.
    - `session`: A pointer to the associated `session` structure.
    - `window`: A pointer to the associated `window` structure.
    - `flags`: An integer representing various flags for the winlink.
    - `entry`: A red-black tree entry for managing winlinks.
    - `wentry`: A tail queue entry for managing winlinks in a list.
    - `sentry`: A tail queue entry for managing winlinks in a separate list.
- **Description**: The `winlink` structure represents a link between a session and a window in a terminal multiplexer, allowing for the management of multiple windows within a session. It contains an index, pointers to the associated session and window, flags for various states (like alerts), and entries for maintaining order in red-black trees and tail queues.


---
### layout_type
- **Type**: `enum`
- **Members**:
    - `LAYOUT_LEFTRIGHT`: Layout type for left-to-right arrangement.
    - `LAYOUT_TOPBOTTOM`: Layout type for top-to-bottom arrangement.
    - `LAYOUT_WINDOWPANE`: Layout type for window pane arrangement.
- **Description**: The `layout_type` enum defines the possible layout arrangements for a user interface, specifying how components are organized visually, including left-to-right, top-to-bottom, and window pane configurations.


---
### layout_cell
- **Type**: `struct`
- **Members**:
    - `type`: An enumeration indicating the layout type of the cell.
    - `parent`: A pointer to the parent `layout_cell`, forming a tree structure.
    - `sx`: The width of the cell in screen units.
    - `sy`: The height of the cell in screen units.
    - `xoff`: The x-offset of the cell from its parent.
    - `yoff`: The y-offset of the cell from its parent.
    - `wp`: A pointer to the associated `window_pane`.
    - `cells`: A linked list of child `layout_cell` structures.
    - `entry`: A queue entry for linking `layout_cell` instances.
- **Description**: The `layout_cell` structure represents a cell in a layout, which can contain a window pane or other layout cells. It includes properties for its dimensions, offsets, and relationships to parent and child cells, allowing for a hierarchical arrangement of layout components. The `type` field specifies the layout orientation, while the `cells` field manages child cells in a linked list.


---
### environ_entry
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the name of the environment variable.
    - `value`: A pointer to a string representing the value of the environment variable.
    - `flags`: An integer representing flags associated with the environment variable, such as visibility.
    - `entry`: A red-black tree entry structure for maintaining a sorted list of environment entries.
- **Description**: The `environ_entry` structure represents an environment variable with its name, value, and associated flags, and is designed to be used within a red-black tree for efficient storage and retrieval of environment variables.


---
### session_group
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the name of the session group.
    - `sessions`: A doubly linked list head for managing a list of `session` structures.
    - `entry`: A red-black tree entry structure for maintaining a sorted collection of session groups.
- **Description**: The `session_group` structure represents a collection of sessions grouped under a common name. It contains a pointer to the group's name, a linked list of sessions that belong to this group, and a red-black tree entry for efficient organization and retrieval of session groups. This structure is essential for managing multiple sessions in a terminal multiplexer environment.


---
### session
- **Type**: `struct`
- **Members**:
    - `id`: A unique identifier for the session.
    - `name`: The name of the session.
    - `cwd`: The current working directory of the session.
    - `creation_time`: The time when the session was created.
    - `last_attached_time`: The time when the session was last attached.
    - `activity_time`: The last recorded activity time of the session.
    - `last_activity_time`: The time of the last activity in the session.
    - `lock_timer`: Timer for session locking.
    - `curw`: Pointer to the current window link.
    - `lastw`: Stack of last window links.
    - `windows`: List of windows associated with the session.
    - `statusat`: Status line index for the session.
    - `statuslines`: Number of status lines for the session.
    - `options`: Pointer to session-specific options.
    - `flags`: Flags indicating session state.
    - `attached`: Count of attached clients.
    - `tio`: Terminal I/O settings for the session.
    - `environ`: Environment variables for the session.
    - `references`: Reference count for the session.
    - `gentry`: Entry for the session in a global list.
    - `entry`: Entry for the session in a red-black tree.
- **Description**: The `session` structure represents a user session in a terminal multiplexer, encapsulating various attributes such as session ID, name, current working directory, timestamps for creation and activity, and associated windows. It also maintains state information like the number of attached clients, session options, and flags for session management, allowing for efficient handling of multiple terminal sessions.


---
### mouse_event
- **Type**: `struct`
- **Members**:
    - `valid`: Indicates if the mouse event is valid.
    - `ignore`: Flag to ignore the event.
    - `key`: Stores the key code associated with the event.
    - `statusat`: Represents the status line index.
    - `statuslines`: Number of status lines.
    - `x`: X coordinate of the mouse event.
    - `y`: Y coordinate of the mouse event.
    - `b`: Mouse button state.
    - `lx`: Last X coordinate.
    - `ly`: Last Y coordinate.
    - `lb`: Last button state.
    - `ox`: Original X coordinate.
    - `oy`: Original Y coordinate.
    - `s`: State of the mouse event.
    - `w`: Width of the mouse event area.
    - `wp`: Width of the pane.
    - `sgr_type`: Type of SGR (Select Graphic Rendition) event.
    - `sgr_b`: SGR button state.
- **Description**: The `mouse_event` structure is designed to encapsulate all relevant information regarding mouse events in a terminal application. It includes fields for the event's validity, coordinates, button states, and additional parameters related to the terminal's status lines and graphical rendering. This structure is essential for handling user interactions through mouse input effectively.


---
### key_event
- **Type**: `struct`
- **Members**:
    - `key`: A `key_code` representing the key event.
    - `m`: A `mouse_event` structure that holds mouse event details.
    - `buf`: A pointer to a character buffer for additional key event data.
    - `len`: A `size_t` representing the length of the buffer.
- **Description**: The `key_event` structure encapsulates information about a keyboard event, including the key pressed, associated mouse event details, and a buffer for any additional data related to the key event. It combines keyboard and mouse interactions, making it useful for handling complex input scenarios in applications.


---
### tty_term
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the terminal's name.
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `features`: An integer representing the features supported by the terminal.
    - `acs`: A 2D array for alternate character set mappings.
    - `codes`: A pointer to a `tty_code` structure containing terminal capabilities.
    - `flags`: An integer representing various flags related to terminal capabilities.
    - `entry`: A linked list entry for managing `tty_term` instances.
- **Description**: The `tty_term` structure defines a terminal type with its associated properties, including its name, capabilities, and features. It contains a pointer to a `tty` structure, an array for alternate character sets, and flags that indicate specific terminal features. This structure is used to manage terminal capabilities in a terminal multiplexer environment.


---
### tty
- **Type**: `struct tty`
- **Members**:
    - `client`: Pointer to the associated client structure.
    - `start_timer`: Event structure for managing the start timer.
    - `clipboard_timer`: Event structure for managing the clipboard timer.
    - `last_requests`: Timestamp of the last requests made.
    - `sx`: Width of the terminal in character cells.
    - `sy`: Height of the terminal in character cells.
    - `xpixel`: Width of a cell in pixels.
    - `ypixel`: Height of a cell in pixels.
    - `cx`: Current cursor x position.
    - `cy`: Current cursor y position.
    - `cstyle`: Style of the screen cursor.
    - `ccolour`: Colour of the cursor.
    - `oflag`: Flag indicating if the drawing area is larger than the terminal.
    - `oox`: Original x offset for drawing.
    - `ooy`: Original y offset for drawing.
    - `osx`: Original screen width.
    - `osy`: Original screen height.
    - `mode`: Current mode of the terminal.
    - `fg`: Foreground colour.
    - `bg`: Background colour.
    - `rlower`: Lower bound of the scroll region.
    - `rupper`: Upper bound of the scroll region.
    - `rleft`: Left bound of the scroll region.
    - `rright`: Right bound of the scroll region.
    - `event_in`: Event structure for input events.
    - `in`: Buffer for input data.
    - `event_out`: Event structure for output events.
    - `out`: Buffer for output data.
    - `timer`: Event structure for managing timers.
    - `discarded`: Count of discarded input.
    - `tio`: Terminal I/O settings.
    - `cell`: Current grid cell.
    - `last_cell`: Last grid cell.
    - `flags`: Flags for various terminal states.
    - `term`: Pointer to the terminal definition.
    - `mouse_last_x`: Last x position of the mouse.
    - `mouse_last_y`: Last y position of the mouse.
    - `mouse_last_b`: Last button pressed on the mouse.
    - `mouse_drag_flag`: Flag indicating if mouse dragging is active.
    - `mouse_scrolling_flag`: Flag indicating if mouse scrolling is active.
    - `mouse_slider_mpos`: Mouse slider position.
    - `mouse_drag_update`: Callback for mouse drag updates.
    - `mouse_drag_release`: Callback for mouse drag release.
    - `key_timer`: Event structure for key events.
    - `key_tree`: Pointer to the key tree structure.
- **Description**: The `tty` structure represents a terminal interface in a client-server architecture, encapsulating various properties such as client connection, screen dimensions, cursor position, event handling, and terminal settings. It manages input and output buffers, mouse interactions, and maintains state flags for terminal behavior, allowing for efficient rendering and user interaction in a terminal environment.


---
### tty_ctx
- **Type**: `struct tty_ctx`
- **Members**:
    - `s`: Pointer to a `screen` structure representing the screen context.
    - `redraw_cb`: Callback function for redrawing the terminal.
    - `set_client_cb`: Callback function for setting the client.
    - `arg`: Pointer to user-defined argument for callbacks.
    - `cell`: Pointer to a `grid_cell` structure representing the current cell.
    - `wrapped`: Integer indicating if the line is wrapped.
    - `num`: Unsigned integer for a numeric identifier.
    - `ptr`: Generic pointer for additional data.
    - `ptr2`: Another generic pointer for additional data.
    - `allow_invisible_panes`: Integer flag indicating if commands can be sent to invisible panes.
    - `ocx`: Unsigned integer for the original cursor x position.
    - `ocy`: Unsigned integer for the original cursor y position.
    - `orupper`: Unsigned integer for the original upper scroll region.
    - `orlower`: Unsigned integer for the original lower scroll region.
    - `xoff`: Unsigned integer for the x offset of the target region.
    - `yoff`: Unsigned integer for the y offset of the target region.
    - `rxoff`: Unsigned integer for the right x offset of the target region.
    - `ryoff`: Unsigned integer for the right y offset of the target region.
    - `sx`: Unsigned integer for the width of the target region.
    - `sy`: Unsigned integer for the height of the target region.
    - `bg`: Unsigned integer for the background color used for clearing.
    - `defaults`: A `grid_cell` structure representing default colors and attributes.
    - `palette`: Pointer to a `colour_palette` structure for color management.
    - `bigger`: Integer flag indicating if the containing region is larger.
    - `wox`: Unsigned integer for the x offset of the containing region.
    - `woy`: Unsigned integer for the y offset of the containing region.
    - `wsx`: Unsigned integer for the width of the containing region.
    - `wsy`: Unsigned integer for the height of the containing region.
- **Description**: The `tty_ctx` structure encapsulates the context for terminal operations, including screen management, cursor positioning, and color settings. It contains various fields for handling the state of the terminal, such as callbacks for redrawing and setting the client, as well as information about the current cell, offsets, and dimensions of the target region. This structure is essential for managing the terminal's display and interaction with the user.


---
### message_entry
- **Type**: `struct`
- **Members**:
    - `msg`: A pointer to a character string representing the message.
    - `msg_num`: An unsigned integer representing the message number.
    - `msg_time`: A `struct timeval` representing the time the message was created.
    - `entry`: A `TAILQ_ENTRY` for linking this message entry in a queue.
- **Description**: The `message_entry` structure is designed to store information about a message, including the message text, a unique message number, the timestamp of when the message was created, and a link to other message entries in a queue. This structure is useful for managing a list of messages in applications that require message logging or communication.


---
### args_type
- **Type**: `enum`
- **Members**:
    - `ARGS_NONE`: Indicates that no arguments are provided.
    - `ARGS_STRING`: Indicates that the argument is a string.
    - `ARGS_COMMANDS`: Indicates that the argument consists of commands.
- **Description**: The `args_type` enum defines the types of arguments that can be processed, allowing for differentiation between no arguments, string arguments, and command arguments in a command-line interface context.


---
### args_value
- **Type**: `struct`
- **Members**:
    - `type`: An enumeration indicating the type of argument.
    - `string`: A pointer to a string value if the type is ARGS_STRING.
    - `cmdlist`: A pointer to a list of commands if the type is ARGS_COMMANDS.
    - `cached`: A cached string representation of the argument value.
    - `entry`: A linked list entry for managing multiple `args_value` instances.
- **Description**: The `args_value` structure is used to represent a single argument value in a command-line interface, supporting multiple types of values through a union. It can hold either a string or a list of commands, and includes a cached representation for efficiency. The structure also integrates with a linked list for easy management of multiple argument values.


---
### args_parse_type
- **Type**: `enum`
- **Members**:
    - `ARGS_PARSE_INVALID`: Indicates an invalid argument parsing type.
    - `ARGS_PARSE_STRING`: Indicates that the argument is expected to be a string.
    - `ARGS_PARSE_COMMANDS_OR_STRING`: Indicates that the argument can be either commands or a string.
    - `ARGS_PARSE_COMMANDS`: Indicates that the argument is expected to be commands.
- **Description**: The `args_parse_type` enum defines various types of argument parsing states used in command-line argument processing, allowing the system to differentiate between invalid inputs, string inputs, and command inputs.


---
### args_parse
- **Type**: `struct`
- **Members**:
    - `template`: A pointer to a string that defines the argument template.
    - `lower`: An integer representing the lower bound for argument parsing.
    - `upper`: An integer representing the upper bound for argument parsing.
    - `cb`: A callback function used for custom argument parsing.
- **Description**: The `args_parse` structure is designed to facilitate the parsing of command-line arguments by defining a template, specifying lower and upper bounds for the number of arguments, and providing a callback function for custom parsing logic.


---
### cmd_find_type
- **Type**: `enum`
- **Members**:
    - `CMD_FIND_PANE`: Represents a command that targets a specific pane.
    - `CMD_FIND_WINDOW`: Represents a command that targets a specific window.
    - `CMD_FIND_SESSION`: Represents a command that targets a specific session.
- **Description**: The `cmd_find_type` enum defines constants used to specify the type of target for commands in a terminal multiplexer context, allowing commands to be directed towards panes, windows, or sessions.


---
### cmd_find_state
- **Type**: `struct`
- **Members**:
    - `flags`: An integer representing various flags for the command find state.
    - `current`: A pointer to the current `cmd_find_state` structure.
    - `s`: A pointer to the associated `session` structure.
    - `wl`: A pointer to the associated `winlink` structure.
    - `w`: A pointer to the associated `window` structure.
    - `wp`: A pointer to the associated `window_pane` structure.
    - `idx`: An integer index for identifying the state.
- **Description**: The `cmd_find_state` structure is used to maintain the state of command finding operations within a terminal multiplexer. It contains various pointers to related structures such as `session`, `winlink`, `window`, and `window_pane`, as well as an integer for flags and an index. This structure facilitates the management of command execution contexts and helps in navigating between different terminal panes and windows.


---
### cmd_list
- **Type**: `struct`
- **Members**:
    - `references`: Counts the number of references to this command list.
    - `group`: Indicates the group to which this command list belongs.
    - `list`: Pointer to a list of commands associated with this command list.
- **Description**: The `cmd_list` structure is designed to manage a list of commands in a command processing system. It contains a reference count to track how many times the command list is referenced, a group identifier to categorize the command list, and a pointer to the actual list of commands, allowing for dynamic command management and execution.


---
### cmd_retval
- **Type**: `enum`
- **Members**:
    - `CMD_RETURN_ERROR`: Indicates an error occurred.
    - `CMD_RETURN_NORMAL`: Indicates normal operation.
    - `CMD_RETURN_WAIT`: Indicates that the command is waiting.
    - `CMD_RETURN_STOP`: Indicates that the command has stopped.
- **Description**: The `cmd_retval` enum defines the possible return values for command execution in the system, allowing for standardized handling of command outcomes such as errors, normal completion, waiting states, and stopping conditions.


---
### cmd_parse_status
- **Type**: `enum`
- **Members**:
    - `CMD_PARSE_ERROR`: Indicates a parsing error occurred.
    - `CMD_PARSE_SUCCESS`: Indicates the parsing was successful.
- **Description**: The `cmd_parse_status` is an enumeration that defines the possible outcomes of a command parsing operation, with two values: `CMD_PARSE_ERROR` for indicating an error during parsing and `CMD_PARSE_SUCCESS` for indicating successful parsing.


---
### cmd_parse_result
- **Type**: `struct`
- **Members**:
    - `status`: Indicates the result of the command parsing process.
    - `cmdlist`: A pointer to a list of commands parsed from the input.
    - `error`: A string that holds any error message encountered during parsing.
- **Description**: The `cmd_parse_result` structure is used to encapsulate the outcome of parsing a command input in a command-line interface. It contains a status field to indicate whether the parsing was successful or if an error occurred, a pointer to a list of commands that were successfully parsed, and an error message string that provides details about any issues encountered during the parsing process.


---
### cmd_parse_input
- **Type**: `struct cmd_parse_input`
- **Members**:
    - `flags`: An integer representing various command parsing options.
    - `file`: A pointer to a string representing the file from which commands are parsed.
    - `line`: An unsigned integer indicating the line number in the file.
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item.
    - `c`: A pointer to a `client` structure representing the client associated with the command.
    - `fs`: A `cmd_find_state` structure representing the state of command finding.
- **Description**: The `cmd_parse_input` structure is used to encapsulate the input parameters required for parsing commands in a command-line interface. It includes flags for controlling the parsing behavior, a reference to the source file and line number for error reporting, and pointers to relevant structures such as the command queue item and client, facilitating the execution of commands within the context of a terminal multiplexer.


---
### cmd_entry_flag
- **Type**: `struct`
- **Members**:
    - `flag`: A character representing a specific command flag.
    - `type`: An enumeration indicating the type of command find operation.
    - `flags`: An integer representing additional flags associated with the command entry.
- **Description**: The `cmd_entry_flag` structure is used to define a command entry with a specific flag, type, and additional flags. It contains a character for the flag, an enumeration to specify the type of command find operation, and an integer for any additional flags that may be relevant to the command's execution or behavior.


---
### cmd_entry
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the command name.
    - `alias`: A pointer to a string representing an alternative name for the command.
    - `args`: An `args_parse` structure that defines the arguments for the command.
    - `usage`: A pointer to a string that describes how to use the command.
    - `source`: A `cmd_entry_flag` structure representing the source of the command.
    - `target`: A `cmd_entry_flag` structure representing the target of the command.
    - `flags`: An integer representing various flags associated with the command.
    - `exec`: A function pointer to the command execution function.
- **Description**: The `cmd_entry` structure defines a command in the system, encapsulating its name, alias, argument parsing rules, usage information, source and target flags, associated flags, and a function pointer for executing the command. This structure is essential for managing commands within the application, allowing for flexible command definitions and execution.


---
### status_line_entry
- **Type**: `struct`
- **Members**:
    - `expanded`: A pointer to a character array representing the expanded status line entry.
    - `ranges`: A `style_ranges` structure that holds style information for different segments of the status line.
- **Description**: The `status_line_entry` structure is designed to represent an individual entry in a status line, containing an expanded string representation and associated style ranges that define how different parts of the string should be styled.


---
### status_line
- **Type**: `struct`
- **Members**:
    - `timer`: An `event` structure representing a timer for the status line.
    - `screen`: A `screen` structure representing the display area for the status line.
    - `active`: A pointer to the currently active `screen`.
    - `references`: An integer tracking the number of references to this status line.
    - `style`: A `grid_cell` structure defining the style attributes of the status line.
    - `entries`: An array of `status_line_entry` structures, limited to `STATUS_LINES_LIMIT`, representing individual entries in the status line.
- **Description**: The `status_line` structure is designed to manage the status line in a terminal multiplexer, encapsulating the necessary components such as a timer for updates, the screen context, a reference count for memory management, style attributes, and an array of entries that can be displayed in the status line. This structure allows for dynamic updates and rendering of status information, enhancing the user experience in terminal applications.


---
### prompt_type
- **Type**: `enum`
- **Members**:
    - `PROMPT_TYPE_COMMAND`: Represents a command prompt type.
    - `PROMPT_TYPE_SEARCH`: Represents a search prompt type.
    - `PROMPT_TYPE_TARGET`: Represents a target prompt type.
    - `PROMPT_TYPE_WINDOW_TARGET`: Represents a window target prompt type.
    - `PROMPT_TYPE_INVALID`: Indicates an invalid prompt type.
- **Description**: The `prompt_type` enum defines various types of prompts that can be used in a terminal multiplexer context, allowing the system to differentiate between command prompts, search prompts, target prompts, and window target prompts, with an additional value to signify an invalid prompt type.


---
### client_file
- **Type**: `struct`
- **Members**:
    - `c`: Pointer to a `client` structure representing the client associated with the file.
    - `peer`: Pointer to a `tmuxpeer` structure representing the peer connection.
    - `tree`: Pointer to a `client_files` structure representing a tree of client files.
    - `references`: Integer representing the number of references to this client file.
    - `stream`: Integer representing the stream identifier for the client file.
    - `path`: Pointer to a string representing the file path.
    - `buffer`: Pointer to an `evbuffer` structure for buffering data.
    - `event`: Pointer to a `bufferevent` structure for handling events.
    - `fd`: Integer representing the file descriptor associated with the client file.
    - `error`: Integer representing any error status related to the client file.
    - `closed`: Integer flag indicating whether the client file is closed.
    - `cb`: Callback function pointer of type `client_file_cb` for handling file events.
    - `data`: Pointer to user-defined data associated with the client file.
    - `entry`: Red-black tree entry for managing client files.
- **Description**: The `client_file` structure represents a file associated with a client in a terminal multiplexer environment. It contains various members that manage the client's connection, file path, event handling, and buffering. The structure is designed to facilitate communication between the client and the server, allowing for efficient data transfer and management of client-specific resources.


---
### client_window
- **Type**: `struct`
- **Members**:
    - `window`: An unsigned integer representing the window identifier.
    - `pane`: A pointer to a `window_pane` structure representing the associated pane.
    - `sx`: An unsigned integer representing the width of the client window.
    - `sy`: An unsigned integer representing the height of the client window.
    - `entry`: A red-black tree entry for managing `client_window` instances.
- **Description**: The `client_window` structure represents a window associated with a client in a terminal multiplexer environment. It contains fields for the window identifier, a pointer to the associated pane, dimensions of the window, and a red-black tree entry for efficient management within a collection of client windows.


---
### overlay_ranges
- **Type**: `struct`
- **Members**:
    - `px`: An array of unsigned integers representing positive overlay ranges.
    - `nx`: An array of unsigned integers representing negative overlay ranges.
- **Description**: The `overlay_ranges` structure is designed to hold two arrays of unsigned integers, `px` and `nx`, which represent positive and negative overlay ranges respectively, with a maximum size defined by `OVERLAY_MAX_RANGES`. This structure is useful in scenarios where overlays need to be managed, such as in terminal applications or graphical interfaces.


---
### client_theme
- **Type**: `enum`
- **Members**:
    - `THEME_UNKNOWN`: Represents an unknown theme.
    - `THEME_LIGHT`: Represents a light theme.
    - `THEME_DARK`: Represents a dark theme.
- **Description**: The `client_theme` enum defines three possible themes for a client interface: `THEME_UNKNOWN`, `THEME_LIGHT`, and `THEME_DARK`, allowing for easy identification and application of different visual styles based on user preferences or system settings.


---
### client
- **Type**: `struct`
- **Members**:
    - `name`: Pointer to the client's name.
    - `peer`: Pointer to the associated `tmuxpeer` structure.
    - `queue`: Pointer to the command queue list.
    - `windows`: Structure holding the client's windows.
    - `control_state`: Pointer to the control state structure.
    - `pause_age`: Unsigned integer representing the pause duration.
    - `pid`: Process ID of the client.
    - `fd`: File descriptor for the client.
    - `out_fd`: Output file descriptor for the client.
    - `event`: Event structure for handling client events.
    - `retval`: Return value from client operations.
    - `creation_time`: Timestamp of when the client was created.
    - `activity_time`: Timestamp of the last activity of the client.
    - `last_activity_time`: Timestamp of the last recorded activity.
    - `environ`: Pointer to the environment variables.
    - `jobs`: Pointer to the job tree associated with the client.
    - `title`: Pointer to the client's title.
    - `path`: Pointer to the current path of the client.
    - `cwd`: Pointer to the current working directory.
    - `term_name`: Pointer to the terminal name.
    - `term_features`: Integer representing terminal features.
    - `term_type`: Pointer to the terminal type.
    - `term_caps`: Array of terminal capabilities.
    - `term_ncaps`: Number of terminal capabilities.
    - `ttyname`: Pointer to the TTY name.
    - `tty`: Structure representing the TTY.
    - `written`: Size of data written by the client.
    - `discarded`: Size of data discarded by the client.
    - `redraw`: Size of data to be redrawn.
    - `repeat_timer`: Event structure for repeat actions.
    - `click_timer`: Event structure for click actions.
    - `click_button`: Button identifier for mouse clicks.
    - `click_event`: Structure for mouse click events.
    - `status`: Structure representing the status line.
    - `theme`: Enumeration representing the client theme.
    - `flags`: Bitmask of various client flags.
    - `exit_type`: Enumeration representing the exit type.
    - `exit_msgtype`: Message type for exit.
    - `exit_session`: Pointer to the exit session.
    - `exit_message`: Pointer to the exit message.
    - `keytable`: Pointer to the key table.
    - `last_key`: Last key pressed by the client.
    - `redraw_panes`: Bitmask for redrawing panes.
    - `redraw_scrollbars`: Bitmask for redrawing scrollbars.
    - `message_ignore_keys`: Flag to ignore certain keys in messages.
    - `message_ignore_styles`: Flag to ignore certain styles in messages.
    - `message_string`: Pointer to the message string.
    - `message_timer`: Event structure for message timing.
    - `prompt_string`: Pointer to the prompt string.
    - `prompt_formats`: Pointer to the prompt formats.
    - `prompt_buffer`: Pointer to the prompt buffer.
    - `prompt_last`: Pointer to the last prompt.
    - `prompt_index`: Index for the prompt.
    - `prompt_inputcb`: Callback for prompt input.
    - `prompt_freecb`: Callback for freeing prompt data.
    - `prompt_data`: Pointer to additional prompt data.
    - `prompt_hindex`: Array of prompt history indices.
    - `prompt_mode`: Enumeration representing the prompt mode.
    - `prompt_saved`: Pointer to saved prompt data.
    - `prompt_flags`: Flags for prompt behavior.
    - `prompt_type`: Enumeration representing the prompt type.
    - `prompt_cursor`: Cursor position in the prompt.
    - `session`: Pointer to the associated session.
    - `last_session`: Pointer to the last session.
    - `references`: Reference count for the client.
    - `pan_window`: Pointer to the pan window.
    - `pan_ox`: Offset in the x direction for the pan.
    - `pan_oy`: Offset in the y direction for the pan.
    - `overlay_check`: Callback for overlay checks.
    - `overlay_mode`: Callback for overlay mode.
    - `overlay_draw`: Callback for overlay drawing.
    - `overlay_key`: Callback for overlay key events.
    - `overlay_free`: Callback for freeing overlay data.
    - `overlay_resize`: Callback for resizing overlay.
    - `overlay_data`: Pointer to overlay data.
    - `overlay_timer`: Event structure for overlay timing.
    - `files`: Structure holding client files.
    - `source_file_depth`: Depth of source files.
    - `clipboard_panes`: Array of clipboard pane identifiers.
    - `clipboard_npanes`: Number of clipboard panes.
    - `entry`: Entry in the linked list of clients.
- **Description**: The `client` structure represents a client connection in a terminal multiplexer, encapsulating various attributes such as the client's name, associated peer, command queue, window management, terminal settings, and event handling. It maintains state information including process identifiers, file descriptors, timestamps for activity, and user-defined settings, allowing for comprehensive management of client interactions within the multiplexer environment.


---
### control_sub_type
- **Type**: `enum`
- **Members**:
    - `CONTROL_SUB_SESSION`: Represents a subscription to a specific session.
    - `CONTROL_SUB_PANE`: Represents a subscription to a specific pane.
    - `CONTROL_SUB_ALL_PANES`: Represents a subscription to all panes.
    - `CONTROL_SUB_WINDOW`: Represents a subscription to a specific window.
    - `CONTROL_SUB_ALL_WINDOWS`: Represents a subscription to all windows.
- **Description**: The `control_sub_type` enum defines various subscription types for a control mechanism, allowing clients to subscribe to specific sessions, panes, or windows in a terminal multiplexer environment. Each enumerator represents a distinct subscription target, facilitating organized control over the terminal's state and interactions.


---
### key_binding
- **Type**: `struct`
- **Members**:
    - `key`: A `key_code` representing the key being bound.
    - `cmdlist`: A pointer to a `cmd_list` structure containing commands associated with the key.
    - `note`: A string providing additional information or notes about the key binding.
    - `flags`: An integer representing various flags for the key binding, including repeat functionality.
    - `entry`: An entry for a red-black tree to manage key bindings.
- **Description**: The `key_binding` structure represents a mapping between a specific key and a command or set of commands in a terminal multiplexer environment. It includes a `key_code` to identify the key, a pointer to a `cmd_list` that holds the commands to be executed when the key is pressed, and a `note` for any additional context. The `flags` field allows for configuration options such as enabling repeat functionality, while the `entry` field integrates the structure into a red-black tree for efficient management of key bindings.


---
### key_table
- **Type**: `struct key_table`
- **Members**:
    - `name`: A pointer to a constant character string representing the name of the key table.
    - `activity_time`: A `struct timeval` representing the last activity time of the key table.
    - `key_bindings`: A `struct key_bindings` that holds the current key bindings for this key table.
    - `default_key_bindings`: A `struct key_bindings` that holds the default key bindings.
    - `references`: An unsigned integer that counts the number of references to this key table.
    - `entry`: An entry point for a red-black tree that organizes key tables.
- **Description**: The `key_table` structure is designed to manage key bindings in a terminal multiplexer environment. It contains fields for the name of the key table, timestamps for activity tracking, and structures to hold both current and default key bindings. Additionally, it includes a reference count to manage memory and a red-black tree entry for efficient organization and retrieval of key tables.


---
### options_value
- **Type**: `union`
- **Members**:
    - `string`: A pointer to a character string.
    - `number`: A 64-bit integer to store numeric values.
    - `style`: A `struct style` to define styling options.
    - `array`: A `struct options_array` to hold an array of options.
    - `cmdlist`: A pointer to a `struct cmd_list` for command options.
- **Description**: The `options_value` union is a flexible data structure that can hold different types of values, including a string, a numeric value, a style definition, an array of options, or a command list. This allows for versatile handling of various option types in a configuration context, enabling the storage and retrieval of different data types as needed.


---
### options_table_type
- **Type**: `enum`
- **Members**:
    - `OPTIONS_TABLE_STRING`: Represents a string option type.
    - `OPTIONS_TABLE_NUMBER`: Represents a numeric option type.
    - `OPTIONS_TABLE_KEY`: Represents a key option type.
    - `OPTIONS_TABLE_COLOUR`: Represents a color option type.
    - `OPTIONS_TABLE_FLAG`: Represents a boolean flag option type.
    - `OPTIONS_TABLE_CHOICE`: Represents a choice option type.
    - `OPTIONS_TABLE_COMMAND`: Represents a command option type.
- **Description**: The `options_table_type` enum defines various types of options that can be used in a configuration system, allowing for different data types such as strings, numbers, keys, colors, flags, choices, and commands to be represented and processed accordingly.


---
### options_table_entry
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the name of the option.
    - `alternative_name`: A pointer to a string representing an alternative name for the option.
    - `type`: An enumeration value indicating the type of the option.
    - `scope`: An integer representing the scope of the option (e.g., session, window, etc.).
    - `flags`: An integer representing various flags associated with the option.
    - `minimum`: An unsigned integer representing the minimum value for numeric options.
    - `maximum`: An unsigned integer representing the maximum value for numeric options.
    - `choices`: A pointer to an array of strings representing possible choices for the option.
    - `default_str`: A pointer to a string representing the default value as a string.
    - `default_num`: A long long integer representing the default numeric value.
    - `default_arr`: A pointer to an array of strings representing the default array value.
    - `separator`: A pointer to a string representing the separator for array values.
    - `pattern`: A pointer to a string representing a pattern for validating the option.
    - `text`: A pointer to a string providing a description or help text for the option.
    - `unit`: A pointer to a string representing the unit of measurement for the option.
- **Description**: The `options_table_entry` structure defines a single entry in an options table, which includes various attributes such as the option's name, type, scope, and default values. It is designed to facilitate the management of configuration options in a software application, allowing for a flexible and extensible way to define and validate user-configurable settings.


---
### options_name_map
- **Type**: `struct`
- **Members**:
    - `from`: A pointer to a string representing the original option name.
    - `to`: A pointer to a string representing the mapped option name.
- **Description**: The `options_name_map` structure is used to define a mapping between two option names, where `from` holds the original name and `to` holds the corresponding mapped name. This structure facilitates the translation or aliasing of option names in a configuration context.


---
### spawn_context
- **Type**: `struct`
- **Members**:
    - `item`: Pointer to a `cmdq_item` structure representing a command queue item.
    - `s`: Pointer to a `session` structure representing the current session.
    - `wl`: Pointer to a `winlink` structure representing the current window link.
    - `tc`: Pointer to a `client` structure representing the target client.
    - `wp0`: Pointer to a `window_pane` structure representing the first window pane.
    - `lc`: Pointer to a `layout_cell` structure representing the layout cell.
    - `name`: Pointer to a constant character string representing the name of the spawn context.
    - `argv`: Pointer to an array of character strings representing the arguments for the command.
    - `argc`: Integer representing the count of arguments.
    - `environ`: Pointer to an `environ` structure representing the environment variables.
    - `idx`: Integer representing the index of the spawn context.
    - `cwd`: Pointer to a constant character string representing the current working directory.
    - `flags`: Integer representing various flags for the spawn context.
- **Description**: The `spawn_context` structure is used to encapsulate the context for spawning new processes in a terminal multiplexer environment. It contains references to various components such as command queue items, sessions, window links, clients, and layout cells, along with command-line arguments and environment variables. The structure also includes flags to control the behavior of the spawn operation, such as whether to kill existing processes, detach from the terminal, or respawn processes.


---
### mode_tree_sort_criteria
- **Type**: `struct`
- **Members**:
    - `field`: An unsigned integer representing the sorting field.
    - `reversed`: An integer indicating whether the sort order is reversed.
- **Description**: The `mode_tree_sort_criteria` structure is used to define the criteria for sorting in a mode tree, consisting of a field identifier and a flag to indicate if the sorting should be in reverse order.


