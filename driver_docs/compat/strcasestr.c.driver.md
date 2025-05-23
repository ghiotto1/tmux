# Purpose
This C source code file implements the [`strcasestr`](#strcasestr) function, which is a case-insensitive version of the standard `strstr` function. The purpose of this function is to locate the first occurrence of a substring (`find`) within a string (`s`), ignoring the case of both strings. The implementation uses the `tolower` function to perform case-insensitive comparisons and the `strncasecmp` function to compare the remaining parts of the strings once a potential match is found. This code is a utility function that can be used in applications where case-insensitive string searching is required. The file also includes a standard Berkeley Software Distribution (BSD) license, allowing for redistribution and use under specified conditions.
# Imports and Dependencies

---
- `ctype.h`
- `string.h`
- `compat.h`


# Functions

---
### strcasestr<!-- {{#callable:strcasestr}} -->
The `strcasestr` function searches for the first occurrence of a substring in a string, ignoring case differences.
- **Inputs**:
    - `s`: The main string in which to search for the substring.
    - `find`: The substring to search for within the main string.
- **Control Flow**:
    - Check if the first character of 'find' is not null; if it is, convert it to lowercase and store it in 'c'.
    - Calculate the length of the remaining 'find' string after the first character.
    - Enter a loop to iterate over the main string 's'.
    - Within the loop, check if the current character of 's' is null; if so, return NULL as the substring cannot be found.
    - Convert the current character of 's' to lowercase and compare it with 'c'.
    - If they match, use `strncasecmp` to compare the rest of the 'find' string with the corresponding part of 's', ignoring case.
    - If `strncasecmp` returns 0, indicating a match, break the loop and return the current position in 's'.
    - If no match is found, continue iterating through 's'.
- **Output**:
    - Returns a pointer to the first occurrence of the substring in the main string, ignoring case, or NULL if the substring is not found.


