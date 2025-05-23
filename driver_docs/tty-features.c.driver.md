# Purpose
This C source code file is part of a terminal multiplexer, likely related to the `tmux` project, which is used to manage terminal sessions. The file defines a set of terminal features and capabilities, encapsulated in the `tty_feature` structure, which includes a name, a list of capabilities, and associated flags. These features cover a wide range of terminal functionalities, such as color support (RGB and 256 colors), mouse support, clipboard operations, hyperlinks, and various text styling options like overline and strikethrough. The code provides mechanisms to add, retrieve, and apply these features to terminal sessions, allowing for dynamic configuration based on the terminal's capabilities.

The file is not an executable but rather a component of a larger system, likely intended to be included in other parts of the software. It does not define public APIs or external interfaces directly but provides internal functionality to manage terminal features. The code includes functions to add features to a terminal, retrieve a list of features, and apply these features to a terminal session. It also includes a function to set default features based on the terminal's name and version, indicating its role in adapting the terminal multiplexer to different terminal environments.
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
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_title_capabilities` is a static constant array of string constants that defines terminal capabilities related to setting the xterm(1) title. It includes escape sequences for starting and finishing the title setting operation. The array is terminated with a `NULL` pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to specify the capabilities required for a terminal to support xterm(1) title setting, and it is referenced in the `tty_feature_title` structure.


---
### tty_feature_title
- **Type**: ``struct tty_feature``
- **Description**: The `tty_feature_title` is a static constant instance of the `struct tty_feature` structure, representing a named terminal feature related to xterm(1) title setting. It contains a name, a list of capabilities, and flags. The capabilities include escape sequences for setting and finalizing the terminal title.
- **Use**: This variable is used to define and manage the terminal's ability to set the xterm(1) title, which is part of the terminal features available in the system.


---
### tty_feature_osc7_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_osc7_capabilities` is a static constant array of string pointers, which defines the capabilities related to the OSC 7 (Operating System Command 7) feature for terminals. This array contains escape sequences that are used to set the working directory in terminal emulators that support OSC 7. The array is terminated with a `NULL` pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to store and provide the specific escape sequences that enable the OSC 7 feature in terminal emulators.


---
### tty_feature_osc7
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_osc7` is a static constant instance of the `struct tty_feature` that represents a terminal feature related to the OSC 7 working directory capability. It contains a name, a list of capabilities, and flags. The capabilities include escape sequences for setting the working directory and finalizing the sequence.
- **Use**: This variable is used to define and manage the OSC 7 working directory feature for terminals, allowing the terminal to recognize and apply this capability.


---
### tty_feature_mouse_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_mouse_capabilities` is a static constant array of strings that defines the capabilities related to mouse support in a terminal. It contains escape sequences that are used to enable mouse tracking in terminal applications.
- **Use**: This variable is used to specify the mouse capabilities of a terminal, allowing applications to interpret mouse events.


---
### tty_feature_mouse
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_mouse` is a constant instance of the `struct tty_feature` that represents the mouse support feature in a terminal. It contains a name, a list of capabilities, and flags associated with the feature. The capabilities list includes the escape sequence for mouse events, specifically `kmous=\E[M`. The flags are set to 0, indicating no special flags are used for this feature.
- **Use**: This variable is used to define and manage the mouse support feature in terminal applications, allowing them to recognize and handle mouse input events.


---
### tty_feature_clipboard_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_clipboard_capabilities` is a static constant array of strings that defines the capabilities related to setting the clipboard in a terminal using the OSC 52 escape sequence. It contains a single string that specifies the escape sequence format for setting the clipboard content, followed by a NULL terminator to indicate the end of the array.
- **Use**: This variable is used to specify the terminal capabilities for clipboard operations, particularly for setting clipboard content using the OSC 52 escape sequence.


---
### tty_feature_clipboard
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_clipboard` is a static constant instance of the `struct tty_feature` that represents the clipboard feature for a terminal. It includes a name, a set of capabilities, and flags. The capabilities array contains the escape sequence for setting the clipboard using OSC 52.
- **Use**: This variable is used to define and manage the clipboard feature capabilities for terminals that support OSC 52.


---
### tty_feature_hyperlinks_capabilities
- **Type**: `const char *`
- **Description**: The `tty_feature_hyperlinks_capabilities` is a static constant array of strings that defines terminal capabilities related to OSC 8 hyperlinks. It is conditionally compiled to include a specific capability string if the code is being compiled on OpenBSD or with a version of ncurses greater than 5.8. The array is terminated with a NULL pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to specify the terminal's support for OSC 8 hyperlinks, which allows for hyperlink functionality in terminal emulators that support this feature.


---
### tty_feature_hyperlinks
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_hyperlinks` is a static constant instance of the `struct tty_feature` that represents the terminal's capability to support OSC 8 hyperlinks. It contains a name, a list of capabilities, and flags, with the capabilities being specific escape sequences for enabling hyperlink support in compatible terminals.
- **Use**: This variable is used to define and manage the terminal's hyperlink capabilities, allowing the terminal to interpret and render hyperlinks correctly.


---
### tty_feature_rgb_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_rgb_capabilities` is a static constant array of strings that defines the capabilities related to RGB color support in a terminal. It includes escape sequences for setting foreground and background RGB colors, as well as fallbacks for setting colors using the 256-color palette. The array is terminated with a NULL pointer to indicate the end of the list.
- **Use**: This variable is used to specify the RGB color capabilities of a terminal, allowing for advanced color settings in terminal applications.


---
### tty_feature_rgb
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_rgb` is a static constant instance of the `struct tty_feature` that represents a terminal feature supporting RGB color capabilities. It includes a name, a list of capabilities, and flags indicating the feature's properties. The capabilities include escape sequences for setting RGB foreground and background colors, as well as compatibility with 256 color palettes.
- **Use**: This variable is used to define and manage the RGB color capabilities of a terminal, allowing for enhanced color support in terminal applications.


---
### tty_feature_256_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_256_capabilities` is a static constant array of strings that defines terminal capabilities related to 256 color support. It includes escape sequences for setting background (`setab`) and foreground (`setaf`) colors using the 256 color palette, as well as a capability identifier `AX`. The array is terminated with a `NULL` pointer to indicate the end of the list.
- **Use**: This variable is used to specify the capabilities of terminals that support 256 colors, allowing the software to apply these capabilities when interacting with such terminals.


---
### tty_feature_256
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_256` is a constant instance of the `struct tty_feature` that represents a terminal feature supporting 256 colors. It includes a name, a list of capabilities, and a set of flags associated with this feature. The capabilities define the escape sequences used to set background and foreground colors in a terminal that supports 256 colors.
- **Use**: This variable is used to define and manage the 256-color capability of a terminal within the terminal multiplexer application.


---
### tty_feature_overline_capabilities
- **Type**: `array of strings`
- **Description**: The `tty_feature_overline_capabilities` is a static constant array of strings that defines terminal capabilities related to the overline feature. It contains escape sequences that enable the overline text attribute in terminals that support it.
- **Use**: This variable is used to specify the escape sequence for enabling the overline feature in terminals.


---
### tty_feature_overline
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_overline` is a constant structure of type `tty_feature` that represents a terminal feature supporting overline text formatting. It contains a name, a list of capabilities, and flags associated with the feature. The capabilities array includes the escape sequence for enabling overline mode in a terminal.
- **Use**: This variable is used to define and manage the overline capability of a terminal within the terminal feature management system.


---
### tty_feature_usstyle_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_usstyle_capabilities` is a static constant array of string constants that define terminal capabilities related to underscore styles. Each string in the array represents a specific terminal escape sequence for different underscore styles, such as setting underline color or style. The array is terminated with a `NULL` pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to store and provide terminal escape sequences for underscore styles, which can be applied to terminals that support these features.


---
### tty_feature_usstyle
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_usstyle` is a constant structure of type `tty_feature` that represents terminal support for underscore styles. It includes a name, a set of capabilities, and flags. The capabilities define specific escape sequences for setting underscore styles and colors.
- **Use**: This variable is used to identify and apply terminal capabilities related to underscore styles.


---
### tty_feature_bpaste_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_bpaste_capabilities` is an array of constant strings that define the terminal capabilities related to bracketed paste mode. It includes escape sequences to enable and disable bracketed paste mode in a terminal.
- **Use**: This variable is used to store the escape sequences for enabling and disabling bracketed paste mode, which are part of the terminal's feature set.


---
### tty_feature_bpaste
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_bpaste` is a static constant instance of the `struct tty_feature` that represents the terminal's support for bracketed paste mode. It contains a name, a list of capabilities, and flags. The capabilities include enabling and disabling bracketed paste mode with specific escape sequences.
- **Use**: This variable is used to identify and apply the bracketed paste feature in terminal applications that support it.


---
### tty_feature_focus_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_focus_capabilities` is a static constant array of strings that defines terminal capabilities related to focus reporting. It includes escape sequences for enabling and disabling focus events in a terminal.
- **Use**: This variable is used to specify the terminal capabilities for focus reporting, which can be applied to a terminal to enable or disable focus events.


---
### tty_feature_focus
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_focus` is a static constant instance of the `struct tty_feature` that represents a terminal feature related to focus reporting. It contains a name, a list of capabilities, and flags. The capabilities include enabling and disabling focus reporting sequences for terminals.
- **Use**: This variable is used to define and manage the focus reporting feature in terminal applications, allowing the terminal to notify when it gains or loses focus.


---
### tty_feature_cstyle_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_cstyle_capabilities` is a static constant array of string constants that defines terminal capabilities related to cursor styles. It includes escape sequences for setting and resetting cursor styles in a terminal.
- **Use**: This variable is used to specify the escape sequences for enabling and disabling cursor styles in terminal applications.


---
### tty_feature_cstyle
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_cstyle` is a constant instance of the `tty_feature` structure, representing a terminal feature related to cursor styles. It includes a name, a set of capabilities, and flags, which are all initialized to specific values.
- **Use**: This variable is used to define and manage the capabilities of terminals that support cursor styles, allowing the application to apply these features when interacting with such terminals.


---
### tty_feature_ccolour_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_ccolour_capabilities` is a static constant array of string literals that define terminal capabilities related to cursor color settings. It includes escape sequences for setting and resetting cursor colors in a terminal that supports these features.
- **Use**: This variable is used to store the escape sequences for terminal capabilities that allow setting and resetting cursor colors, which are part of the `tty_feature_ccolour` structure.


---
### tty_feature_ccolour
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_ccolour` is a static constant instance of the `struct tty_feature` that represents a terminal feature related to cursor colors. It contains a name, a list of capabilities, and flags. The capabilities include escape sequences for setting and resetting cursor colors.
- **Use**: This variable is used to define and manage the cursor color capabilities of a terminal within the terminal multiplexer application.


---
### tty_feature_strikethrough_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_strikethrough_capabilities` is a static constant array of strings that defines terminal capabilities related to the strikethrough feature. It contains escape sequences that enable strikethrough text formatting in terminals that support it.
- **Use**: This variable is used to specify the escape sequence for enabling strikethrough text in terminal applications.


---
### tty_feature_strikethrough
- **Type**: ``struct tty_feature``
- **Description**: The `tty_feature_strikethrough` is a static constant instance of the `struct tty_feature` that represents the terminal's capability to support strikethrough text formatting. It contains a name, a list of capabilities, and flags associated with this feature. The capabilities list includes the escape sequence for enabling strikethrough mode in the terminal.
- **Use**: This variable is used to identify and apply the strikethrough capability to terminals that support it.


---
### tty_feature_sync_capabilities
- **Type**: `array of strings`
- **Description**: The `tty_feature_sync_capabilities` is a static constant array of strings that defines terminal capabilities related to synchronized updates. It contains escape sequences that are used to enable or disable synchronized updates in a terminal.
- **Use**: This variable is used to specify the escape sequences for enabling synchronized updates in terminal features.


---
### tty_feature_sync
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_sync` is a static constant instance of the `struct tty_feature` that represents a terminal feature supporting synchronized updates. It is initialized with the name "sync", a set of capabilities defined in `tty_feature_sync_capabilities`, and a flag value of 0.
- **Use**: This variable is used to define and manage the capabilities of terminals that support synchronized updates, allowing the system to apply these features when interacting with compatible terminals.


---
### tty_feature_extkeys_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_extkeys_capabilities` is a static constant array of strings that defines terminal capabilities related to extended keys. It contains escape sequences that enable and disable extended keys in a terminal environment.
- **Use**: This variable is used to specify the escape sequences for enabling and disabling extended key support in terminals.


---
### tty_feature_extkeys
- **Type**: ``struct tty_feature``
- **Description**: The `tty_feature_extkeys` is a constant instance of the `struct tty_feature` that represents a terminal feature supporting extended keys. It is initialized with the name "extkeys", a set of capabilities defined in `tty_feature_extkeys_capabilities`, and a flag value of 0.
- **Use**: This variable is used to define and manage the extended keys feature for terminals, allowing the system to recognize and apply this capability when interacting with terminal environments.


---
### tty_feature_margins_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_margins_capabilities` is a static constant array of string literals that define terminal capabilities related to DECSLRM margins. Each string in the array represents a specific escape sequence used to enable, disable, clear, or set margins in a terminal that supports these features. The array is terminated with a `NULL` pointer to indicate the end of the capabilities list.
- **Use**: This variable is used to store the escape sequences for managing terminal margins, which are part of the `tty_feature_margins` structure to define terminal features.


---
### tty_feature_margins
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_margins` is a constant structure of type `tty_feature` that represents terminal support for DECSLRM margins. It includes a name, a set of capabilities, and a flag indicating the feature's properties.
- **Use**: This variable is used to define and manage terminal capabilities related to setting margins, which can be applied to terminals that support the DECSLRM feature.


---
### tty_feature_rectfill_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_rectfill_capabilities` is a static constant array of string pointers, which contains terminal capability strings related to rectangle fill operations. It is part of the terminal feature set that supports DECFRA rectangle fill capabilities.
- **Use**: This variable is used to define the capabilities of terminals that support rectangle fill operations, specifically for the DECFRA feature.


---
### tty_feature_rectfill
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_rectfill` is a constant structure of type `tty_feature` that represents a terminal feature supporting DECFRA rectangle fill. It contains a name, a list of capabilities, and associated flags. The capabilities list for this feature includes a single capability, "Rect", and it is associated with the `TERM_DECFRA` flag.
- **Use**: This variable is used to define and manage the DECFRA rectangle fill capability of a terminal within the terminal multiplexer software.


---
### tty_feature_ignorefkeys_capabilities
- **Type**: ``const char *const[]``
- **Description**: The `tty_feature_ignorefkeys_capabilities` is a static constant array of string literals representing terminal capabilities related to ignoring function keys. Each string in the array corresponds to a specific function key capability, denoted by 'kf' followed by a number and an '@' symbol, indicating that the function key should be ignored.
- **Use**: This variable is used to define a set of terminal capabilities that should be ignored, specifically related to function keys, within the context of terminal feature management.


---
### tty_feature_ignorefkeys
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_ignorefkeys` is a constant structure of type `tty_feature` that represents a terminal feature for ignoring function keys. It is initialized with the name "ignorefkeys", a list of capabilities that disable function keys from kf0 to kf63, and a flag value of 0.
- **Use**: This variable is used to define a terminal feature that disables the use of function keys, likely to ensure compatibility with certain terminal environments or applications.


---
### tty_feature_sixel_capabilities
- **Type**: `const char *const[]`
- **Description**: The `tty_feature_sixel_capabilities` is a static constant array of string pointers that defines the capabilities related to the sixel feature of a terminal. It contains a single string "Sxl" followed by a NULL terminator, indicating the presence of sixel capability in the terminal.
- **Use**: This variable is used to specify the sixel capabilities of a terminal in the context of terminal feature management.


---
### tty_feature_sixel
- **Type**: `struct tty_feature`
- **Description**: The `tty_feature_sixel` is a static constant instance of the `struct tty_feature` that represents a terminal feature supporting sixel graphics. It contains a name, a list of capabilities, and a set of flags. The capabilities array for this feature includes the string "Sxl", and it is associated with the `TERM_SIXEL` flag.
- **Use**: This variable is used to define and manage the sixel graphics capability of a terminal within the terminal multiplexer application.


---
### tty_features
- **Type**: ``const struct tty_feature *const[]``
- **Description**: The `tty_features` variable is a static constant array of pointers to `tty_feature` structures. Each element in the array represents a specific terminal feature, such as support for 256 colors, mouse support, or clipboard capabilities, among others. These features are defined as `tty_feature` structures, each containing a name, a list of capabilities, and associated flags.
- **Use**: This variable is used to store and manage a list of terminal features that can be applied to a terminal session, allowing the program to check and apply these features as needed.


# Data Structures

---
### tty_feature
- **Type**: `struct`
- **Members**:
    - `name`: A constant character pointer representing the name of the terminal feature.
    - `capabilities`: A pointer to a constant array of constant character pointers, representing the capabilities associated with the terminal feature.
    - `flags`: An integer representing flags associated with the terminal feature.
- **Description**: The `tty_feature` structure is used to define a named terminal feature, including its associated capabilities and any relevant flags. Each feature has a name, a list of capabilities expressed as terminal control sequences, and flags that may indicate specific properties or behaviors of the feature. This structure is part of a system to manage and apply terminal features in a terminal multiplexer environment, allowing for the customization and extension of terminal capabilities.


# Functions

---
### tty_add_features<!-- {{#callable:tty_add_features}} -->
The `tty_add_features` function adds specified terminal features to a feature set based on a string of feature names separated by given delimiters.
- **Inputs**:
    - `feat`: A pointer to an integer representing the current set of terminal features, where each bit corresponds to a specific feature.
    - `s`: A string containing the names of terminal features to be added, separated by specified delimiters.
    - `separators`: A string containing delimiter characters used to separate feature names in the input string `s`.
- **Control Flow**:
    - The function logs the input string `s` for debugging purposes.
    - It duplicates the input string `s` to a new string `copy` to avoid modifying the original string.
    - The function enters a loop where it uses `strsep` to split `copy` into individual feature names based on the `separators`.
    - For each feature name extracted, it iterates over the `tty_features` array to find a matching feature by comparing names case-insensitively.
    - If a matching feature is found, it checks if the feature is not already set in `feat` using bitwise operations.
    - If the feature is not set, it logs the addition of the feature and updates `feat` to include the new feature by setting the corresponding bit.
    - If no matching feature is found, it logs the unknown feature and breaks out of the loop.
    - Finally, it frees the duplicated string `copy` to prevent memory leaks.
- **Output**:
    - The function does not return a value; it modifies the integer pointed to by `feat` to include the specified terminal features.


---
### tty_get_features<!-- {{#callable:tty_get_features}} -->
The `tty_get_features` function returns a comma-separated string of terminal feature names that are enabled in the given feature bitmask.
- **Inputs**:
    - `feat`: An integer bitmask representing the enabled terminal features.
- **Control Flow**:
    - Initialize a static character array `s` to store the resulting feature names as a string.
    - Iterate over each terminal feature in the `tty_features` array using an index `i`.
    - For each feature, check if the corresponding bit in `feat` is set; if not, continue to the next feature.
    - If the bit is set, append the feature's name to the string `s`, followed by a comma.
    - After the loop, if `s` is not empty, remove the trailing comma by setting the last character to a null terminator.
    - Return the string `s` containing the names of the enabled features.
- **Output**:
    - A pointer to a static character array containing a comma-separated list of enabled terminal feature names.


---
### tty_apply_features<!-- {{#callable:tty_apply_features}} -->
The `tty_apply_features` function applies specified terminal features to a given terminal structure if they are not already applied.
- **Inputs**:
    - `term`: A pointer to a `tty_term` structure representing the terminal to which features are to be applied.
    - `feat`: An integer representing the features to be applied, encoded as bit flags.
- **Control Flow**:
    - Check if the `feat` argument is zero; if so, return 0 immediately as no features need to be applied.
    - Log the features being applied using `tty_get_features(feat)`.
    - Iterate over each feature in the `tty_features` array.
    - For each feature, check if it is already applied to the terminal or not included in `feat`; if either is true, skip to the next feature.
    - Log the application of the current feature.
    - If the feature has associated capabilities, iterate over them and apply each capability to the terminal using `tty_term_apply`.
    - Update the terminal's flags with the feature's flags.
    - After iterating through all features, check if all features in `feat` are already applied; if so, return 0.
    - Otherwise, update the terminal's features with `feat` and return 1.
- **Output**:
    - The function returns 0 if no new features were applied, and 1 if any new features were successfully applied to the terminal.
- **Functions called**:
    - [`tty_get_features`](#tty_get_features)


---
### tty_default_features<!-- {{#callable:tty_default_features}} -->
The `tty_default_features` function assigns default terminal features to a given terminal type and version by matching it against a predefined table of terminal names and their associated features.
- **Inputs**:
    - `feat`: A pointer to an integer where the features will be stored as bit flags.
    - `name`: A string representing the name of the terminal type.
    - `version`: An unsigned integer representing the version of the terminal.
- **Control Flow**:
    - A static table of terminal types, versions, and their associated features is defined.
    - The function iterates over each entry in the table.
    - For each entry, it checks if the terminal name matches the provided name.
    - If a version is specified, it checks if the provided version is greater than or equal to the version in the table.
    - If both checks pass, it calls [`tty_add_features`](#tty_add_features) to add the features from the table entry to the `feat` variable.
- **Output**:
    - The function does not return a value; it modifies the `feat` variable in place to include the features of the matched terminal type.
- **Functions called**:
    - [`tty_add_features`](#tty_add_features)


