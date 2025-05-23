# Purpose
This C source code file is part of the tmux project and is responsible for managing job scheduling and execution within the tmux environment. The code provides functionality to run commands in the background, manage their execution state, and handle their input/output streams. It defines a `job` structure that encapsulates the details of a job, such as its command, process ID, file descriptor, and state. The file includes functions to start a job ([`job_run`](#job_run)), transfer a job's file descriptor ([`job_transfer`](#job_transfer)), free a job ([`job_free`](#job_free)), resize a job's terminal window ([`job_resize`](#job_resize)), and handle job events through callbacks ([`job_read_callback`](#job_read_callback), [`job_write_callback`](#job_write_callback), [`job_error_callback`](#job_error_callback)). Additionally, it provides utility functions to check if jobs are still running, kill all jobs, and print a summary of the current jobs.

The code is designed to be integrated into the tmux application, providing a mechanism to execute and manage background processes. It uses system calls and libraries such as `fork`, `exec`, `ioctl`, and `bufferevent` to handle process creation, communication, and event-driven I/O. The file does not define a public API or external interfaces but rather serves as an internal component of tmux, focusing on job management. The use of callback functions allows for flexible handling of job updates, completion, and cleanup, making it a crucial part of tmux's ability to manage multiple processes efficiently.
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
- **Use**: This variable is used to maintain and manage all the background jobs that are scheduled and run by the program.


# Data Structures

---
### job
- **Type**: `struct`
- **Members**:
    - `state`: An enumeration representing the current state of the job, which can be JOB_RUNNING, JOB_DEAD, or JOB_CLOSED.
    - `flags`: An integer representing various flags associated with the job.
    - `cmd`: A pointer to a character array (string) representing the command to be executed by the job.
    - `pid`: A process ID (pid_t) representing the process ID of the job.
    - `tty`: A character array representing the terminal associated with the job, with a maximum size of TTY_NAME_MAX.
    - `status`: An integer representing the status of the job, typically used to store the exit status.
    - `fd`: An integer representing the file descriptor associated with the job.
    - `event`: A pointer to a bufferevent structure used for handling buffered I/O events for the job.
    - `updatecb`: A callback function pointer for updating the job's state or data.
    - `completecb`: A callback function pointer that is called when the job is completed.
    - `freecb`: A callback function pointer for freeing resources associated with the job.
    - `data`: A void pointer to user-defined data associated with the job.
    - `entry`: A LIST_ENTRY structure used to link this job into a list of jobs.
- **Description**: The 'job' structure is a comprehensive data structure used to manage and track the execution of background jobs in a system. It includes fields for maintaining the job's state, command, process ID, terminal, status, and file descriptor. Additionally, it supports event-driven I/O through a bufferevent, and allows for user-defined callbacks for updating, completing, and freeing the job. The structure is designed to be part of a linked list, enabling efficient management of multiple jobs.


# Functions

---
### job_run<!-- {{#callable:job_run}} -->
The `job_run` function initiates a background job by forking a process to execute a command or a set of arguments, managing its environment, and setting up necessary callbacks and event handling.
- **Inputs**:
    - `cmd`: A string representing the command to be executed; if NULL, the command is constructed from `argc` and `argv`.
    - `argc`: An integer representing the number of arguments in `argv`.
    - `argv`: An array of strings representing the arguments for the command to be executed.
    - `e`: A pointer to an `environ` structure representing the environment variables for the job.
    - `s`: A pointer to a `session` structure, which may provide session-specific options.
    - `cwd`: A string representing the current working directory for the job; if NULL, defaults to the home directory or root.
    - `updatecb`: A callback function to be called when the job's output is updated.
    - `completecb`: A callback function to be called when the job completes.
    - `freecb`: A callback function to be called when the job is freed.
    - `data`: A pointer to user-defined data associated with the job.
    - `flags`: An integer representing various flags that control job behavior, such as whether to use a PTY.
    - `sx`: An integer representing the width of the terminal if a PTY is used.
    - `sy`: An integer representing the height of the terminal if a PTY is used.
- **Control Flow**:
    - Initialize the environment for the job using the session's environment or a provided environment.
    - Determine the shell to use based on flags and session options, defaulting to `/bin/sh` if necessary.
    - Block all signals to prevent interference during job setup.
    - If the `JOB_PTY` flag is set, create a pseudo-terminal with specified dimensions; otherwise, create a socket pair for communication.
    - Fork the process to execute the command or arguments, handling errors appropriately.
    - In the child process, set up the environment, change the working directory, and redirect standard input/output as needed.
    - Execute the command using `execl` or `execvp`, depending on whether `cmd` is provided or not.
    - In the parent process, restore the signal mask and free resources used for setup.
    - Allocate and initialize a `job` structure to track the job's state, command, process ID, and file descriptor.
    - Insert the job into a global list of jobs and set up event handling for job output.
    - Return the initialized `job` structure, or NULL if an error occurred during setup.
- **Output**:
    - Returns a pointer to a `job` structure representing the running job, or NULL if the job could not be started.


---
### job_transfer<!-- {{#callable:job_transfer}} -->
The `job_transfer` function transfers ownership of a job's file descriptor and cleans up the job structure by freeing associated resources.
- **Inputs**:
    - `job`: A pointer to the `job` structure representing the job to be transferred.
    - `pid`: A pointer to a `pid_t` variable where the job's process ID will be stored, if not NULL.
    - `tty`: A pointer to a character array where the job's TTY name will be copied, if not NULL.
    - `ttylen`: The size of the `tty` buffer to ensure safe copying of the TTY name.
- **Control Flow**:
    - Retrieve the file descriptor from the job structure and store it in a local variable `fd`.
    - Log the job transfer operation for debugging purposes.
    - If `pid` is not NULL, store the job's process ID in the provided `pid` variable.
    - If `tty` is not NULL, copy the job's TTY name into the provided `tty` buffer using `strlcpy` to ensure it does not exceed `ttylen`.
    - Remove the job from the global job list using `LIST_REMOVE`.
    - Free the memory allocated for the job's command string.
    - If a free callback (`freecb`) is defined and job data is present, call the free callback to release the job's data.
    - If a bufferevent is associated with the job, free it using `bufferevent_free`.
    - Free the memory allocated for the job structure itself.
    - Return the file descriptor `fd` that was originally associated with the job.
- **Output**:
    - The function returns the file descriptor associated with the job, which can be used by the caller to continue interacting with the job's output or input.


---
### job_free<!-- {{#callable:job_free}} -->
The `job_free` function is responsible for cleaning up and freeing resources associated with a job structure.
- **Inputs**:
    - `job`: A pointer to a `struct job` that represents the job to be freed.
- **Control Flow**:
    - Logs a debug message indicating the job being freed.
    - Removes the job from the list of all jobs using `LIST_REMOVE`.
    - Frees the memory allocated for the job's command string using `free(job->cmd)`.
    - Checks if a free callback (`freecb`) is set and if job data is present, then calls the callback with the job data.
    - If the job has a valid process ID (`pid`), sends a SIGTERM signal to terminate the process.
    - If the job has an associated event (`event`), frees the event using `bufferevent_free`.
    - If the job has a valid file descriptor (`fd`), closes the file descriptor using `close`.
    - Finally, frees the memory allocated for the job structure itself using `free(job)`.
- **Output**:
    - The function does not return any value; it performs cleanup operations and frees resources associated with the job.


---
### job_resize<!-- {{#callable:job_resize}} -->
The `job_resize` function adjusts the terminal window size for a job that uses a pseudo-terminal (PTY).
- **Inputs**:
    - `job`: A pointer to a `struct job` representing the job whose terminal size is to be adjusted.
    - `sx`: An unsigned integer representing the new number of columns for the terminal window.
    - `sy`: An unsigned integer representing the new number of rows for the terminal window.
- **Control Flow**:
    - Check if the job's file descriptor is valid and if the job uses a PTY; if not, return immediately.
    - Log a debug message indicating the job and the new size dimensions.
    - Initialize a `struct winsize` to zero and set its `ws_col` and `ws_row` fields to `sx` and `sy`, respectively.
    - Use the `ioctl` system call to set the terminal window size using the `TIOCSWINSZ` command; if this fails, call `fatal` to handle the error.
- **Output**:
    - The function does not return a value; it performs an operation to resize the terminal window associated with a job.


---
### job_read_callback<!-- {{#callable:job_read_callback}} -->
The `job_read_callback` function is a callback that triggers the update callback function of a job when data is read from a buffer event.
- **Inputs**:
    - `bufev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `data`: A pointer to a `job` structure, representing the job associated with the buffer event.
- **Control Flow**:
    - The function checks if the `updatecb` callback function of the `job` structure is not NULL.
    - If `updatecb` is not NULL, it calls the `updatecb` function, passing the `job` structure as an argument.
- **Output**:
    - The function does not return any value; it performs an action by invoking a callback if it is set.


---
### job_write_callback<!-- {{#callable:job_write_callback}} -->
The `job_write_callback` function handles the write event for a job's buffer, logging the job's status and disabling further write events if the buffer is empty and the job is not flagged to keep writing.
- **Inputs**:
    - `bufev`: A pointer to the bufferevent structure associated with the job, marked as unused in this function.
    - `data`: A pointer to the job structure, which contains information about the job being processed.
- **Control Flow**:
    - Retrieve the job structure from the data pointer.
    - Calculate the length of the output buffer associated with the job's event.
    - Log the current state of the job, including its command, process ID, and remaining output buffer length.
    - Check if the output buffer length is zero and the job is not flagged with JOB_KEEPWRITE.
    - If the buffer is empty and the job is not flagged to keep writing, shut down the write side of the job's file descriptor and disable the write event on the bufferevent.
- **Output**:
    - The function does not return a value; it performs actions based on the state of the job's output buffer.


---
### job_error_callback<!-- {{#callable:job_error_callback}} -->
The `job_error_callback` function handles errors for a job by logging the error and either completing and freeing the job if it is dead, or disabling its read event and marking it as closed.
- **Inputs**:
    - `bufev`: A pointer to the bufferevent structure associated with the job, which is unused in this function.
    - `events`: A short integer representing the events that triggered the callback, which is unused in this function.
    - `data`: A pointer to the job structure that encountered an error.
- **Control Flow**:
    - Log the error details including the job pointer, command, and process ID.
    - Check if the job's state is `JOB_DEAD`.
    - If the job is dead, check if a completion callback (`completecb`) is set and call it if it exists.
    - Free the job using [`job_free`](#job_free) if it is dead.
    - If the job is not dead, disable the read event on the job's bufferevent.
    - Set the job's state to `JOB_CLOSED`.
- **Output**:
    - The function does not return any value; it performs actions based on the job's state and modifies the job's state and event handling accordingly.
- **Functions called**:
    - [`job_free`](#job_free)


---
### job_check_died<!-- {{#callable:job_check_died}} -->
The `job_check_died` function checks if a job with a given process ID has died and updates its status accordingly.
- **Inputs**:
    - `pid`: The process ID of the job to check.
    - `status`: The status code returned by the job, typically from a waitpid() call.
- **Control Flow**:
    - Iterate over the list of all jobs to find the job with the matching process ID (pid).
    - If no job is found with the given pid, return immediately.
    - Check if the job was stopped using WIFSTOPPED(status).
    - If the job was stopped due to SIGTTIN or SIGTTOU, return without further action.
    - If the job was stopped for other reasons, send a SIGCONT signal to the job's process group to continue it.
    - Log the job's death with its details.
    - Update the job's status with the provided status code.
    - If the job's state is JOB_CLOSED, call the job's completion callback if it exists, and then free the job.
    - If the job's state is not JOB_CLOSED, set the job's pid to -1 and update its state to JOB_DEAD.
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
    - The function takes a single argument, a pointer to a `struct job`.
    - It returns the `data` member of the `struct job`.
- **Output**:
    - A void pointer to the data associated with the job.


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
    - The function uses the `LIST_FOREACH` macro to iterate over each job in the `all_jobs` list.
    - For each job, it checks if the job's `pid` is not equal to -1, indicating that the job is running.
    - If the job is running, it sends a SIGTERM signal to the job's process using the `kill` function.
- **Output**:
    - The function does not return any value; it performs an action by sending termination signals to all running jobs.


---
### job_still_running<!-- {{#callable:job_still_running}} -->
The `job_still_running` function checks if there are any jobs in the global job list that are currently running and not marked with the `JOB_NOWAIT` flag.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each job in the global `all_jobs` list using the `LIST_FOREACH` macro.
    - For each job, check if the `JOB_NOWAIT` flag is not set and if the job's state is `JOB_RUNNING`.
    - If a job meets these conditions, return 1 to indicate that there is at least one running job.
    - If no jobs meet the conditions, return 0 after the loop completes.
- **Output**:
    - The function returns an integer: 1 if there is at least one job running that is not marked with `JOB_NOWAIT`, otherwise 0.


---
### job_print_summary<!-- {{#callable:job_print_summary}} -->
The `job_print_summary` function iterates over all jobs and prints a summary of each job's details to a command queue item, optionally starting with a blank line.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure where the job summary will be printed.
    - `blank`: An integer flag indicating whether to print a blank line before the job summaries (non-zero value) or not (zero value).
- **Control Flow**:
    - Initialize a job pointer and a counter `n` to zero.
    - Iterate over each job in the global `all_jobs` list using `LIST_FOREACH`.
    - If `blank` is non-zero, print a blank line to `item` and set `blank` to zero.
    - For each job, print its details (job number, command, file descriptor, process ID, and status) to `item`.
    - Increment the job counter `n` after printing each job's details.
- **Output**:
    - The function does not return a value; it outputs job summaries to the provided `cmdq_item`.


# Function Declarations (Public API)

---
### job_read_callback<!-- {{#callable_declaration:job_read_callback}} -->
Invoke the job's update callback if it is set.
- **Description**: This function is used as a callback for reading data from a job's buffer event. It should be registered with a bufferevent to handle read events. When invoked, it checks if the job's update callback is set and, if so, calls it. This function is typically used in the context of managing asynchronous job execution, where updates to the job's state or output need to be processed.
- **Inputs**:
    - `bufev`: A pointer to a bufferevent structure, which is unused in this function. It is expected to be valid but can be ignored.
    - `data`: A pointer to a job structure. This must not be null and should point to a valid job object whose update callback may be invoked.
- **Output**: None
- **See also**: [`job_read_callback`](#job_read_callback)  (Implementation)


---
### job_write_callback<!-- {{#callable_declaration:job_write_callback}} -->
Handles write events for a job's buffer.
- **Description**: This function is a callback that manages write events for a job's buffer. It is triggered when the buffer's data falls below a certain threshold, typically when it is empty. If all data has been written and the job does not have the JOB_KEEPWRITE flag set, the function will shut down the write side of the job's file descriptor and disable further write events. This function is typically used in the context of managing asynchronous job execution and should be set as a callback for a bufferevent associated with a job.
- **Inputs**:
    - `bufev`: A pointer to the bufferevent structure associated with the job. This parameter is unused in the function.
    - `data`: A pointer to the job structure. This must not be null and should point to a valid job object that is being managed by the bufferevent.
- **Output**: None
- **See also**: [`job_write_callback`](#job_write_callback)  (Implementation)


---
### job_error_callback<!-- {{#callable_declaration:job_error_callback}} -->
Handles errors for a job's buffer event.
- **Description**: This function is used as a callback to handle errors that occur during a job's buffer event operations. It should be registered with a bufferevent to manage error conditions. When an error is detected, the function checks the job's state. If the job is dead, it invokes the job's completion callback if available and then frees the job resources. If the job is not dead, it disables further read events and marks the job as closed. This function is typically used in the context of managing asynchronous job execution and should be set up as part of the job's event handling.
- **Inputs**:
    - `bufev`: A pointer to the bufferevent associated with the job. This parameter is unused in the function.
    - `events`: A short integer representing the events that triggered the callback. This parameter is unused in the function.
    - `data`: A pointer to the job structure associated with the bufferevent. This must not be null and is used to access the job's state and callbacks.
- **Output**: None
- **See also**: [`job_error_callback`](#job_error_callback)  (Implementation)


