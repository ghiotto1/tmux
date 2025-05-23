# Purpose
This C source code file provides a specialized function for performing regular expression-based substitutions on strings. The primary function, [`regsub`](#regsub), takes a pattern, a replacement string, a target text, and flags as input, and returns a new string where occurrences of the pattern in the target text are replaced with the replacement string. The code utilizes the POSIX regex library to compile and execute regular expressions, and it handles the dynamic allocation of memory for the resulting string using helper functions [`regsub_copy`](#regsub_copy) and [`regsub_expand`](#regsub_expand). These helper functions manage the copying and expansion of text segments, ensuring that the resulting string is constructed correctly as matches are found and replaced.

The file is part of a larger project, as indicated by the inclusion of a custom header file "tmux.h", suggesting it is likely a component of the tmux terminal multiplexer. The code is designed to be robust, handling edge cases such as empty matches and anchored patterns. It does not define a public API or external interface directly but provides a utility function that can be used internally within the project to facilitate text processing tasks involving regular expressions. The use of static functions for [`regsub_copy`](#regsub_copy) and [`regsub_expand`](#regsub_expand) indicates that these are intended for internal use only, encapsulating the logic needed to support the main [`regsub`](#regsub) function.
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
    - `buf`: A pointer to a character buffer that will be reallocated and appended with a substring from 'text'.
    - `len`: A pointer to a ssize_t variable representing the current length of the buffer, which will be updated after appending the substring.
    - `text`: The source string from which a substring will be copied.
    - `start`: The starting index in 'text' from which the substring will be copied.
    - `end`: The ending index in 'text' up to which the substring will be copied.
- **Control Flow**:
    - Calculate the length of the substring to be copied as 'end - start'.
    - Reallocate the buffer 'buf' to accommodate the new substring plus a null terminator.
    - Copy the substring from 'text' starting at 'start' and ending at 'end' into the buffer at the current length position.
    - Update the length of the buffer by adding the length of the copied substring.
- **Output**:
    - The function does not return a value but modifies the buffer and its length in place.
- **Functions called**:
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


---
### regsub_expand<!-- {{#callable:regsub_expand}} -->
The `regsub_expand` function processes a substitution template string, expanding back-references to matched text segments and appending the result to a buffer.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the expanded result will be stored.
    - `len`: A pointer to a ssize_t variable that tracks the current length of the buffer.
    - `with`: A string containing the substitution template, which may include back-references to matched text segments.
    - `text`: The original text from which matched segments are extracted.
    - `m`: An array of regmatch_t structures representing the matches found in the text.
    - `n`: The number of valid matches in the array `m`.
- **Control Flow**:
    - Iterate over each character in the substitution template string `with`.
    - If a backslash is encountered, check if it is followed by a digit (0-9).
    - If a digit follows the backslash, interpret it as a back-reference to a matched segment in `text`.
    - If the back-reference is valid (within bounds and non-empty), call [`regsub_copy`](#regsub_copy) to append the matched segment to `buf`.
    - If no valid back-reference is found, or if the character is not a backslash, append the character directly to `buf`.
- **Output**:
    - The function does not return a value; it modifies the buffer `buf` in place to include the expanded substitution result.
- **Functions called**:
    - [`regsub_copy`](#regsub_copy)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


---
### regsub<!-- {{#callable:regsub}} -->
The `regsub` function performs a regular expression-based substitution on a given text, replacing matches of a pattern with a specified replacement string.
- **Inputs**:
    - `pattern`: A string representing the regular expression pattern to search for in the text.
    - `with`: A string representing the replacement text to substitute for each match of the pattern.
    - `text`: The input string on which the substitution is to be performed.
    - `flags`: An integer representing flags that modify the behavior of the regular expression compilation.
- **Control Flow**:
    - Initialize variables and compile the regular expression from the pattern using `regcomp`.
    - If the input text is empty, return an empty string; if `regcomp` fails, return NULL.
    - Iterate over the text using a loop, searching for matches of the pattern using `regexec`.
    - If no match is found, copy the remaining text to the buffer and break the loop.
    - If a match is found, copy the text preceding the match to the buffer.
    - Check if the last match was empty and handle the current match accordingly, either expanding it or moving past it.
    - If the pattern is anchored to the start (begins with '^'), copy the remaining text and break the loop.
    - Terminate the buffer with a null character and free the compiled regular expression resources.
- **Output**:
    - A dynamically allocated string containing the result of the substitution, or NULL if an error occurs during regular expression compilation.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`regsub_copy`](#regsub_copy)
    - [`regsub_expand`](#regsub_expand)


