# Purpose
The provided C source code file implements functions for decoding strings that have been previously encoded using a specific encoding scheme, likely the "vis" encoding from the BSD Unix systems. The primary function, [`unvis`](#unvis), is a state machine-driven decoder that processes individual characters and transitions through various states to interpret escape sequences, metacharacters, control characters, and octal representations. The function [`strunvis`](#strunvis) utilizes [`unvis`](#unvis) to decode an entire string, returning the number of characters decoded or -1 in case of an error. Additionally, [`strnunvis`](#strnunvis) performs a similar operation but with a specified buffer size, ensuring that the decoded string does not exceed the provided space.

This code provides a narrow but essential functionality focused on decoding operations, which are crucial for applications that need to interpret encoded data back into its original form. The file is likely part of a library or utility that deals with data encoding and decoding, and it defines public APIs for use in other parts of a program or by other programs. The functions are designed to handle various encoding scenarios, including handling special characters and sequences, making them robust for different input cases. The inclusion of a state machine allows for efficient and clear handling of the decoding process, ensuring that each character is processed correctly according to its context within the encoded string.
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
    - Check if the `flag` has `UNVIS_END` set; if so, handle end-of-input conditions based on the current state.
    - Use a switch statement to handle different states of the state machine (`S_GROUND`, `S_START`, `S_META`, `S_META1`, `S_CTRL`, `S_OCTAL2`, `S_OCTAL3`).
    - In `S_GROUND`, check if the character is a backslash to start a special sequence or return the character as valid.
    - In `S_START`, handle various escape sequences such as octal digits, meta characters, control characters, and common escape sequences like `\n`, `\r`, etc.
    - In `S_META`, check for continuation of meta character sequences and transition to `S_META1` or `S_CTRL` as needed.
    - In `S_META1`, complete the meta character sequence and return the valid character.
    - In `S_CTRL`, handle control character sequences and return the valid character.
    - In `S_OCTAL2` and `S_OCTAL3`, handle octal digit sequences, transitioning between states as needed, and return the valid character or indicate a pushback if the sequence is complete.
    - Default case handles unknown states by resetting to `S_GROUND` and returning a syntax error.
- **Output**:
    - The function returns an integer indicating the result of the decoding process, such as `UNVIS_VALID`, `UNVIS_VALIDPUSH`, `UNVIS_NOCHAR`, or `UNVIS_SYNBAD`.


---
### strunvis<!-- {{#callable:strunvis}} -->
The `strunvis` function decodes a string encoded with visual representation back to its original form, storing the result in the destination buffer.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the decoded string will be stored.
    - `src`: A pointer to the source string that contains the encoded visual representation to be decoded.
- **Control Flow**:
    - Initialize a character pointer `start` to the beginning of the destination buffer and an integer `state` to 0.
    - Iterate over each character `c` in the source string `src` until the null terminator is reached.
    - For each character, call the [`unvis`](#unvis) function to decode it based on the current state and update the destination buffer `dst` accordingly.
    - If [`unvis`](#unvis) returns `UNVIS_VALID`, increment the destination pointer `dst`.
    - If [`unvis`](#unvis) returns `UNVIS_VALIDPUSH`, increment `dst` and re-evaluate the current character by jumping to the `again` label.
    - If [`unvis`](#unvis) returns `0` or `UNVIS_NOCHAR`, continue to the next character without modifying `dst`.
    - If [`unvis`](#unvis) returns any other value, terminate the destination string with a null character and return -1 to indicate an error.
    - After processing all characters, call [`unvis`](#unvis) with the `UNVIS_END` flag to finalize the decoding process.
    - If the final call to [`unvis`](#unvis) returns `UNVIS_VALID`, increment `dst`.
    - Terminate the destination string with a null character.
    - Return the number of characters written to the destination buffer, excluding the null terminator, by subtracting the initial `start` pointer from the final `dst` pointer.
- **Output**:
    - The function returns the number of characters decoded into the destination buffer, excluding the null terminator, or -1 if an error occurs during decoding.
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
    - If the buffer size `sz` is greater than 0, set the last character of the buffer to the null terminator to ensure proper string termination.
    - Iterate over each character `c` in the source string `src`.
    - For each character, call the [`unvis`](#unvis) function to decode it, handling different return values from [`unvis`](#unvis):
    - If [`unvis`](#unvis) returns `UNVIS_VALID`, store the decoded character `p` in the destination buffer if there is space, and increment the destination pointer `dst`.
    - If [`unvis`](#unvis) returns `UNVIS_VALIDPUSH`, store the decoded character `p` in the destination buffer if there is space, increment the destination pointer `dst`, and repeat the decoding process for the same character.
    - If [`unvis`](#unvis) returns 0 or `UNVIS_NOCHAR`, continue to the next character without storing anything.
    - If [`unvis`](#unvis) returns any other value, terminate the destination string with a null character if there is space, and return -1 to indicate an error.
    - After processing all characters, call [`unvis`](#unvis) with the `UNVIS_END` flag to finalize the decoding process, storing any final character if valid.
    - Ensure the destination string is null-terminated and return the number of characters written to the destination buffer, excluding the null terminator.
- **Output**:
    - The function returns the number of characters written to the destination buffer, excluding the null terminator, or -1 if an error occurs during decoding.
- **Functions called**:
    - [`unvis`](#unvis)


