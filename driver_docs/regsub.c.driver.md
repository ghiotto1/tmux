# Purpose
This C source code file provides a specialized function for performing regular expression-based substitutions on strings. The primary function, [`regsub`](#regsub), takes a pattern, a replacement string, a target text, and flags as input. It uses the POSIX regex library to compile the pattern and execute it against the target text. The function iterates through the text, identifying matches of the pattern and replacing them with the specified replacement string. It handles both non-empty and empty matches, ensuring that the entire text is processed correctly. The [`regsub_copy`](#regsub_copy) and [`regsub_expand`](#regsub_expand) helper functions are used to manage the dynamic allocation and expansion of the result buffer, ensuring that the substitutions are correctly applied and the resulting string is properly constructed.

The code is designed to be part of a larger application, as indicated by the inclusion of "tmux.h", suggesting it is part of the tmux terminal multiplexer project. The file does not define a public API or external interface but rather provides internal functionality that can be utilized by other components of the application. The use of static functions for [`regsub_copy`](#regsub_copy) and [`regsub_expand`](#regsub_expand) indicates that these are intended for use only within this file, encapsulating the logic for handling string manipulation during the substitution process. The code is robust in handling various edge cases, such as empty matches and anchored patterns, making it a reliable utility for text processing within the context of the application.
# Imports and Dependencies

---
- `sys/types.h`
- `regex.h`
- `string.h`
- `tmux.h`


# Functions

---
### regsub_copy<!-- {{#callable:regsub_copy}} -->
The `regsub_copy` function appends a specified substring from a source text to a dynamically allocated buffer, updating the buffer's length accordingly.
- **Inputs**:
    - `buf`: A pointer to a character buffer that will be reallocated and appended with a substring from the source text.
    - `len`: A pointer to a ssize_t variable representing the current length of the buffer, which will be updated after appending the substring.
    - `text`: The source text from which a substring will be copied.
    - `start`: The starting index in the source text from which the substring will be copied.
    - `end`: The ending index in the source text up to which the substring will be copied.
- **Control Flow**:
    - Calculate the length of the substring to be copied by subtracting 'start' from 'end'.
    - Reallocate the buffer pointed to by 'buf' to accommodate the new substring plus a null terminator.
    - Copy the substring from 'text' starting at 'start' and ending at 'end' into the buffer at the current length position.
    - Update the length of the buffer by adding the length of the copied substring.
- **Output**:
    - The function does not return a value, but it modifies the buffer and its length in place.


---
### regsub_expand<!-- {{#callable:regsub_expand}} -->
The `regsub_expand` function processes a substitution template string, expanding back-references to matched text segments and appending the result to a buffer.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the expanded result will be stored.
    - `len`: A pointer to a ssize_t variable that tracks the current length of the buffer.
    - `with`: A string containing the substitution template, which may include back-references to matched text segments.
    - `text`: The original text from which matched segments are extracted.
    - `m`: An array of regmatch_t structures representing the matches found in the text.
    - `n`: The number of matches in the array `m`.
- **Control Flow**:
    - Iterate over each character in the substitution template string `with`.
    - If a backslash is encountered, check if it is followed by a digit, indicating a back-reference.
    - If a valid back-reference is found, copy the corresponding matched text segment from `text` to `buf` using [`regsub_copy`](#regsub_copy).
    - If no valid back-reference is found, or if the character is not a backslash, append the character to `buf`.
- **Output**:
    - The function does not return a value; it modifies the buffer `buf` in place to include the expanded substitution result.
- **Functions called**:
    - [`regsub_copy`](#regsub_copy)


---
### regsub<!-- {{#callable:regsub}} -->
The `regsub` function performs a regular expression-based substitution on a given text, replacing matches of a pattern with a specified replacement string.
- **Inputs**:
    - `pattern`: A string representing the regular expression pattern to search for in the text.
    - `with`: A string representing the replacement text to substitute for each match of the pattern.
    - `text`: The input string on which the regular expression search and substitution is performed.
    - `flags`: An integer representing flags that modify the behavior of the regular expression compilation, such as case sensitivity.
- **Control Flow**:
    - Check if the input text is empty; if so, return an empty string duplicate.
    - Compile the regular expression pattern with the specified flags; if compilation fails, return NULL.
    - Initialize start, last, and end positions for processing the text.
    - Enter a loop to process the text from the start position to the end.
    - Use `regexec` to find matches of the pattern in the text starting from the current position.
    - If no match is found, copy the remaining text to the buffer and break the loop.
    - Copy text from the last match end to the current match start to the buffer.
    - Check if the last match was empty and handle the current match accordingly, either expanding it or moving the start position forward.
    - If the pattern is anchored to the start of the text (begins with '^'), copy the remaining text and break the loop.
    - Terminate the buffer with a null character and free the compiled regular expression resources.
- **Output**:
    - A dynamically allocated string containing the result of the substitution, or NULL if the regular expression compilation fails.
- **Functions called**:
    - [`regsub_copy`](#regsub_copy)
    - [`regsub_expand`](#regsub_expand)


