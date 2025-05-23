# Purpose
This shell script is designed to automate the testing of a `tmux` session management scenario. It provides narrow functionality, specifically for setting up, executing, and verifying the behavior of `tmux` sessions. The script first sets up the environment by defining the `PATH` and `TERM` variables and determining the path to the `tmux` executable. It then creates a temporary file to store a series of `tmux` commands that initiate three new sessions, each running a `sleep` command with different durations. The script uses `tmux` to execute these commands and checks if exactly three sessions are created, indicating successful execution. Finally, it cleans up by killing the `tmux` server and removing the temporary file, ensuring no residual processes or files remain.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. In this script, it is set to include `/bin` and `/usr/bin`, which are common directories for system binaries and user binaries, respectively.
- **Use**: This variable is used by the shell to locate executable files for commands that are run without specifying a full path.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type for the session, which can affect how terminal capabilities are interpreted by applications.
- **Use**: This variable is used to define the terminal type for the session, ensuring compatibility with terminal-based applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that holds the path to the tmux executable. It is initialized by checking if it is unset or empty, and if so, it is assigned the absolute path of the tmux executable located one directory up from the current directory.
- **Use**: This variable is used to construct the `TMUX` command string, which is then used to manage tmux sessions in the script.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that stores the command to execute the tmux program with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to execute tmux commands with a specific socket name for testing purposes.


---
### TMP
- **Type**: `string`
- **Description**: The `TMP` variable is a string that holds the path to a temporary file created by the `mktemp` command. This file is used to store a series of commands that are executed by the `tmux` session.
- **Use**: `TMP` is used to store and manage temporary command data for the `tmux` session, ensuring that the file is deleted upon script exit or interruption.


