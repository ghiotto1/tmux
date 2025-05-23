# Purpose
This shell script is designed to automate testing for the `tmux` terminal multiplexer, specifically focusing on verifying the behavior of environment variables and status formatting within `tmux` sessions. The script sets up two `tmux` servers, `TMUX` and `TMUX2`, and uses them to create isolated testing environments. It checks the output of various `tmux` commands to ensure that environment variables and status formats are correctly interpreted and displayed. The script includes logic to handle different shell environments, preferring `bash` if available, and it disables command history to prevent interference with user settings. Overall, this script provides narrow functionality, serving as a specialized testing tool for `tmux` configurations and behaviors.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate commands that are entered without a full path.
- **Use**: The `PATH` variable is used by the shell to search for executable files in the specified directories when a command is run.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the terminal type that the shell or application should emulate or interact with. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: The `TERM` variable is used to define the terminal type for the shell environment, influencing how terminal capabilities are interpreted.


---
### shell
- **Type**: `string`
- **Description**: The `shell` variable is a global string variable that is conditionally assigned the value 'bash --noprofile --norc +o history' if the Bash shell is available on the system. This setup is used to start a plain Bash session without user configuration files and with command history disabled.
- **Use**: The `shell` variable is used to specify the command for starting a Bash session in a controlled environment for testing purposes.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that holds the absolute path to the `tmux` executable. It is initialized by using the `readlink` command to resolve the relative path `../tmux` to an absolute path. If `TEST_TMUX` is not already set, it is assigned this resolved path.
- **Use**: This variable is used to construct the `TMUX` and `TMUX2` command strings for managing tmux sessions in the script.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a global string variable that holds the command to execute the tmux session with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to manage and control a tmux server session for testing purposes.


---
### TMUX2
- **Type**: `string`
- **Description**: The variable `TMUX2` is a global string variable that stores a command to execute the `tmux` program with a specific socket name `-Ltest2`. This command is used to manage a separate tmux server instance for testing purposes.
- **Use**: `TMUX2` is used to execute and manage a separate tmux server instance for testing by running commands like `kill-server` and `capturep`.


# Functions

---
### check
The `check` function verifies that the output of a tmux command matches expected values and prints an error message if it does not.
- **Inputs**:
    - `$1`: A tmux format string to be displayed and set as the status format.
    - `$2`: The expected output of the tmux display command.
    - `$3`: The expected output of the tmux capture command after processing.
- **Control Flow**:
    - The function executes a tmux display command with the format string $1 and stores the result in variable v.
    - It sets the tmux status format to the format string $1.
    - The function waits for 1 second to ensure the status format is applied.
    - It captures the output of the tmux2 capture command, processes it to remove escape sequences, and stores the result in variable r.
    - The function compares the captured values v and r with the expected values $2 and $3 respectively.
    - If either of the comparisons fails, it prints an error message indicating the mismatch and exits with a status of 1.
- **Output**:
    - The function does not return a value but exits with a status of 1 if the checks fail, otherwise it continues execution.


