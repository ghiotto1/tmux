# Purpose
This C source file, `getopt_long.c`, is part of the OpenSSH project and provides compatibility functions for parsing command-line options, specifically implementing the [`getopt`](#getopt) functionality. The file is designed to be used in environments where the standard [`getopt`](#getopt) and [`getopt_long`](#getopt_long) functions are not available, ensuring that software relying on these functions can still operate correctly. The code includes definitions for handling both short and long command-line options, allowing for flexible command-line parsing. It defines several key variables and structures, such as `optarg`, `optind`, and the `option` struct, which are essential for tracking the state of option parsing and storing option arguments.

The file includes implementations of functions like [`getopt_internal`](#getopt_internal), [`parse_long_options`](#parse_long_options), and [`permute_args`](#permute_args), which are responsible for the core logic of option parsing. These functions handle various scenarios, such as detecting option arguments, managing non-option arguments, and supporting both short and long options. The code also includes error handling for unknown options and missing arguments, providing feedback through error messages. This file is a crucial component for ensuring compatibility with systems that lack native support for [`getopt`](#getopt) and [`getopt_long`](#getopt_long), making it a valuable utility for cross-platform software development.
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
- **Description**: The `opterr` variable is a global integer that determines whether error messages should be printed by the getopt function when an invalid option is encountered. It is initialized to 1, which means error messages are enabled by default.
- **Use**: This variable is used to control the printing of error messages during option parsing in command-line argument processing.


---
### optind
- **Type**: `int`
- **Description**: The `optind` variable is an integer that serves as an index into the parent argument vector (argv). It is used to track the current position in the argument list being processed by the getopt function.
- **Use**: `optind` is used to keep track of the next argument to be processed in the command-line argument list.


---
### optopt
- **Type**: `int`
- **Description**: The `optopt` variable is an integer that stores the character checked for validity during option parsing in command-line argument processing. It is used in conjunction with the `getopt` function to handle invalid or unrecognized options.
- **Use**: `optopt` is used to store the last known option character that was invalid or unrecognized during command-line option parsing.


---
### optreset
- **Type**: `int`
- **Description**: The `optreset` variable is an integer used to reset the state of the `getopt` function, which is a standard library function used for parsing command-line options. It is typically set to 1 to reinitialize the option parsing process, allowing `getopt` to be called multiple times within the same program execution.
- **Use**: `optreset` is used to reset the `getopt` function's internal state, enabling repeated parsing of command-line arguments.


---
### optarg
- **Type**: `char*`
- **Description**: The `optarg` variable is a global pointer to a character string that holds the argument associated with an option when using command-line argument parsing functions like `getopt`. It is used to store the value of an option's argument when the option requires or optionally accepts an argument.
- **Use**: `optarg` is used to retrieve the argument value associated with a command-line option when parsing arguments.


---
### place
- **Type**: `char*`
- **Description**: The `place` variable is a static character pointer initialized to `EMSG`, which is an empty string. It is used in the context of option letter processing within the getopt implementation.
- **Use**: This variable is used to track the current position in the option string being processed.


---
### nonopt_start
- **Type**: `int`
- **Description**: The `nonopt_start` variable is a static integer initialized to -1, used to track the index of the first non-option argument in a command-line argument list. It is part of the logic for permuting arguments in the `getopt` implementation.
- **Use**: This variable is used to identify the starting position of non-option arguments for permutation purposes in the argument parsing process.


---
### nonopt_end
- **Type**: `int`
- **Description**: The `nonopt_end` variable is a static integer initialized to -1, used to track the position of the first option after non-option arguments in a command-line argument list. It is part of the logic for permuting arguments, which allows non-option arguments to be moved to the end of the argument list.
- **Use**: `nonopt_end` is used in the permutation logic to identify and manage the transition between non-option and option arguments in the argument vector.


---
### recargchar
- **Type**: ``const char[]``
- **Description**: The `recargchar` variable is a static constant character array that holds an error message template. This template is used to indicate that an option requires an argument, with a placeholder for a character (`%c`).
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
- **Description**: The `noarg` variable is a static constant character array that holds an error message string. This string is used to indicate that a command-line option does not accept an argument.
- **Use**: This variable is used in error handling to provide a specific error message when a user attempts to pass an argument to an option that does not accept one.


---
### illoptchar
- **Type**: ``const char[]``
- **Description**: The `illoptchar` variable is a static constant character array that holds the error message format string "unknown option -- %c". This string is used to report an error when an unrecognized option character is encountered during command-line argument parsing.
- **Use**: This variable is used in error handling to format and display a message indicating an unknown option character.


---
### illoptstring
- **Type**: ``const char[]``
- **Description**: The `illoptstring` is a static constant character array that holds the error message format string "unknown option -- %s". It is used to format error messages when an unrecognized option is encountered during command-line argument parsing.
- **Use**: This variable is used to provide a template for error messages related to unknown command-line options.


# Data Structures

---
### option
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a string representing the name of the long option.
    - `has_arg`: An integer indicating if the option requires an argument, with possible values being no_argument, required_argument, or optional_argument.
    - `flag`: A pointer to an integer that, if not NULL, is set to val when the option is found.
    - `val`: An integer value that is set to *flag if flag is not NULL, otherwise it is returned as the value of the option.
- **Description**: The `option` structure is used to define long command-line options for parsing functions like `getopt_long`. It includes fields to specify the name of the option, whether it requires an argument, and how it should behave when encountered. The `flag` and `val` fields allow for flexible handling of options, either by setting a flag or returning a specific value.


# Functions

---
### gcd<!-- {{#callable:gcd}} -->
The `gcd` function calculates the greatest common divisor of two integers using the Euclidean algorithm.
- **Inputs**:
    - `a`: The first integer for which the greatest common divisor is to be calculated.
    - `b`: The second integer for which the greatest common divisor is to be calculated.
- **Control Flow**:
    - Initialize variable `c` with the remainder of `a` divided by `b`.
    - Enter a while loop that continues as long as `c` is not zero.
    - Inside the loop, assign the value of `b` to `a`, the value of `c` to `b`, and recalculate `c` as the remainder of `a` divided by `b`.
    - Exit the loop when `c` becomes zero.
    - Return the value of `b`, which is the greatest common divisor of the original `a` and `b`.
- **Output**:
    - The function returns an integer representing the greatest common divisor of the two input integers.


---
### permute_args<!-- {{#callable:permute_args}} -->
The `permute_args` function rearranges elements in an argument vector to move non-option arguments to the end, maintaining their order.
- **Inputs**:
    - `panonopt_start`: The starting index of the non-option arguments in the argument vector.
    - `panonopt_end`: The ending index of the non-option arguments in the argument vector.
    - `opt_end`: The ending index of the option arguments in the argument vector.
    - `nargv`: A constant pointer to an array of character pointers representing the argument vector.
- **Control Flow**:
    - Calculate the number of non-option arguments (`nnonopts`) and option arguments (`nopts`).
    - Determine the number of cycles (`ncycle`) using the greatest common divisor of `nnonopts` and `nopts`.
    - Calculate the length of each cycle (`cyclelen`) as the total number of arguments divided by `ncycle`.
    - Iterate over each cycle, starting from the end of non-option arguments (`cstart`).
    - For each cycle, iterate over its length, adjusting the position (`pos`) by adding `nopts` or subtracting `nnonopts` to swap elements between non-option and option blocks.
    - Perform the swap of elements in the argument vector to achieve the permutation.
- **Output**:
    - The function does not return a value; it modifies the input argument vector `nargv` in place.
- **Functions called**:
    - [`gcd`](#gcd)


---
### parse_long_options<!-- {{#callable:parse_long_options}} -->
The `parse_long_options` function parses long command-line options from an argument vector, handling both exact and partial matches, and returns the appropriate option value or error code.
- **Inputs**:
    - `nargv`: A pointer to an array of command-line argument strings.
    - `options`: A string containing the legitimate option characters.
    - `long_options`: A pointer to an array of `struct option`, which defines the long options to be recognized.
    - `idx`: A pointer to an integer where the index of the matched long option will be stored, if not NULL.
    - `short_too`: An integer flag indicating whether short options should also be considered.
- **Control Flow**:
    - Initialize `current_argv` to point to the current option being processed and set `match` to -1.
    - Increment the global `optind` to move to the next argument.
    - Check if the current argument contains an '=' character to separate option and argument; if so, split them.
    - Iterate over the `long_options` array to find a matching long option name.
    - If an exact match is found, set `match` to the index and break the loop.
    - If a partial match is found and `short_too` is not set or the match is not ambiguous, set `match` to the index.
    - If an ambiguous match is detected, print an error if `PRINT_ERROR` is set and return `BADCH`.
    - If a match is found, check if the option requires an argument and handle accordingly, setting `optarg` if necessary.
    - If the option requires an argument but none is provided, print an error if `PRINT_ERROR` is set and return `BADARG`.
    - If no match is found and `short_too` is set, decrement `optind` and return -1.
    - If no match is found and `short_too` is not set, print an error if `PRINT_ERROR` is set and return `BADCH`.
    - If `idx` is not NULL, store the index of the matched option in `*idx`.
    - If the matched option has a `flag`, set it to the option's `val` and return 0; otherwise, return the option's `val`.
- **Output**:
    - Returns the value associated with the matched long option, `BADCH` for an unknown option, `BADARG` for a missing required argument, or -1 if `short_too` is set and no match is found.
- **Functions called**:
    - [`warnx`](err.c.driver.md#warnx)


---
### getopt_internal<!-- {{#callable:getopt_internal}} -->
The `getopt_internal` function parses command-line arguments, handling both short and long options, and supports GNU and POSIX conventions.
- **Inputs**:
    - `nargc`: The number of arguments in the argument vector `nargv`.
    - `nargv`: An array of strings representing the command-line arguments.
    - `options`: A string containing the legitimate option characters; if a character is followed by a colon, the option requires an argument.
    - `long_options`: An array of `struct option` structures defining the long options.
    - `idx`: A pointer to an integer that, if not NULL, is set to the index of the long option relative to `long_options`.
    - `flags`: Flags that modify the behavior of the function, such as `FLAG_PERMUTE`, `FLAG_ALLARGS`, and `FLAG_LONGONLY`.
- **Control Flow**:
    - Check if `options` is NULL and return -1 if true.
    - Initialize `optind` and `optreset` if `optind` is 0.
    - Determine if POSIXLY_CORRECT is set and adjust flags accordingly.
    - Set `optarg` to NULL and reset non-option indices if `optreset` is true.
    - Iterate over `nargv` to find options, handling non-options and permutations based on flags.
    - Check for long options if `long_options` is not NULL and the current argument is not just '-'.
    - Parse long options using [`parse_long_options`](#parse_long_options) if applicable.
    - Handle short options by checking against `options` and setting `optarg` if the option requires an argument.
    - Return the option character or error codes like `BADCH` or `BADARG` based on the parsing results.
- **Output**:
    - The function returns the option character if a valid option is found, `INORDER` for non-options if `FLAG_ALLARGS` is set, or error codes like `BADCH` or `BADARG` for invalid options or missing arguments.
- **Functions called**:
    - [`permute_args`](#permute_args)
    - [`parse_long_options`](#parse_long_options)
    - [`warnx`](err.c.driver.md#warnx)


---
### getopt<!-- {{#callable:getopt}} -->
The `getopt` function parses command-line options from an argument vector according to a specified options string.
- **Inputs**:
    - `nargc`: The number of arguments in the argument vector `nargv`.
    - `nargv`: The argument vector containing the command-line arguments to be parsed.
    - `options`: A string containing the legitimate option characters; if an option character is followed by a colon, the option requires an argument.
- **Control Flow**:
    - The function calls [`getopt_internal`](#getopt_internal) with the provided arguments and additional parameters set to `NULL` and `0`.
    - [`getopt_internal`](#getopt_internal) handles the parsing of the argument vector based on the options string, without the `FLAG_PERMUTE` flag, which means it does not rearrange non-option arguments.
    - The function returns the result of [`getopt_internal`](#getopt_internal), which is the next option character found, or -1 if there are no more options.
- **Output**:
    - The function returns the next option character found in the argument vector, or -1 if there are no more options to process.
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
    - `long_options`: An array of `struct option` structures defining the long options.
    - `idx`: A pointer to an integer that, if not NULL, is set to the index of the current long option relative to `long_options`.
- **Control Flow**:
    - The function calls [`getopt_internal`](#getopt_internal) with the provided arguments and additional flags `FLAG_PERMUTE|FLAG_LONGONLY`.
    - [`getopt_internal`](#getopt_internal) handles the parsing of both short and long options, with the `FLAG_LONGONLY` allowing long options to be recognized even if they are not prefixed by '--'.
    - The function returns the result of [`getopt_internal`](#getopt_internal), which is the next option character or an error code.
- **Output**:
    - The function returns the next option character parsed, or an error code if an error occurs during parsing.
- **Functions called**:
    - [`getopt_internal`](#getopt_internal)


