# Purpose
This shell script is a test script designed to verify the behavior of the `tmux` terminal multiplexer, specifically focusing on the control mode and window size adjustments. It provides narrow functionality, as it is tailored to test specific features of `tmux`, such as the ability to change window size using the `refresh-client -C` command and the `-x` and `-y` options without requiring the `-d` option for control clients. The script sets up a `tmux` session, executes commands to manipulate and check window dimensions, and compares the output against expected results to ensure correctness. It is not an executable or a library file but rather a utility script for automated testing, using temporary files to capture and verify the output of `tmux` commands.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to find and execute commands without needing to specify their full path.
- **Use**: This variable is used to define the search path for command execution in the shell.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type that the shell or application should emulate. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: This variable is used to define the terminal type for the script's execution environment, ensuring compatibility with terminal-based applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the absolute path to the `tmux` executable. It is initialized by resolving the relative path `../tmux` to its absolute path using the `readlink -f` command. If `TEST_TMUX` is not already set, it will be assigned this resolved path.
- **Use**: This variable is used to construct the `TMUX` command with a specific server name (`-Ltest`) for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that stores the command to execute the tmux program with specific options. It is initialized with the path to the tmux executable, followed by the `-Ltest` option, which specifies a unique server name for tmux sessions, allowing for isolated testing environments.
- **Use**: This variable is used to run tmux commands throughout the script, ensuring they are executed in the context of the test server.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a global variable that stores the path to a temporary file created using the `mktemp` command. This file is used to capture the output of tmux commands executed in control mode.
- **Use**: `TMP` is used to store and manage temporary data generated during the execution of the script, ensuring that the data is isolated and can be cleaned up after use.


---
### OUT
- **Type**: `string`
- **Description**: The variable `OUT` is a global variable that holds the path to a temporary file created using the `mktemp` command. This file is used to store output data from the `tmux` commands executed in the script.
- **Use**: `OUT` is used to store and compare the output of `tmux` commands to verify the expected window dimensions.


