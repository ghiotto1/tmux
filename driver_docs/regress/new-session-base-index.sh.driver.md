# Purpose
This shell script is designed to test the functionality of the `tmux` terminal multiplexer, specifically focusing on the `base-index` setting, which determines the starting index for window numbering. The script sets up a temporary configuration file to initialize a `tmux` session with specific base indices for windows, then verifies that the indices are set correctly by comparing the output against expected values. It is a narrow-purpose script, primarily used for testing and validation of `tmux` configurations, rather than being a general-purpose executable or library. The script includes cleanup operations to remove temporary files and terminate any `tmux` server instances it starts, ensuring that the environment is left in a clean state after execution.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate commands that are entered without a full path.
- **Use**: This variable is used to define the directories that the shell searches for executable files.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type for the session, which can affect how terminal capabilities are interpreted by applications.
- **Use**: This variable is used to define the terminal type for the script's execution environment.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that holds the absolute path to the tmux executable. It is initialized by using the `readlink` command to resolve the path of the tmux executable located one directory up from the current script's location.
- **Use**: This variable is used to construct the `TMUX` command with a specific socket name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that stores the command to execute the tmux session with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to execute various tmux commands, such as starting a session, killing the server, and listing windows, with a specific socket name.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a string that holds the path to a temporary file created by the `mktemp` command. This file is used to store a series of tmux commands that are executed later in the script.
- **Use**: `TMP` is used to store and execute tmux configuration commands and to verify the output of the tmux session.


