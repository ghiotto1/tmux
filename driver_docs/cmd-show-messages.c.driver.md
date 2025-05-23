# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it defines functionality for displaying a log of messages. The file is structured around the [`cmd_show_messages_exec`](#cmd_show_messages_exec) function, which is responsible for executing the "show-messages" command within tmux. This command allows users to view a log of messages, which can include terminal information and job summaries, depending on the options provided. The code defines a command entry structure, `cmd_show_messages_entry`, which specifies the command's name, alias, arguments, usage, and execution function, indicating that this file is part of a larger command handling system within tmux.

The file includes several key components, such as the [`cmd_show_messages_exec`](#cmd_show_messages_exec) function, which processes command-line arguments to determine what information to display. It uses helper functions like [`cmd_show_messages_terminals`](#cmd_show_messages_terminals) to print terminal-related information and `job_print_summary` for job summaries. The code also utilizes a format tree to format and print message logs, demonstrating its integration with tmux's formatting and command queue systems. This file is not a standalone executable but rather a component of the tmux application, providing specific functionality related to message logging and display.
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
- **Description**: The `cmd_show_messages_entry` is a constant structure of type `cmd_entry` that defines the command 'show-messages' with an alias 'showmsgs'. It specifies the command's arguments, usage, flags, and the function to execute when the command is invoked.
- **Use**: This variable is used to register and define the behavior of the 'show-messages' command within the application, allowing it to display message logs.


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
    - Print the terminal's details including its index, name, associated client name, and flags using `cmdq_print`.
    - Increment the terminal counter `n`.
    - Iterate over each terminal code using `tty_term_ncodes()` and print its description using [`tty_term_describe`](tty-term.c.driver.md#tty_term_describe).
    - Return true if at least one terminal was processed, otherwise return false.
- **Output**:
    - The function returns an integer indicating whether any terminals were processed (non-zero if true, zero if false).
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`tty_term_ncodes`](tty-term.c.driver.md#tty_term_ncodes)
    - [`tty_term_describe`](tty-term.c.driver.md#tty_term_describe)


---
### cmd_show_messages_exec<!-- {{#callable:cmd_show_messages_exec}} -->
The `cmd_show_messages_exec` function displays a log of messages, optionally showing terminal or job summaries based on command arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Initialize `done` and `blank` to 0.
    - Check if the 'T' argument is present; if so, call [`cmd_show_messages_terminals`](#cmd_show_messages_terminals) to display terminal information and set `done` to 1.
    - Check if the 'J' argument is present; if so, call [`job_print_summary`](job.c.driver.md#job_print_summary) to display job summary and set `done` to 1.
    - If either 'T' or 'J' was processed, return `CMD_RETURN_NORMAL`.
    - Create a format tree from the target item.
    - Iterate over the message log in reverse order, adding message details to the format tree.
    - Expand the format tree into a string using `SHOW_MESSAGES_TEMPLATE` and print it.
    - Free the format tree and return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval`, specifically `CMD_RETURN_NORMAL`, indicating successful execution.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`cmd_show_messages_terminals`](#cmd_show_messages_terminals)
    - [`job_print_summary`](job.c.driver.md#job_print_summary)
    - [`format_create_from_target`](format.c.driver.md#format_create_from_target)
    - [`format_add_tv`](format.c.driver.md#format_add_tv)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)


