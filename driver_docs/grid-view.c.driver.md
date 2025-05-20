# Purpose
This C source code file is part of the `tmux` terminal multiplexer project and provides a set of functions for manipulating a grid structure that represents the visible screen area. The code defines a series of operations that can be performed on this grid, such as getting and setting cells, inserting and deleting lines and characters, and scrolling regions. These operations are essential for managing the display of text and other content within a terminal window, allowing for dynamic updates and modifications to the screen's content. The functions use a coordinate system relative to the visible screen area, which is adjusted by the `grid_view_x` and `grid_view_y` macros to account for the grid's history size.

The file is not an executable but rather a component of the `tmux` library, intended to be used internally by other parts of the program. It does not define public APIs or external interfaces directly but provides foundational functionality for higher-level operations within `tmux`. The functions interact with a `grid` structure, which is likely defined elsewhere in the `tmux` codebase, and they rely on other grid-related functions such as `grid_get_cell`, `grid_set_cell`, and `grid_move_lines`. This modular approach allows for efficient management of terminal content, supporting features like scrolling, text insertion, and deletion, which are crucial for the responsive behavior of terminal multiplexers.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `tmux.h`


# Functions

---
### grid_view_get_cell<!-- {{#callable:grid_view_get_cell}} -->
The `grid_view_get_cell` function retrieves a cell from a grid using view-relative coordinates.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid from which the cell is to be retrieved.
    - `px`: An unsigned integer representing the x-coordinate of the cell in view-relative terms.
    - `py`: An unsigned integer representing the y-coordinate of the cell in view-relative terms.
    - `gc`: A pointer to a `struct grid_cell` where the retrieved cell data will be stored.
- **Control Flow**:
    - The function calls `grid_view_x` with `gd` and `px` to convert the x-coordinate from view-relative to grid-relative.
    - The function calls `grid_view_y` with `gd` and `py` to convert the y-coordinate from view-relative to grid-relative.
    - The function calls [`grid_get_cell`](grid.c.driver.md#grid_get_cell) with the grid, converted coordinates, and the cell pointer `gc` to retrieve the cell data.
- **Output**:
    - The function does not return a value but populates the `struct grid_cell` pointed to by `gc` with the data of the cell at the specified view-relative coordinates.
- **Functions called**:
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)


---
### grid_view_set_cell<!-- {{#callable:grid_view_set_cell}} -->
The `grid_view_set_cell` function sets a cell in a grid at a specified position using view-relative coordinates.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid where the cell will be set.
    - `px`: An unsigned integer representing the x-coordinate of the cell in view-relative terms.
    - `py`: An unsigned integer representing the y-coordinate of the cell in view-relative terms.
    - `gc`: A pointer to a `struct grid_cell` which contains the data to be set in the specified cell.
- **Control Flow**:
    - The function calls `grid_view_x` with `gd` and `px` to convert the x-coordinate from view-relative to grid-relative.
    - The function calls `grid_view_y` with `gd` and `py` to convert the y-coordinate from view-relative to grid-relative.
    - The function then calls [`grid_set_cell`](grid.c.driver.md#grid_set_cell) with the grid, the converted x and y coordinates, and the grid cell data to set the cell in the grid.
- **Output**:
    - This function does not return any value; it performs an operation on the grid by setting a cell.
- **Functions called**:
    - [`grid_set_cell`](grid.c.driver.md#grid_set_cell)


---
### grid_view_set_padding<!-- {{#callable:grid_view_set_padding}} -->
The `grid_view_set_padding` function sets the padding of a grid view using coordinates relative to the visible screen area.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid to modify.
    - `px`: An unsigned integer representing the x-coordinate for padding in the grid view.
    - `py`: An unsigned integer representing the y-coordinate for padding in the grid view.
- **Control Flow**:
    - The function calls `grid_view_x` with `gd` and `px` to get the x-coordinate relative to the grid view.
    - The function calls `grid_view_y` with `gd` and `py` to get the y-coordinate relative to the grid view.
    - The function then calls [`grid_set_padding`](grid.c.driver.md#grid_set_padding) with `gd`, the calculated x-coordinate, and the calculated y-coordinate to set the padding.
- **Output**:
    - The function does not return a value; it modifies the grid's padding directly.
- **Functions called**:
    - [`grid_set_padding`](grid.c.driver.md#grid_set_padding)


---
### grid_view_set_cells<!-- {{#callable:grid_view_set_cells}} -->
The `grid_view_set_cells` function sets a series of cells in a grid at specified coordinates using a given grid cell structure and string data.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid where cells will be set.
    - `px`: An unsigned integer representing the x-coordinate in the grid view where the cells will start being set.
    - `py`: An unsigned integer representing the y-coordinate in the grid view where the cells will start being set.
    - `gc`: A pointer to a `struct grid_cell` which contains the attributes and data for the cells to be set.
    - `s`: A constant character pointer to a string that contains the data to be set in the grid cells.
    - `slen`: A size_t value representing the length of the string `s`.
- **Control Flow**:
    - The function calls [`grid_set_cells`](grid.c.driver.md#grid_set_cells), passing the grid `gd`, the x-coordinate transformed by `grid_view_x`, the y-coordinate transformed by `grid_view_y`, the grid cell `gc`, the string `s`, and the string length `slen`.
- **Output**:
    - The function does not return a value; it modifies the grid `gd` by setting cells at the specified coordinates.
- **Functions called**:
    - [`grid_set_cells`](grid.c.driver.md#grid_set_cells)


---
### grid_view_clear_history<!-- {{#callable:grid_view_clear_history}} -->
The `grid_view_clear_history` function clears the history of a grid by scrolling used lines into the history and clearing the remaining lines.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid to be cleared.
    - `bg`: An unsigned integer representing the background color to be used when clearing the grid.
- **Control Flow**:
    - Initialize `last` to 0 to track the last used line in the grid.
    - Iterate over each line in the grid using a for loop from 0 to `gd->sy`.
    - For each line, retrieve the line using [`grid_get_line`](grid.c.driver.md#grid_get_line) and check if it has any used cells (`gl->cellused != 0`).
    - If a line has used cells, update `last` to the current line index plus one.
    - If no lines are used (`last == 0`), clear the entire grid using [`grid_view_clear`](#grid_view_clear) and return.
    - If there are used lines, iterate from 0 to `last` and for each line, call [`grid_collect_history`](grid.c.driver.md#grid_collect_history) and [`grid_scroll_history`](grid.c.driver.md#grid_scroll_history) to move lines into history.
    - If `last` is less than `gd->sy`, clear the remaining lines in the grid using [`grid_view_clear`](#grid_view_clear).
    - Set `gd->hscrolled` to 0 to indicate that the grid has been scrolled.
- **Output**:
    - The function does not return a value; it modifies the grid in place by clearing its history and updating its state.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`grid_view_clear`](#grid_view_clear)
    - [`grid_collect_history`](grid.c.driver.md#grid_collect_history)
    - [`grid_scroll_history`](grid.c.driver.md#grid_scroll_history)


---
### grid_view_clear<!-- {{#callable:grid_view_clear}} -->
The `grid_view_clear` function clears a specified rectangular area of a grid, using view-relative coordinates, and fills it with a specified background color.
- **Inputs**:
    - `gd`: A pointer to the grid structure that represents the grid to be cleared.
    - `px`: The x-coordinate of the top-left corner of the area to be cleared, relative to the visible screen area.
    - `py`: The y-coordinate of the top-left corner of the area to be cleared, relative to the visible screen area.
    - `nx`: The width of the area to be cleared.
    - `ny`: The height of the area to be cleared.
    - `bg`: The background color to fill the cleared area with.
- **Control Flow**:
    - Convert the view-relative x-coordinate `px` to an absolute x-coordinate using `grid_view_x` macro.
    - Convert the view-relative y-coordinate `py` to an absolute y-coordinate using `grid_view_y` macro.
    - Call the [`grid_clear`](grid.c.driver.md#grid_clear) function with the converted coordinates and dimensions to clear the specified area and fill it with the background color `bg`.
- **Output**:
    - The function does not return a value; it performs an operation on the grid structure in place.
- **Functions called**:
    - [`grid_clear`](grid.c.driver.md#grid_clear)


---
### grid_view_scroll_region_up<!-- {{#callable:grid_view_scroll_region_up}} -->
The `grid_view_scroll_region_up` function scrolls a specified region of a grid upwards, either by moving lines or by interacting with the grid's history, depending on the grid's flags.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid to be manipulated.
    - `rupper`: An unsigned integer representing the upper boundary of the region to scroll.
    - `rlower`: An unsigned integer representing the lower boundary of the region to scroll.
    - `bg`: An unsigned integer representing the background color or attribute to use when scrolling.
- **Control Flow**:
    - Check if the grid has the `GRID_HISTORY` flag set.
    - If the `GRID_HISTORY` flag is set, call [`grid_collect_history`](grid.c.driver.md#grid_collect_history) to manage the grid's history.
    - If the region to scroll is the entire grid (from 0 to `gd->sy - 1`), call [`grid_scroll_history`](grid.c.driver.md#grid_scroll_history) to scroll the entire history.
    - If the region is not the entire grid, convert `rupper` and `rlower` to their view coordinates using `grid_view_y`, and call [`grid_scroll_history_region`](grid.c.driver.md#grid_scroll_history_region) to scroll only the specified region in the history.
    - If the `GRID_HISTORY` flag is not set, convert `rupper` and `rlower` to their view coordinates using `grid_view_y`, and call [`grid_move_lines`](grid.c.driver.md#grid_move_lines) to move the lines within the specified region upwards.
- **Output**:
    - The function does not return a value; it modifies the grid in place by scrolling the specified region upwards.
- **Functions called**:
    - [`grid_collect_history`](grid.c.driver.md#grid_collect_history)
    - [`grid_scroll_history`](grid.c.driver.md#grid_scroll_history)
    - [`grid_scroll_history_region`](grid.c.driver.md#grid_scroll_history_region)
    - [`grid_move_lines`](grid.c.driver.md#grid_move_lines)


---
### grid_view_scroll_region_down<!-- {{#callable:grid_view_scroll_region_down}} -->
The `grid_view_scroll_region_down` function scrolls a specified region of a grid downwards by moving lines within the grid.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid to be manipulated.
    - `rupper`: An unsigned integer representing the upper boundary of the region to scroll, relative to the visible screen area.
    - `rlower`: An unsigned integer representing the lower boundary of the region to scroll, relative to the visible screen area.
    - `bg`: An unsigned integer representing the background color or attribute to be used when moving lines.
- **Control Flow**:
    - Convert `rupper` and `rlower` from view coordinates to absolute grid coordinates using `grid_view_y` function.
    - Call [`grid_move_lines`](grid.c.driver.md#grid_move_lines) to move lines from `rupper` to `rlower` downwards by one line, filling the vacated line with the background `bg`.
- **Output**:
    - The function does not return a value; it performs an in-place modification of the grid structure.
- **Functions called**:
    - [`grid_move_lines`](grid.c.driver.md#grid_move_lines)


---
### grid_view_insert_lines<!-- {{#callable:grid_view_insert_lines}} -->
The `grid_view_insert_lines` function inserts a specified number of lines into a grid at a given position, shifting existing lines downwards.
- **Inputs**:
    - `gd`: A pointer to the grid structure where lines are to be inserted.
    - `py`: The y-coordinate (row) in the grid where the insertion should begin, relative to the visible screen area.
    - `ny`: The number of lines to insert.
    - `bg`: The background value to use for the new lines.
- **Control Flow**:
    - Convert the visible y-coordinate `py` to the actual grid y-coordinate using `grid_view_y` function.
    - Calculate the actual grid y-coordinate for the bottom of the grid using `grid_view_y` with `gd->sy`.
    - Call [`grid_move_lines`](grid.c.driver.md#grid_move_lines) to shift existing lines downwards from the insertion point by `ny` lines, filling the new lines with the specified background `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by inserting lines and shifting existing lines downwards.
- **Functions called**:
    - [`grid_move_lines`](grid.c.driver.md#grid_move_lines)


---
### grid_view_insert_lines_region<!-- {{#callable:grid_view_insert_lines_region}} -->
The `grid_view_insert_lines_region` function inserts a specified number of lines into a region of a grid, moving existing lines and clearing the affected area with a background color.
- **Inputs**:
    - `gd`: A pointer to the grid structure where lines are to be inserted.
    - `rlower`: The lower boundary of the region in the grid where lines are to be inserted, given in grid coordinates.
    - `py`: The starting y-coordinate in the grid where lines are to be inserted, given in grid coordinates.
    - `ny`: The number of lines to insert into the grid.
    - `bg`: The background color to use when clearing the affected area of the grid.
- **Control Flow**:
    - Convert `rlower` and `py` from grid coordinates to view coordinates using `grid_view_y` function.
    - Calculate `ny2`, the number of lines to move, as `rlower + 1 - py - ny`.
    - Call [`grid_move_lines`](grid.c.driver.md#grid_move_lines) to move `ny2` lines from the position `rlower + 1 - ny2` to `py`, filling the moved lines with the background color `bg`.
    - Call [`grid_clear`](grid.c.driver.md#grid_clear) to clear the area from `py + ny2` to `ny - ny2` with the background color `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by inserting lines and clearing the affected area.
- **Functions called**:
    - [`grid_move_lines`](grid.c.driver.md#grid_move_lines)
    - [`grid_clear`](grid.c.driver.md#grid_clear)


---
### grid_view_delete_lines<!-- {{#callable:grid_view_delete_lines}} -->
The `grid_view_delete_lines` function deletes a specified number of lines from a grid starting at a given position and fills the resulting empty space with a background value.
- **Inputs**:
    - `gd`: A pointer to the grid structure from which lines are to be deleted.
    - `py`: The starting y-coordinate (row) in the grid from which lines will be deleted, relative to the visible screen area.
    - `ny`: The number of lines to delete from the grid starting at the specified y-coordinate.
    - `bg`: The background value used to fill the cleared space after lines are deleted.
- **Control Flow**:
    - Convert the starting y-coordinate `py` to the grid's internal coordinate system using `grid_view_y`.
    - Determine the grid's internal y-coordinate for the bottom of the visible screen area using `grid_view_y` with `gd->sy`.
    - Call [`grid_move_lines`](grid.c.driver.md#grid_move_lines) to move lines in the grid, effectively deleting the specified lines and shifting the remaining lines up.
    - Call [`grid_clear`](grid.c.driver.md#grid_clear) to clear the space at the bottom of the grid that was vacated by the line deletion, filling it with the specified background value.
- **Output**:
    - The function does not return a value; it modifies the grid in place by deleting lines and clearing space.
- **Functions called**:
    - [`grid_move_lines`](grid.c.driver.md#grid_move_lines)
    - [`grid_clear`](grid.c.driver.md#grid_clear)


---
### grid_view_delete_lines_region<!-- {{#callable:grid_view_delete_lines_region}} -->
The `grid_view_delete_lines_region` function deletes a specified number of lines from a grid within a defined region and adjusts the remaining lines accordingly.
- **Inputs**:
    - `gd`: A pointer to the grid structure from which lines are to be deleted.
    - `rlower`: The lower boundary of the region in the grid, given in visible screen coordinates.
    - `py`: The starting y-coordinate in the grid, given in visible screen coordinates, from where lines will be deleted.
    - `ny`: The number of lines to delete from the grid starting at the y-coordinate `py`.
    - `bg`: The background color or attribute to use when clearing lines.
- **Control Flow**:
    - Convert `rlower` from visible screen coordinates to grid coordinates using `grid_view_y` function.
    - Convert `py` from visible screen coordinates to grid coordinates using `grid_view_y` function.
    - Calculate `ny2`, the number of lines that will remain after deletion, as `rlower + 1 - py - ny`.
    - Move the lines in the grid from `py + ny` to `py + ny2` to fill the gap left by the deleted lines using [`grid_move_lines`](grid.c.driver.md#grid_move_lines).
    - Clear the remaining lines from `py + ny2` to `py + ny` using [`grid_clear`](grid.c.driver.md#grid_clear) to ensure the grid is visually consistent.
- **Output**:
    - The function does not return a value; it modifies the grid in place by deleting lines and adjusting the remaining lines.
- **Functions called**:
    - [`grid_move_lines`](grid.c.driver.md#grid_move_lines)
    - [`grid_clear`](grid.c.driver.md#grid_clear)


---
### grid_view_insert_cells<!-- {{#callable:grid_view_insert_cells}} -->
The `grid_view_insert_cells` function inserts a specified number of cells at a given position in a grid, shifting existing cells to the right and filling the new space with a background value.
- **Inputs**:
    - `gd`: A pointer to the grid structure where cells are to be inserted.
    - `px`: The x-coordinate (column) in the grid where the insertion begins, relative to the visible screen area.
    - `py`: The y-coordinate (row) in the grid where the insertion occurs, relative to the visible screen area.
    - `nx`: The number of cells to insert at the specified position.
    - `bg`: The background value used to fill the newly inserted cells.
- **Control Flow**:
    - Convert the visible screen coordinates `px` and `py` to the actual grid coordinates using `grid_view_x` and `grid_view_y` functions.
    - Determine the grid's width `sx` by converting the grid's total width `gd->sx` to the actual grid coordinate using `grid_view_x`.
    - Check if the insertion point `px` is at or beyond the last column (`sx - 1`).
    - If `px` is at or beyond the last column, clear the cell at `px, py` with the background value `bg`.
    - If `px` is not at the last column, move existing cells from `px` to `sx - nx` to the right by `nx` positions, starting from `px + nx`, and fill the new space with the background value `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by inserting cells and adjusting existing cells accordingly.
- **Functions called**:
    - [`grid_clear`](grid.c.driver.md#grid_clear)
    - [`grid_move_cells`](grid.c.driver.md#grid_move_cells)


---
### grid_view_delete_cells<!-- {{#callable:grid_view_delete_cells}} -->
The `grid_view_delete_cells` function deletes a specified number of cells from a grid starting at a given position and shifts the remaining cells to fill the gap.
- **Inputs**:
    - `gd`: A pointer to the grid structure from which cells are to be deleted.
    - `px`: The x-coordinate (column) in the grid from which deletion starts, relative to the visible screen area.
    - `py`: The y-coordinate (row) in the grid from which deletion starts, relative to the visible screen area.
    - `nx`: The number of cells to delete starting from the position (px, py).
    - `bg`: The background value used to fill cleared cells.
- **Control Flow**:
    - Convert the visible screen coordinates (px, py) to absolute grid coordinates using `grid_view_x` and `grid_view_y`.
    - Determine the absolute x-coordinate of the grid's width using `grid_view_x`.
    - Call [`grid_move_cells`](grid.c.driver.md#grid_move_cells) to shift cells to the left, starting from the position after the deleted cells, to fill the gap left by the deletion.
    - Call [`grid_clear`](grid.c.driver.md#grid_clear) to clear the remaining cells at the end of the row, using the background value `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by deleting cells and adjusting the remaining cells accordingly.
- **Functions called**:
    - [`grid_move_cells`](grid.c.driver.md#grid_move_cells)
    - [`grid_clear`](grid.c.driver.md#grid_clear)


---
### grid_view_string_cells<!-- {{#callable:grid_view_string_cells}} -->
The `grid_view_string_cells` function converts a specified number of grid cells, starting from a given position, into a string representation.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid from which cells are to be converted into a string.
    - `px`: An unsigned integer representing the x-coordinate (column) in the grid, relative to the visible screen area.
    - `py`: An unsigned integer representing the y-coordinate (row) in the grid, relative to the visible screen area.
    - `nx`: An unsigned integer representing the number of cells to convert into a string, starting from the specified position.
- **Control Flow**:
    - The function first adjusts the x-coordinate `px` using the `grid_view_x` macro, which currently returns the same value as input.
    - The function then adjusts the y-coordinate `py` using the `grid_view_y` macro, which adds the grid's history size (`hsize`) to the input value, effectively converting the visible screen coordinate to an absolute grid coordinate.
    - Finally, the function calls [`grid_string_cells`](grid.c.driver.md#grid_string_cells), passing the adjusted coordinates and the number of cells `nx` to convert them into a string.
- **Output**:
    - The function returns a pointer to a character string that represents the content of the specified grid cells.
- **Functions called**:
    - [`grid_string_cells`](grid.c.driver.md#grid_string_cells)


