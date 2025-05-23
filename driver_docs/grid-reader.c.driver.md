# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for navigating and manipulating a virtual cursor within a grid structure. The file defines a set of functions that operate on a `grid_reader` structure, which is used to traverse and interact with a grid, presumably representing a terminal screen or a similar text-based interface. The primary operations include initializing the cursor, moving it in various directions (right, left, up, down), and handling line wrapping. Additionally, the code provides functions to navigate through words, jump to specific characters, and manage cursor positioning relative to line indentation and word boundaries.

The file is not an executable but rather a component of a larger system, likely intended to be compiled and linked with other parts of the tmux codebase. It does not define public APIs or external interfaces directly but instead offers internal utility functions for cursor management within the grid. The functions are focused on precise cursor control, including handling special cases like line wrapping and word boundaries, which are essential for text manipulation in terminal applications. The code is structured to ensure that cursor movements respect the grid's boundaries and characteristics, such as line lengths and wrapped lines, making it a crucial part of the text navigation logic in tmux.
# Imports and Dependencies

---
- `tmux.h`
- `string.h`


# Functions

---
### grid_reader_start<!-- {{#callable:grid_reader_start}} -->
The `grid_reader_start` function initializes a grid reader's position by setting its grid and cursor coordinates.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure that will be initialized.
    - `gd`: A pointer to a `grid` structure that the grid reader will read from.
    - `cx`: An unsigned integer representing the initial x-coordinate (column) of the cursor.
    - `cy`: An unsigned integer representing the initial y-coordinate (row) of the cursor.
- **Control Flow**:
    - Assign the grid pointer `gd` to the `gd` member of the `grid_reader` structure `gr`.
    - Assign the x-coordinate `cx` to the `cx` member of the `grid_reader` structure `gr`.
    - Assign the y-coordinate `cy` to the `cy` member of the `grid_reader` structure `gr`.
- **Output**:
    - This function does not return a value; it initializes the `grid_reader` structure with the provided grid and cursor coordinates.


---
### grid_reader_get_cursor<!-- {{#callable:grid_reader_get_cursor}} -->
The `grid_reader_get_cursor` function retrieves the current cursor position from a `grid_reader` structure and stores it in the provided variables.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure from which the cursor position will be retrieved.
    - `cx`: A pointer to an unsigned integer where the x-coordinate of the cursor will be stored.
    - `cy`: A pointer to an unsigned integer where the y-coordinate of the cursor will be stored.
- **Control Flow**:
    - The function directly accesses the `cx` and `cy` fields of the `grid_reader` structure pointed to by `gr`.
    - It assigns the value of `gr->cx` to the location pointed to by `cx`.
    - It assigns the value of `gr->cy` to the location pointed to by `cy`.
- **Output**:
    - The function does not return a value, but it outputs the current cursor position by modifying the values pointed to by `cx` and `cy`.


---
### grid_reader_line_length<!-- {{#callable:grid_reader_line_length}} -->
The function `grid_reader_line_length` returns the length of the line at the current cursor position in a grid.
- **Inputs**:
    - `gr`: A pointer to a `struct grid_reader`, which contains the grid data and the current cursor position.
- **Control Flow**:
    - The function calls `grid_line_length` with the grid data (`gr->gd`) and the current y-coordinate of the cursor (`gr->cy`).
    - It returns the result of the `grid_line_length` function call.
- **Output**:
    - The function returns an unsigned integer representing the length of the line at the current cursor position in the grid.


---
### grid_reader_cursor_right<!-- {{#callable:grid_reader_cursor_right}} -->
The `grid_reader_cursor_right` function moves the cursor one position to the right within a grid, handling line wrapping and padding cells as necessary.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the cursor and the grid it operates on.
    - `wrap`: An integer flag indicating whether the cursor should wrap to the next line if it reaches the end of the current line.
    - `all`: An integer flag indicating whether to consider the entire width of the grid (`sx`) or just the length of the current line when determining the end of the line.
- **Control Flow**:
    - Determine the maximum x-coordinate (`px`) to move the cursor to, based on the `all` flag.
    - If `wrap` is true and the cursor is at or beyond `px`, and not at the last line, move the cursor to the start of the next line.
    - If the cursor is not at the end of the line (`cx < px`), increment the cursor's x-coordinate (`cx`).
    - Continue moving the cursor right while it is within bounds and the current cell is a padding cell, skipping over padding cells.
- **Output**:
    - The function does not return a value; it modifies the `grid_reader` structure in place to update the cursor's position.
- **Functions called**:
    - [`grid_reader_line_length`](#grid_reader_line_length)
    - [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line)
    - [`grid_reader_cursor_down`](#grid_reader_cursor_down)


---
### grid_reader_cursor_left<!-- {{#callable:grid_reader_cursor_left}} -->
The `grid_reader_cursor_left` function moves the cursor one position to the left within a grid, handling line wrapping if necessary.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the cursor and the grid.
    - `wrap`: An integer flag indicating whether line wrapping should be considered when moving the cursor left.
- **Control Flow**:
    - The function enters a loop that decrements the cursor's x-coordinate (`gr->cx`) while it is greater than zero, checking each cell to ensure it is not a padding cell (using the `GRID_FLAG_PADDING` flag).
    - If the cursor reaches the start of a line (`gr->cx == 0`) and the line is wrapped or the `wrap` flag is set, the function moves the cursor up one line and to the end of that line.
    - If the cursor is not at the start of a line, it simply decrements the x-coordinate by one.
- **Output**:
    - The function does not return a value; it modifies the `grid_reader` structure in place to update the cursor's position.
- **Functions called**:
    - [`grid_reader_cursor_up`](#grid_reader_cursor_up)
    - [`grid_reader_cursor_end_of_line`](#grid_reader_cursor_end_of_line)


---
### grid_reader_cursor_down<!-- {{#callable:grid_reader_cursor_down}} -->
The `grid_reader_cursor_down` function moves the cursor down one line in a grid, adjusting the horizontal position to skip over padding cells.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current cursor position and a reference to the grid data.
- **Control Flow**:
    - Check if the current vertical position (`cy`) is less than the maximum allowed (`hsize + sy - 1`), and if so, increment `cy` to move the cursor down one line.
    - Enter a loop to adjust the horizontal position (`cx`) by decrementing it until a non-padding cell is found or `cx` reaches zero.
    - Use `grid_get_cell` to retrieve the cell at the current cursor position and check its flags to determine if it is a padding cell.
- **Output**:
    - The function does not return a value; it modifies the `grid_reader` structure in place to update the cursor position.


---
### grid_reader_cursor_up<!-- {{#callable:grid_reader_cursor_up}} -->
The `grid_reader_cursor_up` function moves the cursor of a grid reader up one line, adjusting the horizontal position to skip over padding cells.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the cursor and the grid it is reading.
- **Control Flow**:
    - Check if the current vertical position (`cy`) is greater than 0; if so, decrement `cy` to move the cursor up one line.
    - Enter a loop to adjust the horizontal position (`cx`) by decrementing it until a non-padding cell is found or `cx` reaches 0.
    - In each iteration of the loop, retrieve the cell at the current cursor position using `grid_get_cell` and check if it is not a padding cell using the `GRID_FLAG_PADDING` flag.
    - Break the loop if a non-padding cell is found.
- **Output**:
    - The function does not return a value; it modifies the `grid_reader` structure in place to update the cursor position.


---
### grid_reader_cursor_start_of_line<!-- {{#callable:grid_reader_cursor_start_of_line}} -->
The function `grid_reader_cursor_start_of_line` moves the cursor to the start of the current line in a grid, optionally considering wrapped lines.
- **Inputs**:
    - `gr`: A pointer to a `struct grid_reader` which contains the current state of the grid reader, including the grid data and the current cursor position.
    - `wrap`: An integer flag indicating whether to consider wrapped lines when moving the cursor to the start of the line.
- **Control Flow**:
    - If the `wrap` flag is set, the function enters a loop that decrements the cursor's y-coordinate (`gr->cy`) while the current line is wrapped, effectively moving the cursor to the start of the wrapped line sequence.
    - The loop continues until it reaches a line that is not wrapped or the top of the grid is reached.
    - After handling wrapped lines, the function sets the cursor's x-coordinate (`gr->cx`) to 0, moving the cursor to the start of the line.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure.


---
### grid_reader_cursor_end_of_line<!-- {{#callable:grid_reader_cursor_end_of_line}} -->
The function `grid_reader_cursor_end_of_line` moves the cursor to the end of the current line in a grid, considering line wrapping and whether to include all columns.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the grid and cursor position.
    - `wrap`: An integer flag indicating whether to consider line wrapping when moving the cursor.
    - `all`: An integer flag indicating whether to move the cursor to the very end of the grid's width or just to the end of the current line's content.
- **Control Flow**:
    - If `wrap` is true, calculate the maximum y-coordinate (`yy`) by adding the grid's history size (`hsize`) and screen height (`sy`), then decrement by one.
    - While the current y-coordinate (`cy`) is less than `yy` and the current line is wrapped, increment the y-coordinate (`cy`).
    - If `all` is true, set the x-coordinate (`cx`) to the grid's width (`sx`).
    - Otherwise, set the x-coordinate (`cx`) to the length of the current line using [`grid_reader_line_length`](#grid_reader_line_length).
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure.
- **Functions called**:
    - [`grid_reader_line_length`](#grid_reader_line_length)


---
### grid_reader_handle_wrap<!-- {{#callable:grid_reader_handle_wrap}} -->
The function `grid_reader_handle_wrap` ensures that the cursor within a grid reader does not exceed the specified horizontal boundary, wrapping to the next line if necessary, and returns zero if the cursor would wrap past the bottom of the grid.
- **Inputs**:
    - `gr`: A pointer to a `struct grid_reader` which contains the current state of the grid reader, including its grid, cursor position, and other relevant data.
    - `xx`: A pointer to an unsigned integer representing the horizontal boundary (column index) that the cursor should not exceed.
    - `yy`: A pointer to an unsigned integer representing the vertical boundary (row index) that the cursor should not exceed.
- **Control Flow**:
    - The function enters a while loop that continues as long as the current cursor's x-coordinate (`gr->cx`) is greater than the value pointed to by `xx`.
    - Inside the loop, it checks if the current cursor's y-coordinate (`gr->cy`) is equal to the value pointed to by `yy`. If true, it returns 0, indicating that the cursor would wrap past the bottom of the grid.
    - If the cursor is not at the bottom, it moves the cursor to the start of the current line using [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line) and then moves the cursor down one line using [`grid_reader_cursor_down`](#grid_reader_cursor_down).
    - It checks if the current line is wrapped by examining the `GRID_LINE_WRAPPED` flag. If the line is wrapped, it sets `*xx` to the maximum x-coordinate of the grid (`gr->gd->sx - 1`), otherwise, it sets `*xx` to the length of the current line using [`grid_reader_line_length`](#grid_reader_line_length).
    - The loop continues until the cursor is within the specified horizontal boundary or it reaches the bottom of the grid.
- **Output**:
    - The function returns an integer: 1 if the cursor is successfully wrapped within the grid's boundaries, or 0 if the cursor would wrap past the bottom of the grid.
- **Functions called**:
    - [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line)
    - [`grid_reader_cursor_down`](#grid_reader_cursor_down)
    - [`grid_reader_line_length`](#grid_reader_line_length)


---
### grid_reader_in_set<!-- {{#callable:grid_reader_in_set}} -->
The `grid_reader_in_set` function checks if the character at the current cursor position in a grid is part of a specified set of characters.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the grid and the current cursor position (cx, cy).
    - `set`: A string representing the set of characters to check against.
- **Control Flow**:
    - The function calls `grid_in_set` with the grid from the `grid_reader`, the current cursor position (cx, cy), and the character set.
    - It directly returns the result of the `grid_in_set` function call.
- **Output**:
    - The function returns an integer indicating whether the character at the current cursor position is in the specified set (non-zero if true, zero if false).


---
### grid_reader_cursor_next_word<!-- {{#callable:grid_reader_cursor_next_word}} -->
The `grid_reader_cursor_next_word` function moves the cursor in a grid reader to the start of the next word, considering specified separators and whitespace.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the grid data and the current cursor position.
    - `separators`: A string of characters that are considered as word separators.
- **Control Flow**:
    - Initialize `xx` to the end of the current line or the grid's width, depending on whether the line is wrapped.
    - Initialize `yy` to the total height of the grid including history size.
    - Check if the cursor can be wrapped to the next line using [`grid_reader_handle_wrap`](#grid_reader_handle_wrap); if not, return immediately.
    - If the current character is not whitespace, check if it is a separator; if so, skip over consecutive separators that are not whitespace.
    - If the current character is not a separator, skip over consecutive non-whitespace and non-separator characters.
    - Finally, skip over any whitespace characters to reach the start of the next word.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure to point to the start of the next word.
- **Functions called**:
    - [`grid_reader_line_length`](#grid_reader_line_length)
    - [`grid_reader_handle_wrap`](#grid_reader_handle_wrap)
    - [`grid_reader_in_set`](#grid_reader_in_set)


---
### grid_reader_cursor_next_word_end<!-- {{#callable:grid_reader_cursor_next_word_end}} -->
The function `grid_reader_cursor_next_word_end` moves the cursor in a grid reader to the end of the next word, considering specified separators and whitespace.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the grid data and the current cursor position.
    - `separators`: A string of characters that are considered as word separators.
- **Control Flow**:
    - Initialize `xx` and `yy` to determine the bounds of the grid based on whether the current line is wrapped or not.
    - Enter a loop that continues as long as [`grid_reader_handle_wrap`](#grid_reader_handle_wrap) returns true, which ensures the cursor stays within the grid bounds.
    - If the current character is whitespace, increment the cursor's x-coordinate (`gr->cx`).
    - If the current character is a separator, continue moving the cursor right until a non-separator or whitespace is found, then return.
    - If the current character is neither whitespace nor a separator, continue moving the cursor right until a separator or whitespace is found, then return.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure to point to the end of the next word.
- **Functions called**:
    - [`grid_reader_line_length`](#grid_reader_line_length)
    - [`grid_reader_handle_wrap`](#grid_reader_handle_wrap)
    - [`grid_reader_in_set`](#grid_reader_in_set)


---
### grid_reader_cursor_previous_word<!-- {{#callable:grid_reader_cursor_previous_word}} -->
The function `grid_reader_cursor_previous_word` moves the cursor in a grid reader to the beginning of the previous word, considering specified separators and whitespace.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which maintains the current position of the cursor in the grid.
    - `separators`: A string containing characters that are considered as word separators.
    - `already`: An integer flag indicating if the cursor is already on a word character.
    - `stop_at_eol`: An integer flag indicating if the cursor should stop at the end of a line if a separator is found.
- **Control Flow**:
    - Check if the cursor is already on a word character or on whitespace; if so, move the cursor backwards until a non-whitespace character is found.
    - Determine if the current character is part of a word by checking if it is not a separator.
    - If the cursor is at the beginning of a line, move up to the previous line and to the end of that line, unless at the top of the grid.
    - If `stop_at_eol` is true, check if the character at the end of the line is a separator and stop if it is.
    - Move the cursor backwards to the beginning of the current word, considering separators and whitespace.
    - Restore the cursor position to the start of the word.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure to point to the start of the previous word.
- **Functions called**:
    - [`grid_reader_in_set`](#grid_reader_in_set)
    - [`grid_reader_cursor_up`](#grid_reader_cursor_up)
    - [`grid_reader_cursor_end_of_line`](#grid_reader_cursor_end_of_line)


---
### grid_reader_cell_equals_data<!-- {{#callable:grid_reader_cell_equals_data}} -->
The function `grid_reader_cell_equals_data` compares a grid cell's data with UTF-8 data to determine if they are equal.
- **Inputs**:
    - `gc`: A pointer to a `grid_cell` structure representing the cell in the grid whose data is to be compared.
    - `ud`: A pointer to a `utf8_data` structure containing the UTF-8 data to compare against the grid cell's data.
- **Control Flow**:
    - Check if the grid cell has the `GRID_FLAG_PADDING` flag set; if so, return 0 indicating inequality.
    - Check if the grid cell has the `GRID_FLAG_TAB` flag set, the UTF-8 data size is 1, and the data is a tab character; if so, return 1 indicating equality.
    - Compare the size of the grid cell's data with the UTF-8 data size; if they are not equal, return 0.
    - Use `memcmp` to compare the actual data of the grid cell and the UTF-8 data; return 1 if they are equal, otherwise return 0.
- **Output**:
    - The function returns an integer: 1 if the grid cell's data is equal to the UTF-8 data, and 0 otherwise.


---
### grid_reader_cursor_jump<!-- {{#callable:grid_reader_cursor_jump}} -->
The `grid_reader_cursor_jump` function moves the cursor in a grid to the first occurrence of a specified UTF-8 character, if found, starting from the current cursor position.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the grid and cursor position.
    - `jc`: A pointer to a `utf8_data` structure representing the UTF-8 character to search for in the grid.
- **Control Flow**:
    - Initialize `px` to the current x-coordinate of the cursor and `yy` to the maximum y-coordinate of the grid.
    - Iterate over each line from the current y-coordinate (`py`) to `yy`.
    - For each line, determine its length (`xx`) and iterate over each cell from `px` to `xx`.
    - Retrieve the cell at the current position and check if it matches the specified UTF-8 character using [`grid_reader_cell_equals_data`](#grid_reader_cell_equals_data).
    - If a match is found, update the cursor position in `gr` to the current coordinates and return 1.
    - If the end of the grid is reached or the line is not wrapped, return 0.
    - Reset `px` to 0 at the start of each new line.
- **Output**:
    - Returns 1 if the specified character is found and the cursor is moved; otherwise, returns 0.
- **Functions called**:
    - [`grid_reader_cell_equals_data`](#grid_reader_cell_equals_data)


---
### grid_reader_cursor_jump_back<!-- {{#callable:grid_reader_cursor_jump_back}} -->
The function `grid_reader_cursor_jump_back` moves the cursor of a grid reader backwards to the first occurrence of a specified UTF-8 character.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the grid reader including its position.
    - `jc`: A pointer to a `utf8_data` structure representing the UTF-8 character to search for.
- **Control Flow**:
    - Initialize `xx` to one more than the current x-coordinate of the cursor (`gr->cx + 1`).
    - Iterate over each line starting from the current line (`gr->cy + 1`) down to the first line.
    - For each line, iterate over each cell from the current x-coordinate (`xx`) down to the first cell.
    - Retrieve the cell at the current position and check if it matches the specified UTF-8 character using [`grid_reader_cell_equals_data`](#grid_reader_cell_equals_data).
    - If a match is found, update the cursor position to the matched cell's coordinates and return 1.
    - If the current line is the first line or is not wrapped, return 0 indicating the character was not found.
    - Update `xx` to the length of the previous line if the line is wrapped.
- **Output**:
    - Returns 1 if the specified character is found and the cursor is moved, otherwise returns 0.
- **Functions called**:
    - [`grid_reader_cell_equals_data`](#grid_reader_cell_equals_data)


---
### grid_reader_cursor_back_to_indentation<!-- {{#callable:grid_reader_cursor_back_to_indentation}} -->
The function `grid_reader_cursor_back_to_indentation` moves the cursor to the first non-blank character of the current line or wrapped lines in a grid.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the grid data and the current cursor position.
- **Control Flow**:
    - Initialize variables `yy`, `oldx`, and `oldy` to store the maximum y-coordinate, and the current cursor x and y positions, respectively.
    - Move the cursor to the start of the current line using [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line).
    - Iterate over each line starting from the current line (`py = gr->cy`) up to the last line (`py <= yy`).
    - For each line, determine its length `xx` using `grid_line_length`.
    - Iterate over each character in the line (`px = 0` to `px < xx`).
    - Retrieve the grid cell at position (px, py) using `grid_get_cell`.
    - Check if the cell is not a space, tab, or padding; if so, update the cursor position to (px, py) and return.
    - If the line is not wrapped, break out of the loop.
    - If no non-blank character is found, restore the cursor to its original position (`oldx`, `oldy`).
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure to point to the first non-blank character.
- **Functions called**:
    - [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line)


