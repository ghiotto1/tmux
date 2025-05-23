# Purpose
This file is a C header file, `vis.h`, which provides definitions and function prototypes for encoding and decoding visibility representations of strings. It includes a set of macros that define various encoding options, such as `VIS_OCTAL` for octal encoding and `VIS_CSTYLE` for C-style escape sequences, as well as flags for encoding specific characters like spaces, tabs, and newlines. Additionally, it defines return codes for the [`unvis`](#unvis) function, which is used to decode encoded strings, indicating the status of the decoding process. The file declares several functions, such as [`vis`](#vis), [`strvis`](#strvis), and [`unvis`](#unvis), which are used to convert strings to and from a visual representation, making non-printable characters visible in a controlled manner. This header is part of the OpenBSD and NetBSD projects, as indicated by the version control comments at the top.
# Global Variables

---
### vis
- **Type**: `function`
- **Description**: The `vis` function is a global function that takes four parameters: a character pointer, and three integers. It is part of a library for encoding characters into a visual representation, often used to make non-printable characters visible in a safe and readable format.
- **Use**: This function is used to encode characters into a visual representation based on the specified flags and options.


# Function Declarations (Public API)

---
### vis<!-- {{#callable_declaration:vis}} -->
Encode a character into a visual representation.
- **Description**: This function encodes a given character into a visual representation, storing the result in the provided destination buffer. It is useful for converting non-printable or special characters into a format that can be safely printed or logged. The function supports various encoding flags that alter its behavior, such as using octal encoding or encoding specific whitespace characters. It is important to ensure that the destination buffer is large enough to hold the encoded result, which may be longer than a single character. The function appends a null terminator to the encoded string.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded character will be stored. The buffer must be large enough to hold the encoded result and a null terminator. The caller retains ownership.
    - `c`: The character to be encoded. It is an integer value representing a character, typically in the range of unsigned char.
    - `flag`: An integer representing encoding options. It can be a combination of predefined flags such as VIS_OCTAL, VIS_CSTYLE, VIS_SP, VIS_TAB, VIS_NL, VIS_SAFE, VIS_DQ, VIS_ALL, VIS_NOSLASH, and VIS_GLOB, which modify the encoding behavior.
    - `nextc`: The next character in the input stream, used to determine if additional encoding is necessary, particularly for null characters followed by octal digits. It is an integer value representing a character, typically in the range of unsigned char.
- **Output**: Returns a pointer to the end of the encoded string in the destination buffer, which is null-terminated.
- **See also**: [`vis`](vis.c.driver.md#vis)  (Implementation)


---
### strvis<!-- {{#callable_declaration:strvis}} -->
Encodes a string into a visually representable format.
- **Description**: This function encodes the input string into a visually representable format and stores the result in the destination buffer. It is useful for converting non-printable characters into a format that can be safely displayed or logged. The function must be called with a valid destination buffer that is large enough to hold the encoded result, including the null terminator. The encoding behavior can be modified using the provided flag, which determines how characters are represented. The function returns the length of the encoded string, excluding the null terminator.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored. Must not be null and should be large enough to hold the encoded result, including the null terminator.
    - `src`: A pointer to the source string to be encoded. Must not be null.
    - `flag`: An integer representing encoding options. It modifies how characters are encoded, using predefined constants such as VIS_OCTAL, VIS_CSTYLE, etc.
- **Output**: Returns the length of the encoded string, excluding the null terminator.
- **See also**: [`strvis`](vis.c.driver.md#strvis)  (Implementation)


---
### stravis<!-- {{#callable_declaration:stravis}} -->
Convert a string to a visual representation with specified encoding flags.
- **Description**: This function converts the input string into a visually encoded format based on the specified flags and stores the result in a newly allocated buffer. It is useful for encoding strings in a way that makes non-printable characters visible. The function must be called with a valid source string and a pointer to a character pointer, which will be updated to point to the newly allocated encoded string. The caller is responsible for freeing the allocated memory. If memory allocation fails, the function returns -1 and the output pointer is not modified.
- **Inputs**:
    - `outp`: A pointer to a character pointer where the address of the newly allocated encoded string will be stored. Must not be null. The caller is responsible for freeing the allocated memory.
    - `src`: The source string to be encoded. Must not be null.
    - `flag`: An integer representing encoding options. It can be a combination of predefined flags such as VIS_OCTAL, VIS_CSTYLE, VIS_SP, etc., to control the encoding behavior.
- **Output**: Returns the length of the encoded string on success, or -1 if memory allocation fails.
- **See also**: [`stravis`](vis.c.driver.md#stravis)  (Implementation)


---
### strnvis<!-- {{#callable_declaration:strnvis}} -->
Encode a string into a visually safe format with a size limit.
- **Description**: This function encodes the input string into a visually safe format, storing the result in the provided destination buffer. It is useful for preparing strings for display or transmission where non-printable or special characters need to be represented in a safe manner. The function ensures that the output does not exceed the specified buffer size, including the null terminator. It should be used when you need to encode strings with specific visibility flags, and it handles truncation gracefully by returning the number of characters that would have been written if there was enough space. The function must be called with a valid destination buffer and size, and the source string must be null-terminated.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored. Must not be null and should have enough space to accommodate the encoded string and a null terminator.
    - `src`: A pointer to the null-terminated source string to be encoded. Must not be null.
    - `siz`: The size of the destination buffer, including space for the null terminator. Must be greater than zero.
    - `flag`: An integer representing encoding flags that modify the behavior of the encoding process. These flags determine which characters are encoded and how.
- **Output**: Returns the number of characters that would have been written to the destination buffer, excluding the null terminator, if there was enough space.
- **See also**: [`strnvis`](vis.c.driver.md#strnvis)  (Implementation)


---
### strvisx<!-- {{#callable_declaration:strvisx}} -->
Encode a string into a visually safe format.
- **Description**: This function encodes a given source string into a visually safe format and stores the result in the destination buffer. It is useful for encoding strings that may contain non-printable or special characters, making them safe for display or transmission. The function processes up to 'len' characters from the source string and applies encoding rules based on the specified 'flag'. The destination buffer must be large enough to hold the encoded result, including the null terminator. The function should be used when there is a need to safely encode strings for environments where non-graphic characters might cause issues.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored. Must not be null and should have enough space to accommodate the encoded string and a null terminator.
    - `src`: A pointer to the source string to be encoded. Must not be null and should point to a valid string of at least 'len' characters.
    - `len`: The number of characters from the source string to encode. Must be greater than zero.
    - `flag`: An integer representing encoding options, which can be a combination of predefined flags such as VIS_OCTAL, VIS_CSTYLE, etc. These flags determine how characters are encoded.
- **Output**: Returns the number of characters written to the destination buffer, excluding the null terminator.
- **See also**: [`strvisx`](vis.c.driver.md#strvisx)  (Implementation)


---
### strunvis<!-- {{#callable_declaration:strunvis}} -->
Decodes a string encoded with vis encoding back to its original form.
- **Description**: Use this function to convert a string that has been encoded using vis encoding back to its original form. This function processes the input string `src` and writes the decoded result into the buffer `dst`. It is essential that `dst` is large enough to hold the decoded string, which will be at most the same length as `src`. The function returns the length of the decoded string, excluding the null terminator, or -1 if an error occurs during decoding. This function should be used when you need to interpret or display data that was previously encoded for safe transmission or storage.
- **Inputs**:
    - `dst`: A pointer to a character buffer where the decoded string will be stored. Must not be null and should be large enough to hold the decoded string.
    - `src`: A pointer to a null-terminated string that contains the vis-encoded data. Must not be null.
- **Output**: Returns the length of the decoded string on success, or -1 if an error occurs during decoding.
- **See also**: [`strunvis`](unvis.c.driver.md#strunvis)  (Implementation)


---
### unvis<!-- {{#callable_declaration:unvis}} -->
Decodes a single character from a visual representation.
- **Description**: This function is used to decode a character from its visual representation, which is typically encoded using escape sequences. It processes one character at a time and updates the state of the decoding process. The function must be called repeatedly for each character in the input sequence, and the caller is responsible for managing the state across calls. It is important to handle the end of the input sequence by passing the UNVIS_END flag to ensure proper completion of the decoding process. The function returns different codes to indicate the result of the decoding operation, such as whether a valid character was produced or if an error occurred.
- **Inputs**:
    - `cp`: A pointer to a character where the decoded character will be stored. Must not be null. The caller retains ownership.
    - `c`: The character to be decoded. It is expected to be part of an encoded sequence.
    - `astate`: A pointer to an integer representing the current state of the decoding process. Must not be null. The state is updated by the function to track the progress of decoding.
    - `flag`: An integer flag that can modify the behavior of the function. The UNVIS_END flag should be used to indicate the end of the input sequence.
- **Output**: The function returns an integer code indicating the result of the decoding operation, such as UNVIS_VALID, UNVIS_VALIDPUSH, UNVIS_NOCHAR, UNVIS_SYNBAD, or UNVIS_ERROR.
- **See also**: [`unvis`](unvis.c.driver.md#unvis)  (Implementation)


---
### strnunvis<!-- {{#callable_declaration:strnunvis}} -->
Decodes a string with visual representations back to its original form.
- **Description**: This function is used to decode a string that has been encoded with visual representations back to its original form. It writes the decoded string into the provided destination buffer, ensuring that the buffer is null-terminated if its size is greater than zero. The function should be called when you need to convert a visually encoded string back to its original format, such as when reading data that was previously encoded for safe display or transmission. It is important to ensure that the destination buffer is large enough to hold the decoded string, including the null terminator. If the decoding process encounters an error, the function returns -1.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the decoded string will be stored. The buffer must be large enough to hold the decoded string and a null terminator. The caller retains ownership and it must not be null.
    - `src`: A pointer to the source string that contains the visually encoded data. The string must be null-terminated, and the caller retains ownership.
    - `sz`: The size of the destination buffer, including space for the null terminator. It must be greater than zero to ensure the buffer is null-terminated.
- **Output**: Returns the number of characters written to the destination buffer, excluding the null terminator, or -1 if an error occurs during decoding.
- **See also**: [`strnunvis`](unvis.c.driver.md#strnunvis)  (Implementation)


