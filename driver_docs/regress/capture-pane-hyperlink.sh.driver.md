# Purpose
This script is a shell script designed to test the functionality of the `tmux` terminal multiplexer, specifically focusing on the OSC 8 hyperlink feature. It provides narrow functionality as it is tailored to execute specific test cases for `tmux`'s ability to handle OSC 8 hyperlinks correctly. The script sets up a testing environment by defining paths, creating temporary files, and ensuring any previous `tmux` server instances are terminated. It then defines a function `do_test` to run test cases by comparing the output of `tmux`'s capture pane command with expected results. The script is not an executable or a library but rather a test script intended to verify specific behavior in `tmux`.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate binary files for execution when a command is entered without a full path.
- **Use**: This variable is used to define the search path for executable files, allowing the shell to find and execute commands.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable that specifies the terminal type being used. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: This variable is used to define the terminal type for the script's execution environment.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the absolute path to the tmux executable. It is initialized by resolving the relative path '../tmux' to its absolute path using the `readlink -f` command. If `TEST_TMUX` is already set, it retains its value.
- **Use**: This variable is used to construct the `TMUX` command with a specific socket name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that holds the command to execute the tmux program with a specific socket name, 'test'. It is constructed by appending the string '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to run tmux commands with a specific socket name for testing purposes.


---
### TMP
- **Type**: `string`
- **Description**: The variable `TMP` is a global variable that stores the path to a temporary file created by the `mktemp` command. This file is used to store the output of the `tmux capturep` command during the execution of the `do_test` function.
- **Use**: `TMP` is used to temporarily store command output for comparison in test cases.


---
### TMP2
- **Type**: `string`
- **Description**: TMP2 is a global variable that stores the name of a temporary file created using the `mktemp` command. This file is used to store expected output data for comparison during test execution.
- **Use**: TMP2 is used to store expected output data for comparison with actual output in the `do_test` function.


# Functions

---
### do_test
The `do_test` function runs a tmux session to capture and compare output against expected results for testing OSC 8 hyperlinks.
- **Inputs**:
    - `$1`: The string to be printed and captured by the tmux session, representing the test input.
    - `$2`: The expected output string to be compared against the captured output from the tmux session.
- **Control Flow**:
    - Start a new detached tmux session with a null configuration file.
    - Print the input string `$1` within the tmux session and capture the output to a temporary file `$TMP`.
    - Write the expected output string `$2` to another temporary file `$TMP2`.
    - Pause execution for 1 second to ensure the tmux session has time to capture the output.
    - Compare the contents of `$TMP` and `$TMP2` using the `cmp` command.
    - If the files differ, exit with status 1 indicating a test failure.
    - If the files are identical, return 0 indicating a test success.
- **Output**:
    - The function returns 0 if the captured output matches the expected output, otherwise it exits with status 1.


