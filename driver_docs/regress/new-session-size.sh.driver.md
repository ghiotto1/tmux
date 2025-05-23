# Purpose
This shell script is a test script designed to verify the functionality of the `tmux` terminal multiplexer, specifically ensuring that new sessions are created with the correct window dimensions. It provides narrow functionality focused on testing and validating the behavior of `tmux` when no clients are attached, and when specific dimensions are set. The script sets up the environment, creates temporary files for output comparison, and uses `tmux` commands to create new sessions with default and specified dimensions. It then checks if the actual window dimensions match the expected values, indicating whether the `tmux` session was correctly initialized. This script is not an executable or a library but rather a utility for automated testing of `tmux` configurations.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to find and execute commands without needing to specify their full path.
- **Use**: This variable is used to define the directories `/bin` and `/usr/bin` as locations where the shell should look for executable files.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It specifies the terminal type that the shell script should emulate or interact with.
- **Use**: This variable is used to define the terminal type for the script's execution environment, ensuring compatibility with terminal-based applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the absolute path to the `tmux` executable. It is initialized by using the `readlink` command to resolve the path to the `tmux` executable located one directory above the current script's location. If `TEST_TMUX` is already set, it retains its value.
- **Use**: This variable is used to construct the `TMUX` command with a specific server name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that holds the command to execute the tmux program with a specific socket name, 'test'. It is constructed by appending the string '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable. This allows for the creation and management of tmux sessions in a controlled test environment.
- **Use**: This variable is used to execute tmux commands with a specific socket name for testing purposes.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a string that holds the path to a temporary file created by the `mktemp` command. This file is used to store the output of the `tmux ls` command, which lists the dimensions of the tmux window.
- **Use**: `TMP` is used to store and later compare the dimensions of a tmux window to ensure they match expected values.


