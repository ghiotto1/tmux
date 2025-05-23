# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it defines functionality for displaying message logs within the tmux environment. The file implements a command, "show-messages," which is used to display messages that have been logged during the operation of tmux. The command is defined with specific options and flags, allowing users to filter and format the output based on their needs. The primary function, [`cmd_show_messages_exec`](#cmd_show_messages_exec), executes the command by either displaying terminal information, job summaries, or the message log itself, depending on the arguments provided.

The code is structured around the [`cmd_show_messages_exec`](#cmd_show_messages_exec) function, which handles the logic for processing command-line arguments and determining the output. It utilizes helper functions and data structures, such as [`cmd_show_messages_terminals`](#cmd_show_messages_terminals) and `format_tree`, to format and print the message log or terminal information. The file defines a public API through the `cmd_entry` structure, which specifies the command's name, alias, arguments, usage, and execution function. This structure allows the command to be integrated into the broader tmux command framework, enabling users to interact with the message log functionality seamlessly.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### cmd_show_messages_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_show_messages_entry` is a constant structure of type `cmd_entry` that defines the command 'show-messages' with an alias 'showmsgs'. It specifies the command's arguments, usage, flags, and the function to execute when the command is called.
- **Use**: This variable is used to register and define the behavior of the 'show-messages' command within the application, allowing it to be executed with specific options and flags.


# Functions

---
### cmd_show_messages_terminals<!-- {{#callable:cmd_show_messages_terminals}} -->
The function `cmd_show_messages_terminals` iterates over terminal entries and prints their details, optionally filtering by a target client and handling blank line printing.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command context.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `blank`: An integer flag indicating whether a blank line should be printed before terminal details.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args(self)` and store them in `args`.
    - Get the target client from the command queue item using `cmdq_get_target_client(item)` and store it in `tc`.
    - Initialize a counter `n` to zero to track the number of terminals processed.
    - Iterate over each terminal in the global `tty_terms` list using `LIST_FOREACH`.
    - For each terminal, check if the '-t' argument is present and if the terminal is not the target client's terminal, skip to the next terminal.
    - If `blank` is true, print a blank line using `cmdq_print(item, "%s", "")` and set `blank` to false.
    - Print the terminal's details including its name, associated client name, and flags using `cmdq_print`.
    - Increment the terminal counter `n`.
    - Iterate over each terminal code using `tty_term_ncodes()` and print its description using `tty_term_describe`.
    - Return true if at least one terminal was processed, otherwise return false.
- **Output**:
    - The function returns a non-zero integer if at least one terminal was processed, otherwise it returns zero.


---
### cmd_show_messages_exec<!-- {{#callable:cmd_show_messages_exec}} -->
The `cmd_show_messages_exec` function displays message logs or summaries based on command arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Initialize `done` and `blank` to 0.
    - Check if the 'T' argument is present; if so, call [`cmd_show_messages_terminals`](#cmd_show_messages_terminals) to display terminal messages and set `done` to 1.
    - Check if the 'J' argument is present; if so, call `job_print_summary` to display job summaries and set `done` to 1.
    - If either 'T' or 'J' was processed, return `CMD_RETURN_NORMAL`.
    - Create a format tree from the target item.
    - Iterate over the message log in reverse order, formatting each message and printing it using `cmdq_print`.
    - Free the format tree and return `CMD_RETURN_NORMAL`.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` after processing the command.
- **Functions called**:
    - [`cmd_show_messages_terminals`](#cmd_show_messages_terminals)


# Function Declarations (Public API)

---
### cmd_show_messages_exec<!-- {{#callable_declaration:cmd_show_messages_exec}} -->
Displays the message log or specific summaries based on provided options.
- **Description**: This function is used to display the message log or specific summaries such as terminal information or job summaries, depending on the options provided. It should be called when there is a need to review messages or summaries in a structured format. The function processes options to determine what information to display, and it formats and prints the messages accordingly. It is important to ensure that the command and command queue item are properly initialized before calling this function. The function handles different options to either show terminal information, job summaries, or the entire message log.
- **Inputs**:
    - `self`: A pointer to a 'struct cmd' representing the command context. Must not be null. The caller retains ownership.
    - `item`: A pointer to a 'struct cmdq_item' representing the command queue item. Must not be null. The caller retains ownership.
- **Output**: Returns an enum cmd_retval indicating the result of the command execution, typically CMD_RETURN_NORMAL.
- **See also**: [`cmd_show_messages_exec`](#cmd_show_messages_exec)  (Implementation)


