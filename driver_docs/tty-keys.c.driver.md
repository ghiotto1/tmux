# Purpose
This C source code file is part of the `tmux` terminal multiplexer project and is responsible for handling key inputs from the terminal. The file defines a comprehensive system for interpreting various key sequences, including standard, extended, and mouse input sequences, as well as handling terminal-specific features and attributes. The code is structured around a ternary tree data structure to efficiently manage and lookup key sequences, which are defined in several static arrays representing different types of keys, such as raw keys, xterm keys, and terminfo keys. The file also includes functions for building, adding, and freeing the key tree, as well as processing input sequences to determine the appropriate key actions.

The file provides narrow functionality focused on key input handling, which is a critical component of `tmux` as it allows the program to interpret user commands and interactions. It includes functions for processing various terminal escape sequences, such as those for clipboard operations, device attributes, and mouse events. The code is designed to be integrated into the larger `tmux` application, interacting with other components like the client and session management systems. It does not define public APIs or external interfaces directly but rather serves as an internal mechanism for managing terminal input within the `tmux` environment.
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
- **Type**: `static struct tty_key *`
- **Description**: The `tty_keys_find1` is a static function that returns a pointer to a `struct tty_key`. It is used to find a specific key node in a ternary tree structure based on the input buffer and its length.
- **Use**: This function is used internally to traverse the key tree and locate a key node that matches the input sequence.


---
### tty_keys_find
- **Type**: `static struct tty_key *`
- **Description**: The `tty_keys_find` function is a static function that returns a pointer to a `tty_key` structure. It is used to look up a key in a ternary tree structure based on input from a terminal.
- **Use**: This function is used to find a specific key in the terminal's key tree by matching a sequence of characters.


---
### tty_default_raw_keys
- **Type**: ``struct tty_default_key_raw[]``
- **Description**: The `tty_default_raw_keys` is a static constant array of `struct tty_default_key_raw`, which maps terminal escape sequences to specific key codes. This array includes mappings for various keys such as application escape, numeric keypad keys, arrow keys, meta arrow keys, xterm keys, rxvt keys, function keys, focus tracking, paste keys, extended keys, and theme reporting. Each entry in the array consists of a string representing the escape sequence and a corresponding key code.
- **Use**: This variable is used to define a base table of supported raw key sequences that are translated into key codes for terminal input handling.


---
### tty_default_xterm_keys
- **Type**: ``static const struct tty_default_key_xterm[]``
- **Description**: The `tty_default_xterm_keys` is a static constant array of `struct tty_default_key_xterm` that defines default key mappings for xterm terminals. Each entry in the array consists of a template string representing an escape sequence and a corresponding key code. This array is used to map xterm-specific key sequences to internal key codes within the application.
- **Use**: This variable is used to initialize key mappings for xterm terminals by associating specific escape sequences with their corresponding key codes.


---
### tty_default_xterm_modifiers
- **Type**: `key_code[]`
- **Description**: The `tty_default_xterm_modifiers` is a static constant array of type `key_code` that defines a set of default modifier keys used in xterm terminal emulation. Each element in the array represents a combination of modifier keys such as SHIFT, META, and CTRL, which are used to modify the behavior of key inputs in the terminal.
- **Use**: This array is used to map xterm key sequences to their corresponding modifier key combinations in terminal input handling.


---
### tty_default_code_keys
- **Type**: ``static const struct tty_default_key_code[]``
- **Description**: The `tty_default_code_keys` is a static constant array of `struct tty_default_key_code` that maps terminal key codes to internal key codes. It includes mappings for function keys, arrow keys, and other special keys with various modifier combinations such as Shift, Ctrl, and Meta.
- **Use**: This variable is used to define default key mappings for terminal input handling, translating terminal-specific key codes into a format that can be processed by the application.


# Data Structures

---
### tty_key
- **Type**: `struct`
- **Members**:
    - `ch`: A character representing a single key input.
    - `key`: A key_code representing the key's code.
    - `left`: A pointer to the left child in the ternary tree structure.
    - `right`: A pointer to the right child in the ternary tree structure.
    - `next`: A pointer to the next node in the sequence of keys.
- **Description**: The `tty_key` structure is used to represent a node in a ternary tree that maps key sequences to their corresponding key codes. Each node contains a character `ch` that represents a part of the key sequence, a `key_code` that represents the complete key sequence if the node is a terminal node, and pointers to `left`, `right`, and `next` nodes to navigate through the tree. This structure is part of a system to handle key inputs from a terminal, allowing for efficient lookup and processing of key sequences.


---
### tty_default_key_raw
- **Type**: `struct`
- **Members**:
    - `string`: A pointer to a constant character string representing a key sequence.
    - `key`: A key_code representing the key associated with the string.
- **Description**: The `tty_default_key_raw` structure is used to define a mapping between raw key sequences and their corresponding key codes. It consists of a string that represents the key sequence and a key_code that identifies the key. This structure is part of a larger system for handling key inputs from a terminal, allowing for the translation of raw input sequences into recognizable key codes that can be processed by the application.


---
### tty_default_key_xterm
- **Type**: `struct`
- **Members**:
    - `template`: A constant character pointer representing a template string for xterm key sequences.
    - `key`: A key_code type representing the key associated with the template.
- **Description**: The `tty_default_key_xterm` structure is used to define default key mappings for xterm terminals. It consists of a template string that represents the escape sequence for a particular key and a key code that identifies the key. This structure is part of a larger system for handling terminal key inputs, allowing the program to interpret and respond to various key sequences sent by xterm-compatible terminals.


---
### tty_default_key_code
- **Type**: `struct`
- **Members**:
    - `code`: An enumeration value of type `tty_code_code` representing a specific terminal key code.
    - `key`: A `key_code` value representing the key associated with the terminal key code.
- **Description**: The `tty_default_key_code` structure is used to map terminal key codes, represented by the `tty_code_code` enum, to specific key codes, represented by the `key_code` type. This mapping is part of a larger system for handling key inputs from a terminal, allowing the program to interpret and respond to various key presses. The structure is used in an array, `tty_default_code_keys`, which serves as a base table of supported keys that are looked up in terminfo and translated into a ternary tree for efficient key handling.


# Functions

---
### tty_keys_add<!-- {{#callable:tty_keys_add}} -->
The `tty_keys_add` function adds a key sequence to a terminal's key tree, either by creating a new entry or updating an existing one.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal to which the key sequence is being added.
    - `s`: A constant character pointer representing the key sequence string to be added.
    - `key`: A `key_code` value representing the key code associated with the key sequence.
- **Control Flow**:
    - The function begins by looking up the string representation of the key code using [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key).
    - It then attempts to find an existing key entry in the terminal's key tree using [`tty_keys_find`](#tty_keys_find).
    - If no existing entry is found, it logs a debug message indicating a new key is being added and calls [`tty_keys_add1`](#tty_keys_add1) to add the key to the tree.
    - If an existing entry is found, it logs a debug message indicating the key is being replaced and updates the key code of the existing entry.
- **Output**:
    - The function does not return a value; it modifies the terminal's key tree in place.
- **Functions called**:
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`tty_keys_find`](#tty_keys_find)
    - [`tty_keys_add1`](#tty_keys_add1)


---
### tty_keys_add1<!-- {{#callable:tty_keys_add1}} -->
The [`tty_keys_add1`](#tty_keys_add1) function recursively adds a key sequence to a ternary tree structure, representing terminal key mappings.
- **Inputs**:
    - `tkp`: A pointer to a pointer to a `tty_key` structure, representing the current node in the ternary tree where the key sequence is being added.
    - `s`: A string representing the key sequence to be added to the tree.
    - `key`: A `key_code` representing the key code associated with the key sequence.
- **Control Flow**:
    - Check if the current tree node (`tk`) is NULL; if so, allocate memory for a new `tty_key` structure and initialize it with the first character of the string `s` and a default key code `KEYC_UNKNOWN`.
    - Compare the current character of the string `s` with the character stored in the current tree node (`tk->ch`).
    - If the characters match, move to the next character in the string `s` and check if it is the end of the string; if so, set the key code of the current node to `key` and return.
    - If the characters do not match, determine whether to traverse to the left or right child of the current node based on the lexicographical order of the characters.
    - Recursively call [`tty_keys_add1`](#tty_keys_add1) with the updated tree node pointer and the remaining string.
- **Output**:
    - The function does not return a value; it modifies the ternary tree structure pointed to by `tkp` to include the new key sequence and its associated key code.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`tty_keys_add1`](#tty_keys_add1)


---
### tty_keys_build<!-- {{#callable:tty_keys_build}} -->
The `tty_keys_build` function initializes a key tree for a terminal by adding default xterm, raw, and code keys, as well as user-defined keys, to the terminal's key tree structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` which represents the terminal for which the key tree is being built.
- **Control Flow**:
    - Check if the terminal's key tree is not NULL, and if so, free the existing key tree using [`tty_keys_free`](#tty_keys_free).
    - Set the terminal's key tree to NULL to prepare for building a new key tree.
    - Iterate over the `tty_default_xterm_keys` array to add xterm keys with various modifiers to the key tree using [`tty_keys_add`](#tty_keys_add).
    - Iterate over the `tty_default_raw_keys` array to add raw keys to the key tree using [`tty_keys_add`](#tty_keys_add).
    - Iterate over the `tty_default_code_keys` array to add code keys to the key tree using [`tty_keys_add`](#tty_keys_add), using [`tty_term_string`](tty-term.c.driver.md#tty_term_string) to get the string representation of each code key.
    - Retrieve user-defined keys from the global options and add them to the key tree using [`tty_keys_add`](#tty_keys_add).
- **Output**:
    - The function does not return a value; it modifies the `tty` structure by building and populating its key tree.
- **Functions called**:
    - [`tty_keys_free`](#tty_keys_free)
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)
    - [`tty_keys_add`](#tty_keys_add)
    - [`tty_term_string`](tty-term.c.driver.md#tty_term_string)
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_index`](options.c.driver.md#options_array_item_index)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`options_array_next`](options.c.driver.md#options_array_next)


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
    - Check if the `next` pointer of the current `tty_key` node is not NULL, and if so, recursively call [`tty_keys_free1`](#tty_keys_free1) on the `next` node.
    - Check if the `left` pointer of the current `tty_key` node is not NULL, and if so, recursively call [`tty_keys_free1`](#tty_keys_free1) on the `left` node.
    - Check if the `right` pointer of the current `tty_key` node is not NULL, and if so, recursively call [`tty_keys_free1`](#tty_keys_free1) on the `right` node.
    - Free the memory allocated for the current `tty_key` node using `free(tk)`.
- **Output**:
    - The function does not return any value; it performs a cleanup operation by freeing memory.
- **Functions called**:
    - [`tty_keys_free1`](#tty_keys_free1)


---
### tty_keys_find<!-- {{#callable:tty_keys_find}} -->
The `tty_keys_find` function searches for a key in a terminal's key tree based on a given input buffer and updates the size of the matched key sequence.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal whose key tree is being searched.
    - `buf`: A constant character pointer representing the input buffer containing the key sequence to be searched.
    - `len`: A size_t value representing the length of the input buffer.
    - `size`: A pointer to a size_t variable where the function will store the size of the matched key sequence.
- **Control Flow**:
    - Initialize the size variable pointed to by `size` to 0.
    - Call the helper function [`tty_keys_find1`](#tty_keys_find1) with the terminal's key tree, the input buffer, its length, and the size pointer.
    - Return the result of the [`tty_keys_find1`](#tty_keys_find1) function call.
- **Output**:
    - Returns a pointer to a `tty_key` structure if a matching key is found, or NULL if no match is found.
- **Functions called**:
    - [`tty_keys_find1`](#tty_keys_find1)


---
### tty_keys_find1<!-- {{#callable:tty_keys_find1}} -->
The [`tty_keys_find1`](#tty_keys_find1) function recursively searches a ternary tree of `tty_key` nodes to find a match for a given input buffer and updates the size of the matched sequence.
- **Inputs**:
    - `tk`: A pointer to the current `tty_key` node in the ternary tree being searched.
    - `buf`: A pointer to the character buffer containing the input sequence to be matched.
    - `len`: The length of the input buffer `buf`.
    - `size`: A pointer to a `size_t` variable that will be updated with the length of the matched sequence.
- **Control Flow**:
    - Check if the input length `len` is zero; if so, return `NULL` as there is no data to match.
    - Check if the current node `tk` is `NULL`; if so, return `NULL` as it indicates the end of the tree with no match found.
    - Compare the character `tk->ch` with the first character of `buf`.
    - If they match, increment the buffer pointer `buf`, decrement `len`, and increment the value pointed to by `size`.
    - Check if `len` is zero or if `tk->next` is `NULL` and `tk->key` is not `KEYC_UNKNOWN`; if so, return the current node `tk` as a match is found.
    - If the characters do not match, decide whether to traverse the left or right subtree based on the comparison of `*buf` and `tk->ch`.
    - Recursively call [`tty_keys_find1`](#tty_keys_find1) with the updated node, buffer, length, and size.
- **Output**:
    - Returns a pointer to the `tty_key` node that matches the input buffer sequence, or `NULL` if no match is found.
- **Functions called**:
    - [`tty_keys_find1`](#tty_keys_find1)


---
### tty_keys_next1<!-- {{#callable:tty_keys_next1}} -->
The `tty_keys_next1` function processes a buffer of input data to identify and return a recognized key code or determine if more data is needed for a complete key sequence.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A pointer to a buffer containing the input data to be processed.
    - `len`: The length of the input data in the buffer.
    - `key`: A pointer to a `key_code` variable where the identified key code will be stored.
    - `size`: A pointer to a `size_t` variable where the size of the recognized key sequence will be stored.
    - `expired`: An integer flag indicating whether the input data has expired or timed out.
- **Control Flow**:
    - Log the input data and its length for debugging purposes.
    - Attempt to find a known key in the input buffer using [`tty_keys_find`](#tty_keys_find).
    - If a known key is found and it is not `KEYC_UNKNOWN`, log the keys in the list and check if more data is needed; if not, set the key and return 0.
    - If the input is not a known key, check if it is valid UTF-8 using [`utf8_open`](utf8.c.driver.md#utf8_open) and [`utf8_append`](utf8.c.driver.md#utf8_append).
    - If the input is valid UTF-8, convert it to a `utf8_char`, set the key, log the UTF-8 key, and return 0.
    - If the input is neither a known key nor valid UTF-8, return -1 indicating an unrecognized or incomplete key sequence.
- **Output**:
    - The function returns 0 if a complete key is identified, 1 if more data is needed, or -1 if the input is unrecognized or incomplete.
- **Functions called**:
    - [`tty_keys_find`](#tty_keys_find)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)
    - [`utf8_from_data`](utf8.c.driver.md#utf8_from_data)


---
### tty_keys_winsz<!-- {{#callable:tty_keys_winsz}} -->
The `tty_keys_winsz` function processes terminal escape sequences to update the window size of a terminal based on character or pixel dimensions.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal for which the window size is being processed.
    - `buf`: A constant character pointer to the buffer containing the escape sequence data.
    - `len`: A size_t value representing the length of the buffer.
    - `size`: A pointer to a size_t where the function will store the number of bytes processed from the buffer.
- **Control Flow**:
    - Initialize the `size` to 0.
    - Check if the `TTY_WINSIZEQUERY` flag is set in `tty->flags`; if not, return -1 to ignore the request.
    - Verify that the first two bytes of `buf` are '\033['; if not, return -1 or 1 depending on the length of `buf`.
    - Iterate through `buf` starting from the third byte to find the end of the sequence, stopping at 't' or any non-numeric and non-semicolon character.
    - If the end of the sequence is not found or exceeds the buffer size, return -1 or 1.
    - Copy the relevant part of `buf` into a temporary buffer `tmp` and null-terminate it.
    - Attempt to parse `tmp` for window size in characters using `sscanf`; if successful, update the terminal size using [`tty_set_size`](tty.c.driver.md#tty_set_size) and return 0.
    - If parsing for character size fails, attempt to parse for window size in pixels; if successful, calculate character dimensions from pixel dimensions, update the terminal size, invalidate the terminal, clear the `TTY_WINSIZEQUERY` flag, and return 0.
    - If neither parsing attempt is successful, log an unrecognized sequence and return -1.
- **Output**:
    - Returns 0 on successful processing of a window size sequence, -1 if the sequence is invalid or unrecognized, and 1 if more data is needed to complete the sequence.
- **Functions called**:
    - [`tty_set_size`](tty.c.driver.md#tty_set_size)
    - [`tty_invalidate`](tty.c.driver.md#tty_invalidate)


---
### tty_keys_next<!-- {{#callable:tty_keys_next}} -->
The `tty_keys_next` function processes input keys from a terminal, identifying and handling complete or partial key sequences, including special keys, mouse events, and device attributes, and manages key timers for delayed input handling.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` which represents the terminal interface and contains input buffer and state information.
- **Control Flow**:
    - Retrieve the key buffer and its length from the terminal input buffer.
    - Check if the buffer is empty; if so, return 0 indicating no keys to process.
    - Log the current keys in the buffer for debugging purposes.
    - Check if the input is a clipboard response, device attributes response, extended device attributes response, colors response, mouse event, or extended key press, handling each case appropriately.
    - If a complete key is found, handle it by logging, removing the key timer, checking for focus events, and firing the key event.
    - If a partial key is detected, set up a timer to wait for more input, adjusting the delay based on active queries.
    - If the key is not partial, process it as a complete key, handling special cases like escape-prefixed keys, backspace, and control codes.
    - Drain the processed data from the buffer and return 1 indicating a key was processed.
- **Output**:
    - Returns 1 if a key was processed, 0 if no keys were present, or handles partial keys by setting a timer for further input.
- **Functions called**:
    - [`tty_keys_clipboard`](#tty_keys_clipboard)
    - [`tty_keys_device_attributes`](#tty_keys_device_attributes)
    - [`tty_keys_device_attributes2`](#tty_keys_device_attributes2)
    - [`tty_keys_extended_device_attributes`](#tty_keys_extended_device_attributes)
    - [`tty_keys_colours`](#tty_keys_colours)
    - [`session_theme_changed`](session.c.driver.md#session_theme_changed)
    - [`tty_keys_mouse`](#tty_keys_mouse)
    - [`tty_keys_extended_key`](#tty_keys_extended_key)
    - [`tty_keys_winsz`](#tty_keys_winsz)
    - [`tty_keys_next1`](#tty_keys_next1)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_update_focus`](window.c.driver.md#window_update_focus)
    - [`notify_client`](notify.c.driver.md#notify_client)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`server_client_handle_key`](server-client.c.driver.md#server_client_handle_key)


---
### tty_keys_callback<!-- {{#callable:tty_keys_callback}} -->
The `tty_keys_callback` function processes key input events for a terminal by repeatedly calling [`tty_keys_next`](#tty_keys_next) if a timer flag is set.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event types.
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
The function `tty_keys_extended_key` processes extended key input sequences from a terminal, parsing them into key codes and modifiers for further handling.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A pointer to a character buffer containing the input sequence to be processed.
    - `len`: The length of the input buffer `buf`.
    - `size`: A pointer to a `size_t` variable where the function will store the number of bytes processed from the input buffer.
    - `key`: A pointer to a `key_code` variable where the function will store the parsed key code.
- **Control Flow**:
    - Initialize `size` to 0.
    - Check if the first two bytes of `buf` are '\033['; if not, return -1.
    - If `len` is 1 or 2, return 1 indicating more data is needed.
    - Search for a terminator character ('~' or 'u') in `buf` starting from the third byte; if not found, return 1.
    - Copy the relevant part of `buf` into a temporary buffer `tmp` and null-terminate it.
    - Parse the key and modifier numbers from `tmp` using `sscanf`; if parsing fails, return -1.
    - Set `*size` to the number of bytes processed.
    - Determine the key code `nkey` based on the parsed number and the terminal's backspace character setting.
    - If `nkey` is not a basic ASCII character, convert it from UTF-32 to an internal representation using UTF-8 functions.
    - Adjust `nkey` based on the parsed modifiers, setting flags for Shift, Meta, and Ctrl as appropriate.
    - Handle special cases like converting Shift-Tab to Backtab and removing Shift from printable characters.
    - Log the parsed key if logging is enabled.
    - Store the final key code in `*key` and return 0 indicating success.
- **Output**:
    - Returns 0 on successful parsing of an extended key, -1 on failure, and 1 if more data is needed to complete the parsing.
- **Functions called**:
    - [`utf8_fromwc`](utf8.c.driver.md#utf8_fromwc)
    - [`utf8_from_data`](utf8.c.driver.md#utf8_from_data)
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)


---
### tty_keys_mouse<!-- {{#callable:tty_keys_mouse}} -->
The `tty_keys_mouse` function processes mouse input sequences from a terminal, interpreting them into a structured mouse event.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A constant character pointer to the buffer containing the input sequence.
    - `len`: The length of the input buffer.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed from the buffer.
    - `m`: A pointer to a `mouse_event` structure where the function will store the parsed mouse event data.
- **Control Flow**:
    - Initialize variables for mouse event data and set `size` to 0.
    - Check if the first two bytes of `buf` are '\033['; if not, return -1 or 1 based on `len`.
    - Determine the type of mouse sequence by checking the third byte of `buf`.
    - If the third byte is 'M', process the standard mouse sequence by reading three characters for button, X, and Y positions.
    - If the third byte is '<', process the SGR extended mouse sequence by reading numbers for button, X, and Y positions, separated by semicolons, and determine the event type based on the trailing character ('M' or 'm').
    - Log the mouse input for debugging purposes.
    - Validate the parsed button, X, and Y values; if invalid, return -2.
    - Fill the `mouse_event` structure with the parsed data and update the last mouse state in the `tty` structure.
    - Return 0 to indicate successful processing of the mouse input.
- **Output**:
    - Returns 0 on successful parsing of a mouse event, -1 for invalid sequences, 1 if more data is needed, and -2 for invalid but ignorable sequences.


---
### tty_keys_clipboard<!-- {{#callable:tty_keys_clipboard}} -->
The `tty_keys_clipboard` function processes OSC 52 clipboard input sequences, decodes the base64-encoded clipboard data, and forwards it to the appropriate window panes.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A constant character pointer to the buffer containing the input data.
    - `len`: The length of the input buffer.
    - `size`: A pointer to a `size_t` variable where the function will store the size of the processed input.
- **Control Flow**:
    - Initialize the `size` to 0.
    - Check if the first five bytes of the buffer match the expected OSC 52 sequence prefix (`\033]52;`).
    - If the prefix is not matched or the buffer length is insufficient, return -1 or 1 accordingly.
    - Search for the terminator in the buffer, which can be either `\007` or `\033\`.
    - If no terminator is found, return 1 indicating partial input.
    - Adjust the buffer and length to skip the initial part and point to the start of the actual data.
    - Extract the second argument by skipping characters until a semicolon is found.
    - If the second argument is not found or is too short, return 0.
    - Check if the clipboard request was made by verifying the `TTY_OSC52QUERY` flag in `tty->flags`.
    - If the request was not made, return 0.
    - Clear the `TTY_OSC52QUERY` flag and delete the clipboard timer event.
    - Allocate memory and copy the base64-encoded data from the buffer.
    - Decode the base64 data into a newly allocated buffer.
    - If decoding fails, free allocated memory and return 0.
    - Log the decoded data and add it to the paste buffer if the `CLIENT_CLIPBOARDBUFFER` flag is set.
    - Forward the decoded clipboard data to each window pane in `c->clipboard_panes`.
    - Free the `clipboard_panes` array and reset `clipboard_npanes` to 0.
    - Return 0 indicating successful processing.
- **Output**:
    - Returns 0 on successful processing of the clipboard data, -1 if the input is invalid, or 1 if the input is partial.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`b64_pton`](compat/base64.c.driver.md#b64_pton)
    - [`paste_add`](paste.c.driver.md#paste_add)
    - [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id)
    - [`input_reply_clipboard`](input.c.driver.md#input_reply_clipboard)


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
    - Verify the first three bytes of `buf` are '\033[?' and return -1 if not, or 1 if the buffer is incomplete.
    - Copy the remaining characters up to 'c' into a temporary buffer `tmp` and set `*size` to the number of bytes processed.
    - Convert the semicolon-separated arguments in `tmp` to numbers and store them in array `p`.
    - Based on the first number in `p`, add terminal features for specific feature codes (4, 21, 28) using [`tty_add_features`](tty-features.c.driver.md#tty_add_features).
    - Log the received primary device attributes and update terminal features using [`tty_update_features`](tty.c.driver.md#tty_update_features).
    - Set the `TTY_HAVEDA` flag on the `tty` structure to indicate that device attributes have been processed.
    - Return 0 to indicate successful processing.
- **Output**:
    - Returns 0 on successful processing of device attributes, -1 if the terminal already has device attributes or if the input is invalid, and 1 if the input buffer is incomplete.
- **Functions called**:
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`tty_add_features`](tty-features.c.driver.md#tty_add_features)
    - [`tty_update_features`](tty.c.driver.md#tty_update_features)


---
### tty_keys_device_attributes2<!-- {{#callable:tty_keys_device_attributes2}} -->
The function `tty_keys_device_attributes2` processes secondary device attributes input from a terminal and updates terminal features accordingly.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `buf`: A constant character pointer to the buffer containing the input data to be processed.
    - `len`: A size_t value representing the length of the input buffer.
    - `size`: A pointer to a size_t variable where the function will store the size of the processed input.
- **Control Flow**:
    - Initialize local variables and check if the TTY_HAVEDA2 flag is set; if so, return -1 indicating failure.
    - Check if the first three bytes of the buffer are '\033[>' and return -1 if not, or 1 if the buffer is too short.
    - Copy the remaining characters up to 'c' into a temporary buffer and update the size variable.
    - Convert the copied arguments into numbers using `strtoul` and store them in an array.
    - Based on the first number in the array, update terminal features for specific terminal types like 'mintty', 'tmux', or 'rxvt-unicode'.
    - Log the received secondary device attributes and update the terminal features.
    - Set the TTY_HAVEDA2 flag and return 0 indicating success.
- **Output**:
    - Returns 0 on successful processing of the secondary device attributes, -1 on failure, or 1 if more data is needed.
- **Functions called**:
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`tty_default_features`](tty-features.c.driver.md#tty_default_features)
    - [`tty_update_features`](tty.c.driver.md#tty_update_features)


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
    - Check if the first four bytes of `buf` match the expected sequence '\033P>|'; if not, return -1 or 1 depending on the length of `buf`.
    - Copy the remaining characters from `buf` into a temporary buffer `tmp` until the sequence '\033\' is found or the buffer limit is reached.
    - If the buffer limit is reached without finding the sequence, return -1 indicating failure.
    - Null-terminate the `tmp` string and update `*size` to reflect the processed length.
    - Compare the `tmp` string with known terminal identifiers (e.g., 'iTerm2', 'tmux', 'XTerm', 'mintty', 'foot') and call [`tty_default_features`](tty-features.c.driver.md#tty_default_features) to add corresponding features.
    - Log the received extended device attributes using `log_debug`.
    - Free the existing `term_type` in the client structure and duplicate the `tmp` string into `term_type`.
    - Call [`tty_update_features`](tty.c.driver.md#tty_update_features) to update the terminal features.
    - Set the `TTY_HAVEXDA` flag in `tty->flags` to indicate that extended device attributes have been processed.
    - Return 0 indicating success.
- **Output**:
    - The function returns 0 on success, -1 on failure, or 1 if more data is needed to complete processing.
- **Functions called**:
    - [`tty_default_features`](tty-features.c.driver.md#tty_default_features)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`tty_update_features`](tty.c.driver.md#tty_update_features)


---
### tty_keys_colours<!-- {{#callable:tty_keys_colours}} -->
The `tty_keys_colours` function processes a terminal escape sequence to extract and set foreground or background color values.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `buf`: A constant character pointer to the buffer containing the escape sequence.
    - `len`: The length of the buffer.
    - `size`: A pointer to a size_t variable where the function will store the number of bytes processed.
    - `fg`: A pointer to an integer where the function will store the parsed foreground color value if applicable.
    - `bg`: A pointer to an integer where the function will store the parsed background color value if applicable.
- **Control Flow**:
    - Initialize `*size` to 0.
    - Check if the first five characters of `buf` match the expected escape sequence prefix `\033]1` followed by `0` or `1` and `;`.
    - If any of the checks fail, return -1 for an invalid sequence or 1 if more data is needed.
    - Copy the remaining characters from `buf` into a temporary buffer `tmp` until a terminator (`\033\` or `\007`) is found or the buffer is full.
    - If the buffer is full without finding a terminator, return -1.
    - Null-terminate the `tmp` string and update `*size` with the number of bytes processed.
    - Parse the color value from `tmp` using [`colour_parseX11`](colour.c.driver.md#colour_parseX11).
    - If a valid color is parsed and the sequence indicates a foreground color (`buf[3] == '0'`), set `*fg` to the parsed color value and log the foreground color.
    - If a valid color is parsed and the sequence indicates a background color, set `*bg` to the parsed color value and log the background color.
    - Return 0 to indicate successful processing.
- **Output**:
    - Returns 0 on successful processing of the color sequence, -1 if the sequence is invalid, or 1 if more data is needed.
- **Functions called**:
    - [`colour_parseX11`](colour.c.driver.md#colour_parseX11)
    - [`colour_tostring`](colour.c.driver.md#colour_tostring)


