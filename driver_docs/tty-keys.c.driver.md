# Purpose
This C source code file is part of the `tmux` terminal multiplexer project and is responsible for handling key inputs from the terminal. The file defines a comprehensive system for interpreting various key sequences, including standard, extended, and mouse input sequences, as well as handling terminal-specific features and attributes. The code is structured around a ternary tree data structure to efficiently map input sequences to corresponding key codes, allowing for quick lookup and processing of key events. This functionality is crucial for `tmux` as it needs to accurately interpret user inputs to manage multiple terminal sessions effectively.

The file includes several static functions for adding, finding, and freeing keys in the ternary tree, as well as functions for processing different types of input sequences, such as clipboard data, device attributes, and mouse events. It also defines default key mappings for various terminal types and configurations, ensuring compatibility across different environments. The code is designed to be integrated into the larger `tmux` application, providing a robust interface for handling terminal input, which is essential for the application's ability to manage and switch between multiple terminal sessions seamlessly.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/time.h`
- `netinet/in.h`
- `ctype.h`
- `limits.h`
- `resolv.h`
- `stdlib.h`
- `string.h`
- `termios.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### tty_keys_find1
- **Type**: ``static struct tty_key *``
- **Description**: The `tty_keys_find1` function is a static function that returns a pointer to a `tty_key` structure. It is used to find a specific key in a ternary tree structure based on the input buffer and its length.
- **Use**: This function is used internally to traverse the key tree and locate a specific key sequence.


---
### tty_keys_find
- **Type**: `static struct tty_key *`
- **Description**: The `tty_keys_find` is a static function that returns a pointer to a `struct tty_key`. It is used to look up a key in a ternary tree structure based on input from a terminal. The function takes a `tty` structure, a buffer containing the key sequence, the length of the buffer, and a pointer to a size variable as parameters.
- **Use**: This function is used to find and return a key from the terminal input key tree based on the provided key sequence.


---
### tty_default_raw_keys
- **Type**: `const struct tty_default_key_raw[]`
- **Description**: The `tty_default_raw_keys` is a static constant array of `struct tty_default_key_raw` that defines a set of default raw key mappings for a terminal. Each entry in the array consists of a string representing an escape sequence and a corresponding key code. This array includes mappings for various keys such as application escape, numeric keypad keys, arrow keys, meta arrow keys, xterm keys, rxvt keys, function keys, focus tracking, paste keys, extended keys, and theme reporting.
- **Use**: This variable is used to initialize a base table of supported keys that are looked up and translated into a ternary tree for handling key inputs from the terminal.


---
### tty_default_xterm_keys
- **Type**: ``struct tty_default_key_xterm[]``
- **Description**: The `tty_default_xterm_keys` is a static constant array of `struct tty_default_key_xterm` that defines default key mappings for xterm terminals. Each element in the array consists of a template string representing an escape sequence and a corresponding key code. This array is used to map xterm-specific key sequences to internal key codes.
- **Use**: This variable is used to initialize key mappings for xterm terminals by associating specific escape sequences with key codes.


---
### tty_default_xterm_modifiers
- **Type**: `key_code[]`
- **Description**: The `tty_default_xterm_modifiers` is a static constant array of type `key_code` that defines a set of default modifier key combinations for xterm terminals. Each element in the array represents a combination of modifier keys such as SHIFT, META, CTRL, and IMPLIED_META, which are used to interpret key events in terminal applications.
- **Use**: This array is used to map xterm key events to their corresponding modifier key combinations in terminal applications.


---
### tty_default_code_keys
- **Type**: `static const struct tty_default_key_code[]`
- **Description**: The `tty_default_code_keys` is a static constant array of `struct tty_default_key_code` that maps terminal key codes to internal key codes used by the application. It includes mappings for function keys, arrow keys, and other special keys with various modifier combinations such as Shift, Ctrl, and Meta.
- **Use**: This variable is used to define a base table of supported keys that are looked up in terminfo and translated into a ternary tree for handling key inputs from the terminal.


# Data Structures

---
### tty_key
- **Type**: `struct`
- **Members**:
    - `ch`: Represents a character associated with the key.
    - `key`: Stores the key code associated with the character.
    - `left`: Pointer to the left child node in the ternary tree.
    - `right`: Pointer to the right child node in the ternary tree.
    - `next`: Pointer to the next node in the sequence of keys.
- **Description**: The `tty_key` structure is used to represent a node in a ternary tree for handling key inputs from a terminal. Each node contains a character and its associated key code, as well as pointers to left and right child nodes for tree traversal, and a next pointer for sequential key processing. This structure is part of a system to translate terminal key inputs into a format that can be processed by the software, allowing for efficient key lookup and handling.


---
### tty_default_key_raw
- **Type**: `struct`
- **Members**:
    - `string`: A constant character pointer representing a string associated with a key.
    - `key`: A key_code type representing the key associated with the string.
- **Description**: The `tty_default_key_raw` structure is used to define a mapping between a string representation of a key and its corresponding key code. This structure is part of a larger system for handling key inputs from a terminal, where each entry in the `tty_default_raw_keys` array represents a specific key sequence and its associated action or key code. The `string` member holds the escape sequence or string that represents the key, while the `key` member holds the key code that the system uses to identify the key action.


---
### tty_default_key_xterm
- **Type**: `struct`
- **Members**:
    - `template`: A constant character pointer representing a template string for key sequences.
    - `key`: A key_code type representing the key associated with the template.
- **Description**: The `tty_default_key_xterm` structure is used to define default key mappings for xterm terminals. It consists of a template string that represents the escape sequence for a key and a corresponding key code that identifies the key. This structure is part of a larger system for handling terminal key inputs, allowing the program to interpret and respond to various key sequences sent by xterm-compatible terminals.


---
### tty_default_key_code
- **Type**: `struct`
- **Members**:
    - `code`: An enumeration value of type `tty_code_code` representing a specific terminal key code.
    - `key`: A `key_code` value representing the key associated with the terminal key code.
- **Description**: The `tty_default_key_code` structure is used to map terminal key codes, represented by the `tty_code_code` enum, to specific key codes, represented by the `key_code` type. This mapping is part of a larger system for handling key inputs from a terminal, allowing the program to interpret and respond to various key presses. The structure is used in an array, `tty_default_code_keys`, which serves as a base table for supported keys that are looked up in terminfo and translated into a ternary tree for efficient key handling.


# Functions

---
### tty_keys_add<!-- {{#callable:tty_keys_add}} -->
The `tty_keys_add` function adds a key sequence to a terminal's key tree, either by creating a new entry or updating an existing one.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal to which the key sequence is being added.
    - `s`: A constant character pointer representing the key sequence string to be added.
    - `key`: A `key_code` representing the key code associated with the key sequence.
- **Control Flow**:
    - The function begins by looking up the string representation of the key code using `key_string_lookup_key`.
    - It then attempts to find an existing key entry in the terminal's key tree using [`tty_keys_find`](#tty_keys_find).
    - If no existing entry is found, it logs a debug message indicating a new key is being added and calls [`tty_keys_add1`](#tty_keys_add1) to add the key to the tree.
    - If an existing entry is found, it logs a debug message indicating the key is being replaced and updates the key code of the existing entry.
- **Output**:
    - The function does not return a value; it modifies the terminal's key tree in place.
- **Functions called**:
    - [`tty_keys_find`](#tty_keys_find)
    - [`tty_keys_add1`](#tty_keys_add1)


---
### tty_keys_add1<!-- {{#callable:tty_keys_add1}} -->
The [`tty_keys_add1`](#tty_keys_add1) function recursively adds a key sequence to a ternary tree structure, representing terminal key mappings.
- **Inputs**:
    - `tkp`: A pointer to a pointer of a `tty_key` structure, representing the current node in the ternary tree where the key sequence is being added.
    - `s`: A string representing the key sequence to be added to the tree.
    - `key`: A `key_code` value representing the key code associated with the key sequence.
- **Control Flow**:
    - Check if the current tree node (`*tkp`) is NULL; if so, allocate memory for a new `tty_key` structure and initialize it with the first character of the string `s` and a default key code `KEYC_UNKNOWN`.
    - Compare the current character of the string `s` with the character stored in the current tree node (`tk->ch`).
    - If the characters match, move to the next character in the string `s` and check if it is the end of the string. If it is, set the key code of the current node to `key` and return.
    - If the characters do not match, determine whether to traverse the left or right subtree based on the lexicographical order of the characters.
    - Recursively call [`tty_keys_add1`](#tty_keys_add1) with the appropriate subtree pointer and the remaining string `s`.
- **Output**:
    - The function does not return a value; it modifies the ternary tree structure pointed to by `tkp` to include the new key sequence and its associated key code.
- **Functions called**:
    - [`tty_keys_add1`](#tty_keys_add1)


---
### tty_keys_build<!-- {{#callable:tty_keys_build}} -->
The `tty_keys_build` function initializes a key tree for a terminal by adding default xterm, raw, and code keys, as well as user-defined keys, to the terminal's key tree structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` which represents the terminal for which the key tree is being built.
- **Control Flow**:
    - Check if the terminal's key tree is not NULL, and if so, free the existing key tree using [`tty_keys_free`](#tty_keys_free).
    - Set the terminal's key tree to NULL to start with a clean slate.
    - Iterate over the default xterm keys, and for each key, iterate over the xterm modifiers starting from index 2, construct the key string by replacing '_' with the modifier index, and add the key to the terminal's key tree using [`tty_keys_add`](#tty_keys_add).
    - Iterate over the default raw keys, and for each key, add it to the terminal's key tree if the key string is not empty.
    - Iterate over the default code keys, retrieve the key string using `tty_term_string`, and add it to the terminal's key tree if the key string is not empty.
    - Retrieve the 'user-keys' option from global options, and if it exists, iterate over each user key, retrieve its index and value, and add it to the terminal's key tree with a key code offset by `KEYC_USER`.
- **Output**:
    - The function does not return a value; it modifies the `tty` structure by populating its `key_tree` with the appropriate key mappings.
- **Functions called**:
    - [`tty_keys_free`](#tty_keys_free)
    - [`tty_keys_add`](#tty_keys_add)


---
### tty_keys_free<!-- {{#callable:tty_keys_free}} -->
The `tty_keys_free` function deallocates the memory used by the key tree associated with a given terminal (`tty`).
- **Inputs**:
    - `tty`: A pointer to a `struct tty` which represents a terminal and contains a key tree that needs to be freed.
- **Control Flow**:
    - The function calls [`tty_keys_free1`](#tty_keys_free1) with the `key_tree` member of the `tty` structure as an argument.
    - [`tty_keys_free1`](#tty_keys_free1) is responsible for recursively freeing each node in the key tree.
- **Output**:
    - The function does not return any value; it performs a cleanup operation by freeing memory.
- **Functions called**:
    - [`tty_keys_free1`](#tty_keys_free1)


---
### tty_keys_free1<!-- {{#callable:tty_keys_free1}} -->
The [`tty_keys_free1`](#tty_keys_free1) function recursively frees a ternary tree of `tty_key` structures.
- **Inputs**:
    - `tk`: A pointer to a `tty_key` structure, which is the root of the ternary tree to be freed.
- **Control Flow**:
    - Check if the `next` pointer of the current `tty_key` is not NULL, and if so, recursively call [`tty_keys_free1`](#tty_keys_free1) on it.
    - Check if the `left` pointer of the current `tty_key` is not NULL, and if so, recursively call [`tty_keys_free1`](#tty_keys_free1) on it.
    - Check if the `right` pointer of the current `tty_key` is not NULL, and if so, recursively call [`tty_keys_free1`](#tty_keys_free1) on it.
    - Free the memory allocated for the current `tty_key` structure.
- **Output**:
    - The function does not return a value; it performs memory deallocation for the `tty_key` tree.
- **Functions called**:
    - [`tty_keys_free1`](#tty_keys_free1)


---
### tty_keys_find<!-- {{#callable:tty_keys_find}} -->
The `tty_keys_find` function searches for a key in the terminal's key tree based on a given input buffer and updates the size of the matched key sequence.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal state, which includes the key tree to search.
    - `buf`: A constant character pointer representing the input buffer containing the key sequence to search for.
    - `len`: A size_t value representing the length of the input buffer.
    - `size`: A pointer to a size_t variable where the function will store the size of the matched key sequence.
- **Control Flow**:
    - Initialize the size to 0 to indicate no match initially.
    - Call the helper function [`tty_keys_find1`](#tty_keys_find1) with the terminal's key tree, input buffer, buffer length, and size pointer.
    - Return the result of the [`tty_keys_find1`](#tty_keys_find1) function call, which is a pointer to the matched `tty_key` structure or NULL if no match is found.
- **Output**:
    - Returns a pointer to a `tty_key` structure if a match is found, or NULL if no match is found.
- **Functions called**:
    - [`tty_keys_find1`](#tty_keys_find1)


---
### tty_keys_find1<!-- {{#callable:tty_keys_find1}} -->
The [`tty_keys_find1`](#tty_keys_find1) function recursively searches a ternary tree of `tty_key` nodes to find a match for a given input buffer and updates the size of the matched sequence.
- **Inputs**:
    - `tk`: A pointer to the current `tty_key` node in the ternary tree being searched.
    - `buf`: A pointer to the character buffer containing the input sequence to be matched.
    - `len`: The length of the input buffer `buf`.
    - `size`: A pointer to a `size_t` variable that tracks the number of characters matched so far.
- **Control Flow**:
    - Check if the input length `len` is zero; if so, return `NULL` as no match is possible.
    - Check if the current node `tk` is `NULL`; if so, return `NULL` as it indicates the end of the tree with no match.
    - Compare the character `tk->ch` with the first character of `buf`.
    - If they match, increment the buffer pointer `buf`, decrement `len`, and increment the dereferenced `size`.
    - If the remaining length `len` is zero or the current node `tk` has no `next` node and a known key, return the current node `tk`.
    - If the characters do not match, decide whether to traverse left or right in the tree based on the comparison of `*buf` and `tk->ch`.
    - Recursively call [`tty_keys_find1`](#tty_keys_find1) with the updated node `tk`, buffer `buf`, length `len`, and size `size`.
- **Output**:
    - Returns a pointer to the `tty_key` node that matches the input sequence, or `NULL` if no match is found.
- **Functions called**:
    - [`tty_keys_find1`](#tty_keys_find1)


---
### tty_keys_next1<!-- {{#callable:tty_keys_next1}} -->
The `tty_keys_next1` function processes a buffer of input data to identify and return the next key code, handling known keys and UTF-8 sequences, and determining if more data is needed.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A pointer to a buffer containing the input data to be processed.
    - `len`: The length of the input data in the buffer.
    - `key`: A pointer to a `key_code` variable where the identified key code will be stored.
    - `size`: A pointer to a `size_t` variable where the size of the processed input will be stored.
    - `expired`: An integer flag indicating whether the input processing has expired or timed out.
- **Control Flow**:
    - Log the current state of the input buffer and the expired flag.
    - Attempt to find a known key in the input buffer using [`tty_keys_find`](#tty_keys_find).
    - If a known key is found and it is not expired, return 1 to indicate more data is needed; otherwise, set the key and return 0.
    - If no known key is found, check if the input is a valid UTF-8 sequence using `utf8_open` and `utf8_append`.
    - If the UTF-8 sequence is valid and complete, convert it to a key code, log it, and return 0.
    - If the input is not a known key or valid UTF-8, return -1 to indicate an error or incomplete data.
- **Output**:
    - The function returns an integer: 0 if a complete key is found, 1 if more data is needed, or -1 if the input is invalid or incomplete.
- **Functions called**:
    - [`tty_keys_find`](#tty_keys_find)


---
### tty_keys_winsz<!-- {{#callable:tty_keys_winsz}} -->
The `tty_keys_winsz` function processes terminal escape sequences to update the window size of a terminal based on character or pixel dimensions.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface.
    - `buf`: A constant character pointer to the buffer containing the escape sequence data.
    - `len`: A size_t value representing the length of the buffer.
    - `size`: A pointer to a size_t where the function will store the number of bytes processed from the buffer.
- **Control Flow**:
    - Initialize the `size` to 0.
    - Check if the `TTY_WINSIZEQUERY` flag is set in `tty->flags`; if not, return -1.
    - Verify that the first two bytes of `buf` are '\033['; if not, return -1 or 1 depending on the length.
    - Iterate through `buf` starting from the third byte to find the end of the sequence, stopping at 't' or a non-numeric/non-semicolon character.
    - If the end of the sequence is not found or exceeds buffer size, return -1 or 1.
    - Copy the relevant part of `buf` into a temporary buffer `tmp` and null-terminate it.
    - Attempt to parse `tmp` for window size in characters using `sscanf`; if successful, update the terminal size and return 0.
    - If parsing for characters fails, attempt to parse for window size in pixels; if successful, update the terminal size and invalidate the terminal, then return 0.
    - If neither parsing succeeds, log an unrecognized sequence and return -1.
- **Output**:
    - Returns 0 on successful parsing and updating of window size, -1 on failure, or 1 if more data is needed.


---
### tty_keys_next<!-- {{#callable:tty_keys_next}} -->
The `tty_keys_next` function processes input keys from a terminal, identifying and handling complete or partial key sequences, including special keys and mouse events, and manages key timers for delayed input handling.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` which represents the terminal interface and contains input buffer and state information.
- **Control Flow**:
    - Retrieve the key buffer and its length from the `tty` input buffer.
    - Check if the buffer is empty; if so, return 0 indicating no keys to process.
    - Log the current keys in the buffer for debugging purposes.
    - Check if the input is a clipboard response, device attributes response, extended device attributes response, colors response, mouse event, or extended key press using respective helper functions.
    - If a complete key is found, handle it accordingly and proceed to `complete_key` label.
    - If a partial key is detected, handle it by setting a timer and return 0 to wait for more input.
    - If no complete or partial key is found, attempt to process the input as a regular key or with an escape prefix (meta modifier).
    - Handle special cases like C-Space and backspace using termios settings.
    - Convert control codes to corresponding Ctrl keys if necessary.
    - If a complete key is identified, log it, remove any active key timer, and handle focus events if applicable.
    - Fire the key event by creating a `key_event` structure and passing it to `server_client_handle_key`.
    - Drain the processed data from the input buffer and return 1 indicating a key was processed.
- **Output**:
    - Returns 1 if a key was successfully processed, or 0 if no complete key was found or if waiting for more input.
- **Functions called**:
    - [`tty_keys_clipboard`](#tty_keys_clipboard)
    - [`tty_keys_device_attributes`](#tty_keys_device_attributes)
    - [`tty_keys_device_attributes2`](#tty_keys_device_attributes2)
    - [`tty_keys_extended_device_attributes`](#tty_keys_extended_device_attributes)
    - [`tty_keys_colours`](#tty_keys_colours)
    - [`tty_keys_mouse`](#tty_keys_mouse)
    - [`tty_keys_extended_key`](#tty_keys_extended_key)
    - [`tty_keys_winsz`](#tty_keys_winsz)
    - [`tty_keys_next1`](#tty_keys_next1)


---
### tty_keys_callback<!-- {{#callable:tty_keys_callback}} -->
The `tty_keys_callback` function processes key input events for a terminal by repeatedly calling [`tty_keys_next`](#tty_keys_next) if a timer flag is set.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `data`: A pointer to a `tty` structure, which contains terminal-related data.
- **Control Flow**:
    - The function casts the `data` pointer to a `tty` structure pointer.
    - It checks if the `TTY_TIMER` flag is set in the `tty` structure's `flags`.
    - If the `TTY_TIMER` flag is set, it enters a loop that repeatedly calls [`tty_keys_next`](#tty_keys_next) with the `tty` structure until it returns false.
- **Output**:
    - The function does not return any value; it operates by side effects on the `tty` structure.
- **Functions called**:
    - [`tty_keys_next`](#tty_keys_next)


---
### tty_keys_extended_key<!-- {{#callable:tty_keys_extended_key}} -->
The function `tty_keys_extended_key` processes extended key input sequences from a terminal, parsing them into internal key codes with modifiers.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A pointer to a character buffer containing the input sequence to be processed.
    - `len`: The length of the input buffer.
    - `size`: A pointer to a `size_t` variable where the function will store the size of the processed input sequence.
    - `key`: A pointer to a `key_code` variable where the function will store the parsed key code.
- **Control Flow**:
    - Initialize `*size` to 0.
    - Check if the first two bytes of `buf` are '\033['; if not, return -1.
    - If `len` is 1 or 2, return 1 indicating more data is needed.
    - Search for a terminator in `buf` starting from the third byte, stopping at '~' or a non-numeric/non-';' character.
    - If no terminator is found or the buffer is too large, return -1.
    - Copy the relevant part of `buf` into a temporary buffer `tmp` and null-terminate it.
    - Parse the key and modifiers from `tmp` using `sscanf`, handling two possible formats.
    - Set `*size` to the length of the processed sequence.
    - Determine the key code, checking for backspace and converting UTF-32 codepoints if necessary.
    - Adjust the key code based on the parsed modifiers, applying bitwise operations to set flags for SHIFT, META, CTRL, etc.
    - Handle special cases like converting S-Tab to Backtab and dealing with isolated SHIFT modifiers.
    - Log the processed key if logging is enabled.
    - Store the final key code in `*key` and return 0 indicating success.
- **Output**:
    - Returns 0 on successful parsing of the key, -1 on failure, or 1 if more data is needed to complete the sequence.


---
### tty_keys_mouse<!-- {{#callable:tty_keys_mouse}} -->
The `tty_keys_mouse` function processes mouse input sequences from a terminal and updates the mouse event structure accordingly.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A pointer to a character buffer containing the input data to be processed.
    - `len`: The length of the input buffer.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed.
    - `m`: A pointer to a `mouse_event` structure where the parsed mouse event data will be stored.
- **Control Flow**:
    - Initialize variables for mouse coordinates, button states, and SGR type.
    - Check if the first two bytes of the buffer are '\033['; if not, return -1.
    - Determine the type of mouse sequence based on the third byte: 'M' for standard sequences or '<' for SGR sequences.
    - For standard sequences, read three characters for button, X, and Y positions, adjust them, and log the input.
    - For SGR sequences, parse button, X, and Y positions as decimal numbers separated by semicolons, adjust them, and log the input.
    - Check for invalid mouse input values and return -2 if found.
    - Update the `mouse_event` structure with the parsed values and update the last known mouse state in the `tty` structure.
    - Return 0 to indicate successful processing of the mouse input.
- **Output**:
    - Returns 0 on successful processing of a mouse input sequence, -1 for invalid sequences, 1 if more data is needed, and -2 for invalid mouse input values.


---
### tty_keys_clipboard<!-- {{#callable:tty_keys_clipboard}} -->
The `tty_keys_clipboard` function processes OSC 52 clipboard input sequences, decodes the base64-encoded clipboard data, and forwards it to the appropriate window panes.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A pointer to a character buffer containing the input data to be processed.
    - `len`: The length of the input buffer `buf`.
    - `size`: A pointer to a `size_t` variable where the function will store the size of the processed input.
- **Control Flow**:
    - Initialize `size` to 0.
    - Check if the first five bytes of `buf` match the expected OSC 52 sequence prefix (`\033]52;`).
    - If the prefix is not matched or the buffer length is insufficient, return -1 or 1 accordingly.
    - Search for a terminator in the buffer, which can be either `\007` or `\033\`.
    - If no terminator is found, return 1 indicating partial input.
    - Adjust `size` to include the terminator and skip the initial part of the buffer.
    - Extract the second argument by skipping characters until a semicolon is found.
    - If the second argument is not found or is too short, return 0.
    - Check if the clipboard request was initiated by the terminal (`TTY_OSC52QUERY` flag).
    - If the request was not initiated, return 0.
    - Cancel the clipboard timer and clear the `TTY_OSC52QUERY` flag.
    - Allocate memory and copy the base64-encoded data from the buffer.
    - Decode the base64 data into a new buffer.
    - If decoding fails, free allocated memory and return 0.
    - Log the decoded data and add it to the paste buffer if the client has the `CLIENT_CLIPBOARDBUFFER` flag.
    - Forward the decoded clipboard data to each window pane in the client's clipboard pane list.
    - Free the client's clipboard pane list and reset the pane count.
    - Return 0 indicating successful processing.
- **Output**:
    - Returns 0 on successful processing of the clipboard data, -1 if the input is invalid, or 1 if the input is partial and more data is needed.


---
### tty_keys_device_attributes<!-- {{#callable:tty_keys_device_attributes}} -->
The function `tty_keys_device_attributes` processes primary device attributes from a terminal input buffer and updates terminal features accordingly.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface.
    - `buf`: A constant character pointer to the input buffer containing the device attributes data.
    - `len`: A size_t value representing the length of the input buffer.
    - `size`: A pointer to a size_t where the function will store the number of bytes processed from the buffer.
- **Control Flow**:
    - Initialize local variables and set `*size` to 0.
    - Check if the terminal already has device attributes (`TTY_HAVEDA` flag) and return -1 if true.
    - Verify the first three bytes of `buf` are '\033[?' and return -1 if not, or 1 if the buffer is too short.
    - Copy characters from `buf` into `tmp` until 'c' is found or the buffer ends, updating `*size` accordingly.
    - Convert the semicolon-separated numbers in `tmp` to integers stored in `p`.
    - Switch on the first number in `p` to determine the terminal level and add features based on subsequent numbers.
    - Log the received primary device attributes and update the terminal features.
    - Set the `TTY_HAVEDA` flag on the `tty` structure and return 0.
- **Output**:
    - Returns 0 on successful processing of device attributes, -1 if the input is invalid or already processed, and 1 if more data is needed.


---
### tty_keys_device_attributes2<!-- {{#callable:tty_keys_device_attributes2}} -->
The function `tty_keys_device_attributes2` processes secondary device attributes input from a terminal and updates terminal features accordingly.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `buf`: A constant character pointer to the buffer containing the input data.
    - `len`: The length of the input buffer.
    - `size`: A pointer to a size_t variable where the function will store the size of the processed input.
- **Control Flow**:
    - Initialize local variables and check if the TTY_HAVEDA2 flag is set; if so, return -1 indicating failure.
    - Check if the first three bytes of the buffer are '\033[>' and return -1 if not, or 1 if the buffer is incomplete.
    - Copy the remaining characters up to 'c' into a temporary buffer and set the size of the processed input.
    - Convert the copied arguments into numbers and store them in an array.
    - Based on the first number in the array, update terminal features for specific terminal types like 'mintty', 'tmux', or 'rxvt-unicode'.
    - Log the received secondary device attributes and update the terminal features.
    - Set the TTY_HAVEDA2 flag and return 0 indicating success.
- **Output**:
    - Returns 0 on success, -1 on failure, or 1 if the input is incomplete.


---
### tty_keys_extended_device_attributes<!-- {{#callable:tty_keys_extended_device_attributes}} -->
The function `tty_keys_extended_device_attributes` processes extended device attributes from a terminal and updates terminal features accordingly.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface.
    - `buf`: A constant character pointer to the buffer containing the input data to be processed.
    - `len`: A size_t value representing the length of the buffer.
    - `size`: A pointer to a size_t where the function will store the size of the processed data.
- **Control Flow**:
    - Initialize the `size` to 0 and check if the `TTY_HAVEXDA` flag is set in `tty->flags`; if so, return -1 indicating failure.
    - Verify that the first four bytes of `buf` match the expected sequence '\033P>|'; if not, return -1 or 1 depending on the length of `buf`.
    - Copy the remaining characters from `buf` into a temporary buffer `tmp` until the sequence '\033\' is found or the buffer limit is reached.
    - If the buffer limit is reached without finding the sequence, return -1 indicating failure.
    - Null-terminate the copied string in `tmp` and update `*size` to reflect the number of processed bytes.
    - Check the beginning of `tmp` to identify the terminal type (e.g., 'iTerm2', 'tmux', 'XTerm', 'mintty', 'foot') and call `tty_default_features` to add the corresponding features.
    - Log the received extended device attributes for debugging purposes.
    - Free the existing `term_type` in the client structure and duplicate the new terminal type string from `tmp`.
    - Call `tty_update_features` to update the terminal features based on the new attributes.
    - Set the `TTY_HAVEXDA` flag in `tty->flags` to indicate that extended device attributes have been processed.
    - Return 0 indicating successful processing.
- **Output**:
    - Returns 0 on successful processing of extended device attributes, -1 on failure, or 1 if more data is needed.


---
### tty_keys_colours<!-- {{#callable:tty_keys_colours}} -->
The `tty_keys_colours` function processes a terminal escape sequence to extract and set foreground or background color values.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `buf`: A constant character pointer to the buffer containing the escape sequence.
    - `len`: The length of the buffer.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed.
    - `fg`: A pointer to an integer where the function will store the parsed foreground color value.
    - `bg`: A pointer to an integer where the function will store the parsed background color value.
- **Control Flow**:
    - Initialize `*size` to 0.
    - Check if the first five characters of `buf` match the expected escape sequence prefix `\033]1` followed by `0` or `1` and `;`.
    - If the prefix is not matched or the buffer length is insufficient, return -1 or 1 accordingly.
    - Copy the remaining characters from `buf` into a temporary buffer `tmp` until a terminator (`\033\` or `\007`) is found or the buffer limit is reached.
    - If the buffer limit is reached without finding a terminator, return -1.
    - Null-terminate the `tmp` string appropriately.
    - Set `*size` to the number of bytes processed from `buf`.
    - Parse the color value from `tmp` using `colour_parseX11`.
    - If a valid color is parsed and the prefix indicates foreground (`0`), set `*fg` to the parsed color; otherwise, set `*bg`.
    - Log the parsed color value for debugging purposes.
    - Return 0 to indicate successful processing.
- **Output**:
    - The function returns 0 on success, -1 on failure, or 1 if more data is needed. It also updates `*size` with the number of bytes processed and sets either `*fg` or `*bg` with the parsed color value.


# Function Declarations (Public API)

---
### tty_keys_add1<!-- {{#callable_declaration:tty_keys_add1}} -->
Adds a key sequence to a ternary tree structure.
- **Description**: This function is used to add a key sequence, represented by a string, to a ternary tree structure for key handling. It is typically called when building or updating a key mapping structure. The function expects a pointer to a pointer to a `tty_key` structure, a string representing the key sequence, and a key code. The function will recursively traverse or create nodes in the tree to represent each character in the string. It must be called with a valid pointer to a `tty_key` pointer, and the string must be null-terminated. The function does not handle invalid input values, so care must be taken to ensure inputs are valid.
- **Inputs**:
    - `tkp`: A pointer to a pointer to a `tty_key` structure. This must not be null, and the caller retains ownership. It represents the current position in the ternary tree where the key sequence is to be added.
    - `s`: A null-terminated string representing the key sequence to be added. The string must not be null, and it is expected to be a valid sequence of characters.
    - `key`: A `key_code` representing the key associated with the sequence. It is used to identify the key at the end of the sequence.
- **Output**: None
- **See also**: [`tty_keys_add1`](#tty_keys_add1)  (Implementation)


---
### tty_keys_add<!-- {{#callable_declaration:tty_keys_add}} -->
Adds or updates a key in the terminal's key tree.
- **Description**: This function is used to add a new key or update an existing key in the terminal's key tree. It should be called when a new key sequence needs to be recognized by the terminal or when an existing key sequence needs to be updated with a new key code. The function checks if the key sequence already exists in the tree and either adds a new entry or updates the existing one. It is important to ensure that the `tty` structure is properly initialized before calling this function.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal. Must not be null, and the caller retains ownership.
    - `s`: A C string representing the key sequence to be added or updated. Must not be null.
    - `key`: A `key_code` representing the key code associated with the key sequence. It should be a valid key code.
- **Output**: None
- **See also**: [`tty_keys_add`](#tty_keys_add)  (Implementation)


---
### tty_keys_free1<!-- {{#callable_declaration:tty_keys_free1}} -->
Frees a ternary tree of tty_key structures.
- **Description**: Use this function to recursively free all nodes in a ternary tree of tty_key structures, starting from the specified node. It is typically called to clean up resources when the key tree is no longer needed. Ensure that the pointer to the root node is valid and that the tree has been properly initialized before calling this function. This function does not return any value and does not handle null pointers, so the caller must ensure the input is valid.
- **Inputs**:
    - `tk`: A pointer to the root node of the tty_key ternary tree to be freed. Must not be null, and the tree should be properly initialized.
- **Output**: None
- **See also**: [`tty_keys_free1`](#tty_keys_free1)  (Implementation)


---
### tty_keys_find1<!-- {{#callable_declaration:tty_keys_find1}} -->
Finds a key in a ternary tree based on a given input buffer.
- **Description**: This function searches for a key in a ternary tree structure using a given input buffer and its length. It is used to match a sequence of characters against the keys stored in the tree. The function should be called with a valid tree node, a non-null buffer, and a positive length. It returns the matching key node if found, or NULL if no match is found. The function also updates the size parameter to reflect the number of characters matched. It is important to ensure that the buffer and length accurately represent the data to be matched.
- **Inputs**:
    - `tk`: A pointer to the root node of the ternary tree where the search begins. It must not be null.
    - `buf`: A pointer to the character buffer containing the sequence to match. It must not be null.
    - `len`: The length of the buffer, indicating how many characters to consider for matching. It must be greater than zero.
    - `size`: A pointer to a size_t variable where the function will store the number of characters matched. It must not be null.
- **Output**: Returns a pointer to the matching tty_key node if a match is found, or NULL if no match is found.
- **See also**: [`tty_keys_find1`](#tty_keys_find1)  (Implementation)


---
### tty_keys_find<!-- {{#callable_declaration:tty_keys_find}} -->
Finds a key in the terminal's key tree based on input buffer.
- **Description**: This function searches for a key in the terminal's key tree using the provided input buffer and its length. It is typically used to interpret input sequences from a terminal and match them against known key sequences. The function must be called with a valid `tty` structure that has been properly initialized with a key tree. The `size` parameter is used to return the number of bytes matched in the buffer. This function does not handle null pointers and expects valid input parameters.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal. Must not be null and should be initialized with a key tree.
    - `buf`: A pointer to a character buffer containing the input sequence to be matched. Must not be null.
    - `len`: The length of the input buffer. Must be greater than zero.
    - `size`: A pointer to a `size_t` variable where the function will store the number of bytes matched in the buffer. Must not be null.
- **Output**: Returns a pointer to a `tty_key` structure if a match is found, or NULL if no match is found.
- **See also**: [`tty_keys_find`](#tty_keys_find)  (Implementation)


---
### tty_keys_next1<!-- {{#callable_declaration:tty_keys_next1}} -->
Processes the next key input from the terminal buffer.
- **Description**: This function is used to process the next key input from a terminal buffer, determining if it corresponds to a known key or a valid UTF-8 sequence. It should be called when there is input data to be processed, and it handles both complete and partial key sequences. The function updates the key and size parameters with the identified key code and the number of bytes processed, respectively. It also considers whether the input has expired, affecting how partial sequences are handled.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal state. Must not be null.
    - `buf`: A pointer to a buffer containing the input data to be processed. Must not be null.
    - `len`: The length of the input data in the buffer. Must be greater than zero.
    - `key`: A pointer to a key_code variable where the identified key code will be stored. Must not be null.
    - `size`: A pointer to a size_t variable where the number of bytes processed will be stored. Must not be null.
    - `expired`: An integer indicating whether the input has expired, affecting the handling of partial sequences. Non-zero for expired input.
- **Output**: Returns 0 if a complete key is found, 1 if more data is needed for a partial key, or -1 if the input is invalid or cannot be processed.
- **See also**: [`tty_keys_next1`](#tty_keys_next1)  (Implementation)


---
### tty_keys_callback<!-- {{#callable_declaration:tty_keys_callback}} -->
Process key input events for a terminal.
- **Description**: This function is used to handle key input events from a terminal, specifically when a timer is active. It should be called in response to a key event when the terminal's timer flag is set. The function processes all available key events until none are left, ensuring that any pending key inputs are handled appropriately. It is typically used in environments where key input needs to be processed asynchronously, such as in a terminal multiplexer or similar application.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored.
    - `events`: This parameter is unused and can be ignored.
    - `data`: A pointer to a `tty` structure, which must not be null. The function expects this to be a valid pointer to a terminal structure that contains the necessary state and flags for processing key events.
- **Output**: None
- **See also**: [`tty_keys_callback`](#tty_keys_callback)  (Implementation)


---
### tty_keys_extended_key<!-- {{#callable_declaration:tty_keys_extended_key}} -->
Parses extended key sequences from a terminal input buffer.
- **Description**: This function is used to interpret extended key sequences from a terminal input buffer, converting them into a key code with associated modifiers. It should be called when processing input from a terminal to handle keys that are represented by escape sequences. The function expects the input buffer to start with the escape sequence '\033[' and will parse until it finds a terminator character ('~' or 'u'). It handles both numeric and modifier components of the key sequence, updating the provided size and key parameters accordingly. The function returns different values to indicate success, failure, or the need for more data.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal state. Must not be null.
    - `buf`: A pointer to a character buffer containing the input data. Must not be null and should start with '\033['.
    - `len`: The length of the input buffer. Must be greater than 2 to allow parsing beyond the initial escape sequence.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed from the buffer. Must not be null.
    - `key`: A pointer to a key_code variable where the function will store the parsed key code. Must not be null.
- **Output**: Returns 0 on success, -1 on failure, and 1 if more data is needed to complete parsing.
- **See also**: [`tty_keys_extended_key`](#tty_keys_extended_key)  (Implementation)


---
### tty_keys_mouse<!-- {{#callable_declaration:tty_keys_mouse}} -->
Parses mouse input sequences from a terminal buffer.
- **Description**: This function is used to interpret mouse input sequences received from a terminal, updating the provided mouse event structure with the parsed data. It supports both standard and SGR extended mouse sequences. The function should be called with a buffer containing the mouse input and will return different values based on the parsing result. It is important to ensure that the buffer is not null and has sufficient length to contain a valid mouse sequence. The function updates the mouse event structure with the parsed coordinates and button states, and also updates the last known mouse state in the tty structure.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal state. Must not be null.
    - `buf`: A pointer to a character buffer containing the mouse input sequence. Must not be null.
    - `len`: The length of the buffer. Must be sufficient to contain a valid mouse sequence.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed from the buffer. Must not be null.
    - `m`: A pointer to a mouse_event structure that will be filled with the parsed mouse event data. Must not be null.
- **Output**: Returns 0 on successful parsing, -1 if the input is not a valid mouse sequence, 1 if more data is needed, and -2 if the sequence is invalid or should be ignored.
- **See also**: [`tty_keys_mouse`](#tty_keys_mouse)  (Implementation)


---
### tty_keys_clipboard<!-- {{#callable_declaration:tty_keys_clipboard}} -->
Processes OSC 52 clipboard input from a terminal.
- **Description**: This function handles OSC 52 clipboard input sequences from a terminal, which are used to transfer clipboard data. It should be called when such input is detected. The function checks for the correct sequence format, processes the data if valid, and updates the clipboard buffer. It requires that the TTY_OSC52QUERY flag is set in the tty structure to process the input. If the input is incomplete or invalid, the function returns early without processing.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal interface. Must not be null and should have the TTY_OSC52QUERY flag set to process the input.
    - `buf`: A pointer to a buffer containing the input data. Must not be null and should start with the OSC 52 sequence '\033]52;'.
    - `len`: The length of the input buffer. Must be at least 5 to contain the initial OSC 52 sequence.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed from the input buffer. Must not be null.
- **Output**: Returns 0 on success, -1 if the input is invalid, and 1 if the input is incomplete. Updates the clipboard buffer if successful.
- **See also**: [`tty_keys_clipboard`](#tty_keys_clipboard)  (Implementation)


---
### tty_keys_device_attributes<!-- {{#callable_declaration:tty_keys_device_attributes}} -->
Processes primary device attributes from a terminal input buffer.
- **Description**: This function is used to handle and process primary device attributes received from a terminal, which are typically sent as a response to a device attributes request. It should be called with a buffer containing the terminal's response, and it will parse the response to extract and update terminal features. The function expects the buffer to start with the escape sequence '\033[?' and end with 'c'. It must be called only if the terminal has not already processed device attributes, as indicated by the TTY_HAVEDA flag. The function updates the terminal's feature set and sets the TTY_HAVEDA flag upon successful processing.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal. Must not be null and should be properly initialized before calling this function.
    - `buf`: A pointer to a character buffer containing the terminal's response. The buffer must start with '\033[?' and end with 'c'. Must not be null.
    - `len`: The length of the buffer 'buf'. Must be at least 4 to contain the minimum valid sequence.
    - `size`: A pointer to a 'size_t' where the function will store the number of bytes processed from the buffer. Must not be null.
- **Output**: Returns 0 on successful processing, -1 if the input is invalid or if the device attributes have already been processed, and 1 if more data is needed.
- **See also**: [`tty_keys_device_attributes`](#tty_keys_device_attributes)  (Implementation)


---
### tty_keys_device_attributes2<!-- {{#callable_declaration:tty_keys_device_attributes2}} -->
Processes secondary device attributes from a terminal.
- **Description**: This function is used to handle and process secondary device attributes received from a terminal, which are typically sent in response to a query. It should be called when such a response is expected, and it updates the terminal's feature set based on the received attributes. The function requires that the terminal has not already processed secondary device attributes, as indicated by the TTY_HAVEDA2 flag. It returns specific values to indicate success, failure, or partial data, and updates the size parameter to reflect the number of bytes processed.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal. Must not be null.
    - `buf`: A pointer to a buffer containing the input data. Must not be null and should start with the expected escape sequence for secondary device attributes.
    - `len`: The length of the data in the buffer. Must be greater than zero.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed. Must not be null.
- **Output**: Returns 0 on success, -1 on failure, or 1 if more data is needed. Updates the size parameter with the number of bytes processed.
- **See also**: [`tty_keys_device_attributes2`](#tty_keys_device_attributes2)  (Implementation)


---
### tty_keys_extended_device_attributes<!-- {{#callable_declaration:tty_keys_extended_device_attributes}} -->
Processes extended device attributes from a terminal.
- **Description**: This function is used to process extended device attributes received from a terminal, which are typically part of terminal identification sequences. It should be called when such sequences are detected in the input buffer. The function updates terminal features and the terminal type based on the received attributes. It expects the input buffer to start with a specific sequence and will return immediately if the sequence is incomplete or invalid. The function must not be called if the terminal already has extended device attributes processed, as indicated by the TTY_HAVEXDA flag.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal. Must not be null.
    - `buf`: A pointer to a character buffer containing the input data. Must not be null and should contain the extended device attributes sequence.
    - `len`: The length of the input buffer. Must be greater than zero.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed from the buffer. Must not be null.
- **Output**: Returns 0 on successful processing, -1 if the sequence is invalid or already processed, and 1 if the sequence is incomplete.
- **See also**: [`tty_keys_extended_device_attributes`](#tty_keys_extended_device_attributes)  (Implementation)


