# Purpose
This shell script is a test script designed to verify the behavior of the `tmux` terminal multiplexer, specifically focusing on the capture-pane functionality and its handling of color sequences. The script sets up a temporary environment, including a custom `tmux` session, and executes a series of commands to print colored text sequences to a pane. It then captures the output and compares it against expected results to ensure that colors are correctly preserved and reset after the SGR (Select Graphic Rendition) 0 command. The script is narrow in functionality, as it is specifically tailored to test a particular aspect of `tmux`'s behavior, and it is intended to be executed as a standalone test rather than being part of a larger application or library.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a string that specifies a list of directories where executable programs are located. It is used by the shell to find and execute commands without needing to specify their full path.
- **Use**: This variable is used to define the search path for executable files, allowing the shell to locate and run programs specified by the user or script.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable that specifies the terminal type being used. In this script, it is set to 'screen', which indicates that the terminal emulation is compatible with the 'screen' terminal type.
- **Use**: This variable is used to define the terminal type for the script's execution environment, ensuring compatibility with terminal-specific features and behaviors.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the absolute path to the `tmux` executable. It is initialized by using the `readlink` command to resolve the path to the `tmux` executable located one directory up from the current script's location. If `TEST_TMUX` is not already set, it is assigned this resolved path.
- **Use**: This variable is used to construct the `TMUX` command with a specific server name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that stores the command to execute the tmux program with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to run tmux commands with a specific server socket name for testing purposes.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a string that holds the path to a temporary file created by the `mktemp` command. This file is used to store the output of the `tmux capturep` command for later comparison.
- **Use**: This variable is used to store temporary data for the duration of the script, ensuring that the data is cleaned up upon script exit or interruption.


