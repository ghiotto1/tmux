# Purpose
This C source code file is part of the tmux terminal multiplexer and is responsible for handling input key translation. The primary function of this file is to map key codes to sequences that can be sent to applications running within tmux panes. It defines a tree structure to store key mappings and provides functions to build this tree from a list of default key mappings. The file includes functionality to handle various types of keys, including function keys, arrow keys, keypad keys, and keys with embedded modifiers. It also supports mouse events and can translate these into sequences that are compatible with the terminal's input expectations.

The file is not a standalone executable but rather a component of the tmux codebase, likely intended to be compiled and linked with other parts of the tmux application. It does not define public APIs or external interfaces but instead provides internal functionality for key translation within tmux. The code includes mechanisms to handle different terminal modes and key encoding formats, such as extended keys and UTF-8 encoding, ensuring compatibility with various terminal behaviors. The file also includes logging for debugging purposes, allowing developers to trace key translation processes.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### input_key_tree
- **Type**: `struct input_key_tree`
- **Description**: The `input_key_tree` is a global variable of type `struct input_key_tree`, which is initialized using the `RB_INITIALIZER` macro. This structure is used to manage a red-black tree of `input_key_entry` structures, which map key codes to their corresponding data strings.
- **Use**: This variable is used to store and manage a tree of input keys, allowing for efficient lookup and insertion of key mappings.


---
### input_key_defaults
- **Type**: `struct input_key_entry[]`
- **Description**: The `input_key_defaults` is a static array of `struct input_key_entry` that defines default key mappings for various keyboard keys and their corresponding escape sequences. This array includes mappings for paste keys, function keys, arrow keys, keypad keys, and keys with embedded modifiers. Each entry in the array consists of a key code and its associated data, which is typically an escape sequence string.
- **Use**: This variable is used to initialize a tree of input keys that translate key codes into sequences suitable for applications running in a terminal pane.


---
### input_key_modifiers
- **Type**: ``key_code[]``
- **Description**: The `input_key_modifiers` is a static constant array of type `key_code` that defines a set of key modifier combinations. Each element in the array represents a combination of key modifiers such as SHIFT, META, and CTRL, which are used to modify the behavior of key inputs in the application.
- **Use**: This array is used to map and apply different key modifier combinations to input keys, allowing the application to handle various key input scenarios with modifiers.


# Data Structures

---
### input_key_entry
- **Type**: `struct`
- **Members**:
    - `key`: Represents the key code associated with the input key entry.
    - `data`: Points to a string that contains the data or sequence associated with the key code.
    - `entry`: A Red-Black tree entry used to link this structure into a Red-Black tree.
- **Description**: The `input_key_entry` structure is used to represent an entry in a Red-Black tree that maps key codes to their corresponding data sequences. This structure is part of a system that translates key codes into sequences suitable for sending to an application running in a terminal pane. Each entry in the tree consists of a key code, a pointer to the associated data string, and a Red-Black tree entry for maintaining the tree structure. The Red-Black tree is used to efficiently store and retrieve key entries based on their key codes.


# Functions

---
### input_key_cmp<!-- {{#callable:input_key_cmp}} -->
The `input_key_cmp` function compares two `input_key_entry` structures based on their `key` values and returns an integer indicating their relative order.
- **Inputs**:
    - `ike1`: A pointer to the first `input_key_entry` structure to be compared.
    - `ike2`: A pointer to the second `input_key_entry` structure to be compared.
- **Control Flow**:
    - Check if the `key` of `ike1` is less than the `key` of `ike2`; if true, return -1.
    - Check if the `key` of `ike1` is greater than the `key` of `ike2`; if true, return 1.
    - If neither condition is met, return 0, indicating the keys are equal.
- **Output**:
    - An integer: -1 if `ike1`'s key is less than `ike2`'s key, 1 if greater, and 0 if they are equal.


---
### input_key_get<!-- {{#callable:input_key_get}} -->
The `input_key_get` function searches for a key code in a red-black tree and returns the corresponding `input_key_entry` if found.
- **Inputs**:
    - `key`: A `key_code` representing the key to be searched in the red-black tree.
- **Control Flow**:
    - A temporary `input_key_entry` structure is created with the provided key code.
    - The `RB_FIND` macro is used to search the `input_key_tree` for an entry matching the temporary structure.
    - The function returns the result of the `RB_FIND` operation, which is a pointer to the found `input_key_entry` or `NULL` if not found.
- **Output**:
    - A pointer to the `input_key_entry` structure if the key is found in the tree, otherwise `NULL`.


---
### input_key_split2<!-- {{#callable:input_key_split2}} -->
The `input_key_split2` function converts a Unicode character into its UTF-8 byte representation and stores it in a destination buffer.
- **Inputs**:
    - `c`: An unsigned integer representing the Unicode character to be converted.
    - `dst`: A pointer to an unsigned character array where the UTF-8 byte representation will be stored.
- **Control Flow**:
    - Check if the input character `c` is greater than 0x7f (i.e., if it requires more than one byte in UTF-8).
    - If `c` is greater than 0x7f, calculate the first byte by shifting `c` right by 6 bits and OR-ing with 0xc0, and store it in `dst[0]`.
    - Calculate the second byte by AND-ing `c` with 0x3f and OR-ing with 0x80, and store it in `dst[1]`.
    - Return 2, indicating that two bytes were used.
    - If `c` is not greater than 0x7f, store `c` directly in `dst[0]`.
    - Return 1, indicating that one byte was used.
- **Output**:
    - The function returns the number of bytes used to represent the character in UTF-8, either 1 or 2.


---
### input_key_build<!-- {{#callable:input_key_build}} -->
The `input_key_build` function constructs a tree of input key entries by processing default key entries and their modifiers, and logs the constructed keys.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each entry in the `input_key_defaults` array.
    - For each entry, check if the key has modifiers using a bitwise AND operation with `KEYC_BUILD_MODIFIERS`.
    - If the key does not have modifiers, insert the entry directly into the `input_key_tree`.
    - If the key has modifiers, iterate over the `input_key_modifiers` array starting from index 2.
    - For each modifier, create a new key by combining the base key with the modifier, and adjust the data string to reflect the modifier.
    - Allocate memory for a new `input_key_entry`, set its key and data, and insert it into the `input_key_tree`.
    - After processing all entries, iterate over the `input_key_tree` and log each key and its associated data.
- **Output**:
    - The function does not return a value; it modifies the global `input_key_tree` by inserting key entries and logs the constructed keys.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)


---
### input_key_pane<!-- {{#callable:input_key_pane}} -->
The `input_key_pane` function processes a key code for a specific window pane, handling mouse events separately and logging the key if logging is enabled.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane where the key event is to be processed.
    - `key`: A `key_code` representing the key event to be processed.
    - `m`: A pointer to a `mouse_event` structure, which may be NULL, representing a mouse event associated with the key event.
- **Control Flow**:
    - Check if logging is enabled by calling `log_get_level()` and log the key event if it is.
    - Determine if the key event is a mouse event using `KEYC_IS_MOUSE(key)`.
    - If the key is a mouse event and the mouse event structure `m` is not NULL, check if the mouse event is associated with the current pane (`m->wp == wp->id`).
    - If the mouse event is associated with the current pane, call `input_key_mouse(wp, m)` to handle the mouse event and return 0.
    - If the key is not a mouse event, call `input_key(wp->screen, wp->event, key)` to process the key event and return its result.
- **Output**:
    - Returns 0 if the key is a mouse event and is handled, otherwise returns the result of `input_key()` for non-mouse key events.
- **Functions called**:
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`input_key_mouse`](#input_key_mouse)
    - [`input_key`](#input_key)


---
### input_key_write<!-- {{#callable:input_key_write}} -->
The `input_key_write` function logs a debug message and writes data to a buffer event.
- **Inputs**:
    - `from`: A string representing the source or context from which the function is called.
    - `bev`: A pointer to a `bufferevent` structure where the data will be written.
    - `data`: A pointer to a character array containing the data to be written.
    - `size`: The size of the data to be written, specified as a `size_t` type.
- **Control Flow**:
    - Logs a debug message using the `log_debug` function, including the source and the data to be written.
    - Writes the specified data to the buffer event using the `bufferevent_write` function.
- **Output**:
    - The function does not return a value; it performs logging and writes data to a buffer event.


---
### input_key_extended<!-- {{#callable:input_key_extended}} -->
The `input_key_extended` function encodes a key code into an extended key escape sequence and writes it to a buffer event.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure where the encoded key sequence will be written.
    - `key`: A `key_code` representing the key to be encoded, which may include modifier flags.
- **Control Flow**:
    - Determine the modifier character based on the key's modifier flags using a switch statement.
    - If the key is a Unicode key, convert it to a wide character; if conversion fails, return -1.
    - Mask the key to remove modifier flags if it is not Unicode.
    - Check the global option 'extended-keys-format' to decide the format of the escape sequence.
    - Format the escape sequence into the `tmp` buffer using [`xsnprintf`](xmalloc.c.driver.md#xsnprintf).
    - Write the formatted escape sequence to the buffer event using [`input_key_write`](#input_key_write).
    - Return 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs during processing, such as an invalid key or failed Unicode conversion.
- **Functions called**:
    - [`utf8_to_data`](utf8.c.driver.md#utf8_to_data)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`input_key_write`](#input_key_write)


---
### input_key_vt10x<!-- {{#callable:input_key_vt10x}} -->
The `input_key_vt10x` function translates a key code into a corresponding output sequence suitable for a terminal in standard mode, handling various key modifiers and control codes.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure used for writing the output key sequence.
    - `key`: A `key_code` representing the key input, which may include modifiers like META or CTRL.
- **Control Flow**:
    - Log the input key code for debugging purposes.
    - Check if the META modifier is present and write the escape character '\033' if so.
    - If the key is a Unicode key, convert it to UTF-8 and write it to the buffer, then return 0.
    - Extract the base key by masking with `KEYC_MASK_KEY` and handle special cases for TAB, CR, and LF by removing the CTRL modifier.
    - If the CTRL modifier is present, attempt to map the key to a C0 control code using a predefined map or calculate it directly; return -1 if mapping fails.
    - Log the output key code for debugging purposes.
    - Write the final key code to the buffer, ensuring it is within the 7-bit ASCII range, and return 0.
- **Output**:
    - The function returns 0 on successful translation and writing of the key sequence, or -1 if an error occurs during key mapping.
- **Functions called**:
    - [`input_key_write`](#input_key_write)
    - [`utf8_to_data`](utf8.c.driver.md#utf8_to_data)


---
### input_key_mode1<!-- {{#callable:input_key_mode1}} -->
The `input_key_mode1` function processes a key code to determine if it should be translated into a VT10x key sequence based on specific modifier conditions.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure, which is used for buffered I/O operations.
    - `key`: A `key_code` representing the key input, which may include various modifiers like CTRL or META.
- **Control Flow**:
    - Log the function entry and the key value for debugging purposes.
    - Check if the key has only the META modifier and, if so, call [`input_key_vt10x`](#input_key_vt10x) to process it as a VT10x key.
    - Extract the base key from the key code by masking with `KEYC_MASK_KEY`.
    - Check if the key has the CTRL modifier and if the base key is one of the specified characters or within certain ranges.
    - If the above condition is met, call [`input_key_vt10x`](#input_key_vt10x) to process it as a VT10x key.
    - If none of the conditions are met, return -1 indicating the key does not match the criteria for VT10x processing.
- **Output**:
    - Returns an integer, either the result of [`input_key_vt10x`](#input_key_vt10x) if the key matches the criteria, or -1 if it does not.
- **Functions called**:
    - [`input_key_vt10x`](#input_key_vt10x)


---
### input_key<!-- {{#callable:input_key}} -->
The `input_key` function processes a key code and translates it into a suitable output sequence for an application running in a terminal pane, handling various key types and modes.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the terminal screen context.
    - `bev`: A pointer to a `bufferevent` structure used for buffered I/O operations.
    - `key`: A `key_code` representing the key input to be processed.
- **Control Flow**:
    - Check if the key is a mouse event and return 0 if true, as mouse keys need a pane.
    - If the key is a literal key, write it directly to the buffer and return 0.
    - Check if the key is a backspace and handle it by potentially remapping it based on global options, then write the result to the buffer and return 0.
    - Check if the key is a backtab and modify it based on the screen mode, then continue processing.
    - Handle trivial cases where the key is a 7-bit key or a UTF-8 key with no modifiers, writing it directly to the buffer and returning 0.
    - Look up the key in the standard VT10x key tree, adjusting for keypad and cursor modes, and write the corresponding data to the buffer if found, then return 0.
    - Ignore internal function key codes and return 0 if the key falls within certain ranges.
    - If no built-in key sequence is found, construct an extended key sequence based on the client mode and return the result of the appropriate function call.
- **Output**:
    - The function returns 0 on successful processing of the key, or the result of a call to another function if an extended key sequence is constructed.
- **Functions called**:
    - [`input_key_write`](#input_key_write)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`utf8_to_data`](utf8.c.driver.md#utf8_to_data)
    - [`input_key_get`](#input_key_get)
    - [`input_key_extended`](#input_key_extended)
    - [`input_key_mode1`](#input_key_mode1)
    - [`input_key_vt10x`](#input_key_vt10x)


---
### input_key_get_mouse<!-- {{#callable:input_key_get_mouse}} -->
The `input_key_get_mouse` function processes mouse events and generates an appropriate escape sequence for the event based on the screen's mode and the mouse event details.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the current screen state and mode.
    - `m`: A pointer to a `mouse_event` structure containing details about the mouse event, such as button states and SGR type.
    - `x`: An unsigned integer representing the x-coordinate of the mouse event.
    - `y`: An unsigned integer representing the y-coordinate of the mouse event.
    - `rbuf`: A pointer to a constant character pointer where the resulting escape sequence buffer will be stored.
    - `rlen`: A pointer to a size_t variable where the length of the resulting escape sequence will be stored.
- **Control Flow**:
    - Initialize the output buffer and length to NULL and 0, respectively.
    - Check if the screen mode allows processing of motion events; if not, return 0 to indicate no output.
    - Check if the event is a release event and if the screen mode allows it; if not, return 0.
    - Determine the appropriate escape sequence format based on the screen mode and mouse event details (SGR, UTF-8, or legacy format).
    - Generate the escape sequence using the selected format and store it in the buffer.
    - Set the output buffer and length pointers to the generated escape sequence and its length.
    - Return 1 to indicate successful generation of the escape sequence.
- **Output**:
    - Returns 1 if a valid escape sequence is generated and sets `rbuf` and `rlen` to the sequence and its length, respectively; returns 0 if no sequence is generated.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`input_key_split2`](#input_key_split2)


---
### input_key_mouse<!-- {{#callable:input_key_mouse}} -->
The `input_key_mouse` function processes mouse events for a specific window pane, translating them into a format suitable for the application running in the pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane where the mouse event occurred.
    - `m`: A pointer to a `mouse_event` structure containing details about the mouse event to be processed.
- **Control Flow**:
    - Check if the mouse event should be ignored or if the pane is not in a mouse mode; if so, return immediately.
    - Determine the x and y coordinates of the mouse event relative to the pane using [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at); if unsuccessful, return.
    - Check if the pane is visible using [`window_pane_visible`](window.c.driver.md#window_pane_visible); if not, return.
    - Attempt to get the mouse event string using [`input_key_get_mouse`](#input_key_get_mouse); if unsuccessful, return.
    - Log the mouse event string and write it to the pane's event buffer using [`input_key_write`](#input_key_write).
- **Output**:
    - The function does not return a value; it writes the processed mouse event data to the pane's event buffer if all conditions are met.
- **Functions called**:
    - [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at)
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`input_key_get_mouse`](#input_key_get_mouse)
    - [`input_key_write`](#input_key_write)


