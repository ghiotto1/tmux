# Purpose
This shell script is a test script designed to verify the behavior of the `tmux` terminal multiplexer, specifically focusing on how it handles output redirection with and without the `-t` option. The script sets up the environment by defining the `PATH` and `TERM` variables and determining the path to the `tmux` executable. It then creates a temporary file to capture output and sets up a trap to ensure the file is deleted upon script exit. The script runs two `tmux` sessions: one without the `-t` option, expecting the output "foo" to be written to the temporary file, and another with the `-t` option, expecting no output. It also checks if the `tmux` pane is in "view-mode" before cleaning up by killing the `tmux` server. This script provides narrow functionality, specifically for testing and validating certain behaviors of `tmux`.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate commands that are entered without a full path.
- **Use**: This variable is used to define the directories `/bin` and `/usr/bin` as locations where the shell should look for executable files.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable that specifies the type of terminal to emulate. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: This variable is used to define the terminal type for the shell environment, affecting how terminal capabilities are interpreted and utilized by applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the absolute path to the `tmux` executable. It is initialized by using the `readlink` command to resolve the path to the `tmux` executable located one directory up from the current script's location.
- **Use**: This variable is used to construct the `TMUX` command with a specific socket name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that holds the command to execute the tmux program with specific options. It is constructed by appending the string '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to execute various tmux commands throughout the script, such as starting a new session or killing the server.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a global string variable that holds the path to a temporary file created by the `mktemp` command. This file is used to store the output of tmux commands executed in the script.
- **Use**: `TMP` is used to capture and verify the output of tmux commands by redirecting command output to the temporary file it represents.


