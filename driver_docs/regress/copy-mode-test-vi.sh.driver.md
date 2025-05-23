# Purpose
This shell script is a test suite for verifying the behavior of text navigation and selection commands in the `tmux` terminal multiplexer, specifically in copy mode. It provides narrow functionality focused on testing various `tmux` commands such as `previous-word`, `next-word-end`, and `next-space`, ensuring they behave correctly with different types of text, including single-letter words, indented lines, symbols, and digits. The script is not an executable or a library but rather a specialized test script that automates the setup of a `tmux` session, executes a series of command sequences, and checks the output against expected results. It is designed to be run in a shell environment and is likely part of a larger suite of tests for `tmux` or similar terminal applications.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a string that specifies a list of directories where executable programs are located. It is used by the shell to locate commands that are entered without a full path.
- **Use**: This variable is used to define the directories `/bin` and `/usr/bin` as locations where the shell will search for executable files.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type for the shell environment, which can affect how terminal capabilities are interpreted and utilized by applications.
- **Use**: This variable is used to define the terminal type as 'screen', which is relevant for terminal emulation and compatibility with terminal-based applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: `TEST_TMUX` is a global variable that stores the path to the `tmux` executable. It is initialized by checking if it is unset or empty, and if so, it assigns the resolved absolute path of `../tmux` using `readlink -f`. This ensures that the script uses a specific `tmux` binary for testing purposes.
- **Use**: `TEST_TMUX` is used to construct the `TMUX` command string, which is then used to execute various `tmux` commands for testing.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that holds the command to execute the tmux terminal multiplexer with specific options. It is initialized with the path to the tmux executable, followed by options to use a null configuration file and a test session name.
- **Use**: This variable is used to execute various tmux commands throughout the script, such as starting a new session, setting options, and sending keys.


