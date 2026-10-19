# Building XyGrib for Windows (MinGW + Qt 5.13.2)

This document describes how to build XyGrib on Windows using the open-source MinGW toolchain. It documents the exact environment, the manual linker workaround, and the runtime deployment steps needed to get a fully functional portable `XyGrib.exe` with the `fix-swell-241` branch.

**Note:** This build process was verified with **CMake 4.3.5**, **Qt 5.13.2 (MinGW 7.3.0 32-bit)**, and **vcpkg**.

---

## 1. Prerequisites

You need the following installed before starting:

- **Qt 5.13.2** with the **MinGW 7.3.0 32-bit** kit.
  (The 32-bit requirement is strict: the original `thirdpartylib.zip` and the `libaec` static libraries are compiled for 32-bit.)
- **CMake** (version 3.14 or newer; tested with 4.3.5).
- **Git** (optional, for cloning the repository).
- **vcpkg** (for installing `libaec` and `proj`).

---

## 2. Setting up the Dependencies

XyGrib depends on several third-party C/C++ libraries. Historically, the project provided a pre-compiled `thirdpartylib.zip` (from 2018). Since those binaries are outdated, modern builds should compile the dependencies using `vcpkg` or place pre-compiled `.a`/`.dll.a` files in a custom directory.

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

## 3. Configuring with CMake

Open the **Qt 5.13.2 (MinGW 7.3.0 32-bit)** command prompt. Navigate to your XyGrib source directory and create a `build` folder:

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

## 4. Compiling (The 99% Linker Error)

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

## 5. Creating the Portable Folder (Runtime)

To run `XyGrib.exe`, you need the runtime `.dll` files and the `data` folder. Do **not** copy the `.dll` files from the compilation folders (like `/opt` or `vcpkg`), as those are linker libraries, not runtime libraries.

### 5.1. Deploy Qt DLLs
Open the Qt command prompt again, navigate to the folder where `XyGrib.exe` was built (`C:\project\xygrib\build\src`), and run:

```cmd
windeployqt XyGrib.exe
```

This will automatically copy the correct Qt `.dll` files (and the necessary plugin folders like `platforms`, `styles`, `imageformats`) into the same directory.

### 5.2. Copy the Data Folder (Icons and Maps)
If you launch the application and the toolbar icons are missing (empty buttons), it means XyGrib cannot find its resources. 

Copy the entire `data` folder from the root of your source repository (`C:\project\xygrib\data`) into the same directory as your `XyGrib.exe`.

Your final portable folder should look like this:
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

## 6. Verification

Once the portable folder is assembled, launch `XyGrib.exe`, load a GRIB2 file (e.g., generated by the `xygrib-noaa.sh` downloader), and verify:
1. The toolbar icons are visible.
2. The "Swell" layer is accessible and displays data (requires the `fix-swell-241` branch).

*(See the screenshots in the main README for an example of a successful run.)*

---

Fair winds! ⛵
