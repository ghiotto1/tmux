# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to switch easily between several programs in one terminal. The file is responsible for handling notifications and hooks within the tmux environment. It defines a set of functions that manage the creation and execution of hooks, which are essentially commands or scripts that are triggered in response to specific events within tmux, such as changes in pane modes, window layouts, or session states. The code provides a mechanism to register these hooks, execute them when the corresponding events occur, and manage the associated resources.

The file includes several key components, such as the `notify_entry` structure, which encapsulates the details of a notification, including its name, associated client, session, window, and pane. Functions like [`notify_add`](#notify_add), [`notify_hook`](#notify_hook), and [`notify_callback`](#notify_callback) are central to the process of adding and executing hooks. The code also interacts with other parts of the tmux system, such as clients, sessions, and windows, to ensure that hooks are executed in the correct context. This file does not define a public API but rather serves as an internal component of tmux, facilitating the dynamic execution of commands in response to user or system-triggered events.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Data Structures

---
### notify_entry
- **Type**: `struct`
- **Members**:
    - `name`: A constant character pointer representing the name of the notification.
    - `fs`: A structure of type cmd_find_state used to store the state of a command find operation.
    - `formats`: A pointer to a format_tree structure used for formatting data.
    - `client`: A pointer to a client structure representing the client associated with the notification.
    - `session`: A pointer to a session structure representing the session associated with the notification.
    - `window`: A pointer to a window structure representing the window associated with the notification.
    - `pane`: An integer representing the pane ID associated with the notification.
    - `pbname`: A constant character pointer representing the name of the paste buffer associated with the notification.
- **Description**: The `notify_entry` structure is used to encapsulate information related to a notification event in the tmux application. It includes details such as the name of the notification, the state of command find operations, and pointers to associated client, session, window, and pane structures. Additionally, it holds a format tree for data formatting and a paste buffer name if applicable. This structure is integral to managing and processing notification events within the tmux environment.


# Functions

---
### notify_insert_one_hook<!-- {{#callable:notify_insert_one_hook}} -->
The `notify_insert_one_hook` function inserts a new command queue item after a specified item if a command list is provided, and logs the hook details if logging is enabled.
- **Inputs**:
    - `item`: A pointer to the current command queue item after which the new item will be inserted.
    - `ne`: A pointer to a `notify_entry` structure containing details about the notification hook.
    - `cmdlist`: A pointer to a `cmd_list` structure representing the list of commands to be executed.
    - `state`: A pointer to a `cmdq_state` structure representing the current state of the command queue.
- **Control Flow**:
    - Check if `cmdlist` is NULL; if so, return the original `item` without modification.
    - If logging is enabled (log level is not 0), print the command list using `cmd_list_print` and log the hook name and command list string using `log_debug`.
    - Free the memory allocated for the command list string after logging.
    - Create a new command queue item using `cmdq_get_command` with the provided `cmdlist` and `state`.
    - Insert the new command queue item after the specified `item` using `cmdq_insert_after` and return the new item.
- **Output**:
    - Returns a pointer to the newly inserted command queue item, or the original item if no insertion occurs.


---
### notify_insert_hook<!-- {{#callable:notify_insert_hook}} -->
The `notify_insert_hook` function inserts a command hook into the command queue based on the specified notify entry and its associated options.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the current command queue item.
    - `ne`: A pointer to a `notify_entry` structure containing information about the notification, including its name and associated formats.
- **Control Flow**:
    - Log the insertion of the hook with the name from the notify entry.
    - Clear the command find state and determine the appropriate options based on the notify entry's state.
    - Attempt to retrieve the options entry for the hook name from various option sources (session, window pane, window).
    - If the options entry is not found, log a debug message and return.
    - Create a new command queue state and add formats from the notify entry to it.
    - If the hook name starts with '@', parse the hook as a string command and handle parsing errors or success accordingly.
    - If the hook name does not start with '@', iterate over the options array items and insert each command list into the command queue.
    - Free the command queue state after processing.
- **Output**:
    - The function does not return a value; it modifies the command queue by inserting hooks based on the notify entry.
- **Functions called**:
    - [`notify_insert_one_hook`](#notify_insert_one_hook)


---
### notify_callback<!-- {{#callable:notify_callback}} -->
The `notify_callback` function processes a notification event by executing specific control functions based on the event name, inserting hooks, and cleaning up resources.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the notification.
    - `data`: A pointer to a `notify_entry` structure containing details about the notification event, such as the event name and associated resources.
- **Control Flow**:
    - Log the event name using `log_debug`.
    - Check the event name against a series of predefined strings, and call the corresponding control function if a match is found.
    - Call [`notify_insert_hook`](#notify_insert_hook) to insert any hooks associated with the notification.
    - Release references to the client, session, window, and any other resources if they are not NULL.
    - Free the memory allocated for the `notify_entry` structure and its associated strings.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful processing of the notification.
- **Functions called**:
    - [`notify_insert_hook`](#notify_insert_hook)


---
### notify_add<!-- {{#callable:notify_add}} -->
The `notify_add` function creates and initializes a notification entry, then appends it to the command queue for processing.
- **Inputs**:
    - `name`: A string representing the name of the notification event.
    - `fs`: A pointer to a `cmd_find_state` structure representing the command find state.
    - `c`: A pointer to a `client` structure representing the client associated with the notification.
    - `s`: A pointer to a `session` structure representing the session associated with the notification.
    - `w`: A pointer to a `window` structure representing the window associated with the notification.
    - `wp`: A pointer to a `window_pane` structure representing the window pane associated with the notification, or NULL if not applicable.
    - `pbname`: A string representing the name of the paste buffer associated with the notification, or NULL if not applicable.
- **Control Flow**:
    - Check if the current command queue item is running and if it has the CMDQ_STATE_NOHOOKS flag set; if so, return immediately.
    - Allocate memory for a new `notify_entry` structure and initialize its fields with the provided arguments.
    - Create a new format tree for the notification entry and add relevant format variables based on the provided arguments.
    - Increment reference counts for the client, session, and window if they are not NULL.
    - Copy the command find state from the provided `fs` to the notification entry's `fs` and add a reference to the session if it exists.
    - Append the notification entry to the command queue with a callback to `notify_callback`.
- **Output**:
    - The function does not return a value; it modifies the command queue by appending a new notification entry.


---
### notify_hook<!-- {{#callable:notify_hook}} -->
The `notify_hook` function initializes a `notify_entry` structure with information from a command queue item and a hook name, then inserts this entry into the notification system.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item from which target information is extracted.
    - `name`: A constant character pointer representing the name of the hook to be notified.
- **Control Flow**:
    - Retrieve the target state from the command queue item using `cmdq_get_target` and store it in a `cmd_find_state` pointer.
    - Initialize a `notify_entry` structure `ne` to zero using `memset`.
    - Assign the hook name to `ne.name`.
    - Copy the target state into `ne.fs` using `cmd_find_copy_state`.
    - Retrieve the client from the command queue item and assign it to `ne.client`.
    - Assign the session, window, and pane information from the target state to `ne.session`, `ne.window`, and `ne.pane`, respectively.
    - Create a format tree for `ne.formats` using `format_create` with no jobs.
    - Add the hook name to the formats using `format_add`.
    - Log the format tree for debugging using `format_log_debug`.
    - Insert the `notify_entry` into the notification system using [`notify_insert_hook`](#notify_insert_hook).
    - Free the format tree using `format_free`.
- **Output**:
    - The function does not return a value; it operates by side effects, specifically by inserting a notification entry into the system.
- **Functions called**:
    - [`notify_insert_hook`](#notify_insert_hook)


---
### notify_client<!-- {{#callable:notify_client}} -->
The `notify_client` function triggers a notification for a specific client by name, using the client's context to find the appropriate command state.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification to be triggered.
    - `c`: A pointer to a `client` structure representing the client for which the notification is being triggered.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call `cmd_find_from_client` to populate `fs` with the command state derived from the client `c`.
    - Invoke [`notify_add`](#notify_add) with the notification name, the populated `fs`, and the client `c` to add the notification to the queue.
- **Output**:
    - The function does not return any value; it performs its operations by side effects, specifically by adding a notification to the command queue.
- **Functions called**:
    - [`notify_add`](#notify_add)


---
### notify_session<!-- {{#callable:notify_session}} -->
The `notify_session` function triggers a notification for a given session by determining its state and invoking the [`notify_add`](#notify_add) function with appropriate parameters.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification event.
    - `s`: A pointer to a `session` structure representing the session to be notified.
- **Control Flow**:
    - Check if the session `s` is alive using `session_alive(s)`.
    - If the session is alive, initialize `fs` using `cmd_find_from_session` with the session `s`.
    - If the session is not alive, initialize `fs` using `cmd_find_from_nothing`.
    - Call [`notify_add`](#notify_add) with the notification name, the initialized `fs`, and the session `s`.
- **Output**:
    - The function does not return a value; it performs its operations by side effects, specifically by calling [`notify_add`](#notify_add) to handle the notification.
- **Functions called**:
    - [`notify_add`](#notify_add)


---
### notify_winlink<!-- {{#callable:notify_winlink}} -->
The `notify_winlink` function triggers a notification for a specific winlink by setting up the necessary state and calling the [`notify_add`](#notify_add) function.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification to be triggered.
    - `wl`: A pointer to a `winlink` structure, which represents the winlink for which the notification is being triggered.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call `cmd_find_from_winlink` to populate `fs` with the state information from the given winlink `wl`.
    - Invoke [`notify_add`](#notify_add) with the notification name, the populated `fs`, and the session and window from the winlink `wl`.
- **Output**:
    - This function does not return any value; it performs its operations by side effects, specifically by triggering a notification.
- **Functions called**:
    - [`notify_add`](#notify_add)


---
### notify_session_window<!-- {{#callable:notify_session_window}} -->
The `notify_session_window` function triggers a notification event for a specific session and window in the tmux environment.
- **Inputs**:
    - `name`: A string representing the name of the notification event to be triggered.
    - `s`: A pointer to the session structure associated with the notification.
    - `w`: A pointer to the window structure associated with the notification.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure `fs`.
    - Call `cmd_find_from_session_window` to populate `fs` with information about the session `s` and window `w`.
    - Invoke [`notify_add`](#notify_add) with the event name, the populated `fs`, and the session and window pointers to trigger the notification.
- **Output**:
    - This function does not return a value; it performs its operation by side effects, specifically by triggering a notification event.
- **Functions called**:
    - [`notify_add`](#notify_add)


---
### notify_window<!-- {{#callable:notify_window}} -->
The `notify_window` function triggers a notification event for a specific window by setting up the necessary state and calling the [`notify_add`](#notify_add) function.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification event to be triggered.
    - `w`: A pointer to a `window` structure representing the window for which the notification is being triggered.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call `cmd_find_from_window` to populate `fs` with the state information of the given window `w`.
    - Invoke [`notify_add`](#notify_add) with the event name, the populated `fs`, and the window `w` to trigger the notification.
- **Output**:
    - The function does not return any value; it performs its operation by side effects, specifically by triggering a notification event for the specified window.
- **Functions called**:
    - [`notify_add`](#notify_add)


---
### notify_pane<!-- {{#callable:notify_pane}} -->
The `notify_pane` function triggers a notification for a specific window pane by finding its state and adding it to the notification system.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification event.
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the notification is being triggered.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call `cmd_find_from_pane` to populate `fs` with the state of the specified window pane `wp`.
    - Invoke [`notify_add`](#notify_add) with the event name, the populated `fs`, and the window pane `wp` to add the notification.
- **Output**:
    - The function does not return any value; it performs its operation by side effects, specifically by adding a notification to the system.
- **Functions called**:
    - [`notify_add`](#notify_add)


---
### notify_paste_buffer<!-- {{#callable:notify_paste_buffer}} -->
The `notify_paste_buffer` function triggers a notification event for changes or deletions of a paste buffer.
- **Inputs**:
    - `pbname`: A constant character pointer representing the name of the paste buffer.
    - `deleted`: An integer flag indicating whether the paste buffer was deleted (non-zero) or changed (zero).
- **Control Flow**:
    - Initialize a `cmd_find_state` structure `fs` and clear its state using `cmd_find_clear_state`.
    - Check if the `deleted` flag is non-zero; if true, call [`notify_add`](#notify_add) with the event name 'paste-buffer-deleted'.
    - If the `deleted` flag is zero, call [`notify_add`](#notify_add) with the event name 'paste-buffer-changed'.
- **Output**:
    - The function does not return a value; it triggers a notification event based on the paste buffer's state.
- **Functions called**:
    - [`notify_add`](#notify_add)


