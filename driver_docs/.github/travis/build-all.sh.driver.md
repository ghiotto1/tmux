# Purpose
This script is a build automation script written in shell (sh) that provides a narrow functionality focused on downloading, configuring, and installing specific software libraries, namely libevent and ncurses. It is not an executable or a library file but rather a utility script intended to automate the setup process for these dependencies, likely as part of a larger software project. The script uses `wget` to download the source tarballs, extracts them, and then configures and installs them with specific options to a designated build directory. It also runs an `autogen.sh` script and configures the environment for further build steps, indicating that it is part of a larger build process. The script is designed to be robust, exiting with an error if any step fails, and it provides a streamlined way to ensure that the necessary libraries are correctly set up for development or deployment.
# Global Variables

---
### BUILD
- **Type**: `string`
- **Description**: The `BUILD` variable is a string that holds the path to the build directory, which is set to the current working directory appended with '/build'. This variable is used to specify the installation prefix for the software being compiled and installed, ensuring that all build artifacts are placed in a specific directory.
- **Use**: This variable is used to define the installation path for the software being built, ensuring all components are installed in a designated directory.


---
### LIBEVENT
- **Type**: `string`
- **Description**: The `LIBEVENT` variable is a string that holds the URL to the tarball of the libevent library version 2.1.11-stable. This URL is used to download the source code for libevent, which is a library that provides asynchronous event notification.
- **Use**: This variable is used to specify the download location for the libevent library, which is then fetched and extracted for building and installation.


---
### NCURSES
- **Type**: `string`
- **Description**: The `NCURSES` variable is a string that holds the URL to the tar.gz archive of the ncurses library version 6.2. This URL points to the GNU FTP server where the ncurses source code can be downloaded.
- **Use**: This variable is used to download the ncurses library source code for building and installation.


