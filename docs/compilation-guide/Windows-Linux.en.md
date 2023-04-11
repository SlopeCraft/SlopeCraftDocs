# Compilation Guide on Windows and Linux

This document tells you how to build SlopeCraft on Windows or Linux.

## Steps

### Install Dependencies

#### Windows:

Install Qt, libzip and libpng manually, or compile it by yourselfs.

#### Linux (Ubuntu):
```bash
# For libpng, libzip and eigen
sudo apt install libzip-dev zipcmp ziptool zipmerge libpng-dev libeigen3-dev

# For Qt6.2.4
sudo apt install libqt6widgets6 libqt6gui6 libqt6network6 libqt6concurrent6 qt6-base-dev qt6-tools-dev-tools qt6-tools-dev qt6-l10n-tools
sudo apt install x11-utils libxcb-xinerama0 libxv1 libgl-dev
```

### Build SlopeCraft

1. Clone this repo

      ```bash
      git clone https://github.com/SlopeCraft/SlopeCraft.git
      ```

2. Configure

      Run cmake to configure this project. The command could be:

      - Windows

         ```bash
         cmake -S . -B ./build -G "MinGW Makefiles" -DCMAKE_CXX_COMPILER=gcc
         ```

      - Linux

         ```bash
         cmake -S . -B ./build -G "Ninja" -DCMAKE_CXX_COMPILER=gcc
         ```

      You can use other generators like Ninja, but it may not work on windows, because `windres.exe` can not process spaces in including directories correctly, which will cause compilation errors. **This is a bug in GNU bintuils, and Ninja have no way to avoid this error, but mingw makefiles does.** On other platforms, `windres` is no longer used, so the bug is also avoided.

3. Build all targets

      ```bash
      cmake --build . --parallel
      ```

4. Install

      Run cmake --install to install all binaries and block files. By default the install directory is `${CMAKE_BINARY_DIR}/install`, where you can find all executables.

5. Deploy

      You may need to deploy shared libs for executables. For windows, run `windeployqt SlopeCraft.exe` in command line. There is no need to deploy shared objects on Linux, users should install Qt, libzip and libpng via package manager.

      **Now `windeployqt` and `macdeployqt` will be executed automatically during installation. So this step can be skipped.**

## Notice

1. If you meet any problem, draw me a new issue.
2. If you are building VisualCraftL on **linux**, some linking problems around libzip may occur. To fix this problem, there are 2 ways possible:
      1. Use a **static archive of libzip** instead of shared lib. But this libzip.a should be compiled with argument **`-fPIC`**, because `libVisualCraftL.so`, a shared lib will link to it. 
         - If your libzip.a links other compress libs(for example, lzma), linking may fail. That is because libzip links to its dependents **privately regardless of whether libzip is built to be a shared lib**. I guess that this is a designing mistake in libzip. So it is **deprecated** to use a static libzip.
      2. Use shared libzip, and copy `libzip.so` and symlinks against it to `CMAKE_CURRENT_BINARY_DIR`. This is what I'm doing now.

