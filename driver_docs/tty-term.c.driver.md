# Purpose
This C source code file is part of the `tmux` terminal multiplexer project and is responsible for managing terminal capabilities and features. It defines structures and functions to handle terminal capabilities, which are essential for `tmux` to interact with different types of terminal emulators. The file includes definitions for terminal capability types, such as strings, numbers, and flags, and provides mechanisms to read, apply, and override these capabilities based on the terminal type. The code uses the `terminfo` database to retrieve terminal capabilities and applies custom overrides and features to ensure compatibility and enhanced functionality across various terminal environments.

The file is not an executable but rather a component of the `tmux` library, intended to be used internally by other parts of the `tmux` codebase. It defines a set of terminal capabilities and provides functions to manipulate these capabilities, such as [`tty_term_create`](#tty_term_create), [`tty_term_apply`](#tty_term_apply), and [`tty_term_free`](#tty_term_free). These functions are used to initialize terminal settings, apply user-defined overrides, and manage terminal-specific features. The code also includes logging for debugging purposes, which helps in tracking the application of terminal capabilities and features. Overall, this file plays a crucial role in ensuring that `tmux` can effectively manage terminal interactions, providing a consistent user experience across different terminal types.
# Imports and Dependencies

---
- `sys/types.h`
- `curses.h`
- `ncurses.h`
- `fnmatch.h`
- `stdlib.h`
- `string.h`
- `term.h`
- `tmux.h`


# Global Variables

---
### tty_term_strip
- **Type**: `function`
- **Description**: The `tty_term_strip` function is a static function that processes a given string, removing any terminal padding sequences denoted by `$<...>`. It returns a newly allocated string with these sequences stripped out.
- **Use**: This function is used to clean up terminal capability strings by removing padding sequences, which are not needed for certain operations.


---
### tty_terms
- **Type**: `struct tty_terms`
- **Description**: The `tty_terms` variable is a global instance of the `struct tty_terms`, initialized using the `LIST_HEAD_INITIALIZER` macro. This suggests that `tty_terms` is intended to be the head of a linked list structure, likely used to manage terminal-related data or configurations.
- **Use**: This variable is used as the head of a linked list to manage terminal configurations or instances within the program.


---
### tty_term_codes
- **Type**: ``struct tty_term_code_entry[]``
- **Description**: The `tty_term_codes` is a static constant array of `struct tty_term_code_entry` that defines terminal capability codes. Each entry in the array consists of a type (such as string, number, or flag) and a corresponding name that represents a terminal capability. This array is used to map terminal capabilities to their respective codes and types.
- **Use**: This variable is used to define and access terminal capabilities and their types for terminal handling in the program.


# Data Structures

---
### tty_code_type
- **Type**: `enum`
- **Members**:
    - `TTYCODE_NONE`: Represents a terminal code type that is not set or is undefined.
    - `TTYCODE_STRING`: Represents a terminal code type that is a string.
    - `TTYCODE_NUMBER`: Represents a terminal code type that is a number.
    - `TTYCODE_FLAG`: Represents a terminal code type that is a boolean flag.
- **Description**: The `tty_code_type` is an enumeration that defines different types of terminal codes used in terminal handling. It categorizes terminal codes into four types: `TTYCODE_NONE` for undefined or unset codes, `TTYCODE_STRING` for codes that are represented as strings, `TTYCODE_NUMBER` for codes that are represented as numbers, and `TTYCODE_FLAG` for codes that are represented as boolean flags. This enumeration is used to manage and interpret terminal capabilities in a structured manner.


---
### tty_code
- **Type**: `struct`
- **Members**:
    - `type`: Specifies the type of the terminal code, which can be a string, number, or flag.
    - `value`: A union that holds the actual value of the terminal code, which can be a string, number, or flag.
- **Description**: The `tty_code` structure is used to represent a terminal capability code in the tmux terminal multiplexer. It consists of a type field, which indicates the kind of value stored (string, number, or flag), and a union named `value` that holds the actual data corresponding to the type. This structure allows tmux to handle various terminal capabilities in a flexible manner, supporting different types of data that a terminal might use to describe its capabilities.


---
### tty_term_code_entry
- **Type**: `struct`
- **Members**:
    - `type`: Specifies the type of terminal code, represented by an enum tty_code_type.
    - `name`: A constant character pointer representing the name of the terminal code.
- **Description**: The `tty_term_code_entry` structure is used to define terminal code entries, which consist of a type and a name. The type is an enumeration that indicates the kind of terminal code (such as string, number, or flag), while the name is a string that identifies the specific terminal capability or feature. This structure is part of a larger system for managing terminal capabilities in a terminal multiplexer like tmux, allowing for the customization and handling of various terminal features.


# Functions

---
### tty_term_ncodes<!-- {{#callable:tty_term_ncodes}} -->
The function `tty_term_ncodes` returns the number of terminal codes defined in the `tty_term_codes` array.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls the `nitems` macro with `tty_term_codes` as its argument.
    - The `nitems` macro calculates the number of elements in the `tty_term_codes` array.
    - The function returns the result of the `nitems` macro call.
- **Output**:
    - The function returns an unsigned integer representing the number of terminal codes available.


---
### tty_term_strip<!-- {{#callable:tty_term_strip}} -->
The `tty_term_strip` function removes terminal padding sequences from a given string and returns a newly allocated copy of the stripped string.
- **Inputs**:
    - `s`: A constant character pointer representing the input string that may contain terminal padding sequences.
- **Control Flow**:
    - Check if the input string contains a '$' character; if not, return a duplicate of the input string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Initialize a buffer and a length counter to store the stripped string.
    - Iterate over each character in the input string.
    - If a '$<' sequence is found, skip characters until a '>' is encountered, effectively ignoring the padding sequence.
    - Copy non-padding characters to the buffer until the buffer is full or the end of the string is reached.
    - Terminate the buffer with a null character.
    - Return a duplicate of the buffer using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
- **Output**:
    - A newly allocated string that is a copy of the input string with terminal padding sequences removed.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### tty_term_override_next<!-- {{#callable:tty_term_override_next}} -->
The function `tty_term_override_next` extracts the next segment from a colon-separated string, handling escaped colons, and updates the offset for subsequent calls.
- **Inputs**:
    - `s`: A constant character pointer representing the input string from which segments are extracted.
    - `offset`: A pointer to a size_t variable that indicates the current position in the string and is updated to the position after the extracted segment.
- **Control Flow**:
    - Initialize a static buffer `value` to store the extracted segment and set `n` to 0 for tracking the buffer index.
    - Set `at` to the current offset value to start processing the string from that position.
    - Check if the character at the current position `at` is the null terminator; if so, return NULL indicating no more segments.
    - Enter a loop to process characters until a null terminator is encountered.
    - If a colon is found, check if it is followed by another colon (indicating an escaped colon), and if so, add a single colon to `value` and skip the next character; otherwise, break the loop as it marks the end of the segment.
    - If the character is not a colon, add it to `value` and increment the position `at`.
    - Check if the buffer `value` is full; if so, return NULL to indicate an error or overflow.
    - After exiting the loop, update the offset to the position after the extracted segment or to the end of the string if no more characters are left.
    - Terminate the `value` buffer with a null character and return it.
- **Output**:
    - Returns a pointer to a static buffer containing the next segment of the input string, or NULL if the end of the string is reached or an error occurs.


---
### tty_term_apply<!-- {{#callable:tty_term_apply}} -->
The `tty_term_apply` function processes terminal capability overrides and updates the terminal's capability codes accordingly.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal whose capabilities are to be modified.
    - `capabilities`: A string containing terminal capability overrides, formatted as a series of key-value pairs separated by colons.
    - `quiet`: An integer flag indicating whether to suppress debug logging (non-zero to suppress, zero to allow logging).
- **Control Flow**:
    - Initialize local variables including pointers and counters.
    - Enter a loop to process each capability override from the `capabilities` string using [`tty_term_override_next`](#tty_term_override_next).
    - For each override, determine if it is a removal (indicated by '@') or an assignment (indicated by '='), and extract the value if applicable.
    - Log the override action if `quiet` is not set.
    - Iterate over all terminal codes to find a matching code name for the current override.
    - If a match is found, update the terminal's code based on the type (string, number, or flag) and the action (remove or assign).
    - Free any dynamically allocated memory for the value string.
- **Output**:
    - The function does not return a value; it modifies the `tty_term` structure in place to reflect the applied capability overrides.
- **Functions called**:
    - [`tty_term_override_next`](#tty_term_override_next)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strunvis`](compat/unvis.c.driver.md#strunvis)
    - [`tty_term_ncodes`](#tty_term_ncodes)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)


---
### tty_term_apply_overrides<!-- {{#callable:tty_term_apply_overrides}} -->
The function `tty_term_apply_overrides` applies terminal capability overrides and updates terminal flags based on the terminal's capabilities.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal whose capabilities and flags are to be updated.
- **Control Flow**:
    - Retrieve the 'terminal-overrides' option from global options and iterate over its array items.
    - For each item, extract the string value and parse it to find terminal-specific overrides.
    - If an override matches the terminal's name, apply it using [`tty_term_apply`](#tty_term_apply).
    - Log the current state of the SIXEL flag based on the terminal's flags.
    - Check if the terminal supports RGB colors and update the TERM_RGBCOLOURS flag accordingly, then log the state of this flag.
    - Check for margin capabilities and update the TERM_DECSLRM flag, logging its state.
    - Check for rectangle capabilities and update the TERM_DECFRA flag, logging its state.
    - Determine if the terminal lacks auto right margin (am) and update the TERM_NOAM flag, logging its state.
    - Initialize the ACS (Alternate Character Set) table, using the terminal's ACSC capability if available, or a default ASCII mapping otherwise.
- **Output**:
    - The function does not return a value; it modifies the `tty_term` structure in place, updating its flags and capabilities.
- **Functions called**:
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`tty_term_override_next`](#tty_term_override_next)
    - [`tty_term_apply`](#tty_term_apply)
    - [`options_array_next`](options.c.driver.md#options_array_next)
    - [`tty_term_has`](#tty_term_has)
    - [`tty_term_flag`](#tty_term_flag)
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_create<!-- {{#callable:tty_term_create}} -->
The `tty_term_create` function initializes a terminal structure with specified capabilities and features, applying necessary overrides and checks for essential terminal capabilities.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `name`: A string representing the name of the terminal.
    - `caps`: An array of strings representing terminal capabilities in the format 'name=value'.
    - `ncaps`: An unsigned integer representing the number of capabilities in the `caps` array.
    - `feat`: A pointer to an integer where terminal features will be stored.
    - `cause`: A pointer to a string where an error message will be stored if the function fails.
- **Control Flow**:
    - Allocate memory for a new `tty_term` structure and initialize it with the provided `tty` and `name`.
    - Insert the new terminal into the global list of terminals.
    - Iterate over the provided capabilities (`caps`) and match them against known terminal codes, setting the appropriate type and value in the terminal's `codes` array.
    - Retrieve and apply terminal features from global options, matching them against the terminal name and updating the `feat` array.
    - Delete any existing curses data if the ncurses version is greater than 5.6.
    - Apply any terminal overrides to adjust capabilities based on features.
    - Check for the presence of essential capabilities (`clear` and `cup`), and if missing, set an error message and exit with failure.
    - Determine if the terminal is VT100-like based on certain capabilities and add related features if applicable.
    - Add RGB color support features if the terminal supports RGB colors but lacks specific RGB setting capabilities.
    - Reapply features and overrides to ensure all settings are up-to-date.
    - Log the terminal's capabilities for debugging purposes.
- **Output**:
    - Returns a pointer to the newly created `tty_term` structure on success, or `NULL` on failure, with an error message stored in `cause`.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`tty_term_ncodes`](#tty_term_ncodes)
    - [`tty_term_strip`](#tty_term_strip)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`tty_term_override_next`](#tty_term_override_next)
    - [`tty_add_features`](tty-features.c.driver.md#tty_add_features)
    - [`options_array_next`](options.c.driver.md#options_array_next)
    - [`tty_term_apply_overrides`](#tty_term_apply_overrides)
    - [`tty_term_has`](#tty_term_has)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`tty_term_string`](#tty_term_string)
    - [`tty_term_flag`](#tty_term_flag)
    - [`tty_apply_features`](tty-features.c.driver.md#tty_apply_features)
    - [`tty_term_describe`](#tty_term_describe)
    - [`tty_term_free`](#tty_term_free)


---
### tty_term_free<!-- {{#callable:tty_term_free}} -->
The `tty_term_free` function deallocates memory and removes a terminal entry from a list.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal to be freed.
- **Control Flow**:
    - Logs the removal of the terminal using its name.
    - Iterates over all terminal codes using a loop from 0 to the number of terminal codes.
    - Checks if each code is of type `TTYCODE_STRING` and frees the associated string value if so.
    - Frees the memory allocated for the terminal's codes array.
    - Removes the terminal from the list of terminals using `LIST_REMOVE`.
    - Frees the memory allocated for the terminal's name.
    - Frees the memory allocated for the terminal structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided terminal structure.
- **Functions called**:
    - [`tty_term_ncodes`](#tty_term_ncodes)


---
### tty_term_read_list<!-- {{#callable:tty_term_read_list}} -->
The `tty_term_read_list` function reads terminal capabilities from the terminfo database for a specified terminal type and file descriptor, storing them in a dynamically allocated list.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the terminal type to be read.
    - `fd`: An integer representing the file descriptor associated with the terminal.
    - `caps`: A pointer to a pointer to a character array, which will be allocated and filled with the terminal capabilities.
    - `ncaps`: A pointer to an unsigned integer where the number of capabilities read will be stored.
    - `cause`: A pointer to a character pointer where an error message will be stored if the function fails.
- **Control Flow**:
    - The function begins by calling `setupterm` to initialize the terminal type specified by `name` and associates it with the file descriptor `fd`.
    - If `setupterm` fails, an error message is formatted and stored in `cause`, and the function returns -1.
    - If `setupterm` succeeds, `ncaps` is initialized to 0 and `caps` is set to NULL.
    - The function iterates over all terminal codes defined in `tty_term_codes`.
    - For each code, it checks the type and retrieves the corresponding capability using `tigetstr`, `tigetnum`, or `tigetflag` based on the type.
    - If a capability is successfully retrieved, it is formatted as a string and added to the `caps` array, incrementing `ncaps`.
    - After processing all codes, the function conditionally deletes the current terminal structure using `del_curterm` if certain conditions are met.
    - The function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, and -1 on failure, with an error message stored in `cause`.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`tty_term_ncodes`](#tty_term_ncodes)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### tty_term_free_list<!-- {{#callable:tty_term_free_list}} -->
The `tty_term_free_list` function deallocates memory for a list of terminal capability strings and the list itself.
- **Inputs**:
    - `caps`: A pointer to an array of strings, where each string represents a terminal capability.
    - `ncaps`: An unsigned integer representing the number of capability strings in the `caps` array.
- **Control Flow**:
    - Iterate over each element in the `caps` array using a for loop, with the loop index `i` ranging from 0 to `ncaps - 1`.
    - For each element `caps[i]`, call `free()` to deallocate the memory allocated for the string.
    - After the loop, call `free()` on `caps` to deallocate the memory allocated for the array itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the input array and its elements.


---
### tty_term_has<!-- {{#callable:tty_term_has}} -->
The `tty_term_has` function checks if a specific terminal capability code is present in a terminal's code set.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal whose capabilities are being checked.
    - `code`: An enumeration value of type `tty_code_code` representing the specific terminal capability code to check for.
- **Control Flow**:
    - The function accesses the `codes` array of the `tty_term` structure using the provided `code` as an index.
    - It checks if the `type` of the `tty_code` at that index is not equal to `TTYCODE_NONE`.
    - The function returns the result of this comparison as a boolean value.
- **Output**:
    - The function returns an integer value, which is non-zero (true) if the specified terminal capability code is present, and zero (false) if it is not.


---
### tty_term_string<!-- {{#callable:tty_term_string}} -->
The `tty_term_string` function retrieves a string value associated with a specific terminal capability code from a terminal structure.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal from which the string capability is to be retrieved.
    - `code`: An enumeration value of type `tty_code_code` representing the specific terminal capability code to be accessed.
- **Control Flow**:
    - Check if the terminal has the specified capability using [`tty_term_has`](#tty_term_has); if not, return an empty string.
    - Verify that the capability type is `TTYCODE_STRING`; if not, terminate the program with an error message.
    - Return the string value associated with the specified capability code from the terminal's codes array.
- **Output**:
    - Returns a constant character pointer to the string value of the specified terminal capability, or an empty string if the capability is not present.
- **Functions called**:
    - [`tty_term_has`](#tty_term_has)


---
### tty_term_string_i<!-- {{#callable:tty_term_string_i}} -->
The `tty_term_string_i` function retrieves and formats a terminal capability string with a single integer parameter.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal.
    - `code`: An enumeration value of type `tty_code_code` that specifies the terminal capability code to retrieve.
    - `a`: An integer parameter used in formatting the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Depending on the availability of `tiparm_s`, `tiparm`, or `tparm`, format the string `x` with the integer `a`.
    - If the formatted string `s` is `NULL`, log a debug message indicating the failure to expand the terminal capability and return an empty string.
    - Return the formatted string `s`.
- **Output**:
    - Returns a formatted terminal capability string, or an empty string if the expansion fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_string_ii<!-- {{#callable:tty_term_string_ii}} -->
The `tty_term_string_ii` function retrieves and formats a terminal capability string with two integer parameters for a given terminal and code.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal.
    - `code`: An enumeration value of type `tty_code_code` representing the terminal capability code.
    - `a`: An integer parameter used in formatting the terminal capability string.
    - `b`: Another integer parameter used in formatting the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string `x` using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Depending on the availability of `tiparm_s`, `tiparm`, or `tparm`, format the string `x` with the parameters `a` and `b`.
    - If the formatted string `s` is `NULL`, log a debug message indicating the failure to expand the terminal capability and return an empty string.
    - Return the formatted string `s`.
- **Output**:
    - Returns a formatted terminal capability string, or an empty string if the formatting fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_string_iii<!-- {{#callable:tty_term_string_iii}} -->
The `tty_term_string_iii` function retrieves and expands a terminal capability string with three integer parameters.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal.
    - `code`: An enumeration value of type `tty_code_code` representing the terminal capability code to be retrieved.
    - `a`: An integer parameter used for expanding the terminal capability string.
    - `b`: An integer parameter used for expanding the terminal capability string.
    - `c`: An integer parameter used for expanding the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Depending on the availability of `tiparm_s`, `tiparm`, or `tparm`, expand the capability string using the parameters `a`, `b`, and `c`.
    - If the expanded string is `NULL`, log a debug message indicating the failure to expand the capability and return an empty string.
    - Return the expanded capability string.
- **Output**:
    - Returns a pointer to the expanded terminal capability string, or an empty string if expansion fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_string_s<!-- {{#callable:tty_term_string_s}} -->
The `tty_term_string_s` function retrieves a terminal capability string and expands it using a single string parameter.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal from which the capability string is to be retrieved.
    - `code`: An enumeration value of type `tty_code_code` that specifies which terminal capability string to retrieve.
    - `a`: A constant character pointer representing a string parameter to be used in expanding the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string `x` using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Check for the availability of `tiparm_s`, `tiparm`, or `tparm` functions to expand the string `x` with the parameter `a`.
    - If `HAVE_TIPARM_S` is defined, use `tiparm_s` to expand `x` with `a`.
    - If `HAVE_TIPARM` is defined, use `tiparm` to expand `x` with `a`.
    - Otherwise, use `tparm` to expand `x` with `a` cast to a long integer.
    - If the expanded string `s` is `NULL`, log a debug message indicating the failure to expand the terminal capability and return an empty string.
    - Return the expanded string `s`.
- **Output**:
    - The function returns a constant character pointer to the expanded terminal capability string, or an empty string if expansion fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_string_ss<!-- {{#callable:tty_term_string_ss}} -->
The `tty_term_string_ss` function retrieves a terminal capability string and expands it using two string parameters.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal.
    - `code`: An enumeration value of type `tty_code_code` representing the terminal capability code to be retrieved.
    - `a`: A string parameter used for expanding the terminal capability string.
    - `b`: Another string parameter used for expanding the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Depending on the availability of `tiparm_s`, `tiparm`, or `tparm`, expand the capability string using the parameters `a` and `b`.
    - If the expansion result `s` is `NULL`, log a debug message indicating the failure to expand the capability and return an empty string.
    - Return the expanded string `s`.
- **Output**:
    - The function returns the expanded terminal capability string, or an empty string if the expansion fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_number<!-- {{#callable:tty_term_number}} -->
The `tty_term_number` function retrieves the numeric value of a specified terminal capability code from a terminal structure if it exists and is of the correct type.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal from which the numeric capability value is to be retrieved.
    - `code`: An enumeration value of type `tty_code_code` representing the specific terminal capability code to be checked and retrieved.
- **Control Flow**:
    - Check if the terminal has the specified capability code using [`tty_term_has`](#tty_term_has); if not, return 0.
    - Verify that the capability code is of type `TTYCODE_NUMBER`; if not, terminate the program with an error message using `fatalx`.
    - Return the numeric value associated with the specified capability code from the terminal structure.
- **Output**:
    - Returns the numeric value of the specified terminal capability code if it exists and is of type `TTYCODE_NUMBER`; otherwise, returns 0 or terminates the program if the type is incorrect.
- **Functions called**:
    - [`tty_term_has`](#tty_term_has)


---
### tty_term_flag<!-- {{#callable:tty_term_flag}} -->
The `tty_term_flag` function checks if a terminal capability code is a flag and returns its boolean value.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal.
    - `code`: An enumeration value of type `tty_code_code` representing the terminal capability code to be checked.
- **Control Flow**:
    - Check if the terminal has the specified capability code using [`tty_term_has`](#tty_term_has); if not, return 0.
    - Verify if the type of the specified code in the terminal is `TTYCODE_FLAG`; if not, call `fatalx` to handle the error.
    - Return the boolean flag value of the specified code from the terminal's codes.
- **Output**:
    - Returns an integer representing the boolean value of the flag (1 for true, 0 for false) if the code is a flag; otherwise, it may terminate the program with an error.
- **Functions called**:
    - [`tty_term_has`](#tty_term_has)


---
### tty_term_describe<!-- {{#callable:tty_term_describe}} -->
The `tty_term_describe` function generates a human-readable description of a terminal capability code from a given terminal structure.
- **Inputs**:
    - `term`: A pointer to a `struct tty_term` which contains terminal capability codes.
    - `code`: An `enum tty_code_code` representing the specific terminal capability code to describe.
- **Control Flow**:
    - The function initializes a static character array `s` to store the description and a local character array `out` for intermediate string processing.
    - It uses a switch statement to determine the type of the terminal code (`TTYCODE_NONE`, `TTYCODE_STRING`, `TTYCODE_NUMBER`, or `TTYCODE_FLAG`).
    - For `TTYCODE_NONE`, it formats a string indicating the code is missing.
    - For `TTYCODE_STRING`, it uses [`strnvis`](compat/vis.c.driver.md#strnvis) to make the string value visible and formats it into the description.
    - For `TTYCODE_NUMBER`, it formats the number value into the description.
    - For `TTYCODE_FLAG`, it formats the flag value as either 'true' or 'false' into the description.
    - Finally, it returns the formatted description string `s`.
- **Output**:
    - A static character string containing the formatted description of the terminal capability code.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`strnvis`](compat/vis.c.driver.md#strnvis)


