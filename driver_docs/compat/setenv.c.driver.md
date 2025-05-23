# Purpose
This C source code file provides implementations for the [`setenv`](#setenv) and [`unsetenv`](#unsetenv) functions, which are used to manipulate environment variables in a Unix-like operating system. The [`setenv`](#setenv) function constructs a new environment variable string in the format "name=value" and uses `putenv` to add it to the environment, ignoring the `overwrite` parameter. The [`unsetenv`](#unsetenv) function removes an environment variable by searching for it in the `environ` array and shifting subsequent entries to fill the gap. The file includes necessary headers for system types and standard library functions, and it references a custom header "compat.h," likely for compatibility purposes. The code is distributed under a permissive license, allowing for free use and modification.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `compat.h`


# Functions

---
### setenv<!-- {{#callable:setenv}} -->
The `setenv` function sets an environment variable to a specified value.
- **Inputs**:
    - `name`: The name of the environment variable to set.
    - `value`: The value to assign to the environment variable.
    - `overwrite`: An unused parameter intended to indicate whether to overwrite an existing variable.
- **Control Flow**:
    - Allocate memory and format a new string `newval` combining `name` and `value` in the format `name=value`.
    - Call `putenv` with `newval` to add or update the environment variable.
- **Output**:
    - Returns the result of the `putenv` function, which is typically 0 on success and a non-zero value on failure.


---
### unsetenv<!-- {{#callable:unsetenv}} -->
The `unsetenv` function removes an environment variable from the environment list.
- **Inputs**:
    - `name`: A pointer to a constant character string representing the name of the environment variable to be removed.
- **Control Flow**:
    - Calculate the length of the input name using `strlen`.
    - Iterate over the environment list using a pointer `envptr` until a match is found or the end of the list is reached.
    - For each environment variable, compare the name with the input name using `strncmp` for the length of the name, and check if the character following the name is '=' or '\0'.
    - If a match is found, break out of the loop.
    - Shift all subsequent environment variables one position up in the list to remove the matched variable.
    - Continue shifting until the end of the list is reached, effectively removing the variable.
    - Return 0 to indicate successful completion.
- **Output**:
    - The function returns 0 to indicate successful removal of the environment variable.


