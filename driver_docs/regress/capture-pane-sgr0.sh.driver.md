# Purpose
This shell script is a test script designed to verify the functionality of the `tmux` terminal multiplexer, specifically focusing on the behavior of the `capture-pane` command with respect to color handling after the SGR (Select Graphic Rendition) reset (SGR 0). The script sets up a temporary `tmux` session, executes a series of `printf` commands to output colored text sequences, and captures the output using `tmux`'s `capture-pane` feature. It then compares the captured output against expected results to ensure that colors are correctly handled and preserved. The script is intended to be run in a testing environment and includes cleanup operations to remove temporary files and terminate any test `tmux` sessions. This code provides narrow functionality, focusing specifically on testing a particular aspect of `tmux`'s color handling capabilities.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate commands that are entered without a full path.
- **Use**: This variable is used to define the directories `/bin` and `/usr/bin` as locations where the shell should look for executable files.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable that specifies the terminal type being used. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: This variable is used to define the terminal type for the script's execution environment, affecting how terminal capabilities are interpreted.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the absolute path to the `tmux` executable. It is initialized by using the `readlink` command to resolve the path to the `tmux` executable located one directory up from the current script's location.
- **Use**: This variable is used to construct the `TMUX` command with a specific socket name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that stores the command to execute the tmux program with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to run tmux commands with a specific server socket name for testing purposes.


---
### TMP
- **Type**: `string`
- **Description**: The variable `TMP` is a global variable that stores the path to a temporary file created using the `mktemp` command. This file is used to store the output of the `tmux capturep` command for later comparison.
- **Use**: `TMP` is used to store temporary data for the duration of the script, ensuring that the data is cleaned up upon script exit or interruption.


