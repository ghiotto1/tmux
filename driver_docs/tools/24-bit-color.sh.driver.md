# Purpose
This Bash script is designed to demonstrate the use of 24-bit color codes in a terminal environment by echoing four gradients. It utilizes ANSI escape sequences to manipulate the terminal's background color, showcasing the capability to render a wide range of colors by specifying red, green, and blue (RGB) values, each ranging from 0 to 255. The script includes functions to set the background color and reset the terminal output to its default state. It also features a function, `rainbowColor`, which calculates RGB values to represent a color along the HSV spectrum, providing a visual gradient effect.

The script determines the appropriate sequence command to use (`seq` or `gseq`) based on the system's available utilities, ensuring compatibility across different environments. It then defines a custom sequence function, `seq1`, to handle both ascending and descending sequences, which is crucial for generating the gradients. The script iterates over a range of values to create gradients for red, green, blue, and a rainbow spectrum, using loops to apply the background color changes incrementally.

Overall, this script serves as a practical example of how to implement and visualize 24-bit color support in terminal applications. It is a standalone executable script that does not define public APIs or external interfaces but rather focuses on demonstrating terminal color capabilities through direct execution.
# Global Variables

---
### SEQ1
- **Type**: `string`
- **Description**: The variable `SEQ1` is a string that is used to determine the appropriate command for generating sequences of numbers. It is set to either 'gseq' or 'seq' based on the availability and version of the sequence command on the system.
- **Use**: `SEQ1` is used to define the command for generating number sequences, which is then utilized in the `seq1` function to create sequences with specific increments.


---
### SEQ
- **Type**: `function`
- **Description**: The `SEQ` variable is a global variable that is assigned a function name based on the availability of certain sequence-generating commands in the system. It is set to either `seq1`, a custom function defined in the script, or `seq`, a standard command, depending on the presence of GNU `seq` or `gseq`.
- **Use**: This variable is used to generate sequences of numbers for iterating over color values in the script.


---
### SEPARATOR
- **Type**: `string`
- **Description**: The `SEPARATOR` variable is a global string variable that is used to define the character used to separate components in escape sequences for setting terminal colors. In this script, it is set to the colon character ':'. This separator is used in the `setBackgroundColor` function to format the escape sequence for setting background colors in the terminal.
- **Use**: This variable is used to construct escape sequences for terminal color settings by separating the components of the sequence.


# Functions

---
### seq1
The `seq1` function generates a sequence of numbers between two given integers, either in ascending or descending order, depending on their values.
- **Inputs**:
    - `$1`: The starting integer of the sequence.
    - `$2`: The ending integer of the sequence.
- **Control Flow**:
    - Check if the first argument ($1) is greater than the second argument ($2).
    - If $1 is greater than $2, use the command stored in SEQ1 to generate a sequence from $1 to $2 with a decrement of 1.
    - If $1 is less than or equal to $2, use the command stored in SEQ1 to generate a sequence from $1 to $2 with an increment of 1.
- **Output**:
    - The function outputs a sequence of numbers from $1 to $2, either in ascending or descending order, depending on the values of $1 and $2.


---
### setBackgroundColor
The `setBackgroundColor` function sets the terminal's background color using 24-bit RGB color codes.
- **Inputs**:
    - `$1`: The red component of the RGB color, an integer between 0 and 255.
    - `$2`: The green component of the RGB color, an integer between 0 and 255.
    - `$3`: The blue component of the RGB color, an integer between 0 and 255.
- **Control Flow**:
    - The function constructs an escape sequence string for setting the background color using the provided RGB values.
    - It uses the escape sequence format '\033[48;2;<r>;<g>;<b>m' where <r>, <g>, and <b> are replaced by the input RGB values.
    - The constructed escape sequence is then echoed to the terminal with the '-en' option to enable interpretation of backslash escapes and to suppress the trailing newline.
- **Output**:
    - The function outputs an escape sequence to the terminal that changes the background color to the specified RGB values.


---
### resetOutput
The `resetOutput` function resets the terminal's output to its default settings by sending an escape sequence.
- **Inputs**:
    - None
- **Control Flow**:
    - The function uses the `echo` command with the `-en` flags to send an escape sequence to the terminal.
    - The escape sequence `\033[0m` is used to reset the terminal's output formatting.
    - A newline character is appended to the escape sequence to move the cursor to the next line.
- **Output**:
    - The function outputs an escape sequence that resets the terminal's output formatting to default settings, followed by a newline.


---
### rainbowColor
The `rainbowColor` function calculates RGB color values based on a given position along the HSV color wheel, specifically for values between 0 and 255.
- **Inputs**:
    - `$1`: An integer representing a position along the HSV color wheel, expected to be between 0 and 255.
- **Control Flow**:
    - Calculate the hue segment `h` by dividing the input by 43.
    - Calculate the fractional part `f` by subtracting 43 times `h` from the input.
    - Calculate `t` as the fractional part of the hue segment scaled to 255.
    - Calculate `q` as 255 minus `t`.
    - Use a series of conditional statements to determine the RGB values based on the hue segment `h`.
    - Output the RGB values as a string in the format "red green blue".
- **Output**:
    - A string containing three space-separated integers representing the red, green, and blue components of the color, each ranging from 0 to 255.


