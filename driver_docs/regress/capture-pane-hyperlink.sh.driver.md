# Purpose
This shell script is a test script designed to verify the functionality of the OSC 8 hyperlink feature in the `tmux` terminal multiplexer. It provides narrow functionality, focusing specifically on testing the ability of `tmux` to handle OSC 8 hyperlinks correctly. The script sets up a temporary testing environment by creating temporary files and ensuring any existing `tmux` server is terminated. It defines a function `do_test` that runs a `tmux` session with a specific input and compares the output to an expected result, exiting with an error if the comparison fails. The script is intended to be executed directly and is not a library or configuration file, as it performs specific tests and exits based on the results.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate binary files for execution when a command is run without specifying an absolute path.
- **Use**: This variable is used to define the directories `/bin` and `/usr/bin` as locations where the shell should look for executable files.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable that specifies the terminal type being used. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: This variable is used to define the terminal type for the script's execution environment, ensuring compatibility with terminal operations.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the absolute path to the tmux executable. It is initialized by resolving the relative path '../tmux' to its absolute path using the `readlink -f` command. If `TEST_TMUX` is not already set, it will be assigned this resolved path.
- **Use**: This variable is used to construct the `TMUX` command with a specific socket name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that stores the command to execute the tmux program with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to run tmux commands with a specific socket name for testing purposes.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a global variable that stores the path to a temporary file created by the `mktemp` command. This file is used to store the output of the `tmux capturep` command during the execution of the `do_test` function.
- **Use**: `TMP` is used to temporarily store command output for comparison in test cases.


---
### TMP2
- **Type**: `string`
- **Description**: TMP2 is a global variable that stores the path to a temporary file created using the mktemp command. This file is used to store expected output data for comparison during test execution.
- **Use**: TMP2 is used to store expected output data for comparison with actual output in the do_test function.


# Functions

---
### do_test
The `do_test` function runs a tmux session to compare the output of a command with an expected result and exits with an error if they differ.
- **Inputs**:
    - `$1`: The command string to be executed within the tmux session.
    - `$2`: The expected output string to be compared against the actual output from the tmux session.
- **Control Flow**:
    - The function starts a new detached tmux session with a null configuration file.
    - It executes the command string `$1` within the tmux session and captures the output to a temporary file `$TMP`.
    - The expected output string `$2` is written to another temporary file `$TMP2`.
    - The function pauses for 1 second to ensure the tmux command has completed execution.
    - It compares the contents of `$TMP` and `$TMP2` using the `cmp` command.
    - If the files differ, the function exits with status 1, indicating an error.
    - If the files are identical, the function returns 0, indicating success.
- **Output**:
    - The function returns 0 if the actual output matches the expected output, otherwise it exits with status 1.


