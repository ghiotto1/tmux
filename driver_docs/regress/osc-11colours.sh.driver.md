# Purpose
This shell script is designed to test the color handling capabilities of the `tmux` terminal multiplexer. It sets up a `tmux` session and executes a series of tests to verify that different color specifications are correctly interpreted and displayed by `tmux`. The script uses ANSI escape codes to set the background color of a `tmux` pane and then checks if the color is displayed as expected. If the displayed color does not match the expected value, the test fails, and the script exits with an error.

The script begins by setting up the environment variables and paths necessary for `tmux` to run. It then initializes a `tmux` session with a specific server name to avoid conflicts with any existing sessions. The `do_test` function is the core of the script, where it splits a `tmux` window, applies a color using an escape sequence, and checks the resulting background color against the expected value. This function is called multiple times with different color specifications, including RGB, CMY, CMYK, and named colors, to ensure comprehensive testing of `tmux`'s color handling.

The script is a collection of tests focused on a single theme: verifying the accuracy of color rendering in `tmux`. It does not define public APIs or external interfaces but serves as an internal testing tool to ensure that `tmux` correctly interprets and displays a wide range of color inputs. The script concludes by cleaning up the `tmux` server to ensure no lingering processes remain after the tests are completed.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate binary files for execution when a command is entered without a full path.
- **Use**: This variable is used to define the directories that the shell searches for executable files.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the type of terminal to emulate when running terminal-based applications. In this script, it is set to 'screen', which is a common terminal type used for terminal multiplexers like tmux.
- **Use**: The `TERM` variable is used to define the terminal type for the session, ensuring compatibility with terminal-based applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that holds the absolute path to the `tmux` executable. It is initialized by using the `readlink` command to resolve the path to `../tmux` if it is not already set.
- **Use**: This variable is used to construct the `TMUX` command with a specific socket name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that stores the command to execute the tmux session with a specific socket name, 'test'. It is constructed by appending the string '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to execute various tmux commands throughout the script, such as starting a new session, splitting windows, and killing panes.


# Functions

---
### do_test
The `do_test` function tests if a given color escape sequence changes the tmux pane background to the expected color.
- **Inputs**:
    - `$1`: A string representing the color escape sequence to be tested.
    - `$2`: A string representing the expected color code of the tmux pane background after applying the escape sequence.
- **Control Flow**:
    - The function uses `tmux splitw` to create a new pane and execute the `printf` command with the color escape sequence `$1`.
    - It waits for 0.25 seconds to allow the color change to take effect.
    - The function captures the current background color of the tmux pane using `tmux display -p '#{pane_bg}'`.
    - The pane is then closed using `tmux kill-pane`.
    - The function checks if the captured color matches the expected color `$2`.
    - If the colors do not match, the function returns 1, indicating failure; otherwise, it returns 0, indicating success.
- **Output**:
    - The function returns 0 if the tmux pane background color matches the expected color, otherwise it returns 1.


