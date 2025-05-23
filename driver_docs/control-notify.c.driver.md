# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to switch between several programs in one terminal. The file provides a set of functions that handle notifications for various events related to panes, windows, sessions, and paste buffers within tmux. The primary purpose of these functions is to notify clients about changes in the state of these components, such as when a window layout changes, a window is linked or unlinked, a session is renamed, or a paste buffer is modified. Each function iterates over a list of clients and sends a formatted message to those that are in control mode, indicating the specific event that has occurred.

The code is structured around a series of notification functions, each dedicated to a specific type of event. These functions utilize a macro, `CONTROL_SHOULD_NOTIFY_CLIENT`, to determine whether a client should be notified based on its flags. The notifications are sent using the `control_write` function, which formats and dispatches messages to the appropriate clients. This file does not define a public API or external interfaces but rather serves as an internal component of the tmux system, facilitating communication between the tmux server and its clients regarding state changes. The use of the TAILQ_FOREACH macro indicates that the code relies on a queue data structure to manage the list of clients, ensuring efficient iteration and notification delivery.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Functions

---
### control_notify_pane_mode_changed<!-- {{#callable:control_notify_pane_mode_changed}} -->
The function `control_notify_pane_mode_changed` notifies all eligible clients about a change in the mode of a specified pane.
- **Inputs**:
    - `pane`: An integer representing the pane whose mode has changed.
- **Control Flow**:
    - Iterates over all clients in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, checks if the client should be notified using the macro `CONTROL_SHOULD_NOTIFY_CLIENT`.
    - If the client should be notified, calls `control_write` to send a notification message with the format `%%pane-mode-changed %%%u`, where `%u` is replaced by the `pane` argument.
- **Output**:
    - The function does not return any value; it performs its operation by sending notifications to clients.


---
### control_notify_window_layout_changed<!-- {{#callable:control_notify_window_layout_changed}} -->
The function `control_notify_window_layout_changed` notifies all relevant clients about a change in the layout of a specified window.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose layout has changed.
- **Control Flow**:
    - Initialize a template string for the layout change notification message.
    - Iterate over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client should be notified and if it is associated with a session; if not, continue to the next client.
    - Retrieve the session associated with the client.
    - Check if the window is part of the session's windows using [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id); if not, continue to the next client.
    - Check if the window has a layout root; if not, continue to the next client as the window will be removed soon.
    - Find the winlink associated with the window in the session's windows using [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window).
    - If a winlink is found, format a notification message using [`format_single`](format.c.driver.md#format_single) with the template and send it to the client using `control_write`.
    - Free the memory allocated for the formatted message.
- **Output**:
    - The function does not return a value; it sends formatted layout change notifications to relevant clients.
- **Functions called**:
    - [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id)
    - [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window)
    - [`format_single`](format.c.driver.md#format_single)


---
### control_notify_window_pane_changed<!-- {{#callable:control_notify_window_pane_changed}} -->
The function `control_notify_window_pane_changed` notifies all eligible clients about a change in the active pane of a specified window.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose active pane has changed.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client should be notified using the `CONTROL_SHOULD_NOTIFY_CLIENT` macro.
    - If the client should be notified, call `control_write` to send a notification message with the window ID and the active pane ID.
- **Output**:
    - The function does not return any value; it performs its operation by sending notifications to clients.


---
### control_notify_window_unlinked<!-- {{#callable:control_notify_window_unlinked}} -->
The function `control_notify_window_unlinked` notifies all clients about a window being unlinked from a session, sending different messages based on whether the window is still linked to any session.
- **Inputs**:
    - `s`: A pointer to a session structure, which is unused in this function.
    - `w`: A pointer to a window structure representing the window that has been unlinked.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client should be notified and if it has an associated session; if not, continue to the next client.
    - Retrieve the session associated with the current client.
    - Check if the window is still linked to the client's session using [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id).
    - If the window is still linked, send a '%%window-close' message to the client with the window ID.
    - If the window is not linked, send a '%%unlinked-window-close' message to the client with the window ID.
- **Output**:
    - The function does not return a value; it sends notifications to clients via the `control_write` function.
- **Functions called**:
    - [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id)


---
### control_notify_window_linked<!-- {{#callable:control_notify_window_linked}} -->
The function `control_notify_window_linked` notifies all clients about the linking of a window to a session, sending different messages based on whether the window is linked to the client's session or not.
- **Inputs**:
    - `s`: A pointer to a session structure, which is unused in this function.
    - `w`: A pointer to a window structure representing the window that has been linked.
- **Control Flow**:
    - Iterate over each client in the global clients list using TAILQ_FOREACH.
    - For each client, check if the client should be notified and if the client has an associated session.
    - Retrieve the session associated with the client.
    - Check if the window is linked to the client's session using [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id).
    - If the window is linked, send a '%%window-add' message to the client with the window's ID.
    - If the window is not linked, send a '%%unlinked-window-add' message to the client with the window's ID.
- **Output**:
    - The function does not return a value; it sends notifications to clients via the `control_write` function.
- **Functions called**:
    - [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id)


---
### control_notify_window_renamed<!-- {{#callable:control_notify_window_renamed}} -->
The function `control_notify_window_renamed` notifies all relevant clients about a window's name change, indicating whether the window is linked or unlinked.
- **Inputs**:
    - `w`: A pointer to a `window` structure representing the window that has been renamed.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client should be notified using `CONTROL_SHOULD_NOTIFY_CLIENT` and if the client is associated with a session.
    - Retrieve the session associated with the client.
    - Check if the window is part of the session's window list using [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id).
    - If the window is found in the session's window list, send a notification to the client with the message format `%%window-renamed @<window_id> <window_name>`.
    - If the window is not found in the session's window list, send a notification to the client with the message format `%%unlinked-window-renamed @<window_id> <window_name>`.
- **Output**:
    - The function does not return a value; it sends formatted notification messages to clients.
- **Functions called**:
    - [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id)


---
### control_notify_client_session_changed<!-- {{#callable:control_notify_client_session_changed}} -->
The function `control_notify_client_session_changed` notifies all clients about a session change for a specific client.
- **Inputs**:
    - `cc`: A pointer to a `client` structure representing the client whose session has changed.
- **Control Flow**:
    - Check if the session of the client `cc` is NULL; if so, return immediately.
    - Assign the session of `cc` to the local variable `s`.
    - Iterate over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client `c`, check if it should be notified and if its session is not NULL; if not, continue to the next client.
    - If the current client `c` is the same as `cc`, send a 'session-changed' notification with the session ID and name.
    - If the current client `c` is different from `cc`, send a 'client-session-changed' notification with the client name, session ID, and session name.
- **Output**:
    - The function does not return a value; it sends notifications to clients via the `control_write` function.


---
### control_notify_client_detached<!-- {{#callable:control_notify_client_detached}} -->
The function `control_notify_client_detached` notifies all clients that a specific client has been detached.
- **Inputs**:
    - `cc`: A pointer to the `client` structure representing the client that has been detached.
- **Control Flow**:
    - Iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, checks if the client should be notified using the `CONTROL_SHOULD_NOTIFY_CLIENT` macro.
    - If the client should be notified, calls `control_write` to send a message indicating that the specified client has been detached.
- **Output**:
    - The function does not return any value; it performs its operation by sending notifications to clients.


---
### control_notify_session_renamed<!-- {{#callable:control_notify_session_renamed}} -->
The function `control_notify_session_renamed` notifies all eligible clients about a session's name change.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session that has been renamed.
- **Control Flow**:
    - Iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, checks if the client should be notified using the `CONTROL_SHOULD_NOTIFY_CLIENT` macro.
    - If the client should be notified, calls `control_write` to send a message with the session's new name and ID to the client.
- **Output**:
    - The function does not return a value; it performs its operation by sending notifications to clients.


---
### control_notify_session_created<!-- {{#callable:control_notify_session_created}} -->
The function `control_notify_session_created` notifies all clients that a session has been created by sending a '%%sessions-changed' message to each client that should be notified.
- **Inputs**:
    - `s`: A pointer to a session structure, which is unused in this function.
- **Control Flow**:
    - Iterate over each client in the global 'clients' list using TAILQ_FOREACH.
    - For each client, check if the client should be notified using the macro CONTROL_SHOULD_NOTIFY_CLIENT.
    - If the client should be notified, send a '%%sessions-changed' message to the client using the control_write function.
- **Output**:
    - The function does not return any value; it performs actions by sending messages to clients.


---
### control_notify_session_closed<!-- {{#callable:control_notify_session_closed}} -->
The function `control_notify_session_closed` notifies all clients that a session has been closed by sending a '%%sessions-changed' message to each client that should be notified.
- **Inputs**:
    - `s`: A pointer to a session structure, which is unused in this function.
- **Control Flow**:
    - Iterate over each client in the global 'clients' list using TAILQ_FOREACH.
    - For each client, check if the client should be notified using the macro CONTROL_SHOULD_NOTIFY_CLIENT.
    - If the client should be notified, send a '%%sessions-changed' message to the client using the control_write function.
- **Output**:
    - The function does not return any value; it performs actions by sending messages to clients.


---
### control_notify_session_window_changed<!-- {{#callable:control_notify_session_window_changed}} -->
The function `control_notify_session_window_changed` notifies all eligible clients about a change in the current window of a given session.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session whose current window has changed.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client should be notified using the `CONTROL_SHOULD_NOTIFY_CLIENT` macro.
    - If the client should be notified, call `control_write` to send a notification message with the session ID and the current window ID.
- **Output**:
    - The function does not return any value; it performs its operation by sending notifications to clients.


---
### control_notify_paste_buffer_changed<!-- {{#callable:control_notify_paste_buffer_changed}} -->
The function `control_notify_paste_buffer_changed` notifies all eligible clients about a change in the paste buffer by sending a formatted message with the buffer's name.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the paste buffer that has changed.
- **Control Flow**:
    - Iterates over each client in the global `clients` list using the `TAILQ_FOREACH` macro.
    - Checks if the client should be notified using the `CONTROL_SHOULD_NOTIFY_CLIENT` macro, which evaluates to true if the client is not NULL and has the `CLIENT_CONTROL` flag set.
    - If the client should be notified, it sends a message to the client using `control_write`, formatted as '%%paste-buffer-changed' followed by the name of the paste buffer.
- **Output**:
    - The function does not return any value; it performs its operation by sending messages to clients.


---
### control_notify_paste_buffer_deleted<!-- {{#callable:control_notify_paste_buffer_deleted}} -->
The function `control_notify_paste_buffer_deleted` notifies all eligible clients that a paste buffer has been deleted by sending a formatted message.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the paste buffer that has been deleted.
- **Control Flow**:
    - Iterates over each client in the global `clients` list using `TAILQ_FOREACH`.
    - Checks if the client should be notified using the `CONTROL_SHOULD_NOTIFY_CLIENT` macro.
    - If the client is eligible, sends a notification message using `control_write` with the format '%%paste-buffer-deleted %s', where `%s` is replaced by the `name` of the deleted paste buffer.
- **Output**:
    - The function does not return any value; it performs an action by sending notifications to clients.


