# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to manage multiple terminal sessions from a single window. The file appears to be the main entry point for the `tmux` application, as indicated by the presence of the [`main`](#main) function. It handles command-line arguments, sets up the environment, and initializes global options and configurations necessary for the operation of `tmux`. The code includes functionality for parsing command-line options, managing environment variables, and setting up default configurations for server, session, and window options.

Key components of the code include functions for handling shell paths, expanding environment variables, and managing socket paths for communication. The code also includes utility functions for setting file descriptor blocking modes, retrieving the current working directory, and determining the user's home directory. The [`main`](#main) function orchestrates the initialization process, including setting locale settings, configuring UTF-8 support, and determining the appropriate shell to use. The file is designed to be compiled into an executable, as it contains the [`main`](#main) function and directly interacts with the operating system through system calls and environment management.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/stat.h`
- `sys/utsname.h`
- `errno.h`
- `fcntl.h`
- `langinfo.h`
- `locale.h`
- `pwd.h`
- `signal.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### global_options
- **Type**: `struct options*`
- **Description**: The `global_options` variable is a pointer to a `struct options` that holds the server options for the application. It is used to store and manage configuration settings that apply globally to the server.
- **Use**: This variable is used to store and manage server-wide configuration options.


---
### global_s_options
- **Type**: `struct options*`
- **Description**: The `global_s_options` is a global pointer to a `struct options` that holds session-specific options in the tmux application. It is used to manage and store configuration settings that apply to individual sessions within tmux.
- **Use**: This variable is used to store and manage session-specific configuration options in the tmux application.


---
### global_w_options
- **Type**: `struct options*`
- **Description**: The `global_w_options` is a global pointer to a `struct options` that holds configuration settings specific to windows in the application. It is part of a set of global options that also includes server and session options.
- **Use**: This variable is used to store and manage window-specific configuration options throughout the application.


---
### global_environ
- **Type**: `struct environ*`
- **Description**: The `global_environ` variable is a pointer to a `struct environ`, which is used to manage environment variables within the program. This structure likely contains entries that represent key-value pairs of environment variables, allowing the program to access and modify the environment settings.
- **Use**: `global_environ` is used to store and manage the environment variables for the program, enabling operations such as retrieving and setting environment variables.


---
### start_time
- **Type**: `struct timeval`
- **Description**: The `start_time` variable is a global variable of type `struct timeval`, which is used to represent a specific point in time with precision up to microseconds. It is typically used to store the start time of a process or event, allowing for time calculations and measurements.
- **Use**: This variable is used to record the start time of a process or event, enabling time-related operations such as calculating elapsed time.


---
### socket_path
- **Type**: `const char*`
- **Description**: The `socket_path` variable is a global constant character pointer that holds the path to the socket used by the application. It is initialized to `NULL` and is later set to a specific path based on command-line arguments or environment variables.
- **Use**: This variable is used to store and access the path of the socket for inter-process communication within the application.


---
### ptm_fd
- **Type**: `int`
- **Description**: The `ptm_fd` variable is an integer that is initialized to -1. It is used to store the file descriptor for a pseudo-terminal master (PTM).
- **Use**: This variable is used to manage the file descriptor for a PTM, which is essential for terminal emulation and communication in the program.


---
### shell_command
- **Type**: `const char*`
- **Description**: The `shell_command` variable is a global constant pointer to a character string, which is intended to store the command for the shell to execute. It is defined as a constant, indicating that the pointer itself can be modified to point to a different string, but the string it points to should not be modified.
- **Use**: This variable is used to store the shell command specified by the user through the command-line option `-c`.


---
### make_label
- **Type**: `function pointer`
- **Description**: `make_label` is a static function pointer that takes two parameters: a constant character pointer and a double character pointer. It returns a character pointer.
- **Use**: This function is used to generate a label for a socket path, ensuring it is unique and valid, and returns the path as a string.


---
### getshell
- **Type**: `function`
- **Description**: The `getshell` function is a static function that returns a constant character pointer to the default shell path. It first checks the `SHELL` environment variable and validates it using the `checkshell` function. If the `SHELL` environment variable is not valid, it retrieves the shell from the user's password entry using `getpwuid` and validates it. If both checks fail, it defaults to the Bourne shell path (`_PATH_BSHELL`).
- **Use**: This function is used to determine and return the default shell path for the user, which is then set as the default shell in the global session options.


# Functions

---
### usage<!-- {{#callable:usage}} -->
The `usage` function prints the usage message for the program to either standard output or standard error based on the status and then exits the program with the given status code.
- **Inputs**:
    - `status`: An integer representing the exit status; if non-zero, the usage message is printed to standard error, otherwise to standard output.
- **Control Flow**:
    - The function checks the value of `status` to determine whether to print the usage message to `stderr` or `stdout`.
    - It uses `fprintf` to print the usage message, which includes the program name and a list of possible command-line options.
    - The program name is retrieved using `getprogname()`.
    - Finally, the function calls `exit(status)` to terminate the program with the provided status code.
- **Output**:
    - The function does not return a value; it terminates the program with the specified exit status.


---
### getshell<!-- {{#callable:getshell}} -->
The `getshell` function determines and returns the appropriate shell for the current user, prioritizing the `SHELL` environment variable, then the user's password entry, and defaults to a predefined shell path if necessary.
- **Inputs**:
    - None
- **Control Flow**:
    - Retrieve the `SHELL` environment variable and store it in `shell`.
    - Check if the `shell` from the environment variable is valid using [`checkshell`](#checkshell); if valid, return it.
    - Retrieve the password entry for the current user using `getpwuid(getuid())` and store it in `pw`.
    - If `pw` is not NULL and the shell from the password entry (`pw->pw_shell`) is valid, return it.
    - If neither the environment variable nor the password entry provides a valid shell, return the default shell path `_PATH_BSHELL`.
- **Output**:
    - The function returns a `const char *` which is the path to the determined shell.
- **Functions called**:
    - [`checkshell`](#checkshell)


---
### checkshell<!-- {{#callable:checkshell}} -->
The `checkshell` function verifies if a given shell path is valid and executable.
- **Inputs**:
    - `shell`: A constant character pointer representing the path to the shell to be checked.
- **Control Flow**:
    - Check if the `shell` is `NULL` or does not start with a '/' character; if so, return 0.
    - Call [`areshell`](#areshell) to check if the shell is the same as the program name; if true, return 0.
    - Use `access` to check if the shell path is executable; if not, return 0.
    - If all checks pass, return 1 indicating the shell is valid and executable.
- **Output**:
    - Returns an integer: 1 if the shell path is valid and executable, otherwise 0.
- **Functions called**:
    - [`areshell`](#areshell)


---
### areshell<!-- {{#callable:areshell}} -->
The `areshell` function checks if the given shell name matches the program name, ignoring any leading path components and a leading dash in the program name.
- **Inputs**:
    - `shell`: A constant character pointer representing the shell path or name to be checked against the program name.
- **Control Flow**:
    - The function uses `strrchr` to find the last occurrence of '/' in the `shell` string to isolate the shell name from its path.
    - If a '/' is found, `ptr` is set to the character following the last '/', otherwise `ptr` is set to the beginning of the `shell` string.
    - The program name is retrieved using `getprogname()`, and if it starts with a '-', the pointer is incremented to skip it.
    - The function compares the isolated shell name (`ptr`) with the program name (`progname`) using `strcmp`.
    - If they match, the function returns 1, indicating the shell name matches the program name; otherwise, it returns 0.
- **Output**:
    - The function returns an integer: 1 if the shell name matches the program name, otherwise 0.


---
### expand_path<!-- {{#callable:expand_path}} -->
The `expand_path` function expands a given path by replacing the home directory or environment variable references with their actual values.
- **Inputs**:
    - `path`: A string representing the path to be expanded, which may start with '~/'' or '$' to indicate a home directory or environment variable, respectively.
    - `home`: A string representing the home directory path, used when the input path starts with '~/''.
- **Control Flow**:
    - Check if the path starts with '~/'; if so, replace it with the home directory path and return the expanded path.
    - If the path starts with '$', extract the environment variable name, find its value in the global environment, and replace the variable with its value in the path.
    - If the path does not start with '~/'' or '$', return a duplicate of the original path.
- **Output**:
    - Returns a newly allocated string with the expanded path, or NULL if expansion fails due to missing home directory or environment variable.


---
### expand_paths<!-- {{#callable:expand_paths}} -->
The `expand_paths` function processes a colon-separated string of paths, expands each path, resolves it to an absolute path, and stores unique, valid paths in a dynamically allocated array.
- **Inputs**:
    - `s`: A colon-separated string of paths to be expanded and resolved.
    - `paths`: A pointer to an array of strings where the expanded and resolved paths will be stored.
    - `n`: A pointer to an unsigned integer that will store the number of valid paths found.
    - `ignore_errors`: An integer flag indicating whether to ignore errors during path resolution (non-zero to ignore errors).
- **Control Flow**:
    - Initialize the `paths` array to NULL and `n` to 0.
    - Duplicate the input string `s` to `copy` and `tmp` for manipulation.
    - Iterate over each path in `tmp` using `strsep` to split by colon (':').
    - For each path, call [`expand_path`](#expand_path) to handle home directory and environment variable expansions.
    - If [`expand_path`](#expand_path) returns NULL, log an invalid path message and continue to the next path.
    - Attempt to resolve the expanded path to an absolute path using `realpath`.
    - If `realpath` fails and `ignore_errors` is set, log the error and continue; otherwise, use the expanded path as is.
    - Check if the resolved path is already in the `paths` array to avoid duplicates.
    - If the path is unique, reallocate the `paths` array to accommodate the new path and add it to the array.
    - Free the duplicated string `copy` after processing all paths.
- **Output**:
    - The function outputs a dynamically allocated array of unique, valid, and resolved paths, updating the `paths` pointer and the count `n`.
- **Functions called**:
    - [`find_home`](#find_home)
    - [`expand_path`](#expand_path)


---
### make_label<!-- {{#callable:make_label}} -->
The `make_label` function constructs a file path for a tmux socket directory based on a given label and user ID, ensuring the directory exists and has safe permissions.
- **Inputs**:
    - `label`: A string representing the label for the socket directory; if NULL, defaults to "default".
    - `cause`: A pointer to a string that will be set with an error message if the function fails.
- **Control Flow**:
    - Initialize `cause` to NULL and set `label` to "default" if it is NULL.
    - Retrieve the current user ID using `getuid()`.
    - Expand the paths for the tmux socket using [`expand_paths`](#expand_paths), storing them in `paths` and the count in `n`.
    - If no paths are found (`n == 0`), set `cause` to an error message and return NULL.
    - Select the first path from `paths` and free the rest.
    - Construct the base directory path using the selected path and user ID, then free the selected path.
    - Attempt to create the directory specified by `base`; if it fails and the error is not `EEXIST`, set `cause` to an error message and go to the fail label.
    - Check if the directory exists and is a directory using `lstat`; if not, set `cause` to an error message and go to the fail label.
    - Verify the directory's ownership and permissions; if they are unsafe, set `cause` to an error message and go to the fail label.
    - Construct the final path by appending the `label` to the `base` directory path, free `base`, and return the constructed path.
    - In the fail label, free `base` and return NULL.
- **Output**:
    - Returns a string representing the full path to the tmux socket directory if successful, or NULL if an error occurs, with `cause` set to an error message.
- **Functions called**:
    - [`expand_paths`](#expand_paths)


---
### shell_argv0<!-- {{#callable:shell_argv0}} -->
The `shell_argv0` function constructs a string representing the shell's name, optionally prefixed with a dash if it is a login shell.
- **Inputs**:
    - `shell`: A constant character pointer representing the full path to the shell executable.
    - `is_login`: An integer flag indicating whether the shell is a login shell (non-zero) or not (zero).
- **Control Flow**:
    - The function searches for the last occurrence of '/' in the `shell` string to determine the shell's name.
    - If a '/' is found and it is not the last character, the shell's name is set to the substring following the last '/'.
    - If no '/' is found, the entire `shell` string is considered as the shell's name.
    - If `is_login` is true, the shell's name is prefixed with a '-' using `xasprintf`.
    - If `is_login` is false, the shell's name is used as is with `xasprintf`.
    - The constructed string is returned as the function's output.
- **Output**:
    - A dynamically allocated string representing the shell's name, prefixed with a '-' if it is a login shell.


---
### setblocking<!-- {{#callable:setblocking}} -->
The `setblocking` function sets the blocking mode of a file descriptor based on the provided state.
- **Inputs**:
    - `fd`: An integer representing the file descriptor whose blocking mode is to be set.
    - `state`: An integer where a non-zero value sets the file descriptor to blocking mode, and zero sets it to non-blocking mode.
- **Control Flow**:
    - Retrieve the current file status flags of the file descriptor using `fcntl` with `F_GETFL`.
    - Check if the retrieval of the file status flags was successful (i.e., not equal to -1).
    - If the `state` is zero, modify the mode to include `O_NONBLOCK` to set the file descriptor to non-blocking mode.
    - If the `state` is non-zero, modify the mode to exclude `O_NONBLOCK` to set the file descriptor to blocking mode.
    - Apply the modified mode back to the file descriptor using `fcntl` with `F_SETFL`.
- **Output**:
    - The function does not return a value; it modifies the blocking mode of the specified file descriptor.


---
### get_timer<!-- {{#callable:get_timer}} -->
The `get_timer` function retrieves the current time in milliseconds using a monotonic clock, falling back to the real-time clock if necessary.
- **Inputs**:
    - None
- **Control Flow**:
    - Declare a `timespec` structure `ts` to hold the time values.
    - Attempt to get the current time using `clock_gettime` with `CLOCK_MONOTONIC`.
    - If `clock_gettime` fails (returns non-zero), attempt to get the time using `CLOCK_REALTIME`.
    - Calculate the time in milliseconds by converting seconds to milliseconds and adding the nanoseconds converted to milliseconds.
    - Return the calculated time in milliseconds as a `uint64_t`.
- **Output**:
    - The function returns the current time in milliseconds as a `uint64_t` value.


---
### sig2name<!-- {{#callable:sig2name}} -->
The `sig2name` function converts a signal number to its corresponding signal name if available, or returns the signal number as a string if not.
- **Inputs**:
    - `signo`: An integer representing the signal number to be converted to a signal name.
- **Control Flow**:
    - Declare a static character array `s` of size 11 to store the signal name or number as a string.
    - Check if the macro `HAVE_SYS_SIGNAME` is defined, indicating the presence of the `sys_signame` array.
    - If `HAVE_SYS_SIGNAME` is defined and `signo` is greater than 0 and less than `NSIG`, return the signal name from the `sys_signame` array corresponding to `signo`.
    - If the above condition is not met, use `xsnprintf` to format the signal number `signo` into the string `s`.
    - Return the string `s`, which contains either the signal name or the formatted signal number.
- **Output**:
    - Returns a pointer to a string containing the signal name if available, or the signal number as a string if not.


---
### find_cwd<!-- {{#callable:find_cwd}} -->
The `find_cwd` function determines the current working directory, preferring the `PWD` environment variable if it matches the actual directory path, to preserve symbolic links.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize two character arrays `resolved1` and `resolved2` for storing resolved paths, and a static array `cwd` for the current working directory.
    - Attempt to get the current working directory using `getcwd` and store it in `cwd`; return `NULL` if it fails.
    - Retrieve the `PWD` environment variable; if it is not set or empty, return `cwd`.
    - Resolve the path in `PWD` to `resolved1` using `realpath`; if it fails, return `cwd`.
    - Resolve the path in `cwd` to `resolved2` using `realpath`; if it fails, return `cwd`.
    - Compare `resolved1` and `resolved2`; if they differ, return `cwd`.
    - If all checks pass, return `pwd` to preserve symbolic links.
- **Output**:
    - The function returns a pointer to a string representing the current working directory, either from `PWD` or `cwd`, or `NULL` if `getcwd` fails.


---
### find_home<!-- {{#callable:find_home}} -->
The `find_home` function retrieves the home directory path of the current user, either from the `HOME` environment variable or from the user's password entry if the environment variable is not set.
- **Inputs**:
    - None
- **Control Flow**:
    - Check if the static variable `home` is already set; if so, return it immediately.
    - Attempt to retrieve the home directory path from the `HOME` environment variable.
    - If the `HOME` environment variable is not set or is empty, use `getpwuid` with the current user's UID to get the password entry and retrieve the home directory from there.
    - If the password entry is not found, set `home` to `NULL`.
    - Return the `home` directory path.
- **Output**:
    - The function returns a pointer to a string containing the home directory path of the current user, or `NULL` if it cannot be determined.


---
### getversion<!-- {{#callable:getversion}} -->
The `getversion` function returns the version of the TMUX software as a string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function directly returns the value of the `TMUX_VERSION` constant.
- **Output**:
    - A constant character pointer to the TMUX version string.


---
### main<!-- {{#callable:main}} -->
The `main` function initializes and configures the tmux client environment, processes command-line options, and then delegates control to the client main function.
- **Inputs**:
    - `argc`: The number of command-line arguments passed to the program.
    - `argv`: An array of strings representing the command-line arguments.
- **Control Flow**:
    - Set the locale to UTF-8 and check for valid locale settings, exiting with an error if invalid.
    - Initialize environment variables and set the current working directory in the environment.
    - Parse command-line options using `getopt`, setting various flags and options based on the provided arguments.
    - Check for conflicts in command-line arguments and exit with usage information if necessary.
    - Initialize pseudo-terminal file descriptor and apply security pledges.
    - Determine UTF-8 support based on environment variables and set the appropriate flag.
    - Create global options for server, session, and window, and set default options based on a predefined options table.
    - Set the default shell and editor options based on environment variables.
    - Determine the socket path from command-line options, environment variables, or default settings, and handle errors if the path cannot be determined.
    - Pass control to the `client_main` function with the initialized parameters and exit.
- **Output**:
    - The function does not return a value as it exits the program after passing control to the `client_main` function.
- **Functions called**:
    - [`find_cwd`](#find_cwd)
    - [`expand_paths`](#expand_paths)
    - [`usage`](#usage)
    - [`getversion`](#getversion)
    - [`getshell`](#getshell)
    - [`make_label`](#make_label)


# Function Declarations (Public API)

---
### usage<!-- {{#callable_declaration:usage}} -->
Display usage information and exit with the given status.
- **Description**: This function is used to display the usage information for the program, which includes the command-line options and their descriptions. It is typically called when the user provides incorrect arguments or requests help. The function outputs the usage message to either standard output or standard error, depending on the provided status, and then terminates the program with the same status code. This function should be called when the program needs to inform the user about the correct usage format and then exit.
- **Inputs**:
    - `status`: An integer representing the exit status. If non-zero, the usage message is printed to standard error; otherwise, it is printed to standard output. The program then exits with this status code.
- **Output**: None
- **See also**: [`usage`](#usage)  (Implementation)


---
### make_label<!-- {{#callable_declaration:make_label}} -->
Generates a file path label for a tmux session.
- **Description**: This function constructs a file path label for a tmux session, using a specified label or a default if none is provided. It ensures the directory for the label exists and has safe permissions. If any errors occur during this process, such as directory creation failure or unsafe permissions, an error message is set in the provided cause pointer. This function should be used when a unique, user-specific file path is needed for tmux operations, and it must be called with a valid cause pointer to capture potential error messages.
- **Inputs**:
    - `label`: A string representing the desired label for the file path. If null, a default label is used. The caller retains ownership and it must be a valid string or null.
    - `cause`: A pointer to a char pointer where an error message will be stored if the function fails. Must not be null, and the caller is responsible for freeing any allocated memory.
- **Output**: Returns a newly allocated string containing the file path label if successful, or NULL if an error occurs. The caller is responsible for freeing the returned string.
- **See also**: [`make_label`](#make_label)  (Implementation)


---
### areshell<!-- {{#callable_declaration:areshell}} -->
Checks if the given shell name matches the current program name.
- **Description**: Use this function to determine if a specified shell name corresponds to the current program's name. This can be useful in scenarios where you need to verify if the shell being referenced is the same as the one currently executing. The function expects a valid shell name and compares it against the program name obtained from the environment. It is important to ensure that the shell parameter is a valid string, as the function does not handle null or invalid pointers.
- **Inputs**:
    - `shell`: A string representing the shell name to be checked. It must be a valid, non-null string. The function does not handle null pointers and expects the input to be a valid shell path or name.
- **Output**: Returns 1 if the shell name matches the current program name, otherwise returns 0.
- **See also**: [`areshell`](#areshell)  (Implementation)


---
### getshell<!-- {{#callable_declaration:getshell}} -->
Retrieves the user's preferred shell.
- **Description**: This function determines the user's preferred shell by first checking the 'SHELL' environment variable. If the value is valid, it returns this shell. If not, it retrieves the shell from the user's password entry using their user ID. If this shell is also invalid, it defaults to returning a standard shell path. This function is useful for applications that need to execute commands in the user's preferred shell environment. It assumes the environment and user ID are correctly set up and accessible.
- **Inputs**:
    - None
- **Output**: Returns a pointer to a string containing the path of the preferred shell, or a default shell path if none is set or valid.
- **See also**: [`getshell`](#getshell)  (Implementation)


