# Purpose
This shell script is a test script designed to verify the behavior of the `tmux` terminal multiplexer, specifically focusing on the control mode and window size adjustments. The script automates the process of starting a `tmux` session, executing commands to check and refresh window dimensions, and validating the results against expected output. It uses temporary files to capture and compare the output of `tmux` commands, ensuring that the window size changes as expected when using the `refresh-client -C` command and when specifying dimensions with `-x` and `-y` options. The script is intended for testing purposes and is not a standalone executable or library, but rather a utility to ensure the correct functionality of specific `tmux` features.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate commands that are entered without a full path.
- **Use**: This variable is used to define the directories that the shell searches for executable files when a command is run.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type that the shell or application should emulate. In this script, it is likely used to ensure compatibility with the tmux terminal multiplexer, which can emulate a 'screen' terminal.
- **Use**: The `TERM` variable is used to define the terminal type for the script's execution environment, ensuring compatibility with terminal-based applications like tmux.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the absolute path to the `tmux` executable. It is initialized by resolving the relative path `../tmux` to its absolute path using the `readlink -f` command. If `TEST_TMUX` is not already set, it will be assigned this resolved path.
- **Use**: This variable is used to construct the `TMUX` command with a specific server name (`-Ltest`) for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that holds the command to execute the tmux program with specific options. It is initialized with the path to the tmux executable, which is determined by the `TEST_TMUX` variable, and includes the `-Ltest` option to specify a tmux server name.
- **Use**: This variable is used to execute various tmux commands throughout the script, such as starting a new session, refreshing the client, and killing the server.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a global variable that stores the path to a temporary file created by the `mktemp` command. This file is used to capture the output of tmux commands executed in control mode.
- **Use**: `TMP` is used to store and manage temporary data output from tmux commands for comparison and validation purposes in the script.


---
### OUT
- **Type**: `string`
- **Description**: The `OUT` variable is a global variable that stores the path to a temporary file created using the `mktemp` command. This file is used to store the output of various `tmux` commands executed in the script.
- **Use**: `OUT` is used to capture and store the output of `tmux` commands for comparison and validation purposes within the script.


