# Purpose
This C source code file is part of the `tmux` terminal multiplexer project and is responsible for managing terminal capabilities and features. It defines structures and functions to handle terminal capabilities, such as strings, numbers, and flags, which are essential for interacting with different types of terminal emulators. The file includes a comprehensive list of terminal capabilities, each associated with a specific type (string, number, or flag), and provides mechanisms to read, apply, and override these capabilities based on the terminal's characteristics.

The code is organized around several key components: the `tty_code` and `tty_term_code_entry` structures, which define the types and names of terminal capabilities; functions like [`tty_term_create`](#tty_term_create), [`tty_term_apply`](#tty_term_apply), and [`tty_term_apply_overrides`](#tty_term_apply_overrides), which manage the initialization and customization of terminal capabilities; and utility functions for reading and freeing terminal capability lists. The file also includes logic to handle terminal-specific features and flags, such as RGB color support and cursor movement capabilities. This code is integral to ensuring that `tmux` can effectively communicate with and control various terminal types, adapting its behavior to the specific features and limitations of each terminal environment.
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
- **Description**: The `tty_term_strip` function is a static function that processes a given string to remove any terminal padding sequences, which are sequences starting with `$<` and ending with `>`. It returns a newly allocated string with these sequences removed.
- **Use**: This function is used to clean up terminal capability strings by stripping out padding sequences, which are not needed for certain operations.


---
### tty_terms
- **Type**: `struct tty_terms`
- **Description**: The `tty_terms` variable is a global instance of the `struct tty_terms`, initialized using the `LIST_HEAD_INITIALIZER` macro. This suggests that `tty_terms` is intended to be the head of a linked list structure, likely used to manage terminal-related data or configurations.
- **Use**: This variable is used as the head of a linked list to manage terminal configurations or instances within the program.


---
### tty_term_codes
- **Type**: ``struct tty_term_code_entry[]``
- **Description**: The `tty_term_codes` is a static constant array of `struct tty_term_code_entry` that defines terminal capability codes. Each entry in the array consists of a type (such as string, number, or flag) and a corresponding name that represents a terminal capability. This array is used to map terminal capabilities to their respective codes and types.
- **Use**: This variable is used to define and access terminal capabilities by their codes and types within the program.


# Data Structures

---
### tty_code_type
- **Type**: `enum`
- **Members**:
    - `TTYCODE_NONE`: Represents a terminal code type that is not set or is undefined.
    - `TTYCODE_STRING`: Represents a terminal code type that is a string.
    - `TTYCODE_NUMBER`: Represents a terminal code type that is a number.
    - `TTYCODE_FLAG`: Represents a terminal code type that is a boolean flag.
- **Description**: The `tty_code_type` enumeration defines different types of terminal codes that can be used in terminal handling. It includes four types: `TTYCODE_NONE` for undefined or unset codes, `TTYCODE_STRING` for codes that are represented as strings, `TTYCODE_NUMBER` for codes that are represented as numbers, and `TTYCODE_FLAG` for codes that are represented as boolean flags. This enumeration is used to categorize terminal capabilities and settings in terminal management systems.


---
### tty_code
- **Type**: `struct`
- **Members**:
    - `type`: An enumeration of type `tty_code_type` that specifies the type of value stored in the union.
    - `value`: A union that can store a string, a number, or a flag, depending on the `type`.
- **Description**: The `tty_code` structure is used to represent a terminal capability code, which can be of different types such as a string, a number, or a flag. The `type` member indicates the kind of data stored in the `value` union, allowing the structure to flexibly handle different types of terminal capabilities. This structure is part of a larger system for managing terminal capabilities in a terminal multiplexer like tmux.


---
### tty_term_code_entry
- **Type**: `struct`
- **Members**:
    - `type`: Specifies the type of terminal code, represented by an enum tty_code_type.
    - `name`: A constant character pointer representing the name of the terminal code.
- **Description**: The `tty_term_code_entry` structure is used to define terminal code entries, each consisting of a type and a name. The type is an enumeration that indicates the kind of terminal code, such as a string, number, or flag, while the name is a string that identifies the specific terminal capability or feature. This structure is part of a larger system for managing terminal capabilities in a terminal multiplexer like tmux.


# Functions

---
### tty_term_ncodes<!-- {{#callable:tty_term_ncodes}} -->
The function `tty_term_ncodes` returns the number of terminal code entries defined in the `tty_term_codes` array.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls the `nitems` macro with `tty_term_codes` as its argument.
    - The `nitems` macro calculates the number of elements in the `tty_term_codes` array.
    - The function returns the result of the `nitems` macro call.
- **Output**:
    - The function returns an unsigned integer representing the number of terminal code entries.


---
### tty_term_strip<!-- {{#callable:tty_term_strip}} -->
The `tty_term_strip` function removes terminal padding sequences from a given string and returns a newly allocated copy of the stripped string.
- **Inputs**:
    - `s`: A constant character pointer representing the input string that may contain terminal padding sequences.
- **Control Flow**:
    - Check if the input string contains a '$' character; if not, return a duplicate of the input string using `xstrdup`.
    - Initialize a buffer and a length counter to store the stripped string.
    - Iterate over each character in the input string.
    - If a '$<' sequence is found, skip characters until a '>' is encountered, effectively ignoring the padding sequence.
    - Copy non-padding characters to the buffer until the buffer is full or the end of the string is reached.
    - Terminate the buffer with a null character.
    - Return a duplicate of the buffer using `xstrdup`.
- **Output**:
    - A newly allocated string that is a copy of the input string with terminal padding sequences removed.


---
### tty_term_override_next<!-- {{#callable:tty_term_override_next}} -->
The `tty_term_override_next` function extracts the next segment from a colon-separated string, handling escaped colons, and updates the offset for subsequent calls.
- **Inputs**:
    - `s`: A constant character pointer to a colon-separated string from which segments are extracted.
    - `offset`: A pointer to a size_t variable that indicates the current position in the string and is updated to the position after the extracted segment.
- **Control Flow**:
    - Initialize a static buffer `value` to store the extracted segment and set `n` to 0 for tracking the buffer index.
    - Check if the character at the current offset in the string `s` is the null terminator; if so, return NULL indicating no more segments.
    - Enter a loop to process characters in the string starting from the current offset until a null terminator is encountered.
    - If a colon is found, check if it is followed by another colon (indicating an escaped colon); if so, add a single colon to `value` and skip the next character.
    - If a colon is found without another colon following it, break the loop as it marks the end of the current segment.
    - Add non-colon characters to the `value` buffer and increment the offset.
    - If the buffer `value` reaches its maximum size minus one, return NULL to prevent overflow.
    - After exiting the loop, update the offset to point to the character after the current segment or to the end of the string if no more characters are left.
    - Terminate the `value` buffer with a null character and return it.
- **Output**:
    - Returns a pointer to a static buffer containing the next segment of the string, or NULL if no more segments are available or if an error occurs.


---
### tty_term_apply<!-- {{#callable:tty_term_apply}} -->
The `tty_term_apply` function processes and applies terminal capability overrides to a given terminal structure based on specified capabilities and a quiet flag.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal to which the capabilities will be applied.
    - `capabilities`: A string containing terminal capability overrides, formatted as key-value pairs separated by colons.
    - `quiet`: An integer flag indicating whether to suppress debug logging (non-zero value) or not (zero value).
- **Control Flow**:
    - Initialize local variables including pointers and counters.
    - Iterate over each capability override string obtained from [`tty_term_override_next`](#tty_term_override_next).
    - For each capability string, determine if it is a removal (ends with '@') or an assignment (contains '='), and extract the value if applicable.
    - Log the override action if the quiet flag is not set.
    - Iterate over all terminal codes to find a matching capability name.
    - If a match is found, update the terminal's code entry based on the type (string, number, or flag) and the action (remove or assign).
    - Free any dynamically allocated memory for the value string after processing.
- **Output**:
    - The function does not return a value; it modifies the `tty_term` structure in place by applying the specified capability overrides.
- **Functions called**:
    - [`tty_term_override_next`](#tty_term_override_next)
    - [`tty_term_ncodes`](#tty_term_ncodes)


---
### tty_term_apply_overrides<!-- {{#callable:tty_term_apply_overrides}} -->
The `tty_term_apply_overrides` function applies terminal capability overrides and updates various terminal flags based on the terminal's capabilities.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal whose capabilities and flags are to be updated.
- **Control Flow**:
    - Retrieve the 'terminal-overrides' option from global options and iterate over its array items.
    - For each item, extract the string value and parse it to find the next override using [`tty_term_override_next`](#tty_term_override_next).
    - If the override matches the terminal's name, apply it using [`tty_term_apply`](#tty_term_apply).
    - Log the current state of the SIXEL flag.
    - Check if the terminal supports RGB colors using [`tty_term_has`](#tty_term_has) for `TTYC_SETRGBF` and `TTYC_SETRGBB`, and update the `TERM_RGBCOLOURS` flag accordingly.
    - Log the current state of the RGBCOLOURS flag.
    - Check for margin capabilities (`TTYC_CMG` and `TTYC_CLMG`) and update the `TERM_DECSLRM` flag accordingly.
    - Log the current state of the DECSLRM flag.
    - Check for rectangle capability (`TTYC_RECT`) and update the `TERM_DECFRA` flag accordingly.
    - Log the current state of the DECFRA flag.
    - Check for the auto right margin flag (`TTYC_AM`) and update the `TERM_NOAM` flag accordingly.
    - Log the current state of the NOAM flag.
    - Initialize the ACS (Alternate Character Set) table to zero and populate it based on the `TTYC_ACSC` capability or a default ASCII mapping.
- **Output**:
    - The function does not return a value; it modifies the `tty_term` structure in place, updating its capabilities and flags.
- **Functions called**:
    - [`tty_term_override_next`](#tty_term_override_next)
    - [`tty_term_apply`](#tty_term_apply)
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
    - Iterate over the provided capabilities (`caps`) and match them against known terminal codes, storing the parsed values in the terminal's `codes` array.
    - Retrieve and apply terminal features from global options, matching them against the terminal name and updating the `feat` array.
    - Delete any existing curses data if the ncurses version is greater than 5.6.
    - Apply any terminal capability overrides from global options.
    - Check for the presence of essential terminal capabilities (`clear` and `cup`), and if missing, set an error message and exit with failure.
    - Determine if the terminal is VT100-like based on certain capabilities and add related features if applicable.
    - Add RGB color support features if the terminal supports RGB colors but lacks specific RGB setting capabilities.
    - Reapply features and overrides to ensure all settings are up-to-date.
    - Log the terminal's capabilities for debugging purposes.
    - Return the initialized terminal structure, or `NULL` if an error occurred.
- **Output**:
    - Returns a pointer to the newly created `tty_term` structure if successful, or `NULL` if an error occurs, with an error message stored in `cause`.
- **Functions called**:
    - [`tty_term_ncodes`](#tty_term_ncodes)
    - [`tty_term_strip`](#tty_term_strip)
    - [`tty_term_override_next`](#tty_term_override_next)
    - [`tty_term_apply_overrides`](#tty_term_apply_overrides)
    - [`tty_term_has`](#tty_term_has)
    - [`tty_term_string`](#tty_term_string)
    - [`tty_term_flag`](#tty_term_flag)
    - [`tty_term_describe`](#tty_term_describe)
    - [`tty_term_free`](#tty_term_free)


---
### tty_term_free<!-- {{#callable:tty_term_free}} -->
The `tty_term_free` function deallocates memory and resources associated with a `tty_term` structure.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure that needs to be freed.
- **Control Flow**:
    - Logs the removal of the terminal using its name.
    - Iterates over each code in the `term->codes` array, checking if the code type is `TTYCODE_STRING`.
    - If the code type is `TTYCODE_STRING`, it frees the associated string value.
    - Frees the `term->codes` array itself.
    - Removes the `term` from the list of terminal entries using `LIST_REMOVE`.
    - Frees the memory allocated for `term->name`.
    - Frees the memory allocated for the `term` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the `tty_term` structure.
- **Functions called**:
    - [`tty_term_ncodes`](#tty_term_ncodes)


---
### tty_term_read_list<!-- {{#callable:tty_term_read_list}} -->
The `tty_term_read_list` function reads terminal capabilities from the terminfo database for a specified terminal type and stores them in a dynamically allocated list.
- **Inputs**:
    - `name`: A string representing the name of the terminal type to be queried.
    - `fd`: An integer file descriptor used for terminal setup.
    - `caps`: A pointer to a pointer to a char array, which will be allocated and filled with terminal capabilities.
    - `ncaps`: A pointer to an unsigned integer that will store the number of capabilities read.
    - `cause`: A pointer to a char pointer that will store an error message if the function fails.
- **Control Flow**:
    - Call `setupterm` to initialize the terminal type specified by `name` using the file descriptor `fd` and check for errors.
    - If `setupterm` fails, set an appropriate error message in `cause` and return -1.
    - Initialize `ncaps` to 0 and `caps` to NULL to prepare for storing capabilities.
    - Iterate over all terminal codes using [`tty_term_ncodes`](#tty_term_ncodes) to determine the number of codes.
    - For each terminal code, determine its type and retrieve its value using `tigetstr`, `tigetnum`, or `tigetflag` as appropriate.
    - If a capability is valid, format it as a string and append it to the `caps` array, incrementing `ncaps`.
    - If the terminal library version is compatible, call `del_curterm` to clean up the current terminal structure.
    - Return 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 on failure with an error message set in `cause`.
- **Functions called**:
    - [`tty_term_ncodes`](#tty_term_ncodes)


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
The `tty_term_has` function checks if a specific terminal capability code is present in a terminal's code list.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal whose capabilities are being checked.
    - `code`: An enumeration value of type `tty_code_code` representing the specific terminal capability code to check for.
- **Control Flow**:
    - The function accesses the `codes` array of the `tty_term` structure using the provided `code` as an index.
    - It checks if the `type` of the `tty_code` at that index is not equal to `TTYCODE_NONE`.
    - The function returns the result of this comparison as a boolean value.
- **Output**:
    - The function returns an integer that is non-zero (true) if the specified terminal capability code is present, and zero (false) if it is not.


---
### tty_term_string<!-- {{#callable:tty_term_string}} -->
The `tty_term_string` function retrieves a string value associated with a specific terminal capability code from a terminal structure.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal from which the string capability is to be retrieved.
    - `code`: An enumeration value of type `tty_code_code` representing the specific terminal capability code to be accessed.
- **Control Flow**:
    - Check if the terminal has the specified capability using [`tty_term_has`](#tty_term_has); if not, return an empty string.
    - Verify that the capability type is `TTYCODE_STRING`; if not, trigger a fatal error with `fatalx`.
    - Return the string value associated with the specified capability code from the terminal's codes array.
- **Output**:
    - Returns a constant character pointer to the string value of the specified terminal capability, or an empty string if the capability is not present.
- **Functions called**:
    - [`tty_term_has`](#tty_term_has)


---
### tty_term_string_i<!-- {{#callable:tty_term_string_i}} -->
The `tty_term_string_i` function retrieves and expands a terminal capability string with a single integer parameter.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal.
    - `code`: An enumeration value of type `tty_code_code` that specifies the terminal capability code to retrieve.
    - `a`: An integer parameter used to expand the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Depending on the availability of `tiparm_s`, `tiparm`, or `tparm`, expand the capability string `x` using the integer parameter `a`.
    - If the expansion result `s` is `NULL`, log a debug message indicating the failure to expand the capability and return an empty string.
    - Return the expanded string `s`.
- **Output**:
    - The function returns a pointer to the expanded terminal capability string, or an empty string if expansion fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_string_ii<!-- {{#callable:tty_term_string_ii}} -->
The `tty_term_string_ii` function retrieves and expands a terminal capability string with two integer parameters for a given terminal and capability code.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal for which the capability string is being retrieved.
    - `code`: An enumeration value of type `tty_code_code` that specifies the terminal capability code to be expanded.
    - `a`: An integer parameter used in the expansion of the terminal capability string.
    - `b`: Another integer parameter used in the expansion of the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string `x` using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Depending on the availability of `tiparm_s`, `tiparm`, or `tparm`, call the appropriate function to expand the capability string `x` with parameters `a` and `b`.
    - If the expanded string `s` is `NULL`, log a debug message indicating the failure to expand the capability and return an empty string.
    - Return the expanded string `s`.
- **Output**:
    - The function returns a pointer to the expanded terminal capability string, or an empty string if expansion fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_string_iii<!-- {{#callable:tty_term_string_iii}} -->
The `tty_term_string_iii` function retrieves and formats a terminal capability string with three integer parameters.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal.
    - `code`: An enumeration value of type `tty_code_code` representing the terminal capability code.
    - `a`: An integer parameter to be used in formatting the terminal capability string.
    - `b`: An integer parameter to be used in formatting the terminal capability string.
    - `c`: An integer parameter to be used in formatting the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Depending on the availability of `tiparm_s`, `tiparm`, or `tparm`, format the string `x` with the parameters `a`, `b`, and `c`.
    - If the formatted string `s` is `NULL`, log a debug message indicating the failure to expand the terminal capability and return an empty string.
    - Return the formatted string `s`.
- **Output**:
    - A formatted terminal capability string, or an empty string if the expansion fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_string_s<!-- {{#callable:tty_term_string_s}} -->
The `tty_term_string_s` function retrieves a terminal capability string and formats it with a single string parameter using the appropriate parameter expansion function.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal from which the capability string is to be retrieved.
    - `code`: An enumeration value of type `tty_code_code` that specifies the terminal capability code to be retrieved.
    - `a`: A constant character pointer representing a string parameter to be used in formatting the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string corresponding to the given `code` using [`tty_term_string`](#tty_term_string).
    - Check for the availability of different parameter expansion functions (`tiparm_s`, `tiparm`, or `tparm`) and use the appropriate one to format the string with the parameter `a`.
    - If the formatted string is `NULL`, log a debug message indicating the failure to expand the terminal capability and return an empty string.
    - Return the formatted terminal capability string.
- **Output**:
    - The function returns a formatted terminal capability string, or an empty string if the expansion fails.
- **Functions called**:
    - [`tty_term_string`](#tty_term_string)


---
### tty_term_string_ss<!-- {{#callable:tty_term_string_ss}} -->
The `tty_term_string_ss` function retrieves a terminal capability string and expands it using two string parameters.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal from which the capability string is retrieved.
    - `code`: An enumeration value of type `tty_code_code` that specifies the terminal capability code to be retrieved.
    - `a`: A string parameter used in the expansion of the terminal capability string.
    - `b`: Another string parameter used in the expansion of the terminal capability string.
- **Control Flow**:
    - Retrieve the terminal capability string using [`tty_term_string`](#tty_term_string) with the given `term` and `code`.
    - Depending on the availability of `tiparm_s`, `tiparm`, or `tparm`, expand the capability string using the parameters `a` and `b`.
    - If the expansion fails (i.e., `s` is `NULL`), log a debug message indicating the failure to expand the capability and return an empty string.
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
    - Verify that the capability code's type is `TTYCODE_NUMBER`; if not, terminate the program with an error message using `fatalx`.
    - Return the numeric value associated with the specified capability code from the terminal structure.
- **Output**:
    - The function returns the numeric value of the specified terminal capability code if it exists and is of type `TTYCODE_NUMBER`; otherwise, it returns 0 or terminates the program if the type is incorrect.
- **Functions called**:
    - [`tty_term_has`](#tty_term_has)


---
### tty_term_flag<!-- {{#callable:tty_term_flag}} -->
The `tty_term_flag` function checks if a terminal capability code is a flag and returns its boolean value.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal.
    - `code`: An enumeration value of type `tty_code_code` representing the terminal capability code to check.
- **Control Flow**:
    - Check if the terminal has the specified capability code using [`tty_term_has`](#tty_term_has); if not, return 0.
    - Verify if the type of the specified code is `TTYCODE_FLAG`; if not, terminate the program with an error message.
    - Return the boolean value of the flag associated with the specified code.
- **Output**:
    - Returns an integer representing the boolean value of the flag (1 for true, 0 for false) if the code is a flag; otherwise, it may terminate the program if the code is not a flag.
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
    - It uses a switch statement to determine the type of the terminal code (none, string, number, or flag) from the `term->codes[code].type`.
    - For `TTYCODE_NONE`, it formats a string indicating the code is missing and stores it in `s`.
    - For `TTYCODE_STRING`, it uses `strnvis` to convert the string value to a visible format and formats it into `s` with the code name and string value.
    - For `TTYCODE_NUMBER`, it formats the number value into `s` with the code name and number value.
    - For `TTYCODE_FLAG`, it formats a boolean flag into `s` with the code name and either 'true' or 'false' based on the flag's value.
    - Finally, the function returns the formatted string `s`.
- **Output**:
    - A pointer to a static character array containing the formatted description of the terminal capability code.


# Function Declarations (Public API)

---
### tty_term_strip<!-- {{#callable_declaration:tty_term_strip}} -->
Removes terminal padding sequences from a string.
- **Description**: Use this function to remove terminal padding sequences from a given string, which are sequences starting with '$<' and ending with '>'. This is useful when you need a clean string without terminal-specific control sequences. The function returns a newly allocated string with the padding sequences removed. It should be called with a valid, non-null string. If the input string does not contain any padding sequences, a duplicate of the original string is returned. The caller is responsible for freeing the returned string.
- **Inputs**:
    - `s`: A constant character pointer to the input string. It must not be null, and the function expects a valid C string. The caller retains ownership of the input string.
- **Output**: A newly allocated string with terminal padding sequences removed. The caller is responsible for freeing this string.
- **See also**: [`tty_term_strip`](#tty_term_strip)  (Implementation)


