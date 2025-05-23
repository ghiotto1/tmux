# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for managing alert notifications within tmux sessions. The code provides functionality to monitor and respond to various alert conditions such as bell, activity, and silence in tmux windows. It defines several static functions to check for these conditions and to queue alerts when they occur. The alerts are managed using a timer and callback mechanism, which ensures that alerts are processed and appropriate actions are taken, such as notifying the user or updating the session status.

The file includes key components such as the [`alerts_timer`](#alerts_timer), [`alerts_callback`](#alerts_callback), and `alerts_check_*` functions, which collectively handle the detection and processing of alert conditions. The code uses data structures like `TAILQ_HEAD` to maintain a list of windows with pending alerts and employs event-driven programming techniques to manage alert timers and callbacks. The file does not define a public API but rather serves as an internal component of the tmux system, focusing on the alert management aspect. It interacts with other parts of the tmux codebase, such as session and window management, to ensure that alerts are appropriately handled and communicated to the user.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### alerts_fired
- **Type**: `int`
- **Description**: The `alerts_fired` variable is a static integer that tracks whether any alerts have been triggered in the system. It is used to ensure that alert checks are only queued once when necessary.
- **Use**: This variable is used to prevent multiple alert checks from being queued simultaneously by setting it to 1 when an alert is queued and resetting it to 0 after the alert callback is executed.


---
### alerts_list
- **Type**: `TAILQ_HEAD(, window)`
- **Description**: The `alerts_list` is a global variable that is a tail queue head for managing a list of `window` structures. It is initialized using the `TAILQ_HEAD_INITIALIZER` macro, which sets up the queue to be empty initially.
- **Use**: This variable is used to maintain a list of windows that have pending alerts, allowing the program to efficiently manage and process alert events for each window.


# Functions

---
### alerts_timer<!-- {{#callable:alerts_timer}} -->
The `alerts_timer` function logs a debug message when an alert timer expires for a window and queues a silence alert for that window.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `arg`: A pointer to a `struct window`, representing the window for which the alert timer has expired.
- **Control Flow**:
    - The function casts the `arg` parameter to a `struct window` pointer and assigns it to `w`.
    - It logs a debug message indicating that the alert timer for the window with ID `w->id` has expired.
    - It calls the [`alerts_queue`](#alerts_queue) function with the window `w` and the `WINDOW_SILENCE` flag to queue a silence alert for the window.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`alerts_queue`](#alerts_queue)


---
### alerts_callback<!-- {{#callable:alerts_callback}} -->
The `alerts_callback` function processes queued alert checks for each window in the alerts list, updating their status and removing them from the list.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `arg`: An unused void pointer for additional arguments.
- **Control Flow**:
    - Iterates over each window in the `alerts_list` using `TAILQ_FOREACH_SAFE` to safely traverse and modify the list.
    - For each window, calls [`alerts_check_all`](#alerts_check_all) to determine the current alert status and logs the result.
    - Resets the `alerts_queued` flag for the window to 0, indicating it is no longer queued for alerts.
    - Removes the window from the `alerts_list` using `TAILQ_REMOVE`.
    - Clears the `WINDOW_ALERTFLAGS` from the window's flags to reset its alert status.
    - Calls [`window_remove_ref`](window.c.driver.md#window_remove_ref) to decrease the reference count of the window, potentially allowing it to be freed if no longer in use.
    - Sets the global `alerts_fired` flag to 0, indicating that no alerts are currently being processed.
- **Output**:
    - The function does not return a value; it operates by modifying the state of windows in the `alerts_list` and updating global and window-specific flags.
- **Functions called**:
    - [`alerts_check_all`](#alerts_check_all)
    - [`window_remove_ref`](window.c.driver.md#window_remove_ref)


---
### alerts_action_applies<!-- {{#callable:alerts_action_applies}} -->
The function `alerts_action_applies` determines if an alert action should be applied to a given window link based on the specified alert type and the current window state.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure representing a window link in a session.
    - `name`: A string representing the name of the alert action to check (e.g., 'bell-action', 'activity-action', 'silence-action').
- **Control Flow**:
    - Retrieve the alert action setting for the given name from the session's options associated with the window link `wl`.
    - Check if the action is `ALERT_ANY`, and if so, return 1 indicating the alert applies to any window.
    - Check if the action is `ALERT_CURRENT`, and if so, return 1 if the window link `wl` is the current window in the session, otherwise return 0.
    - Check if the action is `ALERT_OTHER`, and if so, return 1 if the window link `wl` is not the current window in the session, otherwise return 0.
    - If none of the above conditions are met, return 0 indicating the alert does not apply.
- **Output**:
    - Returns an integer (1 or 0) indicating whether the alert action applies to the given window link based on the specified alert type and current window state.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### alerts_check_all<!-- {{#callable:alerts_check_all}} -->
The `alerts_check_all` function checks for bell, activity, and silence alerts on a given window and returns a combined alert status.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window to check for alerts.
- **Control Flow**:
    - Initialize an integer variable `alerts` to store the alert status.
    - Call [`alerts_check_bell`](#alerts_check_bell) with the window `w` and store the result in `alerts`.
    - Perform a bitwise OR operation on `alerts` with the result of [`alerts_check_activity`](#alerts_check_activity) called with `w`.
    - Perform another bitwise OR operation on `alerts` with the result of [`alerts_check_silence`](#alerts_check_silence) called with `w`.
    - Return the combined `alerts` value.
- **Output**:
    - The function returns an integer representing the combined alert status for bell, activity, and silence alerts on the specified window.
- **Functions called**:
    - [`alerts_check_bell`](#alerts_check_bell)
    - [`alerts_check_activity`](#alerts_check_activity)
    - [`alerts_check_silence`](#alerts_check_silence)


---
### alerts_check_session<!-- {{#callable:alerts_check_session}} -->
The `alerts_check_session` function iterates over all windows in a given session and checks for any alerts in each window.
- **Inputs**:
    - `s`: A pointer to a `session` structure, representing the session whose windows are to be checked for alerts.
- **Control Flow**:
    - The function uses the `RB_FOREACH` macro to iterate over each `winlink` in the session's `windows` red-black tree.
    - For each `winlink`, it calls the [`alerts_check_all`](#alerts_check_all) function with the associated `window` to check for any alerts.
- **Output**:
    - The function does not return any value; it performs its operations directly on the session's windows.
- **Functions called**:
    - [`alerts_check_all`](#alerts_check_all)


---
### alerts_enabled<!-- {{#callable:alerts_enabled}} -->
The `alerts_enabled` function checks if alerts are enabled for a given window based on specified flags for bell, activity, or silence.
- **Inputs**:
    - `w`: A pointer to a `struct window` which contains the options to be checked for alert settings.
    - `flags`: An integer representing the alert flags to be checked, which can include WINDOW_BELL, WINDOW_ACTIVITY, and WINDOW_SILENCE.
- **Control Flow**:
    - Check if the WINDOW_BELL flag is set in the `flags` argument.
    - If WINDOW_BELL is set, check if the 'monitor-bell' option is enabled for the window; if so, return 1.
    - Check if the WINDOW_ACTIVITY flag is set in the `flags` argument.
    - If WINDOW_ACTIVITY is set, check if the 'monitor-activity' option is enabled for the window; if so, return 1.
    - Check if the WINDOW_SILENCE flag is set in the `flags` argument.
    - If WINDOW_SILENCE is set, check if the 'monitor-silence' option is non-zero for the window; if so, return 1.
    - If none of the conditions are met, return 0.
- **Output**:
    - Returns 1 if any of the specified alert conditions are enabled for the window, otherwise returns 0.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### alerts_reset_all<!-- {{#callable:alerts_reset_all}} -->
The `alerts_reset_all` function iterates over all windows and resets their alert states.
- **Inputs**:
    - None
- **Control Flow**:
    - The function uses the `RB_FOREACH` macro to iterate over all windows in the `windows` red-black tree.
    - For each window, it calls the [`alerts_reset`](#alerts_reset) function to reset its alert state.
- **Output**:
    - The function does not return any value; it performs its operations directly on the window structures.
- **Functions called**:
    - [`alerts_reset`](#alerts_reset)


---
### alerts_reset<!-- {{#callable:alerts_reset}} -->
The `alerts_reset` function resets the alert timer for a given window and reinitializes it based on the 'monitor-silence' option.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window for which the alert timer is being reset.
- **Control Flow**:
    - Check if the alert timer for the window is initialized; if not, initialize it with `evtimer_set`.
    - Clear the `WINDOW_SILENCE` flag from the window's flags.
    - Remove the current alert timer event using `event_del`.
    - Clear the `timeval` structure `tv` and set its seconds to the value obtained from the window's options for 'monitor-silence'.
    - Log the reset action with the window ID and the silence time.
    - If the silence time is not zero, add the alert timer event back with the new timeout value.
- **Output**:
    - The function does not return a value; it modifies the state of the window's alert timer and flags.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### alerts_queue<!-- {{#callable:alerts_queue}} -->
The `alerts_queue` function manages the queuing of alert events for a given window based on specified flags, ensuring alerts are processed if enabled.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window for which alerts are being managed.
    - `flags`: An integer representing the alert flags that need to be checked or set for the window.
- **Control Flow**:
    - The function begins by resetting the alerts for the window using `alerts_reset(w)`.
    - It checks if the current window flags do not already include the specified flags; if not, it adds these flags to the window's flags and logs this action.
    - The function then checks if alerts are enabled for the window with the given flags using `alerts_enabled(w, flags)`.
    - If alerts are enabled and the window's alerts are not already queued, it sets `w->alerts_queued` to 1, adds the window to the `alerts_list`, and increments the window's reference count with `window_add_ref(w, __func__)`.
    - If no alerts have been fired yet (`alerts_fired` is false), it logs that an alerts check is queued, schedules an event with `event_once` to trigger `alerts_callback`, and sets `alerts_fired` to 1.
- **Output**:
    - The function does not return a value; it modifies the state of the window and the global alerts system.
- **Functions called**:
    - [`alerts_reset`](#alerts_reset)
    - [`alerts_enabled`](#alerts_enabled)
    - [`window_add_ref`](window.c.driver.md#window_add_ref)


---
### alerts_check_bell<!-- {{#callable:alerts_check_bell}} -->
The `alerts_check_bell` function checks for bell alerts in a window and notifies sessions and winlinks accordingly.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window to check for bell alerts.
- **Control Flow**:
    - Check if the window's flags include `WINDOW_BELL`; if not, return 0.
    - Check if the 'monitor-bell' option is enabled for the window; if not, return 0.
    - Iterate over each winlink in the window's winlinks list and clear the `SESSION_ALERTED` flag for each session.
    - Iterate over each winlink again to process bell alerts:
    - For each winlink, if the session's current winlink is not the current one or the session is not attached, set the `WINLINK_BELL` flag and update the session status.
    - Check if the bell action applies to the winlink using [`alerts_action_applies`](#alerts_action_applies); if not, continue to the next winlink.
    - Notify the winlink of the bell alert using [`notify_winlink`](notify.c.driver.md#notify_winlink).
    - If the session is already alerted, continue to the next winlink.
    - Set the `SESSION_ALERTED` flag for the session and set a visual bell message using [`alerts_set_message`](#alerts_set_message).
    - Return `WINDOW_BELL` to indicate a bell alert was processed.
- **Output**:
    - Returns `WINDOW_BELL` if a bell alert was processed, otherwise returns 0.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`server_status_session`](server-fn.c.driver.md#server_status_session)
    - [`alerts_action_applies`](#alerts_action_applies)
    - [`notify_winlink`](notify.c.driver.md#notify_winlink)
    - [`alerts_set_message`](#alerts_set_message)


---
### alerts_check_activity<!-- {{#callable:alerts_check_activity}} -->
The `alerts_check_activity` function checks for activity in a window and triggers alerts if necessary.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window to check for activity.
- **Control Flow**:
    - Check if the window's flags indicate activity and if activity monitoring is enabled; return 0 if not.
    - Iterate over each winlink in the window's winlinks list, clearing the SESSION_ALERTED flag for each session.
    - Iterate over each winlink again, checking if the winlink already has the WINLINK_ACTIVITY flag set; if so, continue to the next winlink.
    - For each winlink without the WINLINK_ACTIVITY flag, check if the session's current winlink is not the current one or if the session is not attached; if true, set the WINLINK_ACTIVITY flag and update the session status.
    - Check if the activity action applies to the winlink; if not, continue to the next winlink.
    - Notify the winlink of the activity alert.
    - If the session has not been alerted yet, set the SESSION_ALERTED flag and set an activity message for the winlink.
- **Output**:
    - Returns `WINDOW_ACTIVITY` if activity is detected and alerts are triggered, otherwise returns 0.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`server_status_session`](server-fn.c.driver.md#server_status_session)
    - [`alerts_action_applies`](#alerts_action_applies)
    - [`notify_winlink`](notify.c.driver.md#notify_winlink)
    - [`alerts_set_message`](#alerts_set_message)


---
### alerts_check_silence<!-- {{#callable:alerts_check_silence}} -->
The function `alerts_check_silence` checks for silence alerts in a given window and updates the session and winlink flags accordingly.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window to check for silence alerts.
- **Control Flow**:
    - Check if the window's flags do not include `WINDOW_SILENCE`; if so, return 0 immediately.
    - Check if the 'monitor-silence' option is set to 0 for the window; if so, return 0 immediately.
    - Iterate over each winlink in the window's winlinks list and clear the `SESSION_ALERTED` flag for each associated session.
    - Iterate over each winlink again to check for silence conditions:
    - If the winlink's flags already include `WINLINK_SILENCE`, skip to the next winlink.
    - Retrieve the session associated with the current winlink.
    - If the session's current winlink is not the current winlink or the session is not attached, set the `WINLINK_SILENCE` flag for the winlink and update the session status.
    - Check if the silence action applies to the current winlink using [`alerts_action_applies`](#alerts_action_applies); if not, skip to the next winlink.
    - Notify the winlink of a silence alert using [`notify_winlink`](notify.c.driver.md#notify_winlink).
    - If the session's flags already include `SESSION_ALERTED`, skip to the next winlink.
    - Set the `SESSION_ALERTED` flag for the session.
    - Set a silence alert message for the winlink using [`alerts_set_message`](#alerts_set_message).
    - Return `WINDOW_SILENCE` to indicate that a silence alert was processed.
- **Output**:
    - The function returns `WINDOW_SILENCE` if a silence alert is processed, otherwise it returns 0.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`server_status_session`](server-fn.c.driver.md#server_status_session)
    - [`alerts_action_applies`](#alerts_action_applies)
    - [`notify_winlink`](notify.c.driver.md#notify_winlink)
    - [`alerts_set_message`](#alerts_set_message)


---
### alerts_set_message<!-- {{#callable:alerts_set_message}} -->
The `alerts_set_message` function determines and sends the appropriate alert (bell, message, or both) to clients based on the alert type and session options.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure representing the window link associated with the alert.
    - `type`: A string representing the type of alert (e.g., 'Bell', 'Activity', 'Silence').
    - `option`: A string representing the option name to check for visual alert settings (e.g., 'visual-bell', 'visual-activity', 'visual-silence').
- **Control Flow**:
    - Retrieve the visual alert setting for the given option from the session's options associated with the `winlink` structure.
    - Iterate over each client in the global `clients` list.
    - For each client, check if the client is attached to the same session as the `winlink` and is not in control mode.
    - If the visual setting is `VISUAL_OFF` or `VISUAL_BOTH`, send a bell signal to the client's terminal.
    - If the visual setting is `VISUAL_OFF`, skip sending a message and continue to the next client.
    - If the client is viewing the current window (`curw`), set a status message indicating the alert type in the current window.
    - Otherwise, set a status message indicating the alert type in the specific window identified by the `winlink` index.
- **Output**:
    - The function does not return a value; it sends alerts to clients by modifying their terminal state and setting status messages.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`tty_putcode`](tty.c.driver.md#tty_putcode)


