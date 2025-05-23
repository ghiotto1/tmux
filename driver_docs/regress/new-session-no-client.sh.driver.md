# Purpose
This shell script is designed to test the functionality of the `tmux` terminal multiplexer, specifically focusing on creating a new session without a client, which should imply the `-d` (detached) option. The script sets up the environment by defining the `PATH` and `TERM` variables and determines the path to the `tmux` executable. It then creates a temporary configuration file to initiate a new `tmux` session named "test" in a detached state. The script checks if the session was successfully created and subsequently cleans up by killing the `tmux` server and removing the temporary file. This script provides narrow functionality, serving as a test utility for verifying specific behavior of `tmux` sessions.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to find and execute commands without needing to specify their full path.
- **Use**: This variable is used to define the search path for command execution in the shell.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type that the shell or programs should emulate. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: This variable is used to define the terminal type for the shell environment, ensuring compatibility with terminal-based applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that stores the absolute path to the `tmux` executable. It is initialized by using the `readlink` command to resolve the path to the `tmux` executable located one directory up from the current directory.
- **Use**: This variable is used to construct the `TMUX` command with a specific socket name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that holds the command to execute the tmux program with specific options. It is constructed by appending the string '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to execute tmux commands with a specific server name for testing purposes.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a string that holds the path to a temporary file created by the `mktemp` command. This file is used to store a configuration script for the `tmux` command.
- **Use**: This variable is used to store the path of a temporary file that is created and later removed, ensuring that the configuration script is executed and then cleaned up.


