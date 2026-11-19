# Hexedit
Hexedit is a TUI-based hex editor.

![Hexedit editing itself](https://github.com/dcdeepesh/hexedit/blob/master/Documentation/images/Home.png)

# Table of Contents
- [Building Hexedit](#building-hexedit)
- [Download](#download)
- [Usage](#usage)
  - [Main screen](#main-screen)
  - [Navigation](#navigation)
  - [Input sequences](#input-sequences)
    - Save sequence
    - Exit (quit) sequence
  - [Modes](#modes)
    - Normal mode
    - Edit mode
  - [Examples](#examples)
- [Contributing](#contributing)
- [TODO](#todo)
- [Additional Notes](#additional-notes)

## Building Hexedit
Hexedit needs CMake 3.21+, Ninja, a C++17 compiler and curses.
On Windows, [PDCurses](https://github.com/wmcbrine/PDCurses) is downloaded and linked statically, so the result is a single `hexedit.exe`.

**Linux / macOS**
```sh
sudo apt-get install cmake ninja-build libncurses-dev   # or: brew install cmake ninja
cmake --preset default
cmake --build --preset default    # -> build/default/hexedit
```

**Windows (x64 and ARM64)**

Install the tools once (LLVM-MinGW builds for both architectures from any Windows machine):
```powershell
winget install Kitware.CMake Ninja-build.Ninja MartinStorsjo.LLVM-MinGW.UCRT
```

Then, in a new terminal:
```powershell
cmake --preset windows-x64   ; cmake --build --preset windows-x64     # -> build\windows-x64\hexedit.exe
cmake --preset windows-arm64 ; cmake --build --preset windows-arm64   # -> build\windows-arm64\hexedit.exe
```

GitHub Actions builds Linux, Windows x64 and Windows ARM64 binaries for every pull request and push to `master` (see the workflow artifacts).

## Download
Prebuilt binaries are on the [Releases page](https://github.com/shanselman/hexedit/releases/latest):
`hexedit-windows-x64.exe`, `hexedit-windows-arm64.exe` and `hexedit-linux-x64`. No install needed; just download and run.

**Making a release:** push a version tag and CI builds and publishes the release automatically.
```sh
git tag v1.0.0
git push origin v1.0.0
```

## Usage
To use Hexedit, provide the file name or the path to the file as a command line argument.
If no argument is provided, Hexedit will print a short helpful error msg and quit.
If the file is found and is readable, Hexedit opens (in [normal mode](#modes)).

### Main screen
The top line tells about the index in the current row.
The left column is an index for different columns.
The middle column has the file contents in hex (with a default of 16 columns).
The right column has file contents in ASCII.
Finally, there's a status bar at the bottom that gives information about the current mode, sequence, position etc.

The ASCII contents *do not* include extended ASCII.
All control characters, extended ASCII and other unprintable characters are displayed as a dot (`.`).
There is no distinction between actual dots (ASCII: 46, hex: 0x2E) and unprintable characters,
so you have to see the hex code to distinguish between them.

There is a marker (yellow by default) that indicates your current position in the table.
There are two markers, one in the hex data, and one at the corresponding ASCII character.

### Navigation
To move the marker, following keys can be used.
The keys have a map in the home row, which should be familiar if you have used vim.
The marker is not moved, if the movement would cause the marker to go outside the bounds of file contents.

- `k`, `Arrow Up`: Move up one cell
- `j`, `Arrow Down`: Move down one cell
- `h`, `Arrow Left`: Move left one cell
- `l`, `Arrow Right`: Move right one cell

### Input sequences
To perform actions like saving a file or exiting the editor, you need to use input sequences.
Input sequences are series of keystrokes without interruption.
e.g. If you press `s`, you start the save sequence.
Now if you press any key, it will be used as an input in the series.
If this key is a part of the sequence, the corresponding task is done, otherwise the sequence is terminated.
Which means that to exit an input sequence, just press any other key that is not a part of the sequence.

Following are the different input sequences in Hexedit:
- **Save sequence** (`s`) - This sequence deals with saving files. It has following actions:
  - `s` - Overwrite the currently opened file.
  - `n` - Create a new file named `mod_<fileName>` (e.g. `main.cpp` becomes `mod_main.cpp`).
- **Exit (quit) sequence** (`q`) - This sequence deals with quitting the editor.
If there are no unsaved changes, the editor exits immediately. If there are unsaved changes:
  - `q` - Ignore unsaved changes and quit editor.

### Modes
To perform interactive tasks such as editing a byte, you need to enter the corresponding mode.
Hexedit is in Normal Mode upon start.
To enter a mode, press the corresponding key in Normal mode.

Following are the modes in Hexedit:
- **Normal mode** - Upon start, Hexedit is in Normal mode.
A key pressed in normal mode either starts an input sequence, enters a mode, or is ignored if not a part of any sequence or mode.
**Note:** To enter a mode or start an input sequence, you need to be in the Normal mode, otherwise any further keystrokes are
considered part of the mode/sequence you are in. When you are in a sequence, pressing a key that is not a part of that sequence, brings the
editor to Normal mode.
- **Edit mode** (`i`) - When `i` is pressed in Normal mode, the editor goes into edit mode for the currently highlighted byte.
Following are the inputs for edit mode:
  - `Arrow Left`/`Arrow Right` - Move left/right in the current byte (cell).
  - `Enter` - Save changes, if any, and quit edit mode
  - `q` - Ignore any changes and quit edit mode
  
### Examples
**Running without any arguments:**
```sh
./build/default/hexedit
```
Output:
```
No file provided.
Usage: ./hexedit <fileName>
Use option '-h' or --help' to get complete help.
```
(`-h` and `--help` are not supported right now. In fact, using them will most probably give you an error)

**Running with a valid readable file:**
```sh
./build/default/hexedit build/default/hexedit
```
Output:
```
(output similar to the image above)
```

## Contributing
Contributions are welcome in the form of pull requests.
Contributors are requested to be specific about the problem their edits solve, 
keep the commits clean and provide sufficiently detailed commit messages.

## TODO
Note: This TODO list is in no way exhaustive.

- Command mode
- Change indicator
- Optimize file save process

## Additional Notes
- In some terminal emulators, the some colors may not be accurate (esp. black; as in the picture at the beginning of this file)
- If Hexedit is used in a windowed terminal emulator, the effect of resizing the window while Hexedit is running is undefined.
It is recommended not to resize the window while Hexedit is running.
The default config of 16 columns fits without any problem in a standard 80x25 terminal.
