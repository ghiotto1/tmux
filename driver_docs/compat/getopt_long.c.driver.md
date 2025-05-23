# Purpose
This C source code file provides an implementation of the [`getopt`](#getopt) function, which is used for parsing command-line options in programs. The file is part of the OpenSSH project and is designed to offer compatibility with systems that may not have a native implementation of [`getopt`](#getopt) or [`getopt_long`](#getopt_long). The code includes definitions for handling both short and long command-line options, allowing for flexible parsing of arguments passed to a program. It defines several global variables such as `opterr`, `optind`, `optopt`, `optreset`, and `optarg`, which are used to manage the state of option parsing and provide feedback to the calling program about the options processed.

The file includes a structure definition for `struct option`, which is used to describe long options, and several static functions that handle the internal logic of option parsing, such as [`getopt_internal`](#getopt_internal), [`parse_long_options`](#parse_long_options), and [`permute_args`](#permute_args). These functions work together to process the command-line arguments, handle errors, and manage the permutation of non-option arguments. The code is structured to be robust and flexible, supporting various configurations and behaviors through flags and environment variables. This implementation is particularly useful for ensuring consistent option parsing behavior across different platforms, especially in environments where the native [`getopt`](#getopt) functionality may be limited or absent.
# Imports and Dependencies

---
- `compat.h`
- `err.h`
- `getopt.h`
- `errno.h`
- `stdlib.h`
- `string.h`
- `stdarg.h`


# Global Variables

---
### opterr
- **Type**: `int`
- **Description**: The `opterr` variable is a global integer that determines whether error messages should be printed when an invalid option is encountered during command-line argument parsing. It is part of the implementation of the `getopt` function, which is used to parse command-line options.
- **Use**: `opterr` is used to control the printing of error messages when invalid options are processed by the `getopt` function.


---
### optind
- **Type**: `int`
- **Description**: The `optind` variable is an integer that serves as an index into the parent argument vector (argv). It is used to track the position of the next command-line argument to be processed by the getopt function.
- **Use**: `optind` is used to iterate over command-line arguments, allowing the getopt function to parse options and their arguments sequentially.


---
### optopt
- **Type**: `int`
- **Description**: The `optopt` variable is an integer that stores the last known option character that was checked for validity during option parsing. It is initialized to the character '?' to indicate an invalid option by default.
- **Use**: `optopt` is used to store the option character that caused an error, allowing the program to identify and handle invalid options during command-line argument parsing.


---
### optreset
- **Type**: `int`
- **Description**: The `optreset` variable is an integer used to reset the state of the `getopt` function, which is a standard library function for parsing command-line options. It is typically set to 1 to reinitialize the option parsing process, allowing `getopt` to be called multiple times within the same program.
- **Use**: `optreset` is used to reset the `getopt` function, enabling repeated parsing of command-line arguments.


---
### optarg
- **Type**: `char*`
- **Description**: The `optarg` variable is a global pointer to a character string that holds the argument associated with an option when using command-line argument parsing functions like `getopt`. It is used to store the value of an option's argument when the option requires one.
- **Use**: `optarg` is used to retrieve and store the argument value associated with a command-line option that requires an argument.


---
### place
- **Type**: `char*`
- **Description**: The `place` variable is a static character pointer initialized to `EMSG`, which is an empty string. It is used in the context of option letter processing within the getopt implementation. The static keyword restricts its visibility to the file scope, ensuring it is not accessible outside this file.
- **Use**: `place` is used to keep track of the current position in the option string being processed by the getopt functions.


---
### nonopt_start
- **Type**: `int`
- **Description**: The `nonopt_start` variable is a static integer initialized to -1, used to track the index of the first non-option argument in a command-line argument list. It is part of the logic for permuting arguments in the `getopt` implementation.
- **Use**: This variable is used to identify the starting position of non-option arguments for permutation purposes in command-line argument parsing.


---
### nonopt_end
- **Type**: `int`
- **Description**: The `nonopt_end` variable is a static integer initialized to -1, which indicates the position of the first option after non-option arguments in the argument vector. It is used in the context of argument permutation, where non-option arguments are moved to the end of the argument list.
- **Use**: This variable is used to track the position of the first option after non-option arguments for the purpose of permuting arguments in the `getopt` function.


---
### recargchar
- **Type**: ``const char[]``
- **Description**: The `recargchar` variable is a static constant character array that holds the error message format string "option requires an argument -- %c". This message is used to indicate that a command-line option requires an argument, and the placeholder `%c` is intended to be replaced by the specific option character that is missing its argument.
- **Use**: This variable is used to format error messages when a command-line option is missing its required argument.


---
### recargstring
- **Type**: ``const char[]``
- **Description**: The `recargstring` is a static constant character array that holds the error message format string "option requires an argument -- %s". It is used to indicate that a command-line option requires an argument, but none was provided.
- **Use**: This variable is used in error handling to format and display a message when a required argument for an option is missing.


---
### ambig
- **Type**: ``const char[]``
- **Description**: The `ambig` variable is a static constant character array that holds the error message string "ambiguous option -- %.*s". This message is used to indicate that a command-line option provided by the user is ambiguous, meaning it matches more than one possible option.
- **Use**: This variable is used in error handling to provide feedback to the user when an ambiguous command-line option is detected.


---
### noarg
- **Type**: ``const char[]``
- **Description**: The `noarg` variable is a static constant character array that holds the error message "option doesn't take an argument -- %.*s". It is used to inform the user that a particular command-line option does not accept an argument.
- **Use**: This variable is used in error handling to display a message when a command-line option is provided with an argument that it does not accept.


---
### illoptchar
- **Type**: ``const char[]``
- **Description**: The `illoptchar` variable is a static constant character array that holds the error message format string "unknown option -- %c". This string is used to indicate that an unrecognized option character was encountered during option parsing.
- **Use**: This variable is used in error handling to format and display a message when an unknown option character is detected.


---
### illoptstring
- **Type**: ``const char[]``
- **Description**: The `illoptstring` is a static constant character array that holds the error message format string "unknown option -- %s". This string is used to format error messages when an unrecognized option is encountered in command-line argument parsing.
- **Use**: This variable is used to provide a formatted error message when an unknown option is detected during command-line parsing.


# Data Structures

---
### option
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the name of the long option.
    - `has_arg`: An integer indicating if the option requires an argument, with possible values being no_argument, required_argument, or optional_argument.
    - `flag`: A pointer to an integer that, if not NULL, is set to val when the option is found.
    - `val`: An integer value that is set to *flag if flag is not NULL, otherwise it is returned as the value of the option.
- **Description**: The `option` structure is used to define long command-line options for parsing functions like `getopt_long`. It contains fields to specify the name of the option, whether it requires an argument, and how it should behave when encountered. The `flag` and `val` fields allow for flexible handling of options, either by setting a flag or returning a specific value.


# Functions

---
### gcd<!-- {{#callable:gcd}} -->
The `gcd` function calculates the greatest common divisor of two integers using the Euclidean algorithm.
- **Inputs**:
    - `a`: The first integer for which the greatest common divisor is to be calculated.
    - `b`: The second integer for which the greatest common divisor is to be calculated.
- **Control Flow**:
    - Initialize variable `c` as the remainder of `a` divided by `b`.
    - Enter a while loop that continues as long as `c` is not zero.
    - Inside the loop, assign the value of `b` to `a`, the value of `c` to `b`, and recalculate `c` as the remainder of `a` divided by `b`.
    - Exit the loop when `c` becomes zero, indicating that `b` is the greatest common divisor.
- **Output**:
    - Returns the greatest common divisor of the two input integers, `a` and `b`.


---
### permute_args<!-- {{#callable:permute_args}} -->
The `permute_args` function rearranges elements in an argument vector to move non-option arguments to the end, maintaining their relative order.
- **Inputs**:
    - `panonopt_start`: The starting index of the non-option arguments in the argument vector.
    - `panonopt_end`: The ending index of the non-option arguments in the argument vector.
    - `opt_end`: The ending index of the option arguments in the argument vector.
    - `nargv`: The argument vector, which is an array of strings representing command-line arguments.
- **Control Flow**:
    - Calculate the number of non-option arguments (`nnonopts`) and option arguments (`nopts`).
    - Determine the number of cycles (`ncycle`) using the greatest common divisor (GCD) of `nnonopts` and `nopts`.
    - Calculate the length of each cycle (`cyclelen`) as the total number of arguments divided by `ncycle`.
    - Iterate over each cycle, starting from `panonopt_end` and perform swaps to permute the arguments.
    - Within each cycle, adjust the position (`pos`) by adding `nopts` or subtracting `nnonopts` to cycle through the arguments.
    - Swap the elements at the current position with the starting position of the cycle to achieve the permutation.
- **Output**:
    - The function does not return a value; it modifies the input argument vector `nargv` in place to reorder the arguments.
- **Functions called**:
    - [`gcd`](#gcd)


---
### parse_long_options<!-- {{#callable:parse_long_options}} -->
The `parse_long_options` function parses long command-line options from an argument vector, handling both exact and partial matches, and manages option arguments and errors.
- **Inputs**:
    - `nargv`: A pointer to an array of strings representing the argument vector.
    - `options`: A string containing the legitimate option characters.
    - `long_options`: A pointer to an array of `struct option` which defines the long options.
    - `idx`: A pointer to an integer where the index of the matched option will be stored, if not NULL.
    - `short_too`: An integer flag indicating whether short options should also be considered.
- **Control Flow**:
    - Initialize `current_argv` to point to the current option being processed and set `match` to -1.
    - Increment the global `optind` to move to the next argument.
    - Check if the current argument contains an '=' character to separate the option from its argument.
    - Iterate over the `long_options` array to find a matching long option name.
    - If an exact match is found, set `match` to the index and break the loop.
    - If a partial match is found and `short_too` is not set or the length is greater than 1, set `match` to the index.
    - If multiple partial matches are found, print an error if `PRINT_ERROR` is set and return `BADCH`.
    - If a match is found, check if the option requires an argument and handle accordingly, setting `optarg` if necessary.
    - If the option requires an argument but none is provided, print an error if `PRINT_ERROR` is set and return `BADARG`.
    - If no match is found and `short_too` is set, decrement `optind` and return -1.
    - If no match is found and `short_too` is not set, print an error if `PRINT_ERROR` is set and return `BADCH`.
    - If `idx` is not NULL, store the index of the matched option in `*idx`.
    - If the matched option has a `flag`, set it to the option's `val` and return 0; otherwise, return the option's `val`.
- **Output**:
    - Returns the value associated with the matched option, `BADCH` for an unknown option, `BADARG` for a missing required argument, or -1 if `short_too` is set and no match is found.


---
### getopt_internal<!-- {{#callable:getopt_internal}} -->
The `getopt_internal` function parses command-line arguments, handling both short and long options, and supports GNU and POSIX conventions.
- **Inputs**:
    - `nargc`: The number of arguments in the argument vector `nargv`.
    - `nargv`: An array of strings representing the command-line arguments.
    - `options`: A string containing the legitimate option characters; if a character is followed by a colon, the option requires an argument.
    - `long_options`: An array of `struct option` that describes the long options to be recognized.
    - `idx`: A pointer to an integer that is set to the index of the current long option relative to `long_options`.
    - `flags`: Flags that modify the behavior of the function, such as `FLAG_PERMUTE`, `FLAG_ALLARGS`, and `FLAG_LONGONLY`.
- **Control Flow**:
    - Check if `options` is NULL and return -1 if true.
    - Initialize `optind` and `optreset` if `optind` is 0.
    - Determine if POSIXLY_CORRECT is set and adjust flags accordingly.
    - Set `optarg` to NULL and reset non-option indices if `optreset` is true.
    - Iterate over `nargv` to find options, handling non-options based on flags.
    - Check for long options if `long_options` is not NULL and conditions are met.
    - Parse short options and handle errors for unknown options or missing arguments.
    - Return the option character or error code as appropriate.
- **Output**:
    - The function returns the option character if a valid option is found, `INORDER` for non-options when `FLAG_ALLARGS` is set, `BADCH` for unknown options, `BADARG` for missing arguments, or -1 when no more options are available.
- **Functions called**:
    - [`permute_args`](#permute_args)
    - [`parse_long_options`](#parse_long_options)


---
### getopt<!-- {{#callable:getopt}} -->
The `getopt` function parses command-line options from an argument vector according to a specified options string.
- **Inputs**:
    - `nargc`: The number of arguments in the argument vector `nargv`.
    - `nargv`: An array of strings representing the argument vector to be parsed.
    - `options`: A string containing the legitimate option characters; if a character is followed by a colon, the option requires an argument.
- **Control Flow**:
    - The function calls [`getopt_internal`](#getopt_internal) with the provided arguments and additional parameters set to NULL and 0.
    - [`getopt_internal`](#getopt_internal) handles the parsing of the argument vector based on the options string, without the `FLAG_PERMUTE` flag, which is specific to BSD's implementation.
    - The function returns the result of [`getopt_internal`](#getopt_internal), which is the next option character or an error indicator.
- **Output**:
    - The function returns the next option character from the argument vector, or -1 if there are no more options, or '?' if an error occurs.
- **Functions called**:
    - [`getopt_internal`](#getopt_internal)


---
### getopt_long<!-- {{#callable:getopt_long}} -->
The `getopt_long` function parses command-line arguments, supporting both short and long options, and returns the next option character found.
- **Inputs**:
    - `nargc`: The number of arguments in the argument vector `nargv`.
    - `nargv`: An array of strings representing the command-line arguments.
    - `options`: A string containing the legitimate short option characters.
    - `long_options`: A pointer to an array of `struct option` structures, which define the long options.
    - `idx`: A pointer to an integer that, if not NULL, is set to the index of the current long option relative to `long_options`.
- **Control Flow**:
    - The function calls [`getopt_internal`](#getopt_internal) with the provided arguments and the `FLAG_PERMUTE` flag.
    - [`getopt_internal`](#getopt_internal) processes the argument vector, handling both short and long options, and manages option permutations if necessary.
    - The function returns the next option character or a special value indicating the end of options or an error.
- **Output**:
    - The function returns the next option character found, or -1 if there are no more options, or a special value indicating an error.
- **Functions called**:
    - [`getopt_internal`](#getopt_internal)


---
### getopt_long_only<!-- {{#callable:getopt_long_only}} -->
The `getopt_long_only` function parses command-line arguments, allowing both short and long options, with long options being recognized even if they are not prefixed by '--'.
- **Inputs**:
    - `nargc`: The number of arguments in the argument vector `nargv`.
    - `nargv`: An array of strings representing the command-line arguments.
    - `options`: A string containing the legitimate short option characters.
    - `long_options`: A pointer to an array of `struct option` which defines the long options.
    - `idx`: A pointer to an integer that, if not NULL, is set to the index of the long option relative to `long_options`.
- **Control Flow**:
    - The function calls [`getopt_internal`](#getopt_internal) with the provided arguments and additional flags `FLAG_PERMUTE|FLAG_LONGONLY`.
    - [`getopt_internal`](#getopt_internal) handles the parsing of both short and long options, with the `FLAG_LONGONLY` allowing long options to be recognized even if they are not prefixed by '--'.
    - The function returns the result of [`getopt_internal`](#getopt_internal), which is the next option character or an error code.
- **Output**:
    - The function returns an integer representing the next option character found, or an error code if an error occurs during parsing.
- **Functions called**:
    - [`getopt_internal`](#getopt_internal)


# Function Declarations (Public API)

---
### getopt_internal<!-- {{#callable_declaration:getopt_internal}} -->
Parse command-line options from an argument vector.
- **Description**: This function is used to parse command-line options from an argument vector, supporting both short and long options. It is typically called in a loop to process each option in turn. The function handles option arguments, optional arguments, and non-option arguments, and supports GNU-style option permutation. It must be called with a valid options string and can optionally handle long options if provided. The function modifies global variables such as `optind`, `optarg`, and `optopt` to reflect the current state of option parsing.
- **Inputs**:
    - `nargc`: The number of arguments in the argument vector `nargv`. Must be greater than zero.
    - `nargv`: An array of argument strings. The first element is typically the program name, and subsequent elements are the command-line arguments. Must not be null.
    - `options`: A string containing the legitimate option characters. If a character is followed by a colon, the option requires an argument. Must not be null.
    - `long_options`: An optional array of `struct option` that describes the long options. Can be null if long options are not used.
    - `idx`: An optional pointer to an integer that, if not null, is set to the index of the long option relative to `long_options` when a long option is found.
    - `flags`: Flags that modify the behavior of the function, such as `FLAG_PERMUTE` to allow option permutation and `FLAG_LONGONLY` to treat long options as short options if no match is found.
- **Output**: Returns the next option character, `-1` if there are no more options, or a special value indicating an error or non-option argument. Sets global variables `optarg`, `optind`, and `optopt` to reflect the current state of option parsing.
- **See also**: [`getopt_internal`](#getopt_internal)  (Implementation)


---
### parse_long_options<!-- {{#callable_declaration:parse_long_options}} -->
Parse long options in an argument vector.
- **Description**: This function is used to parse long options from an argument vector, typically used in command-line applications. It should be called when you need to handle long-form command-line options (e.g., --option) and can also handle short options if specified. The function updates global variables such as `optind`, `optarg`, and `optopt` to reflect the current state of option parsing. It returns specific error codes for unknown options or missing arguments, and can optionally update an index to the matched option. This function assumes that the argument vector and options are correctly formatted and that the `long_options` array is properly initialized.
- **Inputs**:
    - `nargv`: A pointer to an array of argument strings. The array must be null-terminated, and the caller retains ownership.
    - `options`: A string containing the legitimate option characters. Must not be null.
    - `long_options`: A pointer to an array of `struct option`, which specifies the long options to be recognized. The array must be terminated with an option with a null name.
    - `idx`: A pointer to an integer where the index of the current option is stored if a match is found. Can be null if the index is not needed.
    - `short_too`: An integer flag indicating whether short options should also be considered. Non-zero values enable this behavior.
- **Output**: Returns the value of the matched option, -1 if no match is found and `short_too` is set, or specific error codes for unknown options or missing arguments.
- **See also**: [`parse_long_options`](#parse_long_options)  (Implementation)


---
### gcd<!-- {{#callable_declaration:gcd}} -->
Compute the greatest common divisor of two integers.
- **Description**: Use this function to determine the greatest common divisor (GCD) of two non-negative integers. It is useful in mathematical computations where the GCD is required, such as simplifying fractions or finding common factors. The function expects both input integers to be non-negative, and it will return the GCD even if one of the integers is zero, in which case the GCD is the non-zero integer. The function does not handle negative inputs or non-integer values.
- **Inputs**:
    - `a`: A non-negative integer for which the GCD is to be calculated. If zero, the GCD is the value of b.
    - `b`: A non-negative integer for which the GCD is to be calculated. If zero, the GCD is the value of a.
- **Output**: The function returns the greatest common divisor of the two input integers.
- **See also**: [`gcd`](#gcd)  (Implementation)


---
### permute_args<!-- {{#callable_declaration:permute_args}} -->
Rearranges command-line arguments to move non-option arguments to the end.
- **Description**: This function is used to reorder command-line arguments such that all non-option arguments are moved to the end of the array, preserving the order of both option and non-option arguments within their respective groups. It is typically used in command-line parsing utilities to facilitate the processing of options before non-options. The function must be called with valid indices that define the boundaries of non-option and option arguments within the array. It does not handle invalid indices or null pointers, so care must be taken to ensure these preconditions are met.
- **Inputs**:
    - `panonopt_start`: The starting index of the non-option arguments in the array. Must be a valid index within the bounds of the array.
    - `panonopt_end`: The ending index (exclusive) of the non-option arguments in the array. Must be greater than or equal to panonopt_start and within the bounds of the array.
    - `opt_end`: The ending index (exclusive) of the option arguments in the array. Must be greater than or equal to panonopt_end and within the bounds of the array.
    - `nargv`: An array of strings representing command-line arguments. Must not be null, and the caller retains ownership of the array.
- **Output**: None
- **See also**: [`permute_args`](#permute_args)  (Implementation)


