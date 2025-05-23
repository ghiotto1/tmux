# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for managing alert notifications within tmux sessions. The code provides functionality to monitor and respond to various alert conditions such as bell, activity, and silence in tmux windows. It defines several static functions to check for these conditions and to handle the corresponding alerts by setting flags and queuing notifications. The file includes mechanisms to determine when alerts should be triggered based on user-configurable options, and it manages a list of windows that require alert checks.

The code is structured around several key components, including functions to check for specific alert types (e.g., [`alerts_check_bell`](#alerts_check_bell), [`alerts_check_activity`](#alerts_check_activity), [`alerts_check_silence`](#alerts_check_silence)) and to handle the queuing and processing of these alerts ([`alerts_queue`](#alerts_queue), [`alerts_callback`](#alerts_callback)). It uses data structures like `TAILQ_HEAD` to maintain a list of windows that need alert processing. The file does not define a public API but rather provides internal functionality to be used within the tmux application. The alert system is designed to notify users of events in tmux windows, either through visual messages or terminal bells, depending on the user's configuration settings.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### alerts_fired
- **Type**: `int`
- **Description**: The `alerts_fired` variable is a static integer that acts as a flag to indicate whether any alerts have been triggered in the system. It is used to prevent multiple alert checks from being queued simultaneously.
- **Use**: This variable is used to track if an alert check has already been queued, ensuring that the alert system does not redundantly queue multiple checks.


---
### alerts_list
- **Type**: `TAILQ_HEAD(, window)`
- **Description**: The `alerts_list` is a global variable that is a tail queue head for managing a list of `window` structures. It is initialized using the `TAILQ_HEAD_INITIALIZER` macro, which sets up the queue to be empty initially.
- **Use**: This variable is used to maintain a list of windows that have pending alerts, allowing the program to efficiently manage and process alert events for each window.


# Functions

---
### alerts_timer<!-- {{#callable:alerts_timer}} -->
The `alerts_timer` function logs a debug message indicating that an alert timer has expired for a specific window and queues a silence alert for that window.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `arg`: A pointer to a `struct window`, representing the window for which the alert timer has expired.
- **Control Flow**:
    - The function casts the `arg` parameter to a `struct window` pointer and assigns it to the variable `w`.
    - A debug message is logged indicating that the alert timer for the window with ID `w->id` has expired.
    - The function calls [`alerts_queue`](#alerts_queue) with the window `w` and the `WINDOW_SILENCE` flag to queue a silence alert for the window.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`alerts_queue`](#alerts_queue)


---
### alerts_callback<!-- {{#callable:alerts_callback}} -->
The `alerts_callback` function processes and clears alert flags for each window in the alerts list, resetting their alert status and removing them from the list.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `arg`: An unused void pointer for additional arguments.
- **Control Flow**:
    - Iterates over each window in the `alerts_list` using `TAILQ_FOREACH_SAFE` to safely traverse and modify the list.
    - For each window, calls [`alerts_check_all`](#alerts_check_all) to determine the current alert status and logs the result.
    - Resets the `alerts_queued` flag for the window to 0, indicating that alerts are no longer queued.
    - Removes the window from the `alerts_list` using `TAILQ_REMOVE`.
    - Clears the alert flags from the window by unsetting `WINDOW_ALERTFLAGS`.
    - Calls `window_remove_ref` to decrease the reference count of the window, potentially allowing it to be freed if no longer in use.
    - Sets the global `alerts_fired` flag to 0, indicating that no alerts are currently active.
- **Output**:
    - The function does not return any value; it operates by modifying the state of windows in the `alerts_list` and global variables.
- **Functions called**:
    - [`alerts_check_all`](#alerts_check_all)


---
### alerts_action_applies<!-- {{#callable:alerts_action_applies}} -->
The `alerts_action_applies` function determines if an alert action should be applied to a given window link based on the specified alert type and the current window state.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure representing a window link in a session.
    - `name`: A string representing the name of the alert action to check, such as 'bell-action', 'activity-action', or 'silence-action'.
- **Control Flow**:
    - Retrieve the alert action setting for the given name from the session's options associated with the window link `wl`.
    - Check if the action is `ALERT_ANY`, and if so, return 1 indicating the alert applies to any window.
    - Check if the action is `ALERT_CURRENT`, and if so, return 1 if the window link `wl` is the current window in the session, otherwise return 0.
    - Check if the action is `ALERT_OTHER`, and if so, return 1 if the window link `wl` is not the current window in the session, otherwise return 0.
    - If none of the conditions are met, return 0 indicating the alert does not apply.
- **Output**:
    - Returns an integer: 1 if the alert action applies to the window link based on the specified conditions, otherwise 0.


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
The function `alerts_check_session` iterates over all windows in a given session and checks for any alerts in each window.
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
    - Check if the `flags` include `WINDOW_BELL`; if true, check if the 'monitor-bell' option is enabled for the window, and return 1 if it is.
    - Check if the `flags` include `WINDOW_ACTIVITY`; if true, check if the 'monitor-activity' option is enabled for the window, and return 1 if it is.
    - Check if the `flags` include `WINDOW_SILENCE`; if true, check if the 'monitor-silence' option is enabled for the window, and return 1 if it is.
    - If none of the conditions are met, return 0.
- **Output**:
    - Returns 1 if any of the specified alert conditions are enabled for the window, otherwise returns 0.


---
### alerts_reset_all<!-- {{#callable:alerts_reset_all}} -->
The `alerts_reset_all` function iterates over all windows and resets their alert states.
- **Inputs**:
    - None
- **Control Flow**:
    - The function uses the `RB_FOREACH` macro to iterate over all windows in the `windows` data structure.
    - For each window, it calls the [`alerts_reset`](#alerts_reset) function to reset the alert state of that window.
- **Output**:
    - The function does not return any value; it performs its operations directly on the window data structures.
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
    - Remove the alert timer event using `event_del`.
    - Clear the `timeval` structure `tv` and set its seconds value to the number obtained from the 'monitor-silence' option of the window.
    - Log the reset action with the window ID and the silence monitor time.
    - If the silence monitor time is not zero, add the alert timer event back with the specified time value.
- **Output**:
    - The function does not return a value; it modifies the state of the window's alert timer and flags.


---
### alerts_queue<!-- {{#callable:alerts_queue}} -->
The `alerts_queue` function manages the queuing of alert events for a given window based on specified flags and ensures that alerts are processed if they are enabled.
- **Inputs**:
    - `w`: A pointer to a `window` structure representing the window for which alerts are being queued.
    - `flags`: An integer representing the alert flags that specify the types of alerts to be queued for the window.
- **Control Flow**:
    - The function begins by resetting the alerts for the given window using `alerts_reset(w)`.
    - It checks if the window's current flags do not already include the specified flags; if not, it adds these flags to the window's flags and logs this action.
    - The function then checks if alerts are enabled for the window with the specified flags using `alerts_enabled(w, flags)`.
    - If alerts are enabled and the window's alerts are not already queued, it sets the `alerts_queued` flag, adds the window to the `alerts_list`, and increments the window's reference count.
    - If no alerts have been fired yet (`alerts_fired` is false), it logs that an alerts check has been queued and schedules the `alerts_callback` function to be called once, setting `alerts_fired` to true.
- **Output**:
    - The function does not return a value; it modifies the state of the window and the global alerts system by queuing alerts and scheduling their processing.
- **Functions called**:
    - [`alerts_reset`](#alerts_reset)
    - [`alerts_enabled`](#alerts_enabled)


---
### alerts_check_bell<!-- {{#callable:alerts_check_bell}} -->
The `alerts_check_bell` function checks for bell alerts in a given window and notifies sessions accordingly.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window to check for bell alerts.
- **Control Flow**:
    - Check if the window's flags do not include `WINDOW_BELL`; if so, return 0 immediately.
    - Check if the 'monitor-bell' option is enabled for the window; if not, return 0.
    - Iterate over each winlink in the window's winlinks list and clear the `SESSION_ALERTED` flag for each session.
    - Iterate over each winlink again to process bell alerts:
    - For each winlink, if the session's current window is not the winlink or the session is not attached, set the `WINLINK_BELL` flag and update the session status.
    - Check if the bell action applies to the winlink using [`alerts_action_applies`](#alerts_action_applies); if not, continue to the next winlink.
    - Notify the winlink of a bell alert using `notify_winlink`.
    - If the session's `SESSION_ALERTED` flag is already set, continue to the next winlink.
    - Set the session's `SESSION_ALERTED` flag and set a bell message using [`alerts_set_message`](#alerts_set_message).
    - Return `WINDOW_BELL` to indicate a bell alert was processed.
- **Output**:
    - Returns `WINDOW_BELL` if a bell alert was processed, otherwise returns 0.
- **Functions called**:
    - [`alerts_action_applies`](#alerts_action_applies)
    - [`alerts_set_message`](#alerts_set_message)


---
### alerts_check_activity<!-- {{#callable:alerts_check_activity}} -->
The function `alerts_check_activity` checks for activity in a window and triggers alerts if necessary.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window to check for activity.
- **Control Flow**:
    - Check if the window's flags indicate activity (`WINDOW_ACTIVITY`) and if the 'monitor-activity' option is enabled; return 0 if either condition is false.
    - Iterate over each winlink in the window's winlinks list, clearing the `SESSION_ALERTED` flag for each associated session.
    - For each winlink, check if it already has the `WINLINK_ACTIVITY` flag set; if so, skip further processing for that winlink.
    - For each winlink, if the session's current window is not the winlink or the session is not attached, set the `WINLINK_ACTIVITY` flag and update the session status.
    - Check if the activity action applies to the winlink using [`alerts_action_applies`](#alerts_action_applies); if not, skip further processing for that winlink.
    - Notify the winlink of activity using `notify_winlink`.
    - If the session has not been alerted (`SESSION_ALERTED` flag is not set), set the `SESSION_ALERTED` flag and set an activity message using [`alerts_set_message`](#alerts_set_message).
- **Output**:
    - Returns `WINDOW_ACTIVITY` if activity is detected and processed, otherwise returns 0.
- **Functions called**:
    - [`alerts_action_applies`](#alerts_action_applies)
    - [`alerts_set_message`](#alerts_set_message)


---
### alerts_check_silence<!-- {{#callable:alerts_check_silence}} -->
The function `alerts_check_silence` checks for silence alerts in a given window and updates the session and winlink flags accordingly.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window to check for silence alerts.
- **Control Flow**:
    - Check if the window's flags include `WINDOW_SILENCE`; if not, return 0.
    - Check if the 'monitor-silence' option is enabled; if not, return 0.
    - Iterate over each winlink in the window's winlinks list and clear the `SESSION_ALERTED` flag for each session.
    - Iterate over each winlink again to check for silence conditions.
    - If a winlink already has the `WINLINK_SILENCE` flag, skip it.
    - For each winlink, check if it is not the current window in its session or if the session is not attached; if so, set the `WINLINK_SILENCE` flag and update the session status.
    - Check if the silence action applies to the winlink using [`alerts_action_applies`](#alerts_action_applies); if not, skip it.
    - Notify the winlink of a silence alert using `notify_winlink`.
    - If the session has already been alerted, skip further processing.
    - Set the `SESSION_ALERTED` flag for the session.
    - Set a silence alert message for the winlink using [`alerts_set_message`](#alerts_set_message).
- **Output**:
    - Returns `WINDOW_SILENCE` if the silence alert conditions are met and processed; otherwise, returns 0.
- **Functions called**:
    - [`alerts_action_applies`](#alerts_action_applies)
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
    - For each client, check if it is attached to the same session as the `winlink` and is not in control mode.
    - If the visual setting is `VISUAL_OFF` or `VISUAL_BOTH`, send a bell signal to the client's terminal.
    - If the visual setting is `VISUAL_OFF`, skip sending a message and continue to the next client.
    - If the client is viewing the current window (`curw`), set a status message indicating the alert type in the current window.
    - Otherwise, set a status message indicating the alert type in the specific window identified by the `winlink` index.
- **Output**:
    - The function does not return a value; it sends alerts to clients by modifying their terminal state and setting status messages.


# Function Declarations (Public API)

---
### alerts_timer<!-- {{#callable_declaration:alerts_timer}} -->
Handles the expiration of an alert timer for a window.
- **Description**: This function is used to handle the expiration of an alert timer associated with a window. It should be called when the timer event for a window expires, indicating that a period of silence has been detected. The function logs the expiration event and queues a silence alert for the specified window. It is typically used in the context of event-driven programming where timers are set to monitor window activity.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored. It is typically used for file descriptors in event-driven functions but is not applicable here.
    - `events`: This parameter is unused and can be ignored. It is typically used to specify event types in event-driven functions but is not applicable here.
    - `arg`: A pointer to a 'struct window' object. This parameter must not be null and should point to the window for which the alert timer has expired. The caller retains ownership of the window object.
- **Output**: None
- **See also**: [`alerts_timer`](#alerts_timer)  (Implementation)


---
### alerts_enabled<!-- {{#callable_declaration:alerts_enabled}} -->
Determine if alerts are enabled for a window based on specified flags.
- **Description**: Use this function to check if any alerts are enabled for a given window based on the specified alert flags. It evaluates the window's options to determine if alerts for bell, activity, or silence are enabled. This function is useful when you need to decide whether to trigger alert-related actions for a window. Ensure that the window structure is properly initialized and that the flags parameter contains valid alert flags before calling this function.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window to check. Must not be null, and the window should be properly initialized with valid options.
    - `flags`: An integer representing the alert flags to check. Valid flags include WINDOW_BELL, WINDOW_ACTIVITY, and WINDOW_SILENCE. The function checks these flags to determine which alerts to evaluate.
- **Output**: Returns 1 if any of the specified alerts are enabled for the window, otherwise returns 0.
- **See also**: [`alerts_enabled`](#alerts_enabled)  (Implementation)


---
### alerts_callback<!-- {{#callable_declaration:alerts_callback}} -->
Handles alert callbacks for windows.
- **Description**: This function processes alert callbacks for windows that are queued in the alerts list. It checks for any alerts on each window, logs the alert status, and then removes the window from the alerts list. This function is typically used in an event-driven system where alerts need to be processed asynchronously. It resets the alert flags and ensures that the alerts are not fired again until re-queued.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored.
    - `events`: This parameter is unused and can be ignored.
    - `arg`: This parameter is unused and can be ignored.
- **Output**: None
- **See also**: [`alerts_callback`](#alerts_callback)  (Implementation)


---
### alerts_reset<!-- {{#callable_declaration:alerts_reset}} -->
Resets the alert timer for a window.
- **Description**: This function is used to reset the alert timer associated with a specific window, ensuring that the window's silence alert is properly managed. It should be called when the alert conditions for a window need to be re-evaluated, such as after a change in the window's state or configuration. The function clears any existing silence flags and removes the current alert timer event. If the silence monitoring option is enabled and set to a non-zero value, a new timer event is scheduled. This function must be called with a valid window structure that has been properly initialized.
- **Inputs**:
    - `w`: A pointer to a 'struct window' that represents the window for which the alert timer is being reset. This parameter must not be null and should point to a valid, initialized window structure. The function assumes ownership of managing the alert timer for this window.
- **Output**: None
- **See also**: [`alerts_reset`](#alerts_reset)  (Implementation)


---
### alerts_action_applies<!-- {{#callable_declaration:alerts_action_applies}} -->
Determine if an alert action applies to a given window link.
- **Description**: This function checks whether a specified alert action should be applied to a given window link within a session. It is used to decide if an alert should be triggered based on the current window context and the alert action settings. The function evaluates the alert action setting for the window link and returns a boolean indicating if the alert condition is met. It is typically called when processing alert conditions such as bell, activity, or silence in a window session.
- **Inputs**:
    - `wl`: A pointer to a 'struct winlink' representing the window link to check. Must not be null, and should be part of a valid session.
    - `name`: A string representing the name of the alert action to check (e.g., 'bell-action', 'activity-action', 'silence-action'). Must not be null.
- **Output**: Returns an integer: 1 if the alert action applies, 0 otherwise.
- **See also**: [`alerts_action_applies`](#alerts_action_applies)  (Implementation)


---
### alerts_check_all<!-- {{#callable_declaration:alerts_check_all}} -->
Checks for any alerts in the specified window.
- **Description**: This function evaluates the specified window for any active alerts, including bell, activity, and silence alerts. It should be used when you need to determine if any of these alert conditions are present in a window. The function returns a bitmask indicating which alerts are active. It is important to ensure that the window is properly initialized and valid before calling this function to avoid undefined behavior.
- **Inputs**:
    - `w`: A pointer to a 'struct window' that represents the window to be checked for alerts. Must not be null, and the window should be properly initialized and valid.
- **Output**: An integer bitmask representing the active alerts in the window. Each bit corresponds to a different type of alert (e.g., bell, activity, silence).
- **See also**: [`alerts_check_all`](#alerts_check_all)  (Implementation)


---
### alerts_check_bell<!-- {{#callable_declaration:alerts_check_bell}} -->
Check for bell alerts in a window and notify sessions accordingly.
- **Description**: This function checks if a bell alert is active for a given window and processes it according to the session and window link configurations. It should be used when you need to handle bell alerts in a window, ensuring that the appropriate sessions are notified and any necessary visual or auditory alerts are triggered. The function assumes that the window has been properly initialized and that the 'monitor-bell' option is set for the window. It modifies session and window link flags to reflect the alert status.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window to check for bell alerts. Must not be null. The window should have its flags and options properly set before calling this function.
- **Output**: Returns an integer indicating the presence of a bell alert, specifically the constant 'WINDOW_BELL', if a bell alert is processed.
- **See also**: [`alerts_check_bell`](#alerts_check_bell)  (Implementation)


---
### alerts_check_activity<!-- {{#callable_declaration:alerts_check_activity}} -->
Checks for activity alerts in a window.
- **Description**: This function evaluates whether there is any activity in the specified window that should trigger an alert. It should be called when there is a need to monitor window activity, particularly when the 'monitor-activity' option is enabled for the window. The function will iterate through all winlinks associated with the window, updating session flags and notifying relevant components if activity is detected. It is important to ensure that the window's flags and options are correctly set before calling this function to avoid unnecessary operations.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window to be checked for activity. Must not be null. The window should have its flags and options properly configured, particularly the 'WINDOW_ACTIVITY' flag and 'monitor-activity' option.
- **Output**: Returns 'WINDOW_ACTIVITY' if activity is detected and processed; otherwise, returns 0 if no activity is detected or if monitoring is disabled.
- **See also**: [`alerts_check_activity`](#alerts_check_activity)  (Implementation)


---
### alerts_check_silence<!-- {{#callable_declaration:alerts_check_silence}} -->
Checks for silence alerts in a window.
- **Description**: Use this function to determine if a window should trigger a silence alert based on its current state and configuration. It should be called when you need to check if a window has been silent for a period defined by its options. The function will update session and winlink flags accordingly and notify if a silence alert is applicable. Ensure that the window's silence monitoring is enabled before calling this function.
- **Inputs**:
    - `w`: A pointer to a 'struct window' that represents the window to be checked for silence alerts. Must not be null. The window should have its options configured for silence monitoring.
- **Output**: Returns an integer indicating the presence of a silence alert, specifically returning 'WINDOW_SILENCE' if a silence alert is triggered, or 0 if not.
- **See also**: [`alerts_check_silence`](#alerts_check_silence)  (Implementation)


---
### alerts_set_message<!-- {{#callable_declaration:alerts_set_message}} -->
Send alerts to clients based on session and window link conditions.
- **Description**: This function is used to notify clients of specific alert conditions such as bell, activity, or silence within a session. It determines whether to send a bell, a message, or both to each client attached to the session associated with the given window link. The function checks the session's options to decide the type of alert to send. It should be called when an alert condition is detected, and it handles the notification for all clients in the session, ensuring that the appropriate alert is sent based on the current window and session settings.
- **Inputs**:
    - `wl`: A pointer to a 'winlink' structure representing the window link associated with the session. Must not be null.
    - `type`: A string representing the type of alert (e.g., 'Bell', 'Activity', 'Silence'). Must not be null.
    - `option`: A string representing the option name to check for visual alert settings (e.g., 'visual-bell'). Must not be null.
- **Output**: None
- **See also**: [`alerts_set_message`](#alerts_set_message)  (Implementation)


