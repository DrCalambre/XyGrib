# Building XyGrib for Windows (MinGW + Qt 5.13.2)

This document describes how to build XyGrib on Windows using the open-source MinGW toolchain. It combines the original project setup with real-world fixes for modern environments, including the workaround for the 100% linker error and the missing toolbar icons.

**Note:** This build process was verified with **CMake 4.3.5**, **Qt 5.13.2 (MinGW 7.3.0 32-bit)**, and **vcpkg**.

---

## 1. Prerequisites

You need the following installed before starting:

- **Qt 5.13.2** with the **MinGW 7.3.0 32-bit** kit.
  Get the Qt online installer from [qt.io/download](https://qt.io/download). When choosing components, select exactly `Qt 5.13.2` → `MinGW 7.3.0 32-bit`. This specific version/kit combo matters.
- **CMake** (version 3.14 or newer; tested with 4.3.5). Tick "add to PATH" during the install.
- **Git for Windows**.
- **vcpkg** (for installing `libaec` and `proj`).

---

## 2. Setting up the Dependencies

XyGrib depends on several third-party C/C++ libraries. Historically, the project provided a pre-compiled `thirdpartylib.zip` (from 2018). Since those binaries are often incompatible with modern compilers, the reliable modern approach is to use `vcpkg` for some dependencies and a custom `C:\opt` directory for others.

### 2.1. Install vcpkg dependencies
Open a terminal (PowerShell or CMD) and run:

```cmd
git clone https://github.com/microsoft/vcpkg
cd vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg install libaec:x86-mingw-static proj:x86-mingw-static
```

### 2.2. Prepare the `C:\opt` directory
Some libraries (`bzip2`, `libnova`, `openjpeg`, `png`, `zlib`) are linked directly from a custom path. In the verified build, these were placed in `C:\opt\` with the following structure:

```text
C:\opt\
├── include\   (headers for bzlib.h, libnova, openjpeg.h, png.h, zlib.h)
└── lib\       (libbz2.a, libnova.a, libopenjp2.dll.a, libpng.a, libz.a)
```

*(You can build these from source or download pre-compiled MinGW 32-bit `.a` files from a reliable source.)*

---

## 3. Clone the patched branch

Open Git Bash:

```bash
git clone -b fix-swell-241 https://github.com/DrCalambre/XyGrib C:\project\xygrib
```

*(If the fix has been merged into `frfa/XyGrib` main by the time you read this, you can clone that instead — but double-check it's a clean build first).*

**Note:** the `fix-swell-241` branch actually carries two commits — the swell patch itself, and an earlier build fix that adds a missing `aecunpack.c` file plus a CMakeLists update for g2clib. Both are needed to compile cleanly from a fresh clone.

---

## 4. Set environment variables

In a plain `cmd` or `PowerShell` window (before building):

```cmd
set EXTERNLIBS=C:\opt
set PATH=C:\Qt\Tools\mingw730_32\bin;C:\Qt\5.13.2\mingw73_32\bin;%PATH%
set CC=gcc
set CXX=g++
```

**One gotcha:** if `sh.exe` (from Git Bash) is in your PATH in that terminal, CMake will refuse to generate MinGW Makefiles. Use a plain cmd/PowerShell window without Git Bash's `sh.exe` on the PATH.

---

## 5. Configuring with CMake

Open the **Qt 5.13.2 (MinGW 7.3.0 32-bit)** command prompt. Navigate to your source directory and create a `build` folder:

```cmd
cd C:\project\xygrib
mkdir build
cd build
```

Run CMake, passing the paths to both `vcpkg` and `C:\opt`:

```cmd
cmake -G "MinGW Makefiles" ^
  -DCMAKE_BUILD_TYPE=Release ^
  -DCMAKE_PREFIX_PATH="C:/project/vcpkg/installed/x86-mingw-static;C:/opt" ^
  -DCMAKE_TOOLCHAIN_FILE="C:/project/vcpkg/scripts/buildsystems/vcpkg.cmake" ^
  ..
```

---

## 6. Compiling (The 99% Linker Error)

Run the build:

```cmd
mingw32-make -j4
```

### ⚠️ Known Issue: Fails at 100% (Linker Error)
The compilation of the `.cpp` files will succeed (100%), but the final link step (`c++.exe ... -o XyGrib.exe`) **may fail** with errors like `undefined reference to ...`.

**Why does this happen?**
This is caused by the linker (`ld.exe`) failing to find the correct order or exact paths of the static libraries (`libaec.a`, `libproj.a`, `libbz2.a`, etc.). MinGW's linker is strictly order-dependent (a library must appear *after* the object files that depend on it).

### ✅ The Workaround: Manual Linking
If the automated link fails, open the `link.txt` file located at:
`C:\project\xygrib\build\src\CMakeFiles\XyGrib.dir\link.txt`

Inside, you will find the exact command the linker was trying to run. Copy that entire command, paste it into your terminal, and adjust it manually:
1. Ensure the `.a` library paths match exactly what is in `C:\opt\lib` and `C:\project\vcpkg\installed\x86-mingw-static\debug\lib`.
2. Reorder the `-l` flags so that libraries that depend on others come *first*.
3. Re-run the command. If successful, it will output `XyGrib.exe`.

*(Once `XyGrib.exe` exists, you can safely ignore the `make` error in the future; the executable is already built.)*

---

## 7. Creating the Portable Folder (Runtime)

To run `XyGrib.exe`, you need the runtime `.dll` files and the `data` folder. Do **not** copy the `.dll` files from the compilation folders (like `/opt` or `vcpkg`), as those are linker libraries, not runtime libraries.

### 7.1. Create the portable folder and deploy Qt DLLs
Create a dedicated folder for your portable build (e.g., `C:\XyGrib_Portable`), copy `XyGrib.exe` from `C:\project\xygrib\build\src\` into it, and open a Qt command prompt in that new folder:

```cmd
mkdir C:\XyGrib_Portable
copy C:\project\xygrib\build\src\XyGrib.exe C:\XyGrib_Portable\
cd C:\XyGrib_Portable
windeployqt XyGrib.exe
```

This will automatically copy the correct Qt `.dll` files (and the necessary plugin folders like `platforms`, `styles`, `imageformats`) directly into `C:\XyGrib_Portable`.

### 7.2. Copy the Data Folder (Icons and Maps)
If you launch the application and the toolbar icons are missing (empty buttons), it means XyGrib cannot find its resources. 

Copy the entire `data` folder from the root of your source repository (`C:\project\xygrib\data`) into `C:\XyGrib_Portable\`.

Your final portable folder should look like this:

![Portable Folder Structure](screenshots/build-windows/Xygrib_Windows_fix-swell-241_portable.jpg)

```text
C:\XyGrib_Portable\
├── data\               <-- (Icons, maps, translations)
├── platforms\          <-- (Qt platform plugins)
├── imageformats\       <-- (Qt image plugins)
├── Qt5Core.dll
├── Qt5Gui.dll
├── ...
└── XyGrib.exe
```

---

## 8. Verification

Once the portable folder is assembled, launch `XyGrib.exe`, load a GRIB2 file (e.g., generated by the `xygrib-noaa.sh` downloader), and verify:
1. The toolbar icons are visible.
2. The "Swell" layer is accessible and displays data (requires the `fix-swell-241` branch).

**Map View:**
![Map View with Swell](screenshots/build-windows/Xygrib_Windows_fix-swell-241_01.jpg)

**Meteogram/Data Table:**
![Meteogram showing Swell data](screenshots/build-windows/Xygrib_Windows_fix-swell-241_02.jpg)

---

If this works for you, it might be worth building a release `.exe` and attaching it to the fork's GitHub releases page, so other non-Linux users on this thread don't have to repeat the whole toolchain setup.

Fair winds! ⛵
```
