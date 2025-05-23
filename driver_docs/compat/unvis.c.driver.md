# Purpose
The provided C source code file implements functions for decoding strings that have been encoded using a specific encoding scheme, likely the "vis" encoding from the OpenBSD project. The primary function, [`unvis`](#unvis), is a state machine that decodes individual characters based on their encoding, handling various escape sequences and special characters. It transitions through different states such as `S_GROUND`, `S_START`, `S_META`, and others to interpret sequences like octal values, control characters, and meta characters. The function returns different status codes to indicate the result of the decoding process, such as `UNVIS_VALID`, `UNVIS_NOCHAR`, and `UNVIS_SYNBAD`.

Additionally, the file includes two higher-level functions, [`strunvis`](#strunvis) and [`strnunvis`](#strnunvis), which utilize the [`unvis`](#unvis) function to decode entire strings. [`strunvis`](#strunvis) decodes a source string into a destination buffer, ensuring the result is null-terminated, and returns the number of characters decoded or -1 on error. [`strnunvis`](#strnunvis) performs a similar task but with a specified buffer size, providing a safer alternative that prevents buffer overflows. These functions are designed to be used in applications where encoded strings need to be converted back to their original form, making this file a utility for handling encoded data in C programs.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `compat.h`


# Functions

---
### unvis<!-- {{#callable:unvis}} -->
The `unvis` function decodes a character that was previously encoded by the `vis` function, using a state machine to handle different escape sequences.
- **Inputs**:
    - `cp`: A pointer to a character where the decoded result will be stored.
    - `c`: The character to be decoded.
    - `astate`: A pointer to an integer representing the current state of the decoding state machine.
    - `flag`: An integer flag that can modify the behavior of the function, such as indicating the end of input with `UNVIS_END`.
- **Control Flow**:
    - Check if the `flag` includes `UNVIS_END` to handle end-of-input scenarios, returning `UNVIS_VALID` if in an octal state or `UNVIS_NOCHAR`/`UNVIS_SYNBAD` otherwise.
    - Use a switch statement to handle different states of the state machine (`S_GROUND`, `S_START`, `S_META`, `S_META1`, `S_CTRL`, `S_OCTAL2`, `S_OCTAL3`).
    - In `S_GROUND`, check if the character is a backslash to transition to `S_START`, otherwise store the character and return `UNVIS_VALID`.
    - In `S_START`, handle various escape sequences (e.g., octal, meta, control characters) and transition to appropriate states or return `UNVIS_VALID`.
    - In `S_META`, check for '-' or '^' to transition to `S_META1` or `S_CTRL`, otherwise return `UNVIS_SYNBAD`.
    - In `S_META1`, combine the character with the meta prefix and return `UNVIS_VALID`.
    - In `S_CTRL`, apply control character transformation and return `UNVIS_VALID`.
    - In `S_OCTAL2` and `S_OCTAL3`, handle octal digit sequences, transitioning between states or returning `UNVIS_VALIDPUSH` if complete.
    - Default case resets the state to `S_GROUND` and returns `UNVIS_SYNBAD` for unknown states.
- **Output**:
    - The function returns an integer indicating the result of the decoding process, such as `UNVIS_VALID`, `UNVIS_VALIDPUSH`, `UNVIS_NOCHAR`, or `UNVIS_SYNBAD`.


---
### strunvis<!-- {{#callable:strunvis}} -->
The `strunvis` function decodes a string encoded with visual representation back to its original form, storing the result in the destination buffer.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the decoded string will be stored.
    - `src`: A pointer to the source string that is encoded and needs to be decoded.
- **Control Flow**:
    - Initialize a character pointer `start` to the beginning of `dst` and an integer `state` to 0.
    - Iterate over each character `c` in the source string `src` until the null terminator is reached.
    - For each character, call the [`unvis`](#unvis) function to decode it based on the current `state`.
    - Handle the return value of [`unvis`](#unvis) using a switch statement:
    - If `UNVIS_VALID`, increment the `dst` pointer to store the decoded character.
    - If `UNVIS_VALIDPUSH`, increment the `dst` pointer and reprocess the current character by jumping to the `again` label.
    - If `0` or `UNVIS_NOCHAR`, continue to the next character without modifying `dst`.
    - For any other return value, terminate the destination string with a null character and return -1 indicating an error.
    - After the loop, call [`unvis`](#unvis) with the `UNVIS_END` flag to finalize the decoding process.
    - If the final call to [`unvis`](#unvis) returns `UNVIS_VALID`, increment the `dst` pointer.
    - Terminate the destination string with a null character.
    - Return the number of characters decoded into `dst` by subtracting the initial `start` pointer from the current `dst` pointer.
- **Output**:
    - The function returns the number of characters decoded into the destination buffer `dst`, or -1 if an error occurs during decoding.
- **Functions called**:
    - [`unvis`](#unvis)


---
### strnunvis<!-- {{#callable:strnunvis}} -->
The `strnunvis` function decodes a string encoded with visual representations back into its original form, storing the result in a destination buffer with a specified size limit.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the decoded string will be stored.
    - `src`: A pointer to the source string that contains the encoded characters to be decoded.
    - `sz`: The size of the destination buffer, indicating the maximum number of characters (including the null terminator) that can be stored in the buffer.
- **Control Flow**:
    - Initialize pointers `start` and `end` to mark the beginning and end of the destination buffer, and set the initial state to 0.
    - If the buffer size `sz` is greater than 0, set the last character of the buffer to the null terminator to ensure it is null-terminated.
    - Iterate over each character `c` in the source string `src`.
    - For each character, call the [`unvis`](#unvis) function to decode it, handling different return values from [`unvis`](#unvis):
    - If [`unvis`](#unvis) returns `UNVIS_VALID`, store the decoded character `p` in the destination buffer if there is space, and increment the destination pointer.
    - If [`unvis`](#unvis) returns `UNVIS_VALIDPUSH`, store the decoded character `p` and repeat the decoding process for the same character.
    - If [`unvis`](#unvis) returns 0 or `UNVIS_NOCHAR`, continue to the next character without storing anything.
    - If [`unvis`](#unvis) returns any other value, terminate the destination string with a null character and return -1 to indicate an error.
    - After processing all characters, call [`unvis`](#unvis) with the `UNVIS_END` flag to finalize the decoding process, storing any final character if valid.
    - Ensure the destination buffer is null-terminated and return the number of characters written to the buffer, excluding the null terminator.
- **Output**:
    - The function returns the number of characters written to the destination buffer, excluding the null terminator, or -1 if an error occurs during decoding.
- **Functions called**:
    - [`unvis`](#unvis)


