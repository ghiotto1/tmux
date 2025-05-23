# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for managing images within a terminal screen. The code is focused on handling images, specifically those in the SIXEL format, which is a bitmap graphics format used in some terminal emulators. The file defines several functions to manage the lifecycle of images, including storing, freeing, and checking images within a screen context. It uses a linked list data structure to keep track of images, allowing for efficient insertion and removal operations.

Key components of the code include functions for freeing images ([`image_free`](#image_free) and [`image_free_all`](#image_free_all)), creating text placeholders for images ([`image_fallback`](#image_fallback)), storing images ([`image_store`](#image_store)), and checking or modifying images based on their position on the screen ([`image_check_line`](#image_check_line), [`image_check_area`](#image_check_area), and [`image_scroll_up`](#image_scroll_up)). The code maintains a global list of all images and a count of them, ensuring that the number of stored images does not exceed a certain limit. This file is not a standalone executable but rather a part of a larger system, likely intended to be compiled and linked with other components of the tmux project. It does not define public APIs or external interfaces but provides internal functionality for image management within the tmux application.
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
- **Description**: The `all_images` variable is a static global variable of type `struct images` that is initialized as a tail queue head using the `TAILQ_HEAD_INITIALIZER` macro. This variable is used to maintain a global list of all image structures currently managed by the program.
- **Use**: `all_images` is used to store and manage a global list of image structures, allowing for operations such as insertion, removal, and traversal of images.


---
### all_images_count
- **Type**: `u_int`
- **Description**: The `all_images_count` variable is a static unsigned integer that keeps track of the total number of images currently stored in the global `all_images` queue. It is used to manage the lifecycle of images, ensuring that the number of images does not exceed a certain limit.
- **Use**: This variable is incremented when a new image is added to the `all_images` queue and decremented when an image is removed.


# Functions

---
### image_free<!-- {{#callable:image_free}} -->
The `image_free` function deallocates memory and removes an image from the global and screen-specific image queues.
- **Inputs**:
    - `im`: A pointer to the `struct image` that is to be freed.
- **Control Flow**:
    - Retrieve the screen associated with the image using `im->s`.
    - Remove the image from the global image queue `all_images` using `TAILQ_REMOVE`.
    - Decrement the global image count `all_images_count`.
    - Remove the image from the screen-specific image queue `s->images` using `TAILQ_REMOVE`.
    - Free the memory allocated for the image's data using `sixel_free`.
    - Free the memory allocated for the image's fallback representation using `free`.
    - Free the memory allocated for the image structure itself using `free`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided image structure.


---
### image_free_all<!-- {{#callable:image_free_all}} -->
The `image_free_all` function frees all images associated with a given screen and returns whether any images were present before freeing.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains a list of images to be freed.
- **Control Flow**:
    - Initialize a variable `redraw` to indicate if the screen needs to be redrawn, based on whether the image list is empty.
    - Iterate over each image in the screen's image list using `TAILQ_FOREACH_SAFE`, which allows safe removal of elements during iteration.
    - For each image, call [`image_free`](#image_free) to deallocate its resources and remove it from the list.
    - Return the value of `redraw`, which is true if there were any images in the list initially.
- **Output**:
    - Returns an integer indicating whether the screen needs to be redrawn (1 if images were present, 0 otherwise).
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
    - Calculate the size of the first line of the placeholder, which includes the image dimensions in a label format.
    - Determine the total size needed for the placeholder buffer based on the width and height provided.
    - Allocate memory for the placeholder buffer and store the pointer in `ret`.
    - Render the first line of the placeholder, adjusting the content based on the width `sx` relative to the label size.
    - Fill the remaining lines of the placeholder with '+' characters, each followed by a newline sequence.
    - Free the memory allocated for the label string.
- **Output**:
    - The function outputs a dynamically allocated string stored in `ret`, representing a text placeholder for an image of the specified dimensions.


---
### image_store<!-- {{#callable:image_store}} -->
The `image_store` function allocates and initializes an image structure, stores it in a screen's image list, and manages a global list of images, freeing the oldest image if a limit is reached.
- **Inputs**:
    - `s`: A pointer to a `screen` structure where the image will be stored.
    - `si`: A pointer to a `sixel_image` structure representing the image data to be stored.
- **Control Flow**:
    - Allocate memory for a new `image` structure using `xcalloc` and assign it to `im`.
    - Set the screen pointer `s` and image data `si` to the newly allocated `image` structure `im`.
    - Initialize the image's position `px` and `py` using the current cursor position `cx` and `cy` from the screen `s`.
    - Calculate the size of the image in cells using `sixel_size_in_cells` and store the dimensions in `im->sx` and `im->sy`.
    - Create a text placeholder for the image using [`image_fallback`](#image_fallback) and store it in `im->fallback`.
    - Insert the new image `im` at the tail of the screen's image list `s->images`.
    - Insert the new image `im` at the tail of the global image list `all_images`.
    - Increment the global image count `all_images_count` and check if it reaches 10.
    - If the global image count reaches 10, free the oldest image in the global list using [`image_free`](#image_free).
    - Return the pointer to the newly created `image` structure `im`.
- **Output**:
    - A pointer to the newly created and stored `image` structure.
- **Functions called**:
    - [`image_fallback`](#image_fallback)
    - [`image_free`](#image_free)


---
### image_check_line<!-- {{#callable:image_check_line}} -->
The `image_check_line` function checks if any images on a screen overlap with a specified line range and removes them if they do, indicating if a redraw is necessary.
- **Inputs**:
    - `s`: A pointer to a `screen` structure that contains a list of images.
    - `py`: An unsigned integer representing the starting y-coordinate of the line range to check.
    - `ny`: An unsigned integer representing the number of lines from the starting y-coordinate to check.
- **Control Flow**:
    - Initialize a variable `redraw` to 0 to track if any images are removed.
    - Iterate over each image `im` in the list of images associated with the screen `s` using `TAILQ_FOREACH_SAFE`.
    - For each image, check if the line range (from `py` to `py + ny`) overlaps with the image's vertical range (from `im->py` to `im->py + im->sy`).
    - If there is an overlap, call `image_free(im)` to remove the image and set `redraw` to 1.
    - Continue iterating until all images have been checked.
- **Output**:
    - Returns an integer `redraw`, which is 1 if any images were removed and 0 otherwise.
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
    - For each image, check if it does not overlap with the specified area by comparing the coordinates and dimensions.
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
    - Iterate over each image in the screen's image list using a safe traversal method.
    - For each image, check if its vertical position (`py`) is greater than or equal to the number of lines to scroll; if so, decrease its `py` by `lines` and mark the screen for redraw.
    - If the image's bottom edge (`py + sy`) is less than or equal to `lines`, free the image and mark the screen for redraw.
    - Otherwise, calculate the new height (`sy`) of the image after scrolling and scale the image data accordingly.
    - Free the old image data and update the image with the new scaled data.
    - Reset the image's vertical position to 0 and update its size in cells.
    - Free the old fallback text and generate a new fallback text for the image.
    - Mark the screen for redraw.
- **Output**:
    - Returns an integer indicating whether any images were redrawn (1 if redraw is needed, 0 otherwise).
- **Functions called**:
    - [`image_free`](#image_free)
    - [`image_fallback`](#image_fallback)


