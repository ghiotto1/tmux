# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer, and it provides functionality for navigating and manipulating a virtual cursor within a grid structure. The file defines a set of functions that operate on a `grid_reader` structure, which is used to traverse and interact with a grid, presumably representing a terminal screen or a similar text-based interface. The primary operations include initializing the cursor, moving it in various directions (right, left, up, down), and handling line wrapping. Additionally, the code provides functions to navigate words, jump to specific characters, and manage cursor positioning relative to line indentation and word boundaries.

The file is not an executable but rather a component of a larger system, likely intended to be compiled and linked with other parts of the `tmux` codebase. It does not define public APIs or external interfaces directly but instead offers internal utility functions for cursor management within the grid. The functions are focused on precise cursor control, including handling special cases like line wrapping and word boundaries, which are essential for text manipulation in terminal applications. The code is structured to ensure that cursor movements respect the grid's boundaries and characteristics, such as line wrapping and padding, making it a crucial part of the text rendering and navigation logic in `tmux`.
# Imports and Dependencies

---
- `tmux.h`
- `string.h`


# Functions

---
### grid_reader_start<!-- {{#callable:grid_reader_start}} -->
The `grid_reader_start` function initializes a `grid_reader` structure with a given grid and cursor position.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure that will be initialized.
    - `gd`: A pointer to a `grid` structure that the `grid_reader` will read from.
    - `cx`: An unsigned integer representing the initial x-coordinate (column) of the cursor.
    - `cy`: An unsigned integer representing the initial y-coordinate (row) of the cursor.
- **Control Flow**:
    - Assigns the provided grid pointer `gd` to the `gd` member of the `grid_reader` structure `gr`.
    - Sets the `cx` member of `gr` to the provided x-coordinate `cx`.
    - Sets the `cy` member of `gr` to the provided y-coordinate `cy`.
- **Output**:
    - This function does not return a value; it initializes the `grid_reader` structure in place.


---
### grid_reader_get_cursor<!-- {{#callable:grid_reader_get_cursor}} -->
The function `grid_reader_get_cursor` retrieves the current cursor position from a `grid_reader` structure and stores it in the provided variables.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure from which the cursor position will be retrieved.
    - `cx`: A pointer to an unsigned integer where the current x-coordinate of the cursor will be stored.
    - `cy`: A pointer to an unsigned integer where the current y-coordinate of the cursor will be stored.
- **Control Flow**:
    - The function directly assigns the x-coordinate of the cursor from the `grid_reader` structure to the variable pointed to by `cx`.
    - The function directly assigns the y-coordinate of the cursor from the `grid_reader` structure to the variable pointed to by `cy`.
- **Output**:
    - The function does not return a value; it outputs the cursor position by modifying the values pointed to by `cx` and `cy`.


---
### grid_reader_line_length<!-- {{#callable:grid_reader_line_length}} -->
The function `grid_reader_line_length` returns the length of the line at the current cursor position in a grid.
- **Inputs**:
    - `gr`: A pointer to a `struct grid_reader`, which contains the grid data and the current cursor position.
- **Control Flow**:
    - The function calls [`grid_line_length`](grid.c.driver.md#grid_line_length) with the grid data (`gr->gd`) and the current y-coordinate of the cursor (`gr->cy`).
    - It returns the result of the [`grid_line_length`](grid.c.driver.md#grid_line_length) function call.
- **Output**:
    - The function returns an unsigned integer representing the length of the line at the current cursor position in the grid.
- **Functions called**:
    - [`grid_line_length`](grid.c.driver.md#grid_line_length)


---
### grid_reader_cursor_right<!-- {{#callable:grid_reader_cursor_right}} -->
The `grid_reader_cursor_right` function moves the cursor one position to the right within a grid, handling line wrapping and padding cells as necessary.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the grid and cursor position.
    - `wrap`: An integer flag indicating whether the cursor should wrap to the next line if it reaches the end of the current line.
    - `all`: An integer flag indicating whether to consider the entire width of the grid (`sx`) or just the length of the current line when determining the end of the line.
- **Control Flow**:
    - Determine the maximum x-coordinate (`px`) to move the cursor to, based on the `all` flag.
    - If `wrap` is true and the cursor is at or beyond `px`, and not at the last line, move the cursor to the start of the next line.
    - If the cursor is within the line (`cx < px`), increment the cursor's x-coordinate (`cx`).
    - Continue moving the cursor right while it is within the line and the current cell is a padding cell, skipping over padding cells.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure.
- **Functions called**:
    - [`grid_reader_line_length`](#grid_reader_line_length)
    - [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line)
    - [`grid_reader_cursor_down`](#grid_reader_cursor_down)
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)


---
### grid_reader_cursor_left<!-- {{#callable:grid_reader_cursor_left}} -->
The `grid_reader_cursor_left` function moves the cursor one position to the left within a grid, handling line wrapping if necessary.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure that contains the current state of the grid and cursor position.
    - `wrap`: An integer flag indicating whether line wrapping should be considered when moving the cursor left.
- **Control Flow**:
    - The function enters a loop that continues as long as the cursor's x-coordinate (`gr->cx`) is greater than 0.
    - Within the loop, it retrieves the grid cell at the current cursor position using [`grid_get_cell`](grid.c.driver.md#grid_get_cell) and checks if the cell is not a padding cell (using the `GRID_FLAG_PADDING` flag).
    - If the cell is not a padding cell, the loop breaks, otherwise, the cursor's x-coordinate is decremented.
    - After the loop, if the cursor's x-coordinate is 0 and the y-coordinate (`gr->cy`) is greater than 0, it checks if wrapping is enabled or if the previous line is wrapped (using `GRID_LINE_WRAPPED`).
    - If the conditions for wrapping are met, it moves the cursor up one line and to the end of the previous line using [`grid_reader_cursor_up`](#grid_reader_cursor_up) and [`grid_reader_cursor_end_of_line`](#grid_reader_cursor_end_of_line).
    - If the cursor's x-coordinate is still greater than 0 after the loop, it decrements the x-coordinate by one.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure directly.
- **Functions called**:
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`grid_reader_cursor_up`](#grid_reader_cursor_up)
    - [`grid_reader_cursor_end_of_line`](#grid_reader_cursor_end_of_line)


---
### grid_reader_cursor_down<!-- {{#callable:grid_reader_cursor_down}} -->
The function `grid_reader_cursor_down` moves the cursor down one line in a grid, adjusting the horizontal position to avoid padding cells.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the cursor and the grid it is navigating.
- **Control Flow**:
    - Check if the current vertical position `cy` is less than the maximum allowed position (`hsize + sy - 1`), and if so, increment `cy` to move the cursor down one line.
    - Enter a loop that continues as long as the horizontal position `cx` is greater than 0.
    - Within the loop, retrieve the cell at the current cursor position using [`grid_get_cell`](grid.c.driver.md#grid_get_cell).
    - Check if the cell's flags do not include `GRID_FLAG_PADDING`; if true, break out of the loop.
    - If the cell is a padding cell, decrement `cx` to move left until a non-padding cell is found.
- **Output**:
    - The function does not return a value; it modifies the `grid_reader` structure in place to update the cursor's position.
- **Functions called**:
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)


---
### grid_reader_cursor_up<!-- {{#callable:grid_reader_cursor_up}} -->
The `grid_reader_cursor_up` function moves the cursor of a grid reader up by one line, adjusting the horizontal position to skip over padding cells.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the cursor and the grid it is reading from.
- **Control Flow**:
    - Check if the current vertical position (`cy`) of the cursor is greater than 0; if so, decrement `cy` to move the cursor up one line.
    - Enter a loop that continues as long as the horizontal position (`cx`) is greater than 0.
    - Within the loop, retrieve the cell at the current cursor position using [`grid_get_cell`](grid.c.driver.md#grid_get_cell).
    - Check if the cell's flags do not include `GRID_FLAG_PADDING`; if true, break out of the loop.
    - If the cell is a padding cell, decrement `cx` to move the cursor left, and continue the loop.
- **Output**:
    - The function does not return a value; it modifies the `grid_reader` structure in place to update the cursor position.
- **Functions called**:
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)


---
### grid_reader_cursor_start_of_line<!-- {{#callable:grid_reader_cursor_start_of_line}} -->
The function `grid_reader_cursor_start_of_line` moves the cursor to the start of the current line in a grid, optionally considering wrapped lines.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the cursor and the grid it is reading.
    - `wrap`: An integer flag indicating whether to consider wrapped lines when moving the cursor to the start of the line.
- **Control Flow**:
    - If the `wrap` flag is set, the function enters a loop that decrements the cursor's y-coordinate (`gr->cy`) while the current line is wrapped, effectively moving the cursor to the start of the wrapped line sequence.
    - The loop checks if the line above the current line is wrapped by examining the `GRID_LINE_WRAPPED` flag of the line's flags.
    - After handling wrapped lines, the function sets the cursor's x-coordinate (`gr->cx`) to 0, moving it to the start of the line.
- **Output**:
    - The function does not return a value; it modifies the `grid_reader` structure in place to update the cursor's position.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)


---
### grid_reader_cursor_end_of_line<!-- {{#callable:grid_reader_cursor_end_of_line}} -->
The function `grid_reader_cursor_end_of_line` moves the cursor to the end of the current line in a grid, with options to handle line wrapping and to move to the end of the grid's width.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the cursor and the grid being read.
    - `wrap`: An integer flag indicating whether to consider wrapped lines when moving the cursor.
    - `all`: An integer flag indicating whether to move the cursor to the end of the grid's width or just to the end of the current line's content.
- **Control Flow**:
    - If `wrap` is true, calculate the maximum y-coordinate (`yy`) as the sum of the grid's history size and screen height minus one.
    - While the current y-coordinate (`gr->cy`) is less than `yy` and the current line is wrapped, increment the y-coordinate to move the cursor down to the next line.
    - If `all` is true, set the x-coordinate (`gr->cx`) to the grid's width (`gr->gd->sx`), effectively moving the cursor to the end of the grid's width.
    - If `all` is false, set the x-coordinate (`gr->cx`) to the length of the current line, moving the cursor to the end of the line's content.
- **Output**:
    - The function does not return a value; it modifies the `grid_reader` structure in place to update the cursor's position.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`grid_reader_line_length`](#grid_reader_line_length)


---
### grid_reader_handle_wrap<!-- {{#callable:grid_reader_handle_wrap}} -->
The `grid_reader_handle_wrap` function ensures that the cursor within a grid reader stays within bounds, wrapping to the next line if necessary, and returns zero if the cursor would wrap past the grid's bottom.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the grid reader including its position and the grid it is reading.
    - `xx`: A pointer to an unsigned integer representing the maximum x-coordinate (column) the cursor can move to before needing to wrap to the next line.
    - `yy`: A pointer to an unsigned integer representing the maximum y-coordinate (row) the cursor can move to before reaching the bottom of the grid.
- **Control Flow**:
    - The function enters a loop that continues as long as the current x-coordinate (`gr->cx`) is greater than the value pointed to by `xx`.
    - Within the loop, it checks if the current y-coordinate (`gr->cy`) is equal to the value pointed to by `yy`; if so, it returns 0, indicating the cursor would wrap past the bottom of the grid.
    - If the cursor is not at the bottom, it moves the cursor to the start of the current line using [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line) and then moves the cursor down one line using [`grid_reader_cursor_down`](#grid_reader_cursor_down).
    - It checks if the current line is wrapped by examining the `GRID_LINE_WRAPPED` flag; if wrapped, it sets `*xx` to the grid's width minus one, otherwise it sets `*xx` to the length of the current line using [`grid_reader_line_length`](#grid_reader_line_length).
    - The loop continues until the cursor is within the bounds specified by `xx`.
- **Output**:
    - The function returns an integer: 1 if the cursor is successfully wrapped within the grid bounds, or 0 if the cursor would wrap past the bottom of the grid.
- **Functions called**:
    - [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line)
    - [`grid_reader_cursor_down`](#grid_reader_cursor_down)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`grid_reader_line_length`](#grid_reader_line_length)


---
### grid_reader_in_set<!-- {{#callable:grid_reader_in_set}} -->
The `grid_reader_in_set` function checks if the character at the current cursor position in a grid is part of a specified set of characters.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the grid and the current cursor position (cx, cy).
    - `set`: A string representing the set of characters to check against.
- **Control Flow**:
    - The function calls [`grid_in_set`](grid.c.driver.md#grid_in_set) with the grid from the `grid_reader`, the current cursor position (cx, cy), and the character set.
    - It returns the result of the [`grid_in_set`](grid.c.driver.md#grid_in_set) function call, which checks if the character at the specified position is in the given set.
- **Output**:
    - The function returns an integer indicating whether the character at the current cursor position is in the specified set (non-zero if true, zero if false).
- **Functions called**:
    - [`grid_in_set`](grid.c.driver.md#grid_in_set)


---
### grid_reader_cursor_next_word<!-- {{#callable:grid_reader_cursor_next_word}} -->
The function `grid_reader_cursor_next_word` moves the cursor in a grid reader to the start of the next word, considering specified separators and whitespace.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the grid data and the current cursor position.
    - `separators`: A string of characters that are considered as separators between words.
- **Control Flow**:
    - Initialize `xx` and `yy` to represent the end of the current line and the bottom of the grid, respectively, considering line wrapping.
    - Check if the current line is wrapped and adjust `xx` accordingly.
    - If the cursor is not on whitespace, determine if it is on a separator or a non-whitespace character and skip over subsequent similar characters.
    - Continue moving the cursor forward, skipping over whitespace until a non-whitespace character is encountered.
    - Use [`grid_reader_handle_wrap`](#grid_reader_handle_wrap) to handle line wrapping while moving the cursor.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure to point to the start of the next word.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`grid_reader_line_length`](#grid_reader_line_length)
    - [`grid_reader_handle_wrap`](#grid_reader_handle_wrap)
    - [`grid_reader_in_set`](#grid_reader_in_set)


---
### grid_reader_cursor_next_word_end<!-- {{#callable:grid_reader_cursor_next_word_end}} -->
The function `grid_reader_cursor_next_word_end` moves the cursor to the end of the next word in a grid, considering specified separators and whitespace.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the grid data and the current cursor position.
    - `separators`: A string of characters that are considered as word separators.
- **Control Flow**:
    - Initialize `xx` and `yy` to represent the end of the current line and the last line of the grid, respectively, considering if the line is wrapped.
    - Enter a loop that continues as long as [`grid_reader_handle_wrap`](#grid_reader_handle_wrap) returns true, which manages line wrapping and cursor positioning.
    - If the current character is whitespace, increment the cursor's x-coordinate (`gr->cx`).
    - If the current character is a separator, continue moving the cursor forward until a non-separator or whitespace is encountered, then return.
    - If the current character is neither whitespace nor a separator, continue moving the cursor forward until a separator or whitespace is encountered, then return.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure to point to the end of the next word.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
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
    - `stop_at_eol`: An integer flag indicating whether to stop at the end of a line if a separator is encountered.
- **Control Flow**:
    - Check if the cursor is already on a word character or on whitespace; if so, move the cursor backwards until a non-whitespace character is found.
    - If the cursor reaches the beginning of a line, move up to the previous line and to the end of that line, unless at the top of the grid.
    - If `stop_at_eol` is true, check if the character at the end of the line is a separator and stop if it is.
    - Determine if the current word consists of letters or separators based on the character set.
    - Move the cursor backwards to the beginning of the current word, considering the word type (letters or separators) and whitespace.
    - Restore the cursor position to the start of the word.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure to point to the start of the previous word.
- **Functions called**:
    - [`grid_reader_in_set`](#grid_reader_in_set)
    - [`grid_reader_cursor_up`](#grid_reader_cursor_up)
    - [`grid_reader_cursor_end_of_line`](#grid_reader_cursor_end_of_line)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)


---
### grid_reader_cell_equals_data<!-- {{#callable:grid_reader_cell_equals_data}} -->
The function `grid_reader_cell_equals_data` compares a grid cell's data with UTF-8 data to determine if they are equal.
- **Inputs**:
    - `gc`: A pointer to a `grid_cell` structure representing the grid cell to be compared.
    - `ud`: A pointer to a `utf8_data` structure representing the UTF-8 data to compare against the grid cell.
- **Control Flow**:
    - Check if the grid cell has the `GRID_FLAG_PADDING` flag set; if so, return 0 indicating inequality.
    - Check if the grid cell has the `GRID_FLAG_TAB` flag set, the UTF-8 data size is 1, and the data is a tab character; if so, return 1 indicating equality.
    - Compare the size of the grid cell's data with the UTF-8 data size; if they are not equal, return 0 indicating inequality.
    - Use `memcmp` to compare the actual data of the grid cell and the UTF-8 data; return the result of this comparison (1 for equality, 0 for inequality).
- **Output**:
    - Returns 1 if the grid cell's data is equal to the UTF-8 data, otherwise returns 0.


---
### grid_reader_cursor_jump<!-- {{#callable:grid_reader_cursor_jump}} -->
The `grid_reader_cursor_jump` function moves the cursor in a grid to the first occurrence of a specified UTF-8 character, if found, starting from the current cursor position.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the grid and cursor position.
    - `jc`: A pointer to a `utf8_data` structure representing the UTF-8 character to search for in the grid.
- **Control Flow**:
    - Initialize `px` to the current x-coordinate of the cursor and `yy` to the maximum y-coordinate of the grid.
    - Iterate over each line from the current y-coordinate (`py`) to `yy`.
    - For each line, determine its length `xx` and iterate over each cell from `px` to `xx`.
    - Retrieve the cell at the current position and compare it with the target UTF-8 character using [`grid_reader_cell_equals_data`](#grid_reader_cell_equals_data).
    - If a match is found, update the cursor position in `gr` to the current `px` and `py`, and return 1 to indicate success.
    - If the end of the line is reached without finding a match, check if the line is wrapped; if not, return 0 to indicate failure.
    - Reset `px` to 0 and continue to the next line if the current line is wrapped.
- **Output**:
    - Returns 1 if the specified UTF-8 character is found and the cursor is moved; otherwise, returns 0 if the character is not found.
- **Functions called**:
    - [`grid_line_length`](grid.c.driver.md#grid_line_length)
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`grid_reader_cell_equals_data`](#grid_reader_cell_equals_data)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)


---
### grid_reader_cursor_jump_back<!-- {{#callable:grid_reader_cursor_jump_back}} -->
The function `grid_reader_cursor_jump_back` moves the cursor of a grid reader backwards to the first occurrence of a specified UTF-8 character.
- **Inputs**:
    - `gr`: A pointer to a `grid_reader` structure, which contains the current state of the grid reader including its position and the grid data.
    - `jc`: A pointer to a `utf8_data` structure representing the UTF-8 character to search for in the grid.
- **Control Flow**:
    - Initialize `xx` to one position right of the current cursor's x-coordinate (`gr->cx + 1`).
    - Iterate over each line starting from the line after the current cursor's y-coordinate (`gr->cy + 1`) down to the first line.
    - For each line, iterate over each cell from the position `xx` down to the first cell in the line.
    - Retrieve the cell data at the current position and compare it with the target UTF-8 character using [`grid_reader_cell_equals_data`](#grid_reader_cell_equals_data).
    - If a match is found, update the cursor's position to the matching cell's coordinates and return 1.
    - If the current line is the first line or is not wrapped, return 0 indicating the character was not found.
    - Update `xx` to the length of the previous line if the current line is wrapped and continue searching.
- **Output**:
    - Returns 1 if the specified character is found and the cursor is moved to its position, otherwise returns 0 if the character is not found.
- **Functions called**:
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`grid_reader_cell_equals_data`](#grid_reader_cell_equals_data)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`grid_line_length`](grid.c.driver.md#grid_line_length)


---
### grid_reader_cursor_back_to_indentation<!-- {{#callable:grid_reader_cursor_back_to_indentation}} -->
The function `grid_reader_cursor_back_to_indentation` moves the cursor of a grid reader to the first non-blank character of the current line or the first line that is not wrapped.
- **Inputs**:
    - `gr`: A pointer to a `struct grid_reader` which contains the grid data and the current cursor position.
- **Control Flow**:
    - Initialize `yy` to the last line index of the grid and store the current cursor position in `oldx` and `oldy`.
    - Move the cursor to the start of the current line using [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line).
    - Iterate over each line from the current line (`gr->cy`) to the last line (`yy`).
    - For each line, determine its length `xx` and iterate over each character position `px` in the line.
    - Retrieve the grid cell at position `px` and `py` using [`grid_get_cell`](grid.c.driver.md#grid_get_cell).
    - Check if the cell is not a single space, not a tab, and not padding; if so, update the cursor position to `px` and `py` and return.
    - If the line is not wrapped, break out of the loop.
    - If no non-blank character is found, restore the cursor to its original position `oldx` and `oldy`.
- **Output**:
    - The function does not return a value; it modifies the cursor position within the `grid_reader` structure.
- **Functions called**:
    - [`grid_reader_cursor_start_of_line`](#grid_reader_cursor_start_of_line)
    - [`grid_line_length`](grid.c.driver.md#grid_line_length)
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)


