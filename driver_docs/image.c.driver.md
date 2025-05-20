# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to switch between several programs in one terminal. The file is focused on managing images within a tmux session, specifically handling the storage, rendering, and cleanup of images displayed on the terminal screen. The code provides functionality to store images, create text placeholders for them, and manage their lifecycle, including freeing memory when images are no longer needed or when the screen is scrolled. The file uses a linked list structure to keep track of images, ensuring efficient insertion and removal operations.

Key technical components include the use of the `TAILQ` macros from the BSD queue library to manage lists of images, and functions for creating and freeing images. The [`image_store`](#image_store) function is responsible for storing a new image and managing its position on the screen, while [`image_free`](#image_free) and [`image_free_all`](#image_free_all) handle the cleanup of image resources. The code also includes functions to check and update the screen area occupied by images, such as [`image_check_line`](#image_check_line), [`image_check_area`](#image_check_area), and [`image_scroll_up`](#image_scroll_up), which adjust image positions or remove them as necessary when the screen content changes. This file is not a standalone executable but rather a component of the tmux codebase, providing specific functionality related to image management within the terminal multiplexer environment.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### all_images
- **Type**: `struct images`
- **Description**: The `all_images` variable is a static global variable of type `struct images`, initialized as a tail queue head using the `TAILQ_HEAD_INITIALIZER` macro. It is used to manage a global list of image structures within the program.
- **Use**: This variable is used to store and manage a list of all image structures, allowing for operations such as insertion, removal, and traversal of images in a global context.


---
### all_images_count
- **Type**: `u_int`
- **Description**: The `all_images_count` variable is a static unsigned integer that keeps track of the total number of images currently stored in the global `all_images` queue. It is used to manage the lifecycle of images, ensuring that the number of images does not exceed a certain limit.
- **Use**: This variable is incremented when a new image is added to the `all_images` queue and decremented when an image is removed.


# Functions

---
### image_free<!-- {{#callable:image_free}} -->
The `image_free` function deallocates memory and removes an image from the global and screen-specific image lists.
- **Inputs**:
    - `im`: A pointer to the `image` structure that is to be freed.
- **Control Flow**:
    - Retrieve the screen associated with the image using `im->s`.
    - Remove the image from the global list `all_images` using `TAILQ_REMOVE`.
    - Decrement the global image count `all_images_count`.
    - Remove the image from the screen-specific list `s->images` using `TAILQ_REMOVE`.
    - Free the memory associated with the image's data using [`sixel_free`](image-sixel.c.driver.md#sixel_free).
    - Free the memory allocated for the image's fallback representation using `free`.
    - Free the memory allocated for the image structure itself using `free`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided image structure.
- **Functions called**:
    - [`sixel_free`](image-sixel.c.driver.md#sixel_free)


---
### image_free_all<!-- {{#callable:image_free_all}} -->
The `image_free_all` function frees all images associated with a given screen and returns whether a redraw is necessary.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains a list of images to be freed.
- **Control Flow**:
    - Initialize a variable `redraw` to indicate if the screen needs to be redrawn, based on whether the image list is empty.
    - Iterate over each image in the screen's image list using `TAILQ_FOREACH_SAFE`, which allows safe removal of elements during iteration.
    - For each image, call [`image_free`](#image_free) to deallocate its resources and remove it from the list.
    - Return the value of `redraw`, which indicates if the screen had any images and thus needs to be redrawn.
- **Output**:
    - An integer indicating whether the screen needs to be redrawn (1 if there were images, 0 if not).
- **Functions called**:
    - [`image_free`](#image_free)


---
### image_fallback<!-- {{#callable:image_fallback}} -->
The `image_fallback` function creates a text placeholder for an image using a specified width and height.
- **Inputs**:
    - `ret`: A pointer to a character pointer where the generated text placeholder will be stored.
    - `sx`: The width of the image placeholder in characters.
    - `sy`: The height of the image placeholder in lines.
- **Control Flow**:
    - Calculate the size of the first line of the placeholder, which includes the image dimensions in the format 'SIXEL IMAGE (widthxheight)\r\n'.
    - Determine the total size needed for the placeholder buffer, considering the width and height of the image.
    - Allocate memory for the placeholder buffer and assign it to the `ret` pointer.
    - Render the first line of the placeholder, adjusting the content based on whether the width is sufficient to include the entire label or needs to be truncated with '+' characters.
    - Fill the remaining lines of the placeholder with '+' characters, each followed by a '\r\n'.
    - Free the memory allocated for the label string.
- **Output**:
    - The function outputs a dynamically allocated string stored in `ret`, which represents a text placeholder for an image of the specified dimensions.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### image_store<!-- {{#callable:image_store}} -->
The `image_store` function allocates and initializes an image structure, stores it in a screen's image list, and manages a global list of images, freeing the oldest image if a limit is reached.
- **Inputs**:
    - `s`: A pointer to a `screen` structure where the image will be stored.
    - `si`: A pointer to a `sixel_image` structure representing the image data to be stored.
- **Control Flow**:
    - Allocate memory for a new `image` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and assign it to `im`.
    - Set the screen pointer `s` and image data `si` to the newly allocated `image` structure `im`.
    - Initialize the image's position `px` and `py` using the current cursor position `cx` and `cy` from the screen `s`.
    - Calculate the size of the image in cells using [`sixel_size_in_cells`](image-sixel.c.driver.md#sixel_size_in_cells) and store the dimensions in `im->sx` and `im->sy`.
    - Create a text placeholder for the image using [`image_fallback`](#image_fallback) and store it in `im->fallback`.
    - Insert the new image `im` at the tail of the screen's image list `s->images`.
    - Insert the new image `im` at the tail of the global image list `all_images`.
    - Check if the global image count `all_images_count` has reached 10; if so, free the oldest image in the global list using [`image_free`](#image_free).
    - Return the pointer to the newly created `image` structure `im`.
- **Output**:
    - A pointer to the newly created and initialized `image` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`sixel_size_in_cells`](image-sixel.c.driver.md#sixel_size_in_cells)
    - [`image_fallback`](#image_fallback)
    - [`image_free`](#image_free)


---
### image_check_line<!-- {{#callable:image_check_line}} -->
The function `image_check_line` checks if any images on a screen overlap with a specified line range and removes them if they do, indicating if a redraw is necessary.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains a list of images to be checked.
    - `py`: An unsigned integer representing the starting y-coordinate of the line range to check.
    - `ny`: An unsigned integer representing the number of lines from `py` to check for overlap with images.
- **Control Flow**:
    - Initialize a variable `redraw` to 0 to track if any images are removed.
    - Iterate over each image in the screen's image list using `TAILQ_FOREACH_SAFE`.
    - For each image, check if the line range (`py` to `py + ny`) overlaps with the image's vertical range (`im->py` to `im->py + im->sy`).
    - If an overlap is detected, call `image_free(im)` to remove the image and set `redraw` to 1.
    - Continue checking the next image until all images have been processed.
    - Return the value of `redraw` to indicate if any images were removed.
- **Output**:
    - An integer value indicating whether any images were removed (1 if any images were removed, 0 otherwise).
- **Functions called**:
    - [`image_free`](#image_free)


---
### image_check_area<!-- {{#callable:image_check_area}} -->
The `image_check_area` function checks if any images on a screen overlap with a specified rectangular area and removes them if they do, indicating if a redraw is necessary.
- **Inputs**:
    - `s`: A pointer to a `screen` structure that contains a list of images.
    - `px`: The x-coordinate of the top-left corner of the area to check.
    - `py`: The y-coordinate of the top-left corner of the area to check.
    - `nx`: The width of the area to check.
    - `ny`: The height of the area to check.
- **Control Flow**:
    - Initialize a variable `redraw` to 0 to track if any images are removed.
    - Iterate over each image in the screen's image list using `TAILQ_FOREACH_SAFE`.
    - For each image, check if it does not overlap with the specified area by comparing coordinates and dimensions.
    - If the image does not overlap, continue to the next image.
    - If the image overlaps, call [`image_free`](#image_free) to remove the image and set `redraw` to 1.
    - After checking all images, return the value of `redraw`.
- **Output**:
    - Returns an integer indicating whether any images were removed (1 if any images were removed, 0 otherwise).
- **Functions called**:
    - [`image_free`](#image_free)


---
### image_scroll_up<!-- {{#callable:image_scroll_up}} -->
The `image_scroll_up` function scrolls images on a screen upwards by a specified number of lines, adjusting or removing images as necessary.
- **Inputs**:
    - `s`: A pointer to a `screen` structure that contains a list of images to be scrolled.
    - `lines`: An unsigned integer representing the number of lines to scroll the images upwards.
- **Control Flow**:
    - Initialize a pointer to iterate over images and a redraw flag set to 0.
    - Iterate over each image in the screen's image list using a safe loop to allow for removal of images during iteration.
    - If the image's vertical position (`py`) is greater than or equal to the number of lines to scroll, decrease `py` by `lines` and set the redraw flag.
    - If the image's bottom edge (`py + sy`) is less than or equal to the number of lines to scroll, free the image and set the redraw flag.
    - For images partially scrolled off the top, calculate the new height (`sy`) after scrolling and scale the image data accordingly.
    - Free the old image data and update the image with the new scaled data.
    - Reset the image's vertical position to 0 and update its size in cells.
    - Free the old fallback text and generate a new fallback text for the image.
    - Set the redraw flag to indicate that changes have been made.
- **Output**:
    - Returns an integer indicating whether any images were redrawn (1 if any image was redrawn, 0 otherwise).
- **Functions called**:
    - [`image_free`](#image_free)
    - [`sixel_scale`](image-sixel.c.driver.md#sixel_scale)
    - [`sixel_free`](image-sixel.c.driver.md#sixel_free)
    - [`sixel_size_in_cells`](image-sixel.c.driver.md#sixel_size_in_cells)
    - [`image_fallback`](#image_fallback)


