# Purpose
This C source code file is part of the tmux terminal multiplexer project and provides a set of functions for manipulating a grid structure that represents the visible screen area. The code defines a series of operations that can be performed on this grid, such as getting and setting individual cells, inserting and deleting lines or characters, and scrolling regions of the grid. These operations are essential for managing the display of text and other content within a terminal window, allowing for dynamic updates and modifications to the screen's content.

The file is not an executable but rather a component of a larger system, likely intended to be compiled and linked with other parts of the tmux project. It does not define public APIs or external interfaces directly but instead provides internal functionality for handling grid-based screen operations. The functions use a coordinate system relative to the visible screen area, which is adjusted by the grid's history size, allowing for seamless integration with the terminal's scrolling and history features. This code is crucial for maintaining the integrity and performance of the terminal's display, ensuring that content is rendered correctly and efficiently.
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
    - The function calls `grid_get_cell` with the grid, converted x and y coordinates, and the `gc` pointer to retrieve the cell data.
- **Output**:
    - The function does not return a value but populates the `struct grid_cell` pointed to by `gc` with the data of the cell at the specified view-relative coordinates.


---
### grid_view_set_cell<!-- {{#callable:grid_view_set_cell}} -->
The `grid_view_set_cell` function sets a cell in a grid at a specified position using view-relative coordinates.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid where the cell will be set.
    - `px`: An unsigned integer representing the x-coordinate of the cell in view-relative terms.
    - `py`: An unsigned integer representing the y-coordinate of the cell in view-relative terms.
    - `gc`: A pointer to a `struct grid_cell` which contains the data to be set in the specified cell.
- **Control Flow**:
    - The function calls `grid_view_x` to convert the view-relative x-coordinate `px` to an absolute x-coordinate.
    - The function calls `grid_view_y` to convert the view-relative y-coordinate `py` to an absolute y-coordinate.
    - The function calls `grid_set_cell` with the grid, the converted x and y coordinates, and the grid cell data to set the cell in the grid.
- **Output**:
    - This function does not return a value; it performs an operation on the grid by setting a cell.


---
### grid_view_set_padding<!-- {{#callable:grid_view_set_padding}} -->
The `grid_view_set_padding` function sets the padding of a grid using coordinates relative to the visible screen area.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid to modify.
    - `px`: An unsigned integer representing the x-coordinate for padding.
    - `py`: An unsigned integer representing the y-coordinate for padding.
- **Control Flow**:
    - The function calls `grid_view_x` with `gd` and `px` to get the x-coordinate relative to the visible screen area.
    - The function calls `grid_view_y` with `gd` and `py` to get the y-coordinate relative to the visible screen area.
    - The function then calls `grid_set_padding` with `gd`, the calculated x-coordinate, and the calculated y-coordinate to set the padding.
- **Output**:
    - The function does not return a value; it modifies the grid's padding in place.


---
### grid_view_set_cells<!-- {{#callable:grid_view_set_cells}} -->
The `grid_view_set_cells` function sets a series of cells in a grid at specified coordinates using a given grid cell template and string data.
- **Inputs**:
    - `gd`: A pointer to the grid structure where the cells will be set.
    - `px`: The x-coordinate (column) in the grid view where the cells will start being set.
    - `py`: The y-coordinate (row) in the grid view where the cells will start being set.
    - `gc`: A pointer to a `grid_cell` structure that defines the attributes of the cells to be set.
    - `s`: A pointer to a string that contains the data to be set in the grid cells.
    - `slen`: The length of the string `s`.
- **Control Flow**:
    - The function calculates the actual grid coordinates by converting the view coordinates `px` and `py` using `grid_view_x` and `grid_view_y` macros.
    - It then calls the `grid_set_cells` function with the calculated grid coordinates, the grid cell template `gc`, the string `s`, and its length `slen`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by setting the specified cells.


---
### grid_view_clear_history<!-- {{#callable:grid_view_clear_history}} -->
The `grid_view_clear_history` function clears the grid's history by scrolling used lines into the history and clearing the remaining lines.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid to be cleared.
    - `bg`: An unsigned integer representing the background color or attribute to be used when clearing lines.
- **Control Flow**:
    - Initialize `last` to 0 to track the last used line in the grid.
    - Iterate over each line in the grid to find the last line that has been used (i.e., has non-zero cells).
    - If no lines are used (`last` is 0), clear the entire grid using the background color and return.
    - If there are used lines, scroll each used line into the history using `grid_collect_history` and `grid_scroll_history`.
    - If there are unused lines after the last used line, clear them using the background color.
    - Reset the `hscrolled` flag of the grid to 0.
- **Output**:
    - The function does not return a value; it modifies the grid in place by clearing its history and updating its state.
- **Functions called**:
    - [`grid_view_clear`](#grid_view_clear)


---
### grid_view_clear<!-- {{#callable:grid_view_clear}} -->
The `grid_view_clear` function clears a specified rectangular area of a grid, using coordinates relative to the visible screen area, and fills it with a specified background color.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid to be modified.
    - `px`: The x-coordinate of the top-left corner of the area to clear, relative to the visible screen area.
    - `py`: The y-coordinate of the top-left corner of the area to clear, relative to the visible screen area.
    - `nx`: The width of the area to clear.
    - `ny`: The height of the area to clear.
    - `bg`: The background color to fill the cleared area with.
- **Control Flow**:
    - Convert the x-coordinate `px` from view-relative to grid-relative using `grid_view_x` macro.
    - Convert the y-coordinate `py` from view-relative to grid-relative using `grid_view_y` macro.
    - Call the `grid_clear` function with the converted coordinates and dimensions to clear the specified area and fill it with the background color.
- **Output**:
    - The function does not return a value; it modifies the grid in place.


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
    - If the `GRID_HISTORY` flag is set, call `grid_collect_history` to manage the grid's history.
    - If the region to scroll is the entire grid (from `rupper` 0 to `rlower` equal to the grid's height minus one), call `grid_scroll_history` to scroll the entire history.
    - If the region is not the entire grid, convert `rupper` and `rlower` to their view coordinates using `grid_view_y`, and call `grid_scroll_history_region` to scroll only the specified region in the history.
    - If the `GRID_HISTORY` flag is not set, convert `rupper` and `rlower` to their view coordinates using `grid_view_y`, and call `grid_move_lines` to move the lines within the specified region upwards.
- **Output**:
    - The function does not return a value; it performs operations directly on the grid structure passed as an argument.


---
### grid_view_scroll_region_down<!-- {{#callable:grid_view_scroll_region_down}} -->
The `grid_view_scroll_region_down` function scrolls a specified region of a grid downwards by moving lines within the grid.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid to be manipulated.
    - `rupper`: An unsigned integer representing the upper boundary of the region to scroll, relative to the visible screen area.
    - `rlower`: An unsigned integer representing the lower boundary of the region to scroll, relative to the visible screen area.
    - `bg`: An unsigned integer representing the background value to be used when moving lines.
- **Control Flow**:
    - Convert `rupper` and `rlower` from visible screen coordinates to grid coordinates using `grid_view_y` function.
    - Call `grid_move_lines` to move lines from `rupper` to `rlower` down by one position, filling the vacated line with the background value `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place.


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
    - Call `grid_move_lines` to shift existing lines downwards from the insertion point by `ny` lines, filling the new lines with the specified background `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by inserting lines.


---
### grid_view_insert_lines_region<!-- {{#callable:grid_view_insert_lines_region}} -->
The `grid_view_insert_lines_region` function inserts a specified number of lines into a region of a grid, moving existing lines and clearing the newly inserted area.
- **Inputs**:
    - `gd`: A pointer to the grid structure where lines are to be inserted.
    - `rlower`: The lower boundary of the region in the grid where lines are to be inserted, relative to the visible screen area.
    - `py`: The starting y-coordinate in the grid where lines are to be inserted, relative to the visible screen area.
    - `ny`: The number of lines to insert into the grid.
    - `bg`: The background value to use when clearing the newly inserted lines.
- **Control Flow**:
    - Convert `rlower` and `py` to their corresponding positions in the grid using `grid_view_y` function.
    - Calculate `ny2`, the number of lines to move, as `rlower + 1 - py - ny`.
    - Call `grid_move_lines` to move `ny2` lines starting from `rlower + 1 - ny2` to `py`, filling the moved lines with the background `bg`.
    - Call `grid_clear` to clear the area from `py + ny2` to `ny - ny2` with the background `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by inserting lines and clearing the specified region.


---
### grid_view_delete_lines<!-- {{#callable:grid_view_delete_lines}} -->
The `grid_view_delete_lines` function deletes a specified number of lines from a grid starting at a given position and clears the remaining space with a background color.
- **Inputs**:
    - `gd`: A pointer to the grid structure from which lines are to be deleted.
    - `py`: The starting y-coordinate (row) in the grid from which lines will be deleted, relative to the visible screen area.
    - `ny`: The number of lines to delete from the grid.
    - `bg`: The background color to use when clearing the space left by the deleted lines.
- **Control Flow**:
    - Convert the starting y-coordinate `py` to the grid's internal coordinate system using `grid_view_y` function.
    - Determine the grid's internal y-coordinate for the bottom of the visible screen area using `grid_view_y` with `gd->sy`.
    - Call `grid_move_lines` to move lines in the grid, effectively deleting the specified lines starting from `py`.
    - Call `grid_clear` to clear the area that was occupied by the deleted lines, filling it with the specified background color `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by deleting lines and clearing the affected area.


---
### grid_view_delete_lines_region<!-- {{#callable:grid_view_delete_lines_region}} -->
The `grid_view_delete_lines_region` function deletes a specified number of lines from a region within a grid, moving the remaining lines up and clearing the vacated space.
- **Inputs**:
    - `gd`: A pointer to the grid structure from which lines are to be deleted.
    - `rlower`: The lower boundary of the region in the grid from which lines are to be deleted, specified in grid coordinates.
    - `py`: The starting y-coordinate in the grid from which lines are to be deleted, specified in grid coordinates.
    - `ny`: The number of lines to delete from the specified region.
    - `bg`: The background color or attribute to use when clearing the vacated space.
- **Control Flow**:
    - Convert `rlower` and `py` from grid view coordinates to actual grid coordinates using `grid_view_y` function.
    - Calculate `ny2`, the number of lines that will remain after deletion, as `rlower + 1 - py - ny`.
    - Call `grid_move_lines` to move the lines above the deleted region up by `ny` lines.
    - Call `grid_clear` to clear the vacated space in the grid using the specified background color or attribute.
- **Output**:
    - The function does not return a value; it modifies the grid in place by deleting lines and clearing the vacated space.


---
### grid_view_insert_cells<!-- {{#callable:grid_view_insert_cells}} -->
The `grid_view_insert_cells` function inserts a specified number of cells at a given position in a grid, shifting existing cells to the right and filling the new space with a background value if necessary.
- **Inputs**:
    - `gd`: A pointer to the grid structure where cells are to be inserted.
    - `px`: The x-coordinate (column) in the grid where cells are to be inserted, relative to the visible screen area.
    - `py`: The y-coordinate (row) in the grid where cells are to be inserted, relative to the visible screen area.
    - `nx`: The number of cells to insert at the specified position.
    - `bg`: The background value to use for any newly created cells.
- **Control Flow**:
    - Convert the provided x-coordinate `px` to the grid's view coordinate using `grid_view_x` function.
    - Convert the provided y-coordinate `py` to the grid's view coordinate using `grid_view_y` function.
    - Determine the grid's width in view coordinates by converting `gd->sx` using `grid_view_x`.
    - Check if the insertion point `px` is at or beyond the last column (`sx - 1`).
    - If `px` is at or beyond the last column, clear the cell at the position `px, py` with the background value `bg`.
    - If `px` is not at the last column, move existing cells from `px` to `px + nx` to the right, making space for the new cells, and fill the new space with the background value `bg`.
- **Output**:
    - The function does not return a value; it modifies the grid in place by inserting cells and potentially clearing or moving existing cells.


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
    - Convert the visible screen coordinates (px, py) to actual grid coordinates using `grid_view_x` and `grid_view_y`.
    - Determine the grid's width in terms of visible screen coordinates using `grid_view_x`.
    - Call `grid_move_cells` to shift cells to the left, starting from the position after the deleted cells, to fill the gap left by the deletion.
    - Call `grid_clear` to clear the remaining cells at the end of the row, filling them with the background value.
- **Output**:
    - The function does not return a value; it modifies the grid in place by deleting cells and adjusting the remaining cells accordingly.


---
### grid_view_string_cells<!-- {{#callable:grid_view_string_cells}} -->
The `grid_view_string_cells` function converts a specified number of grid cells, starting from a given position, into a string representation.
- **Inputs**:
    - `gd`: A pointer to a `struct grid`, representing the grid from which cells are to be converted into a string.
    - `px`: An unsigned integer representing the x-coordinate (column) in the grid, relative to the visible screen area.
    - `py`: An unsigned integer representing the y-coordinate (row) in the grid, relative to the visible screen area.
    - `nx`: An unsigned integer representing the number of cells to convert into a string, starting from the specified position.
- **Control Flow**:
    - The function first adjusts the x-coordinate `px` using the `grid_view_x` macro, which currently returns the same value.
    - The function then adjusts the y-coordinate `py` using the `grid_view_y` macro, which adds the grid's history size (`hsize`) to the given y-coordinate to account for the visible screen area.
    - Finally, the function calls `grid_string_cells`, passing the adjusted coordinates and the number of cells `nx`, along with additional parameters set to `NULL` and `0`, to obtain the string representation of the specified grid cells.
- **Output**:
    - The function returns a pointer to a character string that represents the content of the specified grid cells.


