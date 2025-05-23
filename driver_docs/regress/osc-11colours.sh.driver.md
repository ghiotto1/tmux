# Purpose
This script is a shell script designed to test the color handling capabilities of the `tmux` terminal multiplexer. It sets up a `tmux` session and executes a series of tests to verify that different color codes are correctly interpreted and displayed by `tmux`. The script uses ANSI escape sequences to set the background color of a `tmux` pane and then checks if the color is displayed as expected. If the displayed color does not match the expected value, the script exits with an error.

The script begins by setting up the environment variables and paths necessary for `tmux` to run. It then initializes a `tmux` session with a specific server name to avoid conflicts with other sessions. The `do_test` function is the core of the script, where it splits a `tmux` window, applies a color using an escape sequence, and checks the resulting background color against the expected value. This function is called multiple times with different color codes, including RGB, CMY, CMYK, and named colors, to ensure comprehensive testing of `tmux`'s color handling.

The script is a collection of test cases focused on verifying the accuracy of color representation in `tmux`. It does not define public APIs or external interfaces but serves as a utility for developers to ensure that `tmux` correctly interprets and displays a wide range of color specifications. The script concludes by cleaning up the `tmux` server to ensure no lingering processes remain after the tests are completed.
# Global Variables

---
### PATH
- **Type**: `string`
- **Description**: The `PATH` variable is a global environment variable that specifies a list of directories where executable programs are located. It is used by the shell to locate commands that are entered without a full path.
- **Use**: The `PATH` variable is used to determine the directories that the shell searches for executable files when a command is run.


---
### TERM
- **Type**: `string`
- **Description**: The `TERM` variable is a global environment variable set to the value 'screen'. It is used to specify the type of terminal to emulate when running a shell session.
- **Use**: This variable is used to define the terminal type for the shell environment, which can affect how terminal capabilities are interpreted and utilized by applications.


---
### TEST_TMUX
- **Type**: `string`
- **Description**: The `TEST_TMUX` variable is a global string variable that holds the path to the `tmux` executable. If `TEST_TMUX` is not already set, it is initialized to the absolute path of the `tmux` executable located one directory up from the current script's location.
- **Use**: `TEST_TMUX` is used to construct the `TMUX` command with a specific socket name for testing purposes.


---
### TMUX
- **Type**: `string`
- **Description**: The `TMUX` variable is a string that holds the command to execute the tmux session with a specific socket name, 'test'. It is constructed by appending '-Ltest' to the path of the tmux executable, which is determined by the `TEST_TMUX` variable.
- **Use**: This variable is used to execute various tmux commands throughout the script, such as starting a new session, splitting windows, and setting options.


# Functions

---
### do_test
The `do_test` function tests if a given color escape sequence changes the tmux pane background to the expected color.
- **Inputs**:
    - `$1`: The escape sequence string that sets the pane background color.
    - `$2`: The expected color value in hexadecimal format.
- **Control Flow**:
    - The function uses `tmux splitw` to create a new pane and execute a command that prints the escape sequence `$1`.
    - It waits for 0.25 seconds to allow the command to take effect.
    - The current background color of the pane is retrieved using `tmux display -p '#{pane_bg}'`.
    - The pane is then closed using `tmux kill-pane`.
    - The retrieved color is compared to the expected color `$2`.
    - If the colors do not match, the function returns 1, indicating failure.
    - If the colors match, the function returns 0, indicating success.
- **Output**:
    - The function returns 0 if the pane background color matches the expected color, otherwise it returns 1.


