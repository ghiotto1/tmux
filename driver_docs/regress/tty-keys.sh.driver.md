# Purpose
This script is a shell script designed to test key bindings and their corresponding outputs in the `tmux` terminal multiplexer environment. It sets up two `tmux` sessions, `TMUX` and `TMUX2`, and uses them to simulate key presses and verify that the expected key names are correctly recognized and displayed by `tmux`. The script is structured to handle a wide range of key inputs, including control keys, function keys, arrow keys, and extended keys, ensuring that `tmux` correctly interprets these inputs.

The script begins by setting up the environment, including defining the `PATH` and `TERM` variables, and ensuring that any existing `tmux` servers are terminated to avoid conflicts. It then creates a temporary file to store output and sets up a trap to clean up this file upon script exit. The core functionality is provided by the `assert_key` function, which takes a key sequence and an expected key name, sends the key sequence to the `tmux` session, and checks if the output matches the expected key name. If the output does not match, it logs a failure message and sets an exit status to indicate an error.

The script is comprehensive in its coverage of key bindings, testing a wide array of keys and combinations, including numeric keypad keys, arrow keys, function keys, and various control and meta key combinations. This makes it a valuable tool for developers working with `tmux` to ensure that key bindings are correctly configured and functioning as expected. The script concludes by cleaning up the `tmux` sessions and exiting with the appropriate status based on the test results.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable in Unix-like operating systems that specifies a set of directories where executable programs are located. It is used by the shell to locate executable files for commands that are run without specifying a full path.
- **Use**: `PATH` is used to determine the directories the shell searches for executable files when a command is entered.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It specifies the terminal type that the script should emulate or interact with. This is important for ensuring that terminal-based applications behave correctly according to the expected terminal capabilities.
- **Use**: `TERM` is used to define the terminal type for the script, ensuring compatibility with terminal-based applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that holds the path to the `tmux` executable. If `TEST_TMUX` is not already set, it is initialized to the absolute path of the `tmux` executable located one directory up from the current directory.
- **Use**: `TEST_TMUX` is used to construct the `TMUX` and `TMUX2` variables, which are then used to manage `tmux` server instances in the script.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that holds the command to execute the tmux session with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable. This allows for the creation and management of a tmux server instance with a unique identifier.
- **Use**: This variable is used to execute tmux commands targeting a specific tmux server instance identified by the 'test' socket name.


---
### TMUX2
- **Type**: `string`
- **Description**: The `TMUX2` variable is a string that represents a command to run a tmux server with a specific socket name, 'test2'. It is constructed by appending '-Ltest2' to the path of the tmux executable, which is stored in the `TEST_TMUX` variable. This allows for the creation and management of a separate tmux server instance for testing purposes.
- **Use**: `TMUX2` is used to execute commands on a separate tmux server instance for testing.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a string that holds the path to a temporary file created by the `mktemp` command. This file is used to store intermediate data during the execution of the script.
- **Use**: `TMP` is used to store temporary data output from the `tmux` command-prompt for later comparison in the script.


---
### exit_status
- **Type**: `integer`
- **Description**: The `exit_status` variable is a global integer variable initialized to 0. It is used to track the success or failure of the key assertion tests within the script.
- **Use**: The variable is set to 1 if any key assertion test fails, and its value is used as the exit code of the script to indicate overall success or failure.


# Functions

---
### format_string
The `format_string` function formats a given string by wrapping it in either double or single quotes based on the presence of a single quote character in the input.
- **Inputs**:
    - `$1`: The input string to be formatted.
- **Control Flow**:
    - The function checks if the input string contains a single quote character ('), using a case statement.
    - If the input string contains a single quote, the function outputs a string wrapped in double quotes with escaped percent signs ("%%%%").
    - If the input string does not contain a single quote, the function outputs a string wrapped in single quotes with escaped percent signs ('%%%%').
- **Output**:
    - The function outputs a formatted string, either wrapped in double quotes or single quotes, with escaped percent signs.


---
### assert_key
The `assert_key` function tests if a given key sequence produces the expected key name in a tmux session.
- **Inputs**:
    - `keys`: A string representing the key sequence to be sent to the tmux session.
    - `expected_name`: A string representing the expected key name that should be displayed by the tmux session.
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
    - The function does not return a value but prints pass/fail messages and sets the exit status based on the comparison result.


---
### assert_extended_key
The `assert_extended_key` function tests the handling of extended key sequences in a terminal multiplexer environment.
- **Inputs**:
    - `code`: A numeric code representing the key to be tested.
    - `key_name`: A string representing the human-readable name of the key.
- **Control Flow**:
    - The function calls `assert_key` with a key sequence for Control (C) and the given key, formatted as `Escape [<code>;5u`, and expects the output to be `C-<key_name>`.
    - It then calls `assert_key` again with a key sequence for Control-Meta (C-M) and the given key, formatted as `Escape [<code>;7u`, and expects the output to be `C-M-<key_name>`.
- **Output**:
    - The function does not return a value but affects the global `exit_status` variable based on the success or failure of the key assertions.


