# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides operating system-dependent functionality for process management and information retrieval on OpenBSD systems. The file includes functions that interact with the system's process control and file system to obtain specific details about processes and their execution environment. The primary functions defined in this file are [`osdep_get_name`](#osdep_get_name), [`osdep_get_cwd`](#osdep_get_cwd), and [`osdep_event_init`](#osdep_event_init). These functions are designed to retrieve the name of a process associated with a terminal, the current working directory of a process group, and to initialize an event handling system, respectively.

The code leverages system calls and structures such as `sysctl`, `stat`, and `tcgetpgrp` to gather process-related information. The [`cmp_procs`](#cmp_procs) function is a utility that compares two processes based on their state, CPU usage, sleep time, and process ID to determine which process is more significant or active. This file is intended to be part of a larger codebase, likely included as a module that provides platform-specific implementations for tmux's functionality. It does not define public APIs or external interfaces directly but rather serves as an internal component that facilitates interaction with the operating system's process management features.
# Imports and Dependencies

---
- `sys/param.h`
- `sys/proc.h`
- `sys/stat.h`
- `sys/sysctl.h`
- `errno.h`
- `limits.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### cmp_procs
- **Type**: `function pointer`
- **Description**: `cmp_procs` is a function that takes two pointers to `struct kinfo_proc2` and returns a pointer to the `struct kinfo_proc2` that is considered 'better' based on certain criteria. The criteria include whether the process is runnable, stopped, its estimated CPU usage, sleep time, and process ID.
- **Use**: This function is used to compare two process structures and determine which one should be prioritized or selected based on predefined criteria.


# Functions

---
### cmp_procs<!-- {{#callable:cmp_procs}} -->
The `cmp_procs` function compares two process structures and returns the one that is more favorable based on a series of criteria.
- **Inputs**:
    - `p1`: A pointer to the first `kinfo_proc2` structure representing a process.
    - `p2`: A pointer to the second `kinfo_proc2` structure representing a process.
- **Control Flow**:
    - Check if `p1` is runnable and `p2` is not; if true, return `p1`.
    - Check if `p2` is runnable and `p1` is not; if true, return `p2`.
    - Check if `p1` is stopped and `p2` is not; if true, return `p1`.
    - Check if `p2` is stopped and `p1` is not; if true, return `p2`.
    - Compare the estimated CPU usage (`p_estcpu`) of `p1` and `p2`; return the process with higher usage.
    - Compare the sleep time (`p_slptime`) of `p1` and `p2`; return the process with less sleep time.
    - Compare the process IDs (`p_pid`) of `p1` and `p2`; return the process with the higher ID.
- **Output**:
    - Returns a pointer to the `kinfo_proc2` structure that is considered more favorable based on the comparison criteria.


---
### osdep_get_name<!-- {{#callable:osdep_get_name}} -->
The `osdep_get_name` function retrieves the name of the process group leader associated with a given file descriptor and terminal device.
- **Inputs**:
    - `fd`: An integer representing the file descriptor for which the process group leader's name is to be retrieved.
    - `tty`: A character pointer to the terminal device name associated with the file descriptor, though it is marked as unused in the function signature.
- **Control Flow**:
    - The function first checks if the terminal device specified by `tty` exists using `stat`; if not, it returns NULL.
    - It retrieves the process group ID associated with the file descriptor using `tcgetpgrp`; if this fails, it returns NULL.
    - The function initializes a `mib` array to specify the sysctl query for process information related to the process group.
    - It attempts to determine the size of the buffer needed to store process information using `sysctl`; if this fails, it returns NULL.
    - The function allocates memory for the buffer to store process information and retries the `sysctl` call to fill the buffer; if memory allocation fails, it goes to the error handling section.
    - It iterates over the process information buffer to find the process whose terminal device matches the one specified by `tty`.
    - The function uses [`cmp_procs`](#cmp_procs) to determine the best process if multiple processes are found with the same terminal device.
    - If a suitable process is found, it duplicates the process's command name (`p_comm`) into a new string.
    - The function frees the allocated buffer and returns the duplicated process name, or NULL if no suitable process was found.
- **Output**:
    - The function returns a dynamically allocated string containing the name of the process group leader's command, or NULL if an error occurs or no suitable process is found.
- **Functions called**:
    - [`cmp_procs`](#cmp_procs)


---
### osdep_get_cwd<!-- {{#callable:osdep_get_cwd}} -->
The `osdep_get_cwd` function retrieves the current working directory of the process group associated with a given file descriptor.
- **Inputs**:
    - `fd`: An integer representing the file descriptor for which the current working directory of the associated process group is to be retrieved.
- **Control Flow**:
    - The function begins by declaring a static character array `target` to store the resulting path and a variable `pgrp` to hold the process group ID.
    - It attempts to get the process group ID associated with the file descriptor `fd` using `tcgetpgrp`. If this fails, the function returns `NULL`.
    - If the macro `KERN_PROC_CWD` is defined, it sets up a `mib` array for a `sysctl` call to retrieve the current working directory of the process group. If successful, it returns the `target` array containing the path.
    - If `KERN_PROC_CWD` is not defined, it constructs a path to the `/proc` filesystem to read the symbolic link for the current working directory of the process group. It uses `readlink` to read the link into `target`. If successful, it null-terminates the string and returns `target`.
    - If neither method succeeds, the function returns `NULL`.
- **Output**:
    - The function returns a pointer to a static character array containing the current working directory path of the process group associated with the given file descriptor, or `NULL` if it fails to retrieve the path.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### osdep_event_init<!-- {{#callable:osdep_event_init}} -->
The function `osdep_event_init` initializes and returns a new event base using the `event_init` function.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `event_init()` to initialize a new event base.
    - The result of `event_init()` is returned directly.
- **Output**:
    - The function returns a pointer to a newly initialized `event_base` structure.


