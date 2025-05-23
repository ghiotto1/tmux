# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to switch between several programs in one terminal, detach them, and reattach them later. The file appears to be the main entry point for the `tmux` application, as indicated by the presence of the [`main`](#main) function. It handles command-line arguments, sets up the environment, and initializes global options and configurations for the `tmux` server and client. The code includes functionality for parsing command-line options, setting locale and environment variables, and determining the appropriate shell to use. It also manages socket paths for communication between the `tmux` client and server, and sets up default options based on user environment variables like `SHELL`, `VISUAL`, and `EDITOR`.

The file includes several utility functions that support the main functionality, such as [`getshell`](#getshell) to determine the user's shell, [`expand_path`](#expand_path) to handle path expansions, and [`setblocking`](#setblocking) to configure file descriptor blocking modes. It also defines global variables for server, session, and window options, as well as the environment and socket path. The code is structured to handle various configurations and setups, making it a critical component of the `tmux` application, responsible for initializing and managing the core settings and environment before handing control over to the client-side logic.
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
- **Use**: This variable is used to initialize and manage server-wide configuration options.


---
### global_s_options
- **Type**: `struct options*`
- **Description**: `global_s_options` is a global pointer to a `struct options`, which is used to store session-specific options in the program. It is initialized by calling `options_create(NULL)` and is used to set default session options and configurations.
- **Use**: This variable is used to manage and store configuration options specific to a session, allowing the program to handle session-specific settings.


---
### global_w_options
- **Type**: `struct options*`
- **Description**: The `global_w_options` is a global pointer to a `struct options` that holds configuration settings specific to windows in the application. It is part of a set of global options that also includes server and session options.
- **Use**: This variable is used to store and manage window-specific configuration options throughout the application.


---
### global_environ
- **Type**: `struct environ*`
- **Description**: `global_environ` is a pointer to a `struct environ`, which is likely a data structure used to manage environment variables within the program. This structure is used to store and manipulate environment variables that are accessible globally throughout the application.
- **Use**: This variable is used to store and manage the environment variables for the application, allowing for operations such as finding, setting, and expanding environment variables.


---
### start_time
- **Type**: `struct timeval`
- **Description**: The `start_time` variable is a global variable of type `struct timeval`, which is used to store time values with second and microsecond precision. It is typically used to record the start time of a process or event.
- **Use**: This variable is used to track the start time of the program or a specific operation, allowing for time calculations or performance measurements.


---
### socket_path
- **Type**: `const char*`
- **Description**: The `socket_path` variable is a global constant character pointer that holds the path to the socket used by the application. It is initialized to `NULL` and is later set to a specific path based on command-line arguments or environment variables.
- **Use**: This variable is used to store and access the path of the socket for inter-process communication within the application.


---
### ptm_fd
- **Type**: `int`
- **Description**: The `ptm_fd` variable is an integer that is initialized to -1. It is used to store the file descriptor for a pseudo-terminal master (PTM).
- **Use**: This variable is used to hold the file descriptor for the PTM, which is obtained by calling the `getptmfd()` function.


---
### shell_command
- **Type**: `const char*`
- **Description**: The `shell_command` variable is a global constant character pointer that stores the shell command specified by the user. It is used to hold the command string passed as an argument to the program, typically through the '-c' option in the command line.
- **Use**: This variable is used to execute a specific shell command when the program is run with the '-c' option.


---
### make_label
- **Type**: `static char *`
- **Description**: The `make_label` variable is a static function pointer that returns a character pointer. It takes two parameters: a constant character pointer and a double character pointer. This function is used to create a label for a socket path, ensuring it is unique and valid for the current user.
- **Use**: This variable is used to generate a unique and valid socket path label for the tmux session, based on the provided label and user ID.


---
### getshell
- **Type**: `function`
- **Description**: The `getshell` function is a static function that returns a constant character pointer to the default shell path. It first checks the `SHELL` environment variable and validates it using the `checkshell` function. If the `SHELL` environment variable is not valid, it retrieves the shell from the user's password entry using `getpwuid` and validates it. If neither is valid, it defaults to `_PATH_BSHELL`, which is typically `/bin/sh`.
- **Use**: This function is used to determine and return the default shell path for the user, which is then set as the default shell in the global session options.


# Functions

---
### usage<!-- {{#callable:usage}} -->
The `usage` function prints the usage message for the program to either standard output or standard error based on the status and then exits the program with the given status code.
- **Inputs**:
    - `status`: An integer that determines whether the usage message is printed to standard output (stdout) or standard error (stderr); a non-zero value indicates stderr, while zero indicates stdout.
- **Control Flow**:
    - The function checks the value of 'status' to decide whether to print the usage message to stderr or stdout.
    - It uses the `fprintf` function to print the usage message, which includes the program name obtained from `getprogname()` and a list of command-line options.
    - Finally, the function calls `exit` with the provided status to terminate the program.
- **Output**:
    - The function does not return a value; it terminates the program with the given exit status.


---
### getshell<!-- {{#callable:getshell}} -->
The `getshell` function determines and returns the appropriate shell for the current user, prioritizing the `SHELL` environment variable, then the user's shell from the password entry, and defaults to a predefined shell path if neither is valid.
- **Inputs**:
    - None
- **Control Flow**:
    - Retrieve the `SHELL` environment variable and store it in `shell`.
    - Check if `shell` is a valid shell using [`checkshell`](#checkshell); if valid, return `shell`.
    - Retrieve the password entry for the current user using `getpwuid(getuid())` and store it in `pw`.
    - If `pw` is not NULL and `pw->pw_shell` is a valid shell, return `pw->pw_shell`.
    - If neither the environment variable nor the password entry provides a valid shell, return the default shell path `_PATH_BSHELL`.
- **Output**:
    - Returns a constant character pointer to the determined shell path.
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
    - `shell`: A string representing the shell path or name to be checked against the program name.
- **Control Flow**:
    - The function searches for the last occurrence of '/' in the shell string to isolate the shell name from its path.
    - If a '/' is found, the pointer `ptr` is set to the character following the last '/'; otherwise, `ptr` is set to the start of the shell string.
    - The program name is retrieved using `getprogname()`, and if it starts with a '-', the pointer is incremented to skip it.
    - The function compares the isolated shell name (`ptr`) with the program name (`progname`).
    - If they match, the function returns 1; otherwise, it returns 0.
- **Output**:
    - Returns 1 if the shell name matches the program name, otherwise returns 0.


---
### expand_path<!-- {{#callable:expand_path}} -->
The `expand_path` function expands a given path by replacing the home directory or environment variables with their actual values.
- **Inputs**:
    - `path`: A string representing the path to be expanded, which may start with '~/' for the home directory or '$' for an environment variable.
    - `home`: A string representing the home directory path, used when the input path starts with '~/'.
- **Control Flow**:
    - Check if the path starts with '~/'.
    - If it does, and if the home directory is provided, concatenate the home directory with the rest of the path and return it.
    - If the path starts with '$', find the end of the environment variable name by looking for a '/' or the end of the string.
    - Extract the environment variable name and look it up in the global environment.
    - If the environment variable is found, concatenate its value with the rest of the path and return it.
    - If the path does not start with '~/ or '$', return a duplicate of the path.
- **Output**:
    - A dynamically allocated string containing the expanded path, or NULL if expansion fails.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
    - [`environ_find`](environ.c.driver.md#environ_find)


---
### expand_paths<!-- {{#callable:expand_paths}} -->
The `expand_paths` function processes a colon-separated string of paths, expands each path, resolves it to an absolute path, and stores unique, valid paths in a dynamically allocated array.
- **Inputs**:
    - `s`: A colon-separated string of paths to be expanded and resolved.
    - `paths`: A pointer to a pointer to a char array, which will be allocated and filled with the resolved paths.
    - `n`: A pointer to an unsigned integer that will store the number of resolved paths.
    - `ignore_errors`: An integer flag indicating whether to ignore errors during path resolution (non-zero to ignore errors).
- **Control Flow**:
    - Initialize `paths` to NULL and `n` to 0.
    - Duplicate the input string `s` to `copy` and `tmp` for manipulation.
    - Iterate over each path in `tmp` using [`strsep`](compat/strsep.c.driver.md#strsep) to split by colon.
    - For each path, call [`expand_path`](#expand_path) to handle home directory and environment variable expansion.
    - If [`expand_path`](#expand_path) returns NULL, log an invalid path message and continue to the next path.
    - Attempt to resolve the expanded path to an absolute path using `realpath`.
    - If `realpath` fails and `ignore_errors` is set, log the error and continue; otherwise, use the expanded path as is.
    - Check if the resolved path is already in the `paths` array to avoid duplicates.
    - If the path is unique, reallocate the `paths` array to accommodate the new path and add it to the array.
    - Free the duplicated string `copy` after processing all paths.
- **Output**:
    - The function outputs a dynamically allocated array of unique, resolved paths through the `paths` parameter and updates the count of paths in `n`.
- **Functions called**:
    - [`find_home`](#find_home)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`expand_path`](#expand_path)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### make_label<!-- {{#callable:make_label}} -->
The `make_label` function constructs a file path for a tmux socket directory based on a given label and user ID, ensuring the directory exists and has safe permissions.
- **Inputs**:
    - `label`: A string representing the label for the socket directory; if NULL, defaults to "default".
    - `cause`: A pointer to a string that will be set with an error message if the function fails.
- **Control Flow**:
    - Initialize the `cause` pointer to NULL and set `label` to "default" if it is NULL.
    - Retrieve the current user ID using `getuid()`.
    - Call [`expand_paths`](#expand_paths) to get possible socket paths and store them in `paths`; if no paths are found, set an error message in `cause` and return NULL.
    - Use the first path from `paths` as the base path and free the rest.
    - Construct a directory path using the base path and user ID, then attempt to create the directory with `mkdir`; if it fails and the error is not `EEXIST`, set an error message and go to the fail label.
    - Check if the directory exists and is a directory using `lstat`; if not, set an error message and go to the fail label.
    - Verify the directory's ownership and permissions; if they are unsafe, set an error message and go to the fail label.
    - Construct the final path by appending the label to the base path, free the base path, and return the constructed path.
    - In the fail label, free the base path and return NULL.
- **Output**:
    - Returns a string representing the constructed path for the tmux socket directory, or NULL if an error occurs, with `cause` set to an error message.
- **Functions called**:
    - [`expand_paths`](#expand_paths)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### shell_argv0<!-- {{#callable:shell_argv0}} -->
The `shell_argv0` function constructs a string representing the shell's name, optionally prefixed with a dash if it is a login shell.
- **Inputs**:
    - `shell`: A constant character pointer representing the full path to the shell executable.
    - `is_login`: An integer flag indicating whether the shell is a login shell (non-zero) or not (zero).
- **Control Flow**:
    - Find the last occurrence of '/' in the shell path to determine the shell's name.
    - If a '/' is found and it is not the last character, set the name to the substring after the last '/'.
    - If no '/' is found, set the name to the entire shell path.
    - Use [`xasprintf`](xmalloc.c.driver.md#xasprintf) to format the shell name into `argv0`, prefixing with '-' if `is_login` is true.
    - Return the constructed `argv0` string.
- **Output**:
    - Returns a dynamically allocated string representing the shell's name, prefixed with '-' if it is a login shell.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


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
    - If the `state` is non-zero, modify the mode to remove `O_NONBLOCK` to set the file descriptor to blocking mode.
    - Set the modified file status flags back to the file descriptor using `fcntl` with `F_SETFL`.
- **Output**:
    - The function does not return a value; it modifies the blocking mode of the specified file descriptor.


---
### get_timer<!-- {{#callable:get_timer}} -->
The `get_timer` function retrieves the current time in milliseconds using a monotonic clock, falling back to the real-time clock if necessary.
- **Inputs**:
    - None
- **Control Flow**:
    - Declare a `timespec` structure `ts` to hold the time values.
    - Attempt to get the current time using [`clock_gettime`](compat/clock_gettime.c.driver.md#clock_gettime) with `CLOCK_MONOTONIC` to fill `ts`.
    - If [`clock_gettime`](compat/clock_gettime.c.driver.md#clock_gettime) with `CLOCK_MONOTONIC` fails, attempt to get the time using `CLOCK_REALTIME`.
    - Calculate the time in milliseconds by converting seconds to milliseconds and adding the nanoseconds converted to milliseconds.
    - Return the calculated time in milliseconds as a `uint64_t`.
- **Output**:
    - The function returns the current time in milliseconds as a `uint64_t` value.
- **Functions called**:
    - [`clock_gettime`](compat/clock_gettime.c.driver.md#clock_gettime)


---
### sig2name<!-- {{#callable:sig2name}} -->
The `sig2name` function returns the name of a signal given its signal number, or the signal number as a string if the name is not available.
- **Inputs**:
    - `signo`: An integer representing the signal number for which the name is to be retrieved.
- **Control Flow**:
    - Declare a static character array `s` of size 11 to store the signal name or number as a string.
    - Check if the macro `HAVE_SYS_SIGNAME` is defined, indicating the presence of the `sys_signame` array.
    - If `HAVE_SYS_SIGNAME` is defined and `signo` is greater than 0 and less than `NSIG`, return the signal name from `sys_signame[signo]`.
    - If the above condition is not met, format the signal number as a string using [`xsnprintf`](xmalloc.c.driver.md#xsnprintf) and store it in `s`.
    - Return the string `s`.
- **Output**:
    - The function returns a pointer to a string containing the signal name if available, or the signal number as a string if not.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


---
### find_cwd<!-- {{#callable:find_cwd}} -->
The `find_cwd` function determines the current working directory, preferring the `PWD` environment variable if it matches the actual directory path, to maintain symbolic links.
- **Inputs**:
    - None
- **Control Flow**:
    - Declare two arrays `resolved1` and `resolved2` to store resolved paths, and a static array `cwd` to store the current working directory path.
    - Retrieve the current working directory using `getcwd` and store it in `cwd`; return `NULL` if it fails.
    - Get the `PWD` environment variable; if it is `NULL` or empty, return `cwd`.
    - Resolve the `PWD` path to `resolved1`; if it fails, return `cwd`.
    - Resolve the `cwd` path to `resolved2`; if it fails, return `cwd`.
    - Compare `resolved1` and `resolved2`; if they are not equal, return `cwd`.
    - If all checks pass, return `pwd`.
- **Output**:
    - The function returns a pointer to a string representing the current working directory, either from `PWD` or the actual directory path.


---
### find_home<!-- {{#callable:find_home}} -->
The `find_home` function retrieves the home directory path for the current user, either from the `HOME` environment variable or from the user's password entry if the environment variable is not set.
- **Inputs**:
    - None
- **Control Flow**:
    - Check if the static variable `home` is already set; if so, return it immediately.
    - Attempt to retrieve the home directory from the `HOME` environment variable.
    - If the `HOME` environment variable is not set or is empty, use `getpwuid` with the current user's UID to get the password entry and retrieve the home directory from there.
    - If the password entry is not found, set `home` to `NULL`.
    - Return the value of `home`.
- **Output**:
    - The function returns a pointer to a string containing the path to the user's home directory, or `NULL` if it cannot be determined.


---
### getversion<!-- {{#callable:getversion}} -->
The `getversion` function returns the current version of the TMUX software as a string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function directly returns the value of the `TMUX_VERSION` macro or constant.
    - There are no conditional statements or loops; it is a straightforward return statement.
- **Output**:
    - The function outputs a constant character pointer to the version string of TMUX.


---
### main<!-- {{#callable:main}} -->
The `main` function initializes and configures the tmux client environment, processes command-line options, and then delegates control to the client main function.
- **Inputs**:
    - `argc`: The number of command-line arguments passed to the program.
    - `argv`: An array of strings representing the command-line arguments.
- **Control Flow**:
    - Set the locale to UTF-8 and check for valid locale settings, exiting with an error if invalid.
    - Initialize environment variables and set the current working directory in the environment.
    - Parse command-line options using getopt, setting various flags and options based on the provided arguments.
    - Check for invalid combinations of options and arguments, exiting with usage information if necessary.
    - Initialize pseudo-terminal master file descriptor and set security pledges.
    - Determine UTF-8 support based on environment variables and set the CLIENT_UTF8 flag if applicable.
    - Create global options structures and set default options based on the options table.
    - Set the default shell and editor options based on environment variables.
    - Determine the socket path from command-line options, environment variables, or default settings, and handle errors if the path cannot be determined.
    - Free any allocated memory for labels and paths.
    - Call the client main function with the initialized event system, arguments, flags, and features, and exit with its return value.
- **Output**:
    - The function does not return a value; it exits the program with the status code returned by [`client_main`](client.c.driver.md#client_main).
- **Functions called**:
    - [`errx`](compat/err.c.driver.md#errx)
    - [`environ_create`](environ.c.driver.md#environ_create)
    - [`environ_put`](environ.c.driver.md#environ_put)
    - [`find_cwd`](#find_cwd)
    - [`expand_paths`](#expand_paths)
    - [`tty_add_features`](tty-features.c.driver.md#tty_add_features)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`usage`](#usage)
    - [`getversion`](#getversion)
    - [`log_add_level`](log.c.driver.md#log_add_level)
    - [`getptmfd`](compat/fdforkpty.c.driver.md#getptmfd)
    - [`err`](compat/err.c.driver.md#err)
    - [`strcasestr`](compat/strcasestr.c.driver.md#strcasestr)
    - [`options_create`](options.c.driver.md#options_create)
    - [`options_default`](options.c.driver.md#options_default)
    - [`getshell`](#getshell)
    - [`options_set_number`](options.c.driver.md#options_set_number)
    - [`make_label`](#make_label)
    - [`client_main`](client.c.driver.md#client_main)


