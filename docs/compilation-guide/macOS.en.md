# Compilation Guide on macOS

This document tells you how to build SlopeCraft on macOS.

!!! tip "Tip"
    This document tries to make sure that most users can follow it, so it may contain some basic content (for example, how to install Clang). If you already know the part you are reading, feel free to skip it.
    Since the lead developer ToKiNoBug does not have a macOS device, this document is mainly written based on CI of SlopeCraft, so it's not the only way. If you are familiar with development on macOS, you are welcome to build SlopeCraft however you like.

## Prerequisites

This document involves command line operations. We recommend using iTerm 2 as your terminal emulator and getting familiar with basic command line usage before you start.

## Steps

### Install dependencies

First, make sure that [Homebrew](https://brew.sh/) is installed (we will use Homebrew to install the dependencies). If you have not installed Homebrew yet, run the following command in the terminal:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Then we need to install the dependencies that SlopeCraft's build scripts will not install automatically:

1. Apple clang

    The default C/C++ compiler on macOS is Apple clang, which ships together with the Xcode Command Line Tools and has no separate installer, so installing the command line tools is enough. If it is not installed yet, run the following command in the terminal:

    ```bash
    xcode-select --install
    ```

    A window will then pop up: click "Install" and wait for the download and installation to finish. If you already have the full Xcode installed, the command line tools are installed together with Xcode and you can skip this step; if the system reports `xcode-select: error: command line tools are already installed`, that means it is already installed too.

    It is recommended to check the compiler version with the following command. The version should preferably be at least 21, otherwise it is hard to guarantee good C++23 support. Theoritically you can also use the LLVM clang from Homebrew, but it may run into OpenMP problems as well.
    ```bash
    /usr/bin/clang++ --version
    ```

    If everything is fine, you should see output similar to the following (the exact version numbers change with your Xcode / command line tools version):
    ```bash
    Apple clang version 21.0.0 (clang-2100.1.1.101)
    Target: x86_64-apple-darwin25.6.0
    Thread model: posix
    InstalledDir: /Applications/Xcode_26.6.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
    ```
    Note that Apple clang does not ship an OpenMP runtime, so `libomp` must be installed in step 4.

2. Ninja and CMake

    SlopeCraft is built with CMake, and Ninja is the recommended CMake generator, so we need to install CMake and Ninja:

    ```bash
    brew install cmake ninja
    ```

    Use `ninja --version` to check whether Ninja has been installed successfully. If everything is fine, you should see output similar to the following:

    ```bash
    1.11.1
    ```

3. Rust toolchain

    SlopeCraft uses mc_schem, a schematic library written in Rust, so a Rust toolchain is required. We recommend installing and managing Rust with rustup; run the following command in the terminal:

    ```bash
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    ```

    This script downloads rustup and lists several installation options; just press Enter to choose the default `1) Proceed with standard installation`. When it is done, rustup installs `cargo`, `rustc`, `rustup` and the other commands into `~/.cargo/bin` and adds that directory to the shell's `PATH`, **so you need to open a new terminal (or run `source "$HOME/.cargo/env"`) before you can use these commands.**

    Then run the following commands to confirm that the toolchain is installed and that `stable` is the default toolchain:

    ```bash
    rustup default stable
    rustc --version
    cargo --version
    ```

    If everything is fine, both `rustc` and `cargo` will print their versions. If you prefer to manage Rust with Homebrew, you can also run `brew install rustup` and then `rustup-init`; the result is the same.

4. Third-party dependencies

    ```bash
    brew install libzip libpng xz bzip2 boost magic_enum xsimd eigen sevenzip libomp
    ```

5. Qt

    Install the shared version of Qt 6:
    ```bash
    brew install qt
    ```

    You can also use a statically linked Qt, but that usually means building Qt yourself. Building Qt is a rather involved process; here is a sample script:
    ```bash
    git clone https://github.com/qt/qtbase.git --recursive
    cd qtbase 
    git checkout v6.12.0
    cd ..

    git clone https://github.com/qt/qttools.git --recursive
    cd qttools
    git checkout v6.12.0
    cd ..
    
    mkdir build install
    cd build

    # Configure, build and install qtbase
    ../qtbase/configure --prefix=../install -static -release -nomake tests -skip qtdoc
    cmake --build . --parallel
    cmake --install .
    rm -rf ./*

    # Configure, build and install qttools
    ../install/bin/qt-configure-module ../qttools
    cmake --build . --parallel
    cmake --install .
    rm -rf ./*
    ```
    The script above can be run in any directory (as long as there are no permission problems); the static Qt 6.12.0 libraries will be installed into the `install` subdirectory of the current directory.

### Build SlopeCraft

1. Clone the SlopeCraft repository and enter the repository directory

    ```bash
    git clone https://github.com/SlopeCraft/SlopeCraft.git && cd SlopeCraft
    ```

2. Configure CMake

    Configure CMake with the following command:

    ```bash
    LIBOMP_PREFIX="$(brew --prefix libomp)"
    cmake -S . -B ./build -G Ninja \
        -DCMAKE_C_COMPILER=/usr/bin/clang \
        -DCMAKE_CXX_COMPILER=/usr/bin/clang++ \
        -DCMAKE_INSTALL_PREFIX=./build/install \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_OSX_SYSROOT="$(xcrun --sdk macosx --show-sdk-path)" \
        -DCMAKE_OSX_DEPLOYMENT_TARGET=15.0 \
        -DSlopeCraft_GPU_API:STRING=None \
        -DMC_SCHEM_HAS_mdspan=OFF \
        -DCMAKE_PREFIX_PATH="$LIBOMP_PREFIX;/usr/local"
    ```


    In the command above we specify several parameters:

    - `CMAKE_C_COMPILER` specifies the C compiler; it should be `/usr/bin/clang`, because Apple clang is currently the primary target
    - `CMAKE_CXX_COMPILER` specifies the path of the C++ compiler; it should be `/usr/bin/clang++`. In theory both compilers can be switched to LLVM Clang, but they must always match — never mix compilers from different sources
    - `CMAKE_INSTALL_PREFIX` specifies the installation path; this guide sets it to `./build/install`, you can change it to whatever suits you
    - `CMAKE_BUILD_TYPE` specifies the build type, such as Debug or Release
    - `CMAKE_OSX_DEPLOYMENT_TARGET` specifies the minimum macOS version. It is recommended to set:
      - `13.3` for Apple Silicon devices
      - `15.0` for Intel devices
    - `CMAKE_PREFIX_PATH` tells CMake where to find packages. It can contain multiple paths separated by `;`.

    **If you want to use a statically built Qt, you must add the Qt installation directory to `CMAKE_PREFIX_PATH`; otherwise CMake will not find Qt, or will find another Qt.**

    In addition, you can use `SlopeCraft_vectorize` to enable or disable vectorization. If it is `ON`, AVX and AVX2 are used on Intel chips and Neon is used on Apple Silicon chips. **Vectorization is enabled by default.**

    Although `SlopeCraft_GPU_API` lets you choose the GPU acceleration API, SlopeCraft currently only supports Vulkan for GPU acceleration, and Vulkan has no official support on macOS (yes, Apple really is that self-absorbed — if it can be incompatible, it will be). Community Vulkan implementations have not been tested yet.


3. Change into the build directory and build SlopeCraft

    Use the following commands to change into `build` and build SlopeCraft:

    ```bash
    cd build
    cmake --build . --parallel
    ```

4. Install

    Use the following command to install SlopeCraft:

    ```bash
    cmake --install .
    ```

### Clean up existing build files

If you want to rebuild from scratch, you can delete the existing build files with the following command:

```bash
rm -rf 3rdParty binaries build
```
