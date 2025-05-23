# Purpose
This shell script is a test suite designed to validate the format string functionality of the `tmux` terminal multiplexer, as described in its manual (tmux(1) FORMATS). The script provides narrow functionality, focusing specifically on testing various format string scenarios, including basic string substitutions, conditional expressions, and logical operations within `tmux`. It is not an executable or a library file but rather a collection of test cases that automate the verification of `tmux`'s format handling capabilities. The script sets up a `tmux` session, executes a series of format tests, and checks the output against expected results, ensuring that `tmux` behaves correctly under different conditions and format string configurations.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to find and execute commands without needing to specify their full path.
- **Use**: This variable is used to define the search path for executable files, allowing the shell to locate and run programs efficiently.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type that the shell or application should emulate or interact with. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: The `TERM` variable is used to define the terminal type for the shell environment, ensuring compatibility with terminal-based applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that holds the path to the tmux executable. It is initialized by checking if the variable is empty and, if so, assigns it the absolute path of the tmux executable located one directory up from the current script's location.
- **Use**: This variable is used to construct the `TMUX` command with a specific socket name for testing purposes.


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
    - Assign the first input argument to the variable `fmt` and the second input argument to the variable `exp`.
    - Execute the `tmux display-message -p` command with the format string `fmt` and store the output in the variable `out`.
    - Compare the output `out` with the expected result `exp`.
    - If the output does not match the expected result, print an error message indicating the failure, showing both the expected and actual outputs, and exit the script with a status of 1.
- **Output**:
    - The function does not return a value but exits the script with a status of 1 if the format test fails, otherwise it continues execution.


---
### test_conditional_with_pane_in_mode
The function `test_conditional_with_pane_in_mode` tests a format string to verify its output based on the `pane_in_mode` condition in tmux.
- **Inputs**:
    - `fmt`: A format string to be tested, which may include conditional expressions based on `pane_in_mode`.
    - `exp_true`: The expected output of the format string when `pane_in_mode` is true.
    - `exp_false`: The expected output of the format string when `pane_in_mode` is false.
- **Control Flow**:
    - Enter copy mode using `$TMUX copy-mode` to set `pane_in_mode` to true.
    - Call `test_format` with the format string and `exp_true` to verify the output when `pane_in_mode` is true.
    - Exit copy mode using `$TMUX send-keys -X cancel` to set `pane_in_mode` to false.
    - Call `test_format` with the format string and `exp_false` to verify the output when `pane_in_mode` is false.
- **Output**:
    - The function does not return a value but will exit with an error message if the format string does not produce the expected results for either condition.


---
### test_conditional_with_session_name
The function `test_conditional_with_session_name` tests a format string to produce different expected results based on the current session name in tmux.
- **Inputs**:
    - `fmt`: A format string to be evaluated.
    - `exp_summer`: The expected result of the format string when the session name is 'Summer'.
    - `exp_winter`: The expected result of the format string when the session name is 'Winter'.
- **Control Flow**:
    - Rename the current tmux session to 'Summer'.
    - Call `test_format` with the format string and `exp_summer` to verify the output matches the expected result for 'Summer'.
    - Rename the current tmux session to 'Winter'.
    - Call `test_format` with the format string and `exp_winter` to verify the output matches the expected result for 'Winter'.
    - Rename the session back to 'Summer' to restore the default state.
- **Output**:
    - The function does not return a value but will exit with an error message if the format string does not produce the expected results for the given session names.


