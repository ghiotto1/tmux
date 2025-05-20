# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for job scheduling and management. The code defines a system for running commands in the background, capturing their output, and managing their lifecycle. The primary structure defined in this file is the `job` structure, which encapsulates the state and properties of a background job, including its command, process ID, file descriptor, and various callback functions for handling updates, completion, and cleanup.

The file includes functions for starting a job ([`job_run`](#job_run)), transferring a job's file descriptor ([`job_transfer`](#job_transfer)), freeing a job ([`job_free`](#job_free)), resizing a job's terminal window ([`job_resize`](#job_resize)), and handling job events such as reading, writing, and errors through callback functions. It also provides utility functions to check if jobs are still running, kill all jobs, and print a summary of all jobs. The code is designed to be integrated into the larger tmux application, leveraging system calls and libraries to manage process execution and inter-process communication. The use of callback functions allows for flexible handling of job events, making this code a crucial component for managing asynchronous tasks within tmux.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/ioctl.h`
- `sys/socket.h`
- `sys/wait.h`
- `fcntl.h`
- `signal.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### all_jobs
- **Type**: `LIST_HEAD(joblist, job)`
- **Description**: The `all_jobs` variable is a static global variable that represents a linked list head for managing a list of `job` structures. It is initialized using the `LIST_HEAD_INITIALIZER` macro, which sets up the list to be empty initially.
- **Use**: This variable is used to maintain and manage all the jobs that are currently being tracked by the system, allowing for operations such as insertion, removal, and iteration over the list of jobs.


# Data Structures

---
### job
- **Type**: `struct`
- **Members**:
    - `state`: An enumeration representing the current state of the job, which can be JOB_RUNNING, JOB_DEAD, or JOB_CLOSED.
    - `flags`: An integer representing various flags associated with the job.
    - `cmd`: A pointer to a character array (string) representing the command to be executed by the job.
    - `pid`: A process ID (pid_t) representing the process ID of the job.
    - `tty`: A character array representing the terminal associated with the job, with a maximum size defined by TTY_NAME_MAX.
    - `status`: An integer representing the status of the job, typically used to store the exit status.
    - `fd`: An integer representing the file descriptor associated with the job.
    - `event`: A pointer to a bufferevent structure used for handling buffered I/O events for the job.
    - `updatecb`: A callback function pointer for updating the job.
    - `completecb`: A callback function pointer for when the job is complete.
    - `freecb`: A callback function pointer for freeing resources associated with the job.
    - `data`: A void pointer to user-defined data associated with the job.
    - `entry`: A LIST_ENTRY structure used to link this job into a list of jobs.
- **Description**: The 'job' structure is a comprehensive data structure used to manage and track the execution of background jobs in a system. It includes fields for maintaining the job's state, command, process ID, terminal, status, and file descriptor. Additionally, it supports event-driven I/O through a bufferevent pointer and allows for user-defined callbacks for job updates, completion, and resource cleanup. The structure is designed to be part of a linked list, enabling the management of multiple jobs concurrently.


# Functions

---
### job_run<!-- {{#callable:job_run}} -->
The `job_run` function initiates a background job by forking a process to execute a command or a set of arguments, managing its environment, and setting up necessary callbacks and file descriptors.
- **Inputs**:
    - `cmd`: A string representing the command to be executed; if NULL, the command is constructed from argc and argv.
    - `argc`: An integer representing the number of arguments in argv.
    - `argv`: An array of strings representing the arguments for the command.
    - `e`: A pointer to an `environ` structure representing the environment variables for the job.
    - `s`: A pointer to a `session` structure, which may provide session-specific options.
    - `cwd`: A string representing the current working directory for the job; if NULL, defaults to the home directory or root.
    - `updatecb`: A callback function to be called when the job's output is updated.
    - `completecb`: A callback function to be called when the job completes.
    - `freecb`: A callback function to be called when the job is freed.
    - `data`: A pointer to user-defined data to be passed to the callbacks.
    - `flags`: An integer representing various flags that control job behavior, such as whether to use a PTY.
    - `sx`: An integer representing the width of the terminal if a PTY is used.
    - `sy`: An integer representing the height of the terminal if a PTY is used.
- **Control Flow**:
    - Initialize the environment for the job using the provided session and environment variables.
    - Determine the shell to use based on the flags and session options.
    - Block all signals to prevent interference during job setup.
    - If the JOB_PTY flag is set, create a pseudo-terminal with the specified dimensions; otherwise, create a socket pair for communication.
    - Fork the process to create a child process for executing the command or arguments.
    - In the child process, set up the environment, change the working directory, and redirect standard input/output/error as needed.
    - Execute the command using the specified shell or directly using execvp if no command is provided.
    - In the parent process, restore the signal mask, free the environment, and allocate memory for the job structure.
    - Initialize the job structure with the command, process ID, and file descriptors, and insert it into the global job list.
    - Set up a bufferevent for asynchronous I/O and enable it for reading and writing.
    - Return the job structure pointer if successful, or NULL if an error occurs.
- **Output**:
    - Returns a pointer to a `job` structure representing the running job, or NULL if the job could not be started.
- **Functions called**:
    - [`environ_for_session`](environ.c.driver.md#environ_for_session)
    - [`environ_copy`](environ.c.driver.md#environ_copy)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`checkshell`](tmux.c.driver.md#checkshell)
    - [`shell_argv0`](tmux.c.driver.md#shell_argv0)
    - [`fdforkpty`](compat/fdforkpty.c.driver.md#fdforkpty)
    - [`proc_clear_signals`](proc.c.driver.md#proc_clear_signals)
    - [`find_home`](tmux.c.driver.md#find_home)
    - [`environ_push`](environ.c.driver.md#environ_push)
    - [`environ_free`](environ.c.driver.md#environ_free)
    - [`setenv`](compat/setenv.c.driver.md#setenv)
    - [`cmd_copy_argv`](cmd.c.driver.md#cmd_copy_argv)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`cmd_stringify_argv`](cmd.c.driver.md#cmd_stringify_argv)
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)
    - [`setblocking`](tmux.c.driver.md#setblocking)


---
### job_transfer<!-- {{#callable:job_transfer}} -->
The `job_transfer` function transfers ownership of a job's file descriptor and cleans up the job's resources.
- **Inputs**:
    - `job`: A pointer to the `job` structure representing the job to be transferred.
    - `pid`: A pointer to a `pid_t` variable where the job's process ID will be stored, if not NULL.
    - `tty`: A pointer to a character array where the job's TTY name will be copied, if not NULL.
    - `ttylen`: The size of the `tty` buffer to ensure safe copying of the TTY name.
- **Control Flow**:
    - Retrieve the file descriptor from the job structure and store it in a local variable `fd`.
    - Log the transfer operation with the job's command.
    - If `pid` is not NULL, store the job's process ID in the provided `pid` variable.
    - If `tty` is not NULL, copy the job's TTY name into the provided `tty` buffer using [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy) to ensure it does not exceed `ttylen`.
    - Remove the job from the list of jobs using `LIST_REMOVE`.
    - Free the memory allocated for the job's command string.
    - If a free callback (`freecb`) is defined and job data is present, call the free callback with the job's data.
    - If the job has an associated event, free the event using `bufferevent_free`.
    - Free the memory allocated for the job structure itself.
    - Return the file descriptor `fd` of the job.
- **Output**:
    - The function returns the file descriptor of the job that was transferred.
- **Functions called**:
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)


---
### job_free<!-- {{#callable:job_free}} -->
The `job_free` function is responsible for cleaning up and freeing resources associated with a job structure.
- **Inputs**:
    - `job`: A pointer to a `struct job` that represents the job to be freed.
- **Control Flow**:
    - Logs a debug message indicating the job being freed.
    - Removes the job from the list of all jobs using `LIST_REMOVE`.
    - Frees the memory allocated for the job's command string using `free(job->cmd)`.
    - Checks if a custom free callback (`freecb`) is set and if job data is present, then calls the callback with the job data.
    - If the job has a valid process ID (`pid`), sends a SIGTERM signal to terminate the process using `kill`.
    - If the job has an associated event (`event`), frees the event using `bufferevent_free`.
    - If the job has a valid file descriptor (`fd`), closes it using `close`.
    - Finally, frees the memory allocated for the job structure itself using `free(job)`.
- **Output**:
    - The function does not return any value; it performs cleanup operations and frees memory associated with the job.


---
### job_resize<!-- {{#callable:job_resize}} -->
The `job_resize` function adjusts the terminal window size for a job with a pseudo-terminal (PTY).
- **Inputs**:
    - `job`: A pointer to a `struct job` representing the job whose terminal size is to be adjusted.
    - `sx`: An unsigned integer representing the new number of columns for the terminal window.
    - `sy`: An unsigned integer representing the new number of rows for the terminal window.
- **Control Flow**:
    - Check if the job's file descriptor is valid and if the job has a PTY; if not, return immediately.
    - Log the resize operation with the job pointer and new dimensions.
    - Initialize a `struct winsize` to zero and set its `ws_col` and `ws_row` fields to `sx` and `sy`, respectively.
    - Use the `ioctl` system call to apply the new window size to the job's file descriptor; if it fails, call `fatal` to handle the error.
- **Output**:
    - The function does not return a value; it performs an operation to resize the terminal window associated with a job.


---
### job_read_callback<!-- {{#callable:job_read_callback}} -->
The `job_read_callback` function is a callback that triggers the update callback function for a job when data is read from a buffer event.
- **Inputs**:
    - `bufev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `data`: A pointer to a `job` structure, representing the job associated with the buffer event.
- **Control Flow**:
    - The function checks if the `updatecb` callback function of the `job` structure is not NULL.
    - If `updatecb` is not NULL, it calls the `updatecb` function, passing the `job` structure as an argument.
- **Output**:
    - The function does not return any value.


---
### job_write_callback<!-- {{#callable:job_write_callback}} -->
The `job_write_callback` function handles the write event for a job's buffer, logging the job's status and disabling the write event if the buffer is empty and the job is not flagged to keep writing.
- **Inputs**:
    - `bufev`: A pointer to the bufferevent structure associated with the job, marked as unused in this function.
    - `data`: A pointer to the job structure, which contains information about the job being processed.
- **Control Flow**:
    - Retrieve the job structure from the provided data pointer.
    - Calculate the length of the output buffer associated with the job's event.
    - Log the current state of the job, including its command, process ID, and remaining output buffer length.
    - Check if the output buffer length is zero and the job is not flagged with JOB_KEEPWRITE.
    - If the conditions are met, shut down the write side of the job's file descriptor and disable the write event on the bufferevent.
- **Output**:
    - The function does not return a value; it performs actions based on the state of the job's output buffer and modifies the job's bufferevent accordingly.


---
### job_error_callback<!-- {{#callable:job_error_callback}} -->
The `job_error_callback` function handles errors for a job by logging the error, checking the job's state, and either completing and freeing the job or disabling its read event and marking it as closed.
- **Inputs**:
    - `bufev`: A pointer to the bufferevent structure associated with the job, marked as unused in this function.
    - `events`: A short integer representing the events that triggered the callback, marked as unused in this function.
    - `data`: A pointer to the job structure that contains information about the job being processed.
- **Control Flow**:
    - The function begins by casting the `data` pointer to a `job` structure pointer.
    - It logs a debug message indicating an error with the job, including the job's command and process ID.
    - The function checks if the job's state is `JOB_DEAD`.
    - If the job is dead and has a `completecb` callback, it calls this callback and then frees the job using [`job_free`](#job_free).
    - If the job is not dead, it disables the read event on the job's bufferevent and sets the job's state to `JOB_CLOSED`.
- **Output**:
    - The function does not return any value; it performs actions based on the job's state and modifies the job's state and event handling as necessary.
- **Functions called**:
    - [`job_free`](#job_free)


---
### job_check_died<!-- {{#callable:job_check_died}} -->
The `job_check_died` function checks if a job with a given process ID has died and updates its status accordingly.
- **Inputs**:
    - `pid`: The process ID of the job to check.
    - `status`: The status code returned by the job, typically from a `waitpid` call.
- **Control Flow**:
    - Iterate over the list of all jobs to find the job with the matching process ID (`pid`).
    - If no matching job is found, return immediately.
    - Check if the job was stopped using `WIFSTOPPED(status)`.
    - If the job was stopped due to `SIGTTIN` or `SIGTTOU`, return without further action.
    - If the job was stopped for other reasons, send a `SIGCONT` signal to continue the job and return.
    - Log the job's death with its command and process ID.
    - Update the job's status with the provided `status`.
    - If the job's state is `JOB_CLOSED`, call its completion callback if it exists, then free the job.
    - If the job's state is not `JOB_CLOSED`, set its process ID to -1 and change its state to `JOB_DEAD`.
- **Output**:
    - The function does not return a value; it updates the job's status and potentially frees the job or changes its state.
- **Functions called**:
    - [`job_free`](#job_free)


---
### job_get_status<!-- {{#callable:job_get_status}} -->
The `job_get_status` function retrieves the status of a given job.
- **Inputs**:
    - `job`: A pointer to a `struct job` from which the status is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `status` field of the `job` structure and returns its value.
- **Output**:
    - The function returns an integer representing the status of the job.


---
### job_get_data<!-- {{#callable:job_get_data}} -->
The `job_get_data` function retrieves the data associated with a given job structure.
- **Inputs**:
    - `job`: A pointer to a `struct job` from which the data is to be retrieved.
- **Control Flow**:
    - The function directly returns the `data` member of the `job` structure.
- **Output**:
    - A void pointer to the data associated with the specified job.


---
### job_get_event<!-- {{#callable:job_get_event}} -->
The `job_get_event` function retrieves the `bufferevent` associated with a given job.
- **Inputs**:
    - `job`: A pointer to a `struct job` from which the `bufferevent` is to be retrieved.
- **Control Flow**:
    - The function directly returns the `event` member of the `job` structure.
- **Output**:
    - A pointer to a `struct bufferevent` that is associated with the given job.


---
### job_kill_all<!-- {{#callable:job_kill_all}} -->
The `job_kill_all` function iterates over all jobs in the global job list and sends a SIGTERM signal to terminate each job that is currently running.
- **Inputs**:
    - None
- **Control Flow**:
    - The function iterates over each job in the `all_jobs` list using the `LIST_FOREACH` macro.
    - For each job, it checks if the job's `pid` is not equal to -1, indicating that the job is running.
    - If the job is running, it sends a SIGTERM signal to the job's process using the `kill` function.
- **Output**:
    - The function does not return any value; it performs an action of sending termination signals to all running jobs.


---
### job_still_running<!-- {{#callable:job_still_running}} -->
The `job_still_running` function checks if there are any jobs in the global job list that are currently running and not marked with the `JOB_NOWAIT` flag.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each job in the global `all_jobs` list using the `LIST_FOREACH` macro.
    - For each job, check if the job's flags do not include `JOB_NOWAIT` and if the job's state is `JOB_RUNNING`.
    - If a job meets these conditions, return 1 to indicate that there is at least one running job.
    - If no jobs meet the conditions, return 0 after the loop completes.
- **Output**:
    - The function returns an integer: 1 if there is at least one job running that is not marked with `JOB_NOWAIT`, otherwise 0.


---
### job_print_summary<!-- {{#callable:job_print_summary}} -->
The `job_print_summary` function prints a summary of all jobs in a list, optionally starting with a blank line.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is used to print the job summary.
    - `blank`: An integer flag indicating whether to print a blank line before the job summary (non-zero value) or not (zero value).
- **Control Flow**:
    - Initialize a job pointer and a counter `n` to zero.
    - Iterate over each job in the `all_jobs` list using `LIST_FOREACH`.
    - If `blank` is non-zero, print a blank line using `cmdq_print` and set `blank` to zero.
    - For each job, print its details including the job number, command, file descriptor, process ID, and status using `cmdq_print`.
    - Increment the job counter `n` after printing each job's details.
- **Output**:
    - The function does not return a value; it outputs the job summary directly via the `cmdq_print` function.


