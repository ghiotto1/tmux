# Purpose
This script is a shell script designed to test key bindings and their corresponding output codes in a terminal multiplexer environment, specifically using `tmux`. The script sets up a `tmux` session and uses a function `assert_key` to send various key combinations to a new `tmux` window, capturing the output to verify that the expected key codes are returned. The script covers a wide range of key inputs, including control keys, function keys, and extended keys, ensuring that each key press results in the correct output code. This is crucial for verifying the correct behavior of key bindings in terminal applications.

The script begins by setting up the environment, including the `PATH` and `TERM` variables, and initializes a `tmux` session with specific parameters. It then defines the `assert_key` function, which is responsible for sending a key to a `tmux` window, capturing the output, and comparing it to the expected result. If the actual output matches the expected output, the test passes; otherwise, it fails, and the script records the failure. The script iterates over a comprehensive list of key combinations, including control keys (e.g., `C-a`, `C-b`), meta keys (e.g., `M-C-a`), and function keys (e.g., `F1`, `F2`), to ensure that all key bindings are functioning as intended.

Additionally, the script includes a section for testing extended keys, which are keys that may have additional modifiers such as Shift, Control, or Meta. The `assert_extended_key` function is used to test these combinations, ensuring that the extended key codes are correctly interpreted by `tmux`. The script concludes by terminating the `tmux` server and exiting with the appropriate status code, indicating whether all tests passed successfully. This script is a critical tool for developers and testers who need to ensure that key bindings in `tmux` are correctly configured and functioning across different environments and configurations.
# Functions

---
### assert_key
The `assert_key` function tests if a given key press in a tmux session produces the expected output code and recursively handles additional key assertions if specified.
- **Inputs**:
    - `key`: The key press to be tested, represented as a string.
    - `expected_code`: The expected output code that should result from the key press, represented as a string.
- **Control Flow**:
    - A new tmux window is created and its identifier is stored in variable W.
    - The specified key is sent to the tmux window, and the script waits for a short period to allow processing.
    - The actual output code from the tmux window is captured and processed to remove any trailing characters after 'EOL'.
    - The tmux window is closed after capturing the output.
    - The actual output code is compared with the expected code.
    - If the codes match and verbosity is enabled, a pass message is printed; otherwise, a fail message is printed and the exit status is set to 1.
    - The function checks for additional key assertions by examining if the next argument is '--', and if so, recursively calls itself with the remaining arguments.
- **Output**:
    - The function does not return a value but prints pass/fail messages and sets an exit status based on the comparison of actual and expected output codes.


---
### assert_extended_key
The `assert_extended_key` function tests various combinations of extended key inputs against expected output patterns in a tmux session.
- **Inputs**:
    - `extended_key`: The name of the extended key to be tested, such as 'F1', 'Up', 'Home', etc.
    - `expected_code_pattern`: The expected output pattern for the extended key, with a placeholder (';_') for the modifier code.
- **Control Flow**:
    - The function takes two arguments: an extended key and an expected code pattern.
    - It uses `printf` and `sed` to replace the placeholder ';_' in the expected code pattern with different modifier codes (2 to 8) to generate expected codes for different key combinations.
    - For each generated expected code, it calls the `assert_key` function with a combination of the extended key and the expected code.
    - The combinations tested include Shift (S-), Meta (M-), Shift-Meta (S-M-), Control (C-), Shift-Control (S-C-), Control-Meta (C-M-), and Shift-Control-Meta (S-C-M-).
- **Output**:
    - The function does not return a value but calls `assert_key` to verify that the actual key codes match the expected codes for various key combinations, potentially affecting the global `exit_status` variable if a mismatch occurs.


