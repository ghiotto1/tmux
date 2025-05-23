# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to switch between several programs in one terminal. The file is responsible for handling notifications and hooks within the tmux environment. It defines a set of functions that manage the creation and execution of hooks, which are essentially commands or scripts that are triggered by specific events within tmux, such as changes in window layout, session creation, or client detachment. The code provides a mechanism to register these hooks, execute them when the corresponding events occur, and manage the associated resources.

The file includes several key components, such as the `notify_entry` structure, which encapsulates the details of a notification, including its name, associated client, session, window, and pane. Functions like [`notify_add`](#notify_add), [`notify_hook`](#notify_hook), and [`notify_callback`](#notify_callback) are central to the operation of this notification system, as they handle the registration, execution, and cleanup of hooks. The code is designed to be integrated into the broader tmux application, providing a modular way to extend its functionality through user-defined hooks. It does not define a public API but rather serves as an internal component of tmux, facilitating event-driven behavior within the application.
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
    - If logging is enabled (log level is not 0), print the command list to a string and log the hook name and command list details.
    - Free the memory allocated for the command list string after logging.
    - Retrieve a new command queue item using [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command) with the provided `cmdlist` and `state`.
    - Insert the new command queue item after the specified `item` using [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after).
- **Output**:
    - Returns a pointer to the newly inserted command queue item.
- **Functions called**:
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)


---
### notify_insert_hook<!-- {{#callable:notify_insert_hook}} -->
The `notify_insert_hook` function inserts a hook command into the command queue based on the specified notify entry and its associated options.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the current command queue item.
    - `ne`: A pointer to a `notify_entry` structure containing information about the hook to be inserted, including its name and associated formats.
- **Control Flow**:
    - Log the insertion of the hook with the hook's name.
    - Clear the command find state `fs` and determine its validity based on `ne->fs`.
    - Set the options `oo` based on the session, window pane, or window linked to `fs`.
    - Retrieve the option entry `o` for the hook name from the determined options `oo`.
    - If the option entry `o` is not found, log a debug message and return.
    - Create a new command queue state `state` and add formats from `ne` to it.
    - If the hook name starts with '@', parse the hook's command string and handle parsing errors or success accordingly.
    - If the hook name does not start with '@', iterate over the options array items and insert each command list into the command queue.
    - Free the command queue state `state` after processing.
- **Output**:
    - The function does not return a value; it modifies the command queue by inserting hooks based on the notify entry.
- **Functions called**:
    - [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state)
    - [`cmd_find_empty_state`](cmd-find.c.driver.md#cmd_find_empty_state)
    - [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state)
    - [`cmd_find_from_nothing`](cmd-find.c.driver.md#cmd_find_from_nothing)
    - [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state)
    - [`options_get`](options.c.driver.md#options_get)
    - [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state)
    - [`cmdq_add_formats`](cmd-queue.c.driver.md#cmdq_add_formats)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`notify_insert_one_hook`](#notify_insert_one_hook)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`options_array_next`](options.c.driver.md#options_array_next)
    - [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state)


---
### notify_callback<!-- {{#callable:notify_callback}} -->
The `notify_callback` function processes a notification event by executing specific control functions based on the event name, inserting hooks, and cleaning up resources.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the notification.
    - `data`: A pointer to a `notify_entry` structure containing details about the notification event, such as the event name and associated resources like client, session, window, and pane.
- **Control Flow**:
    - Log the function name and the notification event name for debugging purposes.
    - Check the event name against a series of predefined strings and call the corresponding control function if a match is found.
    - Call [`notify_insert_hook`](#notify_insert_hook) to insert any hooks associated with the notification event.
    - If the `client` field in the `notify_entry` is not NULL, decrement its reference count using [`server_client_unref`](server-client.c.driver.md#server_client_unref).
    - If the `session` field in the `notify_entry` is not NULL, decrement its reference count using [`session_remove_ref`](session.c.driver.md#session_remove_ref).
    - If the `window` field in the `notify_entry` is not NULL, decrement its reference count using [`window_remove_ref`](window.c.driver.md#window_remove_ref).
    - If the `fs.s` field in the `notify_entry` is not NULL, decrement its reference count using [`session_remove_ref`](session.c.driver.md#session_remove_ref).
    - Free the `formats` field in the `notify_entry` using [`format_free`](format.c.driver.md#format_free).
    - Free the memory allocated for the `name` and `pbname` fields in the `notify_entry`.
    - Free the memory allocated for the `notify_entry` itself.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful completion of the notification processing.
- **Functions called**:
    - [`control_notify_pane_mode_changed`](control-notify.c.driver.md#control_notify_pane_mode_changed)
    - [`control_notify_window_layout_changed`](control-notify.c.driver.md#control_notify_window_layout_changed)
    - [`control_notify_window_pane_changed`](control-notify.c.driver.md#control_notify_window_pane_changed)
    - [`control_notify_window_unlinked`](control-notify.c.driver.md#control_notify_window_unlinked)
    - [`control_notify_window_linked`](control-notify.c.driver.md#control_notify_window_linked)
    - [`control_notify_window_renamed`](control-notify.c.driver.md#control_notify_window_renamed)
    - [`control_notify_client_session_changed`](control-notify.c.driver.md#control_notify_client_session_changed)
    - [`control_notify_client_detached`](control-notify.c.driver.md#control_notify_client_detached)
    - [`control_notify_session_renamed`](control-notify.c.driver.md#control_notify_session_renamed)
    - [`control_notify_session_created`](control-notify.c.driver.md#control_notify_session_created)
    - [`control_notify_session_closed`](control-notify.c.driver.md#control_notify_session_closed)
    - [`control_notify_session_window_changed`](control-notify.c.driver.md#control_notify_session_window_changed)
    - [`control_notify_paste_buffer_changed`](control-notify.c.driver.md#control_notify_paste_buffer_changed)
    - [`control_notify_paste_buffer_deleted`](control-notify.c.driver.md#control_notify_paste_buffer_deleted)
    - [`notify_insert_hook`](#notify_insert_hook)
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)
    - [`session_remove_ref`](session.c.driver.md#session_remove_ref)
    - [`window_remove_ref`](window.c.driver.md#window_remove_ref)
    - [`format_free`](format.c.driver.md#format_free)


---
### notify_add<!-- {{#callable:notify_add}} -->
The `notify_add` function creates and queues a notification entry with associated context and formats for later execution.
- **Inputs**:
    - `name`: A string representing the name of the notification event.
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current command find state.
    - `c`: A pointer to a `client` structure representing the client associated with the notification, or NULL if not applicable.
    - `s`: A pointer to a `session` structure representing the session associated with the notification, or NULL if not applicable.
    - `w`: A pointer to a `window` structure representing the window associated with the notification, or NULL if not applicable.
    - `wp`: A pointer to a `window_pane` structure representing the window pane associated with the notification, or NULL if not applicable.
    - `pbname`: A string representing the name of the paste buffer associated with the notification, or NULL if not applicable.
- **Control Flow**:
    - Check if the current command queue item is running and if it has the CMDQ_STATE_NOHOOKS flag set; if so, return immediately.
    - Allocate memory for a new `notify_entry` structure and initialize it with the provided arguments.
    - Create a format tree for the notification and add relevant format entries based on the provided client, session, window, and pane.
    - Log the format tree for debugging purposes.
    - Increment reference counts for the client, session, and window if they are not NULL.
    - Copy the provided command find state into the notification entry and add a reference to the session if it exists.
    - Append the notification entry to the command queue with a callback to `notify_callback`.
- **Output**:
    - The function does not return a value; it operates by side effects, specifically by queuing a notification entry for later execution.
- **Functions called**:
    - [`cmdq_running`](cmd-queue.c.driver.md#cmdq_running)
    - [`cmdq_get_flags`](cmd-queue.c.driver.md#cmdq_get_flags)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_log_debug`](format.c.driver.md#format_log_debug)
    - [`session_add_ref`](session.c.driver.md#session_add_ref)
    - [`window_add_ref`](window.c.driver.md#window_add_ref)
    - [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)


---
### notify_hook<!-- {{#callable:notify_hook}} -->
The `notify_hook` function initializes a `notify_entry` structure with information from a `cmdq_item` and a hook name, then inserts this entry into the notification system.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which represents a command queue item containing context information for the command being executed.
    - `name`: A constant character pointer representing the name of the hook to be notified.
- **Control Flow**:
    - Retrieve the target state from the `cmdq_item` using [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target) and store it in a `cmd_find_state` pointer.
    - Initialize a `notify_entry` structure `ne` to zero using `memset`.
    - Set the `name` field of `ne` to the provided hook name.
    - Copy the target state into the `fs` field of `ne` using [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state).
    - Retrieve the client from the `cmdq_item` and assign it to the `client` field of `ne`.
    - Assign the session, window, and pane from the target state to the corresponding fields in `ne`.
    - Create a format tree for `ne` using [`format_create`](format.c.driver.md#format_create) and add the hook name to it with `format_add`.
    - Log the format tree for debugging purposes using [`format_log_debug`](format.c.driver.md#format_log_debug).
    - Call [`notify_insert_hook`](#notify_insert_hook) to insert the `notify_entry` into the notification system.
    - Free the format tree using [`format_free`](format.c.driver.md#format_free).
- **Output**:
    - The function does not return a value; it operates by side effects, specifically by inserting a notification entry into the system.
- **Functions called**:
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmd_find_copy_state`](cmd-find.c.driver.md#cmd_find_copy_state)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_log_debug`](format.c.driver.md#format_log_debug)
    - [`notify_insert_hook`](#notify_insert_hook)
    - [`format_free`](format.c.driver.md#format_free)


---
### notify_client<!-- {{#callable:notify_client}} -->
The `notify_client` function triggers a notification for a specific client by name, using the client's current state.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification to be triggered.
    - `c`: A pointer to a `client` structure representing the client for which the notification is being triggered.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call [`cmd_find_from_client`](cmd-find.c.driver.md#cmd_find_from_client) to populate `fs` with the state of the client `c`.
    - Call [`notify_add`](#notify_add) with the notification name, the populated `fs`, and the client `c` to add the notification to the queue.
- **Output**:
    - The function does not return any value; it performs its operation by side effects, specifically by adding a notification to the queue.
- **Functions called**:
    - [`cmd_find_from_client`](cmd-find.c.driver.md#cmd_find_from_client)
    - [`notify_add`](#notify_add)


---
### notify_session<!-- {{#callable:notify_session}} -->
The `notify_session` function triggers a notification for a given session by determining its state and invoking the [`notify_add`](#notify_add) function with appropriate parameters.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification event.
    - `s`: A pointer to a `session` structure representing the session to be notified.
- **Control Flow**:
    - Check if the session `s` is alive using `session_alive(s)`.
    - If the session is alive, initialize `fs` using `cmd_find_from_session(&fs, s, 0)`.
    - If the session is not alive, initialize `fs` using `cmd_find_from_nothing(&fs, 0)`.
    - Call [`notify_add`](#notify_add) with `name`, `fs`, and the session `s` as arguments, along with other NULL parameters.
- **Output**:
    - The function does not return a value; it performs its operation by side effects, specifically by calling [`notify_add`](#notify_add) to handle the notification.
- **Functions called**:
    - [`session_alive`](session.c.driver.md#session_alive)
    - [`cmd_find_from_session`](cmd-find.c.driver.md#cmd_find_from_session)
    - [`cmd_find_from_nothing`](cmd-find.c.driver.md#cmd_find_from_nothing)
    - [`notify_add`](#notify_add)


---
### notify_winlink<!-- {{#callable:notify_winlink}} -->
The `notify_winlink` function triggers a notification for a specific winlink by setting up the necessary state and calling the [`notify_add`](#notify_add) function.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification to be triggered.
    - `wl`: A pointer to a `winlink` structure, which contains information about the session and window associated with the winlink.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call [`cmd_find_from_winlink`](cmd-find.c.driver.md#cmd_find_from_winlink) to populate `fs` with the state information from the given `winlink` `wl`.
    - Invoke [`notify_add`](#notify_add) with the notification name, the populated `fs`, and the session and window from the `winlink` `wl`.
- **Output**:
    - This function does not return any value; it performs its operation by side effects, specifically by triggering a notification.
- **Functions called**:
    - [`cmd_find_from_winlink`](cmd-find.c.driver.md#cmd_find_from_winlink)
    - [`notify_add`](#notify_add)


---
### notify_session_window<!-- {{#callable:notify_session_window}} -->
The `notify_session_window` function triggers a notification event for a specific session and window in the tmux environment.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification event to be triggered.
    - `s`: A pointer to a `session` structure representing the tmux session associated with the notification.
    - `w`: A pointer to a `window` structure representing the tmux window associated with the notification.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call [`cmd_find_from_session_window`](cmd-find.c.driver.md#cmd_find_from_session_window) to populate `fs` with information about the session `s` and window `w`.
    - Invoke [`notify_add`](#notify_add) with the event name, the populated `fs`, and the session and window pointers to trigger the notification.
- **Output**:
    - The function does not return any value; it performs its operation by side effects, specifically by triggering a notification event.
- **Functions called**:
    - [`cmd_find_from_session_window`](cmd-find.c.driver.md#cmd_find_from_session_window)
    - [`notify_add`](#notify_add)


---
### notify_window<!-- {{#callable:notify_window}} -->
The `notify_window` function triggers a notification event for a specific window by finding its state and passing it to the notification system.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification event to be triggered.
    - `w`: A pointer to a `window` structure representing the window for which the notification is being triggered.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call [`cmd_find_from_window`](cmd-find.c.driver.md#cmd_find_from_window) to populate `fs` with the state information of the given window `w`.
    - Call [`notify_add`](#notify_add) with the event name, the populated `fs`, and the window `w` to trigger the notification.
- **Output**:
    - The function does not return any value; it performs its operation by side effects, specifically by triggering a notification event.
- **Functions called**:
    - [`cmd_find_from_window`](cmd-find.c.driver.md#cmd_find_from_window)
    - [`notify_add`](#notify_add)


---
### notify_pane<!-- {{#callable:notify_pane}} -->
The `notify_pane` function triggers a notification for a specific window pane by finding its state and adding it to the notification system.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the notification event.
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the notification is being triggered.
- **Control Flow**:
    - Initialize a `cmd_find_state` structure named `fs`.
    - Call [`cmd_find_from_pane`](cmd-find.c.driver.md#cmd_find_from_pane) to populate `fs` with the state information of the given window pane `wp`.
    - Call [`notify_add`](#notify_add) with the notification name, the populated `fs`, and the window pane `wp` to add the notification to the system.
- **Output**:
    - The function does not return any value; it performs its operation by side effects, specifically by adding a notification to the system.
- **Functions called**:
    - [`cmd_find_from_pane`](cmd-find.c.driver.md#cmd_find_from_pane)
    - [`notify_add`](#notify_add)


---
### notify_paste_buffer<!-- {{#callable:notify_paste_buffer}} -->
The `notify_paste_buffer` function triggers a notification event for changes or deletions of a paste buffer.
- **Inputs**:
    - `pbname`: The name of the paste buffer that is being changed or deleted.
    - `deleted`: An integer flag indicating whether the paste buffer was deleted (non-zero) or changed (zero).
- **Control Flow**:
    - Initialize a `cmd_find_state` structure `fs` and clear its state using [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state).
    - Check if the `deleted` flag is non-zero; if true, call [`notify_add`](#notify_add) with the event name 'paste-buffer-deleted' and the paste buffer name `pbname`.
    - If the `deleted` flag is zero, call [`notify_add`](#notify_add) with the event name 'paste-buffer-changed' and the paste buffer name `pbname`.
- **Output**:
    - The function does not return a value; it triggers a notification event based on the paste buffer's state.
- **Functions called**:
    - [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state)
    - [`notify_add`](#notify_add)


