# Purpose
This shell script is a test suite designed to validate the functionality of format strings as described in the `tmux(1)` manual, specifically focusing on the format and conditional expressions used within `tmux`. The script provides narrow functionality, primarily aimed at testing various scenarios and edge cases for format string evaluation in `tmux`, such as basic string substitutions, conditional expressions, and logical operations. It includes functions to test specific conditions, like whether a pane is in copy mode or the session name, and uses these to verify the expected output of format strings. The script is not an executable or a library but rather a collection of test cases intended to be run in a shell environment to ensure the correct behavior of `tmux` format strings.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to find and execute commands without needing to specify their full path.
- **Use**: `PATH` is used by the shell to locate executable files for commands entered by the user.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type that the shell or application should emulate.
- **Use**: This variable is used to define the terminal type for the shell environment, which can affect how terminal-based applications behave.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that holds the path to the `tmux` executable. It is initialized by checking if the variable is empty and, if so, assigns it the absolute path of the `tmux` executable located one directory up from the current script's location.
- **Use**: This variable is used to construct the `TMUX` command with a specific server name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that stores the command to execute the tmux application with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to run tmux commands within the script, ensuring they are executed in the context of the 'test' tmux server.


# Functions

---
### test_format
The `test_format` function verifies that a given format string produces the expected output when processed by the `tmux` command.
- **Inputs**:
    - `fmt`: A format string to be evaluated by the `tmux` command.
    - `exp`: The expected output string that should result from evaluating the format string.
- **Control Flow**:
    - Assign the first input argument to the variable `fmt` and the second to `exp`.
    - Execute the `tmux display-message -p` command with the format string `fmt` and capture the output in the variable `out`.
    - Compare the output `out` with the expected result `exp`.
    - If the output does not match the expected result, print an error message indicating the failure, showing both the expected and actual outputs, and exit the script with a status of 1.
- **Output**:
    - The function does not return a value but will exit the script with an error message if the format test fails.


---
### test_conditional_with_pane_in_mode
The function `test_conditional_with_pane_in_mode` tests a format string to verify its output based on the `pane_in_mode` condition in tmux.
- **Inputs**:
    - `format`: A format string to be tested, which may include conditional expressions based on the `pane_in_mode` variable.
    - `exp1`: The expected output of the format string when `pane_in_mode` is true.
    - `exp2`: The expected output of the format string when `pane_in_mode` is false.
- **Control Flow**:
    - The function enters copy mode in tmux using `$TMUX copy-mode`, which sets `pane_in_mode` to true.
    - It calls `test_format` with the provided format and `exp1` to verify the output when `pane_in_mode` is true.
    - The function exits copy mode using `$TMUX send-keys -X cancel`, setting `pane_in_mode` to false.
    - It calls `test_format` again with the provided format and `exp2` to verify the output when `pane_in_mode` is false.
- **Output**:
    - The function does not return a value but will exit with an error message if the format string does not produce the expected results for either condition.


---
### test_conditional_with_session_name
The function `test_conditional_with_session_name` tests a format string to produce different expected results based on the current session name in tmux.
- **Inputs**:
    - `fmt`: A format string to be evaluated in tmux.
    - `exp_summer`: The expected result of the format string when the session name is 'Summer'.
    - `exp_winter`: The expected result of the format string when the session name is 'Winter'.
- **Control Flow**:
    - Rename the current tmux session to 'Summer'.
    - Call `test_format` with the format string and the expected result for 'Summer'.
    - Rename the current tmux session to 'Winter'.
    - Call `test_format` with the format string and the expected result for 'Winter'.
    - Rename the session back to 'Summer' to restore the default state.
- **Output**:
    - The function does not return a value but will exit with an error if the format string does not produce the expected results for the given session names.


