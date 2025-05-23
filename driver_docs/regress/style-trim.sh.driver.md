# Purpose
This shell script is designed to test the functionality of the `tmux` terminal multiplexer by setting up and manipulating various environment variables and configurations within `tmux` sessions. It provides narrow functionality, specifically focused on verifying the behavior of `tmux` with different environment variable settings and ensuring that the output matches expected results. The script checks the display and formatting of variables within `tmux` sessions, using a series of assertions to validate the output against predefined expectations. It is not an executable or a library file intended for import but rather a standalone test script that automates the process of starting `tmux` sessions, configuring them, and checking their behavior. The script is useful for developers or testers who need to ensure that `tmux` behaves correctly under various conditions, particularly in terms of environment variable handling and display formatting.
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
    - The function sleeps for 1 second to allow the tmux status to update.
    - It captures the output of the tmux2 capture command, processes it to remove escape sequences, and stores the result in variable r.
    - The function compares the captured values v and r with the expected values $2 and $3 respectively.
    - If either of the comparisons fails, it prints an error message indicating the mismatch and exits with status 1.
- **Output**:
    - The function does not return a value but exits with status 1 and prints an error message if the actual and expected outputs do not match.


