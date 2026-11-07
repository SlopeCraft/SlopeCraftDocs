# Compilation Guide on Windows and Linux

This document tells you how to build SlopeCraft on Windows and Linux. Clang (LLVM 20 or later) is strongly recommended; SlopeCraft does not guarantee that MSVC, GCC or a Clang < 20 can build it successfully (although logically they should).

## Steps

### Install Dependencies

#### Windows

Install the dependencies manually, or build them yourself.

Or use vcpkg:

```bash
vcpkg install --triplet=x64-windows boost-uuid boost-multi-array zlib libpng bzip2  liblzma libzip eigen3 xsimd magic-enum gapp 
# Install the rust toolchain (a separately distributed rust toolchain also works)
rustup toolchain install stable
# If you want GPU acceleration for VisualCraft, the Vulkan components are required
vcpkg install --triplet=x64-windows glslang[tools] vulkan-headers vulkan-memory-allocator vulkan-loader
```
Besides that, building on Windows also requires Qt. You can use the official Qt installer, or build and install Qt yourself (with MSVC). Qt 6.5 is the minimum; 6.12 LTS is recommended.

#### Linux (Ubuntu)

```bash
# Install the build tools and the dependencies such as Eigen, libzip and boost
sudo apt install cmake ninja-build clang-20 libomp-20-dev xz-utils bzip2 libzip-dev zipcmp ziptool zipmerge libpng-dev libeigen3-dev libexpected-dev p7zip-full libboost-all-dev libboost-tools-dev libboost-doc
# Install the rust toolchain (a rust toolchain installed by the package manager also works)
rustup toolchain install stable
# Install Qt6 on Ubuntu. Other Linux distributions are similar
sudo apt install libqt6widgets6 libqt6gui6 libqt6network6 qt6-base-dev qt6-tools-dev-tools qt6-tools-dev qt6-l10n-tools x11-utils libxcb-xinerama0 libxv1 libgl-dev
# Dependencies of the Vulkan SDK
sudo apt-get install libglm-dev libxcb-dri3-0 libxcb-present0 libpciaccess0 libxcb-keysyms1-dev libxcb-dri3-dev libx11-dev libwayland-dev libxrandr-dev libxcb-randr0-dev libxcb-ewmh-dev python-is-python3 bison libx11-xcb-dev liblz4-dev libzstd-dev ocaml-core pkg-config libxml2-dev wayland-protocols python3-jsonschema clang-format qtbase5-dev
# Download and extract the Vulkan SDK
wget https://sdk.lunarg.com/sdk/download/1.4.363.0/linux/vulkansdk-linux-x86_64-1.4.363.0.tar.xz
echo 197962f5cbf80baf2775a03336a01cee7c8745686c65aaa70d3f751ade4d7e43 vulkansdk-linux-x86_64-1.4.363.0.tar.xz > vulkanSDK.sha256
sha256sum --check vulkanSDK.sha256
tar xf vulkansdk-linux-x86_64-1.4.363.0.tar.xz
```

### Build SlopeCraft

1. Clone the SlopeCraft repository and enter the repository directory

      ```bash
      git clone https://github.com/SlopeCraft/SlopeCraft.git
      ```

2. Configure CMake

      Run CMake to configure this project. Command line examples:

      - Windows

         ```bash
         .../vcvars64.bat
         cmake -S . -B ./build -G Ninja -DCMAKE_C_COMPILER=clang-cl -DCMAKE_CXX_COMPILER=clang-cl -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=install -DSlopeCraft_GPU_API=Vulkan -DCMAKE_PREFIX_PATH=...
         ```

         - When compiling with clang-cl, you should first call vcvars64.bat to set the environment variables required by MSVC (clang-cl mimics the MSVC compiler). This file can be found in the Visual Studio installation directory. Its default location is probably `C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat`
         - You should put the installation directories of Qt, vcpkg, vulkan and rust into `CMAKE_PREFIX_PATH`.
         - If the installation directory of vulkan or rust has already been added to the `PATH` environment variable, it does not need to be added to `CMAKE_PREFIX_PATH`. If there are multiple paths, separate them with a semicolon `;`.

      - Linux

         ```bash
         cmake -S . -B ./build -G Ninja -DCMAKE_C_COMPILER=clang-20 -DCMAKE_CXX_COMPILER=clang++-20 -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=install  -DSlopeCraft_GPU_API=Vulkan -DCMAKE_PREFIX_PATH=...
         ```
         - If Vulkan is enabled, you should put the installation directory of vulkan into `CMAKE_PREFIX_PATH`.
         - If the default Clang of your system is newer than 20, there is no need to specify the Clang version.
         - The Vulkan SDK downloaded from apt on Ubuntu 24 is older than 1.3, SlopeCraft **cannot** use it and will run into a lot of compilation errors.

3. Build all targets

      ```bash
      cmake --build ./build --parallel
      ```

4. Install

      Run `cmake --install` to install all binaries and block files. By default the install directory is `${CMAKE_BINARY_DIR}/install`, where you can find all executables.

5. Package

      Run `cpack -G <package type>` to install again and create a package. The supported package types are currently `7Z`, `ZIP`, `TXZ` and `DEB`.

## Notice

1. If you meet any problem, draw us a new issue.
2. If you are building VisualCraftL on **linux**, some linking problems around libzip may occur. To fix this problem, there are 2 ways possible:
      1. Use a **static archive of libzip** instead of shared lib. But this libzip.a should be compiled with argument **`-fPIC`**, because `libVisualCraftL.so`, a shared lib will link to it.
         - If your libzip.a links other compress libs(for example, lzma), linking may fail. That is because libzip links to its dependents **privately regardless of whether libzip is built to be a shared lib**. I guess that this is a designing mistake in libzip. So it is **deprecated** to use a static libzip.
      2. Use shared libzip, and copy `libzip.so` and symlinks against it to `CMAKE_CURRENT_BINARY_DIR`. This is what I'm doing now.
