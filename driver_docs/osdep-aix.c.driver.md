# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and provides operating system-dependent functionality specific to process and terminal management on systems that support the `/proc` filesystem, such as OpenBSD. The file defines three functions: [`osdep_get_name`](#osdep_get_name), [`osdep_get_cwd`](#osdep_get_cwd), and [`osdep_event_init`](#osdep_event_init). These functions are designed to interact with system-level process information and terminal control, offering a narrow but essential set of utilities for tmux's operation.

The [`osdep_get_name`](#osdep_get_name) function retrieves the name of the process group leader associated with a given terminal, using the `/proc` filesystem to access process information. The [`osdep_get_cwd`](#osdep_get_cwd) function determines the current working directory of a process group leader, again leveraging the `/proc` filesystem to read symbolic links. The [`osdep_event_init`](#osdep_event_init) function initializes an event base, likely for handling asynchronous events within tmux. This file is intended to be part of a larger codebase, providing specific implementations of functions that are likely abstracted in a broader API, allowing tmux to operate across different Unix-like systems by using system-specific code where necessary.
# Imports and Dependencies

---
- `sys/param.h`
- `sys/procfs.h`
- `stdlib.h`
- `string.h`
- `fcntl.h`
- `tmux.h`


# Functions

---
### osdep_get_name<!-- {{#callable:osdep_get_name}} -->
The `osdep_get_name` function retrieves the name of the process associated with a given terminal device.
- **Inputs**:
    - `fd`: An unused integer file descriptor.
    - `tty`: A string representing the path to the terminal device.
- **Control Flow**:
    - Open the terminal device specified by `tty` in read-only mode without making it the controlling terminal.
    - If opening the terminal fails, return `NULL`.
    - Use `ioctl` to get the process group ID (pgrp) of the terminal device.
    - Close the terminal device file descriptor.
    - If `ioctl` fails, return `NULL`.
    - Format a string to create the path to the process information file in `/proc` using the process group ID.
    - Open the process information file in read-only mode.
    - Free the memory allocated for the path string.
    - If opening the process information file fails, return `NULL`.
    - Read the process information into a `psinfo` structure.
    - Close the process information file descriptor.
    - If the read operation does not return the expected number of bytes, return `NULL`.
    - Return a duplicate of the process name (`pr_fname`) from the `psinfo` structure.
- **Output**:
    - A dynamically allocated string containing the process name, or `NULL` if any step fails.


---
### osdep_get_cwd<!-- {{#callable:osdep_get_cwd}} -->
The `osdep_get_cwd` function retrieves the current working directory of the process group associated with a given file descriptor.
- **Inputs**:
    - `fd`: An integer representing a file descriptor, typically associated with a terminal.
- **Control Flow**:
    - Retrieve the terminal path associated with the file descriptor using `ptsname`.
    - Open the terminal path in read-only mode without making it the controlling terminal using `open`.
    - Use `ioctl` to get the process group ID associated with the terminal file descriptor.
    - Close the terminal file descriptor.
    - If any of the above steps fail, return `NULL`.
    - Construct the path to the current working directory of the process group using the process group ID.
    - Read the symbolic link at the constructed path to get the current working directory into the `target` buffer.
    - If the read is successful, null-terminate the string and remove any trailing slash if present.
    - Return the `target` buffer containing the current working directory path.
    - If the read fails, return `NULL`.
- **Output**:
    - A pointer to a static buffer containing the current working directory path of the process group, or `NULL` if an error occurs.


---
### osdep_event_init<!-- {{#callable:osdep_event_init}} -->
The `osdep_event_init` function initializes and returns a new event base by calling the `event_init` function.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls the `event_init` function.
    - The result of `event_init` is returned directly.
- **Output**:
    - A pointer to a newly initialized `event_base` structure is returned.


