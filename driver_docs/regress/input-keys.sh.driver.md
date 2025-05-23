# Purpose
This script is a shell script designed to test key bindings and their corresponding output codes in the `tmux` terminal multiplexer. The script sets up a `tmux` session with specific configurations and then uses a function, `assert_key`, to send various key combinations to a new `tmux` window. It captures the output to verify that the key press results in the expected control code. The script covers a wide range of key inputs, including control keys, function keys, and extended keys, ensuring that each key press produces the correct output sequence.

The script begins by setting up the environment variables and initializing a `tmux` session. It then defines the `assert_key` function, which is responsible for sending a key to a `tmux` window, capturing the output, and comparing it to the expected result. If the actual output matches the expected output, the test passes; otherwise, it fails, and the script records the failure. The script includes a comprehensive list of key combinations, including control keys (e.g., `C-a`), meta keys (e.g., `M-a`), and function keys (e.g., `F1`), each with its expected output.

Additionally, the script includes a section for testing extended keys, which are keys that can have multiple modifiers (e.g., Shift, Control, Meta). The `assert_extended_key` function is used to test these keys by generating the expected output pattern for each combination of modifiers. The script concludes by terminating the `tmux` server and exiting with a status code that reflects the success or failure of the tests. This script is a utility for developers to ensure that `tmux` key bindings are correctly configured and functioning as expected.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to find and execute commands without needing to specify their full path. In this script, `PATH` is set to include `/bin` and `/usr/bin`, which are common directories for system binaries.
- **Use**: `PATH` is used to determine the directories the shell searches for executable files when a command is run.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It specifies the terminal type that the shell script should emulate or interact with. This is important for ensuring that terminal-based applications behave correctly according to the expected terminal capabilities.
- **Use**: The `TERM` variable is used to define the terminal type for the script's execution environment, affecting how terminal-based applications interpret input and output.


---
### exit_status
- **Type**: `integer`
- **Description**: The `exit_status` variable is a global integer variable initialized to 0. It is used to track the success or failure of the key assertion tests within the script.
- **Use**: This variable is updated to 1 if any key assertion test fails, and its value is used as the exit code of the script to indicate overall success or failure.


# Functions

---
### assert_key
The `assert_key` function tests if a given key press in a tmux session produces the expected output code.
- **Inputs**:
    - `key`: The key press to be tested, represented as a string.
    - `expected_code`: The expected output code that should result from the key press, represented as a string.
- **Control Flow**:
    - A new tmux window is created and its identifier is stored in variable `W`.
    - The specified `key` is sent to the tmux window `W`.
    - The function captures the output from the tmux window and extracts the first line, removing any trailing 'EOL' characters, storing it in `actual_code`.
    - The tmux window `W` is then closed.
    - The function compares `actual_code` with `expected_code`.
    - If they match, and if the `VERBOSE` environment variable is set, a pass message is printed.
    - If they do not match, a fail message is printed and the `exit_status` is set to 1.
    - The function checks if there are additional arguments after a '--' separator and recursively calls `assert_key` with the remaining arguments if present.
- **Output**:
    - The function does not return a value but prints a pass or fail message and modifies the `exit_status` variable based on the comparison result.


---
### assert_extended_key
The `assert_extended_key` function tests various combinations of extended key inputs against expected output patterns in a tmux session.
- **Inputs**:
    - `extended_key`: The name of the extended key to be tested, such as 'F1', 'Up', 'Home', etc.
    - `expected_code_pattern`: The expected output pattern for the extended key, with a placeholder (';_') for the modifier code.
- **Control Flow**:
    - The function takes an extended key and an expected code pattern as inputs.
    - It modifies the expected code pattern by replacing the placeholder ';_' with different modifier codes (2 to 8) to represent Shift, Meta, Control, and their combinations.
    - For each modified expected code, it calls the `assert_key` function with the corresponding key combination (e.g., 'S-', 'M-', 'C-', etc.) and the modified expected code.
    - The `assert_key` function checks if the actual output from the tmux session matches the expected code and reports pass or fail.
- **Output**:
    - The function does not return a value but affects the global `exit_status` variable, which indicates if any key assertion failed.


