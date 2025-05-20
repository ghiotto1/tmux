# Purpose
This script is a shell script designed to test key bindings and their corresponding outputs in the `tmux` terminal multiplexer. It sets up two `tmux` sessions with different socket names (`-Ltest` and `-Ltest2`) to ensure that the tests do not interfere with any existing `tmux` sessions. The script then uses these sessions to simulate key presses and verify that the expected key names are correctly recognized and displayed by `tmux`.

The script defines a function `assert_key` that takes a key sequence and an expected key name as arguments. It uses `tmux` commands to send the key sequence to a `tmux` session and captures the output. The output is then compared to the expected key name to determine if the test passes or fails. The script includes a comprehensive list of key sequences, including control keys, function keys, arrow keys, and extended keys, to ensure that `tmux` correctly interprets a wide range of inputs.

Overall, this script serves as a test suite for verifying the correct mapping of key sequences to their respective names in `tmux`. It is a specialized tool for developers or maintainers of `tmux` to ensure that key bindings function as expected across different terminal environments and configurations. The script concludes by cleaning up the `tmux` sessions it created, ensuring no residual processes are left running.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate binary files for execution when a command is entered without a full path.
- **Use**: `PATH` is used to define the directories that the shell searches for executable files when a command is run.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type for the shell environment.
- **Use**: This variable is used to define the terminal type as 'screen', which can affect how terminal capabilities are interpreted by applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the path to the `tmux` executable. If `TEST_TMUX` is not already set, it is initialized to the absolute path of `../tmux` using the `readlink -f` command.
- **Use**: This variable is used to construct the `TMUX` and `TMUX2` command strings for managing tmux sessions in the script.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that represents a command to run the tmux terminal multiplexer with a specific session name. It is constructed by appending the string `-Ltest` to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to execute tmux commands with a specific session name for testing purposes.


---
### TMUX2
- **Type**: `string`
- **Description**: The variable `TMUX2` is a string that represents a command to start a tmux server with a specific session name, 'test2'. It is constructed by appending the string '-Ltest2' to the path of the tmux executable, which is stored in the `TEST_TMUX` variable. This allows for the creation and management of a separate tmux session for testing purposes.
- **Use**: `TMUX2` is used to execute tmux commands on a separate test session named 'test2'.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a global variable that holds the path to a temporary file created using the `mktemp` command. This file is used to store intermediate data during the execution of the script.
- **Use**: `TMP` is used to store temporary data output from commands, which is later read or processed by the script.


---
### exit_status
- **Type**: `integer`
- **Description**: The `exit_status` variable is a global integer variable initialized to 0. It is used to track the success or failure of the key assertion tests within the script.
- **Use**: The variable is set to 1 if any key assertion test fails, and its value is used as the exit code of the script.


# Functions

---
### format_string
The `format_string` function formats a given string by wrapping it in either single or double quotes based on the presence of a single quote character.
- **Inputs**:
    - `$1`: The input string to be formatted.
- **Control Flow**:
    - The function uses a case statement to check if the input string contains a single quote character ('').
    - If the input string contains a single quote, the function outputs the string wrapped in double quotes.
    - If the input string does not contain a single quote, the function outputs the string wrapped in single quotes.
- **Output**:
    - The function outputs a formatted string, wrapped in either single or double quotes, depending on the presence of a single quote in the input.


---
### assert_key
The `assert_key` function tests if a given key sequence produces the expected key name in a tmux session.
- **Inputs**:
    - `keys`: A string representing the key sequence to be sent to the tmux session.
    - `expected_name`: A string representing the expected key name that should be displayed by tmux.
    - `...`: Additional arguments that, if present, will recursively call `assert_key` with the remaining arguments.
- **Control Flow**:
    - The function starts by formatting the expected key name using the `format_string` function.
    - It sends a command to the tmux session to display a message with the formatted expected key name, redirecting the output to a temporary file.
    - The function then sends the specified key sequence to the tmux session.
    - It waits for the tmux command to complete and reads the actual key name from the temporary file, removing any whitespace.
    - The function compares the actual key name with the expected key name.
    - If they match, it prints a pass message if verbose mode is enabled; otherwise, it prints a fail message and sets the exit status to 1.
    - If the third argument is '--', the function shifts the arguments and recursively calls itself with the remaining arguments.
- **Output**:
    - The function does not return a value but prints pass/fail messages and modifies the global `exit_status` variable based on the test results.


---
### assert_extended_key
The `assert_extended_key` function tests the handling of extended key sequences in a terminal multiplexer by sending specific key codes and verifying the expected key names.
- **Inputs**:
    - `code`: A numeric code representing the key sequence to be tested.
    - `key_name`: The expected name of the key corresponding to the given code.
- **Control Flow**:
    - The function calls `assert_key` with a key sequence formatted as `Escape [<code>;5u` and expects the key name to be `C-<key_name>`.
    - It then calls `assert_key` again with a key sequence formatted as `Escape [<code>;7u` and expects the key name to be `C-M-<key_name>`.
- **Output**:
    - The function does not return a value but affects the global `exit_status` variable based on the success or failure of the key assertions.


