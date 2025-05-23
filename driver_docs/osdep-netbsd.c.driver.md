# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and provides operating system-dependent functionality for process management and information retrieval. The file includes functions that interact with the system's process control and file system to obtain specific details about processes and their execution environment. The primary functions include [`osdep_get_name`](#osdep_get_name), which retrieves the name of the process associated with a given file descriptor and terminal, and [`osdep_get_cwd`](#osdep_get_cwd), which determines the current working directory of a process group. These functions utilize system calls like `sysctl` and `readlink` to gather information, demonstrating a focus on process and environment introspection.

The file also defines a utility function, [`cmp_procs`](#cmp_procs), which compares two process structures (`kinfo_proc2`) based on their state, CPU usage, sleep time, and process ID, to determine which process is more "active" or relevant. This function is used internally to select the most appropriate process when multiple candidates are found. Additionally, the file includes [`osdep_event_init`](#osdep_event_init), which initializes an event base, likely for handling asynchronous events within the tmux application. Overall, this file provides a narrow set of functionalities focused on process management and environment querying, tailored to the needs of the tmux application on systems that support these specific system calls and structures.
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
    - `p1`: A pointer to the first process structure of type `struct kinfo_proc2` to be compared.
    - `p2`: A pointer to the second process structure of type `struct kinfo_proc2` to be compared.
- **Control Flow**:
    - Check if the first process is runnable and the second is not; if true, return the first process.
    - Check if the second process is runnable and the first is not; if true, return the second process.
    - Check if the first process is stopped and the second is not; if true, return the first process.
    - Check if the second process is stopped and the first is not; if true, return the second process.
    - Compare the estimated CPU usage (`p_estcpu`) of both processes; return the process with higher usage.
    - Compare the sleep time (`p_slptime`) of both processes; return the process with less sleep time.
    - Compare the process IDs (`p_pid`) of both processes; return the process with the higher ID.
- **Output**:
    - The function returns a pointer to the `struct kinfo_proc2` that is considered more favorable based on the comparison criteria.


---
### osdep_get_name<!-- {{#callable:osdep_get_name}} -->
The `osdep_get_name` function retrieves the name of the process group leader associated with a given file descriptor and terminal device.
- **Inputs**:
    - `fd`: An integer representing the file descriptor for which the process group leader's name is to be retrieved.
    - `tty`: A character pointer to the terminal device name, which is unused in the function but required for the `stat` call to obtain device information.
- **Control Flow**:
    - The function begins by using `stat` to obtain information about the terminal device specified by `tty`; if this fails, it returns `NULL`.
    - It then attempts to get the process group ID associated with the file descriptor `fd` using `tcgetpgrp`; if this fails, it returns `NULL`.
    - The function sets up a `mib` array to specify the sysctl query for process information related to the process group.
    - A loop is initiated to handle potential `ENOMEM` errors from the `sysctl` call, reallocating the buffer as needed until successful or another error occurs.
    - The function iterates over the process information buffer to find processes associated with the terminal device, using [`cmp_procs`](#cmp_procs) to determine the best candidate based on process state and other criteria.
    - If a suitable process is found, its command name is duplicated into a new string, which is returned; otherwise, `NULL` is returned.
- **Output**:
    - The function returns a dynamically allocated string containing the name of the process group leader's command, or `NULL` if an error occurs or no suitable process is found.
- **Functions called**:
    - [`cmp_procs`](#cmp_procs)


---
### osdep_get_cwd<!-- {{#callable:osdep_get_cwd}} -->
The `osdep_get_cwd` function retrieves the current working directory of the process group associated with a given file descriptor.
- **Inputs**:
    - `fd`: An integer representing the file descriptor for which the current working directory of the associated process group is to be retrieved.
- **Control Flow**:
    - The function begins by attempting to get the process group ID associated with the file descriptor using `tcgetpgrp(fd)`.
    - If `tcgetpgrp(fd)` returns -1, indicating an error, the function returns `NULL`.
    - If the `KERN_PROC_CWD` macro is defined, the function sets up a `sysctl` call to retrieve the current working directory using the process group ID and stores it in the `target` buffer.
    - If the `sysctl` call is successful, the function returns the `target` buffer containing the current working directory.
    - If the `KERN_PROC_CWD` macro is not defined, the function constructs a path to the `/proc` filesystem to read the symbolic link for the current working directory of the process group.
    - The function uses `readlink` to read the symbolic link into the `target` buffer and null-terminates the string.
    - If `readlink` is successful, the function returns the `target` buffer containing the current working directory.
    - If neither method is successful, the function returns `NULL`.
- **Output**:
    - The function returns a pointer to a static buffer containing the current working directory of the process group associated with the given file descriptor, or `NULL` if an error occurs.


---
### osdep_event_init<!-- {{#callable:osdep_event_init}} -->
The function `osdep_event_init` initializes and returns a new event base by calling the `event_init` function.
- **Inputs**:
    - None
- **Control Flow**:
    - The function directly calls the `event_init` function.
    - The result of `event_init` is returned as the output of `osdep_event_init`.
- **Output**:
    - The function returns a pointer to a newly initialized `event_base` structure, which is used for managing events.


