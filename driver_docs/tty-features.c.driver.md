# Purpose
This C source code file is part of a terminal multiplexer, likely related to the `tmux` project, which is used to manage terminal sessions. The file defines a set of terminal features and capabilities, encapsulated in the `tty_feature` structure, which includes a name, a list of capabilities, and associated flags. These features cover a wide range of terminal functionalities, such as support for RGB colors, mouse interactions, clipboard operations, and various text styles like strikethrough and overline. The code is designed to identify, manage, and apply these features to terminal sessions, allowing for enhanced terminal interactions and customizations based on the terminal's capabilities.

The file includes functions to add, retrieve, and apply terminal features, as well as to set default features based on the terminal type and version. The [`tty_add_features`](#tty_add_features) function parses a string of feature names and updates a feature set accordingly, while [`tty_get_features`](#tty_get_features) returns a string representation of the enabled features. The [`tty_apply_features`](#tty_apply_features) function applies the specified features to a terminal session, updating its capabilities. Additionally, the [`tty_default_features`](#tty_default_features) function sets default features for known terminal types, ensuring compatibility and optimal functionality. This code is integral to ensuring that the terminal multiplexer can adapt to different terminal environments and provide a consistent user experience.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `curses.h`
- `ncurses.h`
- `tmux.h`


# Global Variables

---
### tty_feature_title_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_title_capabilities` is a static constant array of string pointers that defines terminal capabilities related to setting the xterm(1) title. It includes escape sequences for starting and finishing the title setting operation.
- **Use**: This variable is used to specify the capabilities required for a terminal to support xterm(1) title setting, and it is referenced in the `tty_feature_title` structure.


---
### tty_feature_title
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_title` is a static constant instance of the `struct tty_feature` that represents a terminal feature related to setting the xterm(1) title. It contains a name, a list of capabilities, and flags. The capabilities include escape sequences for setting and finalizing the title.
- **Use**: This variable is used to define and manage the terminal's ability to set the xterm(1) title through specific capabilities.


---
### tty_feature_osc7_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_osc7_capabilities` is a static constant array of string pointers, which defines terminal capabilities related to the OSC 7 (Operating System Command 7) feature. This feature is used to set the working directory in terminal emulators that support it.
- **Use**: This variable is used to store the escape sequences that enable the OSC 7 feature in terminals, allowing the terminal to recognize and apply these capabilities.


---
### tty_feature_osc7
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_osc7` is a constant instance of the `struct tty_feature` that represents a terminal feature related to the OSC 7 working directory capability. It contains a name, a list of capabilities, and flags associated with this feature.
- **Use**: This variable is used to define and manage the OSC 7 working directory feature for terminals, allowing the terminal to recognize and apply this capability.


---
### tty_feature_mouse_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_mouse_capabilities` is a static constant array of strings that defines the capabilities related to mouse support in a terminal. It contains escape sequences that are used to enable mouse functionality, specifically the sequence for mouse events ("kmous=\\E[M").
- **Use**: This variable is used to specify the mouse capabilities of a terminal, which can be applied to a terminal feature set to enable mouse support.


---
### tty_feature_mouse
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_mouse` is a constant instance of the `struct tty_feature` that represents the mouse support feature in a terminal. It contains a name, a list of capabilities, and flags associated with the mouse feature. The capabilities array includes escape sequences that enable mouse functionality in terminal applications.
- **Use**: This variable is used to define and manage the mouse support feature in terminal applications, allowing them to recognize and handle mouse input.


---
### tty_feature_clipboard_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_clipboard_capabilities` is a static constant array of strings that defines the capabilities related to setting the clipboard in a terminal using the OSC 52 escape sequence. It contains a single string that specifies the escape sequence format for setting the clipboard content, followed by a NULL terminator to indicate the end of the array.
- **Use**: This variable is used to specify the terminal capabilities for clipboard operations, allowing the terminal to set clipboard content using the defined escape sequence.


---
### tty_feature_clipboard
- **Type**: ``struct tty_feature``
- **Description**: The `tty_feature_clipboard` is a static constant instance of the `struct tty_feature` that represents the clipboard feature for a terminal. It includes a name, a set of capabilities, and flags, specifically designed to handle clipboard operations using the OSC 52 escape sequence.
- **Use**: This variable is used to define and manage the clipboard capabilities of a terminal, allowing it to set the clipboard content through specific escape sequences.


---
### tty_feature_hyperlinks_capabilities
- **Type**: `const char *`
- **Description**: The `tty_feature_hyperlinks_capabilities` is an array of strings that defines the capabilities related to OSC 8 hyperlinks for terminals. It includes a specific escape sequence for enabling hyperlinks, but this is conditionally compiled based on the operating system and ncurses version. The array is terminated with a NULL pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to specify the terminal capabilities for supporting OSC 8 hyperlinks, which are applied when the terminal supports this feature.


---
### tty_feature_hyperlinks
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_hyperlinks` is a static constant instance of the `struct tty_feature` that represents the terminal's capability to support OSC 8 hyperlinks. It contains a name, a list of capabilities, and flags, which are all initialized to specific values.
- **Use**: This variable is used to define and manage the terminal's hyperlink capabilities, allowing the terminal to interpret and handle hyperlink sequences.


---
### tty_feature_rgb_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_rgb_capabilities` is a static constant array of strings that defines the capabilities related to RGB color support in a terminal. It includes escape sequences for setting foreground and background RGB colors, as well as fallbacks for setting colors from the 256-color palette. The array is terminated with a NULL pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to specify the RGB color capabilities of a terminal, allowing for advanced color settings in terminal applications.


---
### tty_feature_rgb
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_rgb` is a static constant instance of the `struct tty_feature` that represents a terminal feature supporting RGB color capabilities. It includes a name, a list of capabilities, and flags indicating the feature's properties. The capabilities include sequences for setting RGB foreground and background colors, as well as compatibility with 256 color palettes.
- **Use**: This variable is used to define and manage the RGB color capabilities of a terminal, allowing for enhanced color support in terminal applications.


---
### tty_feature_256_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_256_capabilities` is a static constant array of strings that defines terminal capabilities related to 256 color support. It includes escape sequences for setting background (`setab`) and foreground (`setaf`) colors using the 256 color palette, as well as a capability identifier `AX`. The array is terminated with a `NULL` pointer to indicate the end of the list.
- **Use**: This variable is used to specify the capabilities of terminals that support 256 colors, allowing the software to apply these capabilities when interacting with such terminals.


---
### tty_feature_256
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_256` is a constant structure of type `tty_feature` that represents a terminal feature supporting 256 colors. It includes a name, a list of capabilities, and a set of flags associated with this feature. The capabilities array contains terminal control sequences for setting background and foreground colors using the 256-color palette.
- **Use**: This variable is used to define and manage terminal features related to 256-color support, allowing the terminal to apply these capabilities when needed.


---
### tty_feature_overline_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_overline_capabilities` is a static constant array of strings that defines terminal capabilities related to the overline feature. It contains escape sequences that enable the overline text attribute in terminals that support it.
- **Use**: This variable is used to specify the escape sequence for enabling the overline feature in terminals.


---
### tty_feature_overline
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_overline` is a constant structure of type `tty_feature` that represents a terminal feature supporting overline text formatting. It contains a name, a list of capabilities, and flags associated with the feature. The capabilities array includes the escape sequence for enabling overline mode in a terminal.
- **Use**: This variable is used to define and manage the overline capability of a terminal within the terminal feature management system.


---
### tty_feature_usstyle_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_usstyle_capabilities` is a static constant array of strings that defines terminal capabilities related to underscore styles. Each string in the array represents a specific terminal escape sequence for setting or modifying underscore styles, such as setting underline color or style. The array is terminated with a NULL pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to store and provide terminal escape sequences for underscore style capabilities in terminal feature configurations.


---
### tty_feature_usstyle
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_usstyle` is a constant structure of type `tty_feature` that represents terminal support for underscore styles. It includes a name, a list of capabilities related to underscore styles, and a flags field set to 0.
- **Use**: This variable is used to define and store the capabilities of a terminal that supports various underscore styles, which can be applied to terminal sessions.


---
### tty_feature_bpaste_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_bpaste_capabilities` is a static constant array of string pointers that defines the terminal capabilities related to bracketed paste mode. It contains escape sequences to enable and disable bracketed paste mode, which are used to control how pasted text is handled by the terminal.
- **Use**: This variable is used to store the escape sequences for enabling and disabling bracketed paste mode in terminals.


---
### tty_feature_bpaste
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_bpaste` is a constant instance of the `struct tty_feature` that represents the terminal's support for bracketed paste mode. This feature allows terminals to distinguish between pasted text and typed text, which can be useful for applications that need to handle input differently based on its source.
- **Use**: This variable is used to define and store the capabilities related to bracketed paste mode for terminals.


---
### tty_feature_focus_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_focus_capabilities` is a static constant array of strings that defines terminal capabilities related to focus reporting. It contains escape sequences for enabling and disabling focus reporting in a terminal, specifically using the sequences `\E[?1004h` to enable and `\E[?1004l` to disable focus reporting.
- **Use**: This variable is used to store the escape sequences that enable or disable focus reporting capabilities in a terminal.


---
### tty_feature_focus
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_focus` is a static constant instance of the `struct tty_feature` that represents a terminal feature related to focus reporting. It contains a name, a list of capabilities, and flags. The capabilities include enabling and disabling focus reporting sequences for terminals.
- **Use**: This variable is used to define and manage the focus reporting feature in terminal applications, allowing the terminal to notify when it gains or loses focus.


---
### tty_feature_cstyle_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_cstyle_capabilities` is a static constant array of strings that defines terminal capabilities related to cursor styles. It includes escape sequences for setting and resetting cursor styles, specifically for enabling and disabling cursor styles using terminal control sequences.
- **Use**: This variable is used to store terminal escape sequences that define how cursor styles can be manipulated in a terminal environment.


---
### tty_feature_cstyle
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_cstyle` is a constant instance of the `tty_feature` structure, representing a terminal feature related to cursor styles. It contains a name, a list of capabilities, and flags. The capabilities include escape sequences for setting and ending cursor styles.
- **Use**: This variable is used to define and manage terminal capabilities related to cursor styles within the terminal feature management system.


---
### tty_feature_ccolour_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_ccolour_capabilities` is a static constant array of string literals that define terminal capabilities related to cursor color settings. It includes escape sequences for setting and resetting cursor colors in a terminal that supports these features.
- **Use**: This array is used to specify the capabilities of terminals that support cursor color customization, and it is part of the `tty_feature_ccolour` structure.


---
### tty_feature_ccolour
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_ccolour` is a static constant instance of the `struct tty_feature` that represents a terminal feature related to cursor colors. It contains a name, a list of capabilities, and flags. The capabilities include escape sequences for setting and resetting cursor colors.
- **Use**: This variable is used to define and manage the cursor color capabilities of a terminal within the tmux application.


---
### tty_feature_strikethrough_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_strikethrough_capabilities` is an array of constant character pointers that defines terminal capabilities related to the strikethrough feature. It contains escape sequences that enable strikethrough text formatting in terminals that support this feature.
- **Use**: This variable is used to store the escape sequence for enabling strikethrough text formatting in compatible terminal emulators.


---
### tty_feature_strikethrough
- **Type**: ``struct tty_feature``
- **Description**: The `tty_feature_strikethrough` is a static constant instance of the `struct tty_feature` that represents the strikethrough capability of a terminal. It contains a name, a list of capabilities, and flags. The capabilities array includes the escape sequence for enabling strikethrough text in the terminal.
- **Use**: This variable is used to define and manage the strikethrough text feature in terminal applications that support this capability.


---
### tty_feature_sync_capabilities
- **Type**: `array of strings`
- **Description**: The `tty_feature_sync_capabilities` is a static constant array of strings that defines terminal capabilities related to synchronized updates. It contains escape sequences that are used to enable or disable synchronized updates in terminal emulators.
- **Use**: This variable is used to store the escape sequences for enabling synchronized updates in terminal emulators.


---
### tty_feature_sync
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_sync` is a static constant instance of the `struct tty_feature` that represents a terminal feature supporting synchronized updates. It is initialized with the name "sync", a set of capabilities defined in `tty_feature_sync_capabilities`, and a flag value of 0.
- **Use**: This variable is used to define and manage the synchronized updates feature for terminals, allowing it to be recognized and applied when configuring terminal capabilities.


---
### tty_feature_extkeys_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_extkeys_capabilities` is a static constant array of string literals that define terminal capabilities related to extended keys. Each string in the array represents a specific capability, encoded in a format that can be interpreted by terminal emulators. The array is terminated with a `NULL` pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to store and provide the extended keys capabilities for terminals, which can be applied to terminal configurations to enable or modify extended key functionalities.


---
### tty_feature_extkeys
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_extkeys` is a static constant instance of the `struct tty_feature` that represents a terminal feature supporting extended keys. It is initialized with the name "extkeys", a set of capabilities defined in `tty_feature_extkeys_capabilities`, and a flag value of 0.
- **Use**: This variable is used to define and manage the extended keys feature for terminals, allowing the system to recognize and apply this capability when interacting with terminal features.


---
### tty_feature_margins_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_margins_capabilities` is a static constant array of strings that defines terminal capabilities related to DECSLRM margins. Each string in the array represents a specific terminal escape sequence for enabling, disabling, clearing, or setting margins. The array is terminated with a NULL pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to store the escape sequences for terminal margin capabilities, which are then referenced by the `tty_feature_margins` structure to apply these capabilities to a terminal.


---
### tty_feature_margins
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_margins` is a constant structure of type `tty_feature` that represents terminal support for DECSLRM margins. It contains a name, a list of capabilities, and a flag indicating the feature's properties. The capabilities include enabling and disabling margins, clearing margins, and setting margins with specific parameters.
- **Use**: This variable is used to define and manage terminal capabilities related to DECSLRM margins, allowing the terminal to handle margin settings.


---
### tty_feature_rectfill_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_rectfill_capabilities` is a static constant array of string pointers that defines the capabilities related to the DECFRA rectangle fill feature for a terminal. It contains a single string "Rect" followed by a NULL terminator, indicating the end of the array.
- **Use**: This variable is used to specify the capabilities of a terminal that supports the DECFRA rectangle fill feature.


---
### tty_feature_rectfill
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_rectfill` is a constant instance of the `struct tty_feature` that represents a terminal feature supporting DECFRA rectangle fill. It contains a name, a list of capabilities, and a flag indicating the feature's properties.
- **Use**: This variable is used to define and manage the DECFRA rectangle fill capability in terminal applications.


---
### tty_feature_ignorefkeys_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_ignorefkeys_capabilities` is a static constant array of string literals, each representing a terminal capability related to function keys. The array contains entries like "kf0@", "kf1@", up to "kf63@", and is terminated with a `NULL` pointer.
- **Use**: This array is used to define terminal capabilities that ignore function keys, which is part of the `tty_feature_ignorefkeys` structure.


---
### tty_feature_ignorefkeys
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_ignorefkeys` is a static constant instance of the `struct tty_feature` that represents a terminal feature where function keys are ignored. It is initialized with the name "ignorefkeys", a list of capabilities that denote the function keys to be ignored, and a flag value of 0.
- **Use**: This variable is used to define a terminal feature that disables the use of function keys, likely to ensure compatibility or to enforce a specific terminal behavior.


---
### tty_feature_sixel_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_sixel_capabilities` is a static constant array of string pointers that defines the capabilities related to the sixel feature of a terminal. It contains a single string "Sxl" followed by a NULL terminator, indicating the presence of sixel capability in the terminal.
- **Use**: This variable is used to specify the sixel capability in the `tty_feature_sixel` structure, which is part of the terminal feature set.


---
### tty_feature_sixel
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_sixel` is a constant instance of the `struct tty_feature` that represents a terminal feature supporting sixel graphics. It includes a name, a set of capabilities, and associated flags. The capabilities array for this feature contains the string "Sxl", indicating the terminal's ability to handle sixel graphics.
- **Use**: This variable is used to define and identify the sixel graphics capability of a terminal within the terminal features management system.


---
### tty_features
- **Type**: `const struct tty_feature *const[]`
- **Description**: The `tty_features` variable is a static constant array of pointers to `tty_feature` structures. Each element in the array represents a specific terminal feature, such as support for 256 colors, mouse support, or clipboard capabilities. These features are defined by their name, associated capabilities, and flags.
- **Use**: This variable is used to store and manage a list of terminal features that can be applied to a terminal session.


# Data Structures

---
### tty_feature
- **Type**: `struct`
- **Members**:
    - `name`: A constant character pointer representing the name of the terminal feature.
    - `capabilities`: A constant pointer to an array of constant character pointers, representing the capabilities associated with the terminal feature.
    - `flags`: An integer representing flags associated with the terminal feature.
- **Description**: The `tty_feature` structure is used to define a named terminal feature, including its associated capabilities and any relevant flags. Each feature has a name, a list of capabilities expressed as terminal control sequences, and flags that may indicate specific properties or behaviors of the feature. This structure is part of a system to manage and apply terminal features in a terminal multiplexer environment, allowing for the customization and extension of terminal capabilities.


# Functions

---
### tty_add_features<!-- {{#callable:tty_add_features}} -->
The `tty_add_features` function adds specified terminal features to a feature set based on a string of feature names separated by given delimiters.
- **Inputs**:
    - `feat`: A pointer to an integer representing the current set of terminal features, where each bit corresponds to a specific feature.
    - `s`: A string containing the names of terminal features to be added, separated by characters specified in the 'separators' argument.
    - `separators`: A string of characters used to delimit the feature names in the 's' string.
- **Control Flow**:
    - The function logs the start of adding terminal features with the provided string 's'.
    - It duplicates the string 's' into 'copy' and uses 'loop' to iterate over it.
    - The function uses 'strsep' to split 'loop' into individual feature names based on the 'separators'.
    - For each feature name, it iterates over the 'tty_features' array to find a matching feature by name.
    - If a matching feature is found, it checks if the feature is not already set in 'feat'.
    - If the feature is not set, it logs the addition of the feature and updates 'feat' to include the new feature.
    - If no matching feature is found, it logs an unknown feature and breaks out of the loop.
    - Finally, it frees the memory allocated for 'copy'.
- **Output**:
    - The function modifies the integer pointed to by 'feat' to include the specified terminal features.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strsep`](compat/strsep.c.driver.md#strsep)


---
### tty_get_features<!-- {{#callable:tty_get_features}} -->
The `tty_get_features` function returns a comma-separated string of terminal feature names that are enabled in the given feature bitmask.
- **Inputs**:
    - `feat`: An integer bitmask representing the enabled terminal features.
- **Control Flow**:
    - Initialize a static character array `s` to store the resulting feature names as a string.
    - Iterate over each terminal feature in the `tty_features` array using an index `i`.
    - For each feature, check if the corresponding bit in `feat` is set; if not, continue to the next feature.
    - If the feature is enabled, append its name to the string `s` followed by a comma.
    - After the loop, if `s` is not empty, remove the trailing comma by setting the last character to a null terminator.
    - Return the string `s` containing the names of the enabled features.
- **Output**:
    - A pointer to a static character array containing a comma-separated list of enabled terminal feature names.
- **Functions called**:
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


---
### tty_apply_features<!-- {{#callable:tty_apply_features}} -->
The `tty_apply_features` function applies specified terminal features to a given terminal structure if they are not already applied.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal to which features will be applied.
    - `feat`: An integer representing the features to be applied, encoded as bit flags.
- **Control Flow**:
    - Check if `feat` is zero; if so, return 0 immediately as no features need to be applied.
    - Log the features being applied using `tty_get_features(feat)`.
    - Iterate over each feature in `tty_features` using a loop indexed by `i`.
    - For each feature, check if it is already applied to `term` or not included in `feat`; if either condition is true, skip to the next feature.
    - Log the application of the current feature's name.
    - If the feature has associated capabilities, iterate over them and apply each capability to `term` using [`tty_term_apply`](tty-term.c.driver.md#tty_term_apply).
    - Update `term->flags` with the feature's flags.
    - After the loop, check if all features in `feat` are already applied to `term`; if so, return 0.
    - Otherwise, update `term->features` with `feat` and return 1.
- **Output**:
    - The function returns 0 if no new features were applied, or 1 if any features were successfully applied to the terminal.
- **Functions called**:
    - [`tty_get_features`](#tty_get_features)
    - [`tty_term_apply`](tty-term.c.driver.md#tty_term_apply)


---
### tty_default_features<!-- {{#callable:tty_default_features}} -->
The `tty_default_features` function assigns default terminal features to a given terminal type and version by matching against a predefined table of terminal names and their associated features.
- **Inputs**:
    - `feat`: A pointer to an integer where the features will be stored as bit flags.
    - `name`: A string representing the name of the terminal type.
    - `version`: An unsigned integer representing the version of the terminal.
- **Control Flow**:
    - A static table of terminal types and their associated features is defined, with each entry containing a terminal name, a version, and a string of features.
    - The function iterates over each entry in the table.
    - For each entry, it checks if the terminal name matches the provided `name` argument.
    - If the `version` argument is non-zero, it also checks if the provided version is greater than or equal to the version in the table entry.
    - If both conditions are met, it calls [`tty_add_features`](#tty_add_features) to add the features from the table entry to the `feat` argument, using a comma as the separator.
- **Output**:
    - The function does not return a value; it modifies the `feat` argument in place to include the features of the matched terminal type.
- **Functions called**:
    - [`tty_add_features`](#tty_add_features)


