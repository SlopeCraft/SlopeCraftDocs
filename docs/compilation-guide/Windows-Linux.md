# Windows 与 Linux 编译指南

本文介绍了如何在 Windows 与 Linux 上编译 SlopeCraft。非常推荐使用Clang（LLVM 20及以上）编译，SlopeCraft不保证MSVC、GCC或者版本低于20的Clang能够成功编译（虽然理论上应该可以）。

## 编译步骤

### 安装依赖

#### Windows

手动安装，或者自己编译

或者使用 vcpkg：

```bash
vcpkg install --triplet=x64-windows boost-uuid boost-multi-array zlib libpng bzip2  liblzma libzip eigen3 xsimd magic-enum gapp 
# 安装rust toolchain （单独分发安装的rust toolchain也可以）
rustup toolchain install stable
# 如果需要为VisualCraft启用显卡加速，需安装Vulkan相关
vcpkg install --triplet=x64-windows glslang[tools] vulkan-headers vulkan-memory-allocator vulkan-loader
```
除此以外，windows编译还需要安装Qt。可以使用Qt官方的安装器，也可以自行编译安装（使用MSVC）。至少需要Qt6.5，推荐安装6.12LTS。

#### Linux (Ubuntu)

```bash
# 安装编译工具、Eigen、libzip、boost等依赖
sudo apt install cmake ninja-build clang-20 libomp-20-dev xz-utils bzip2 libzip-dev zipcmp ziptool zipmerge libpng-dev libeigen3-dev libexpected-dev p7zip-full libboost-all-dev libboost-tools-dev libboost-doc
# 安装rust toolchain （包管理器安装的rust toolchain也可以）
rustup toolchain install stable
# 安装 Qt6，ubuntu。其他Linux发行版同理
sudo apt install libqt6widgets6 libqt6gui6 libqt6network6 qt6-base-dev qt6-tools-dev-tools qt6-tools-dev qt6-l10n-tools x11-utils libxcb-xinerama0 libxv1 libgl-dev
# Vulkan SDK的依赖
sudo apt-get install libglm-dev libxcb-dri3-0 libxcb-present0 libpciaccess0 libxcb-keysyms1-dev libxcb-dri3-dev libx11-dev libwayland-dev libxrandr-dev libxcb-randr0-dev libxcb-ewmh-dev python-is-python3 bison libx11-xcb-dev liblz4-dev libzstd-dev ocaml-core pkg-config libxml2-dev wayland-protocols python3-jsonschema clang-format qtbase5-dev
# 下载解压 vulkan SDK
wget https://sdk.lunarg.com/sdk/download/1.4.363.0/linux/vulkansdk-linux-x86_64-1.4.363.0.tar.xz
echo 197962f5cbf80baf2775a03336a01cee7c8745686c65aaa70d3f751ade4d7e43 vulkansdk-linux-x86_64-1.4.363.0.tar.xz > vulkanSDK.sha256
sha256sum --check vulkanSDK.sha256
tar xf vulkansdk-linux-x86_64-1.4.363.0.tar.xz
```

### 编译 SlopeCraft

1. 克隆 SlopeCraft 仓库并进入仓库目录

      ```bash
      git clone https://github.com/SlopeCraft/SlopeCraft.git
      ```

2. 配置 CMake

      运行 CMake 配置这个项目。命令行示例：

      - Windows

         ```bash
         .../vcvars64.bat
         cmake -S . -B ./build -G Ninja -DCMAKE_C_COMPILER=clang-cl -DCMAKE_CXX_COMPILER=clang-cl -DMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=install -DSlopeCraft_GPU_API=Vulkan -DCMAKE_PREFIX_PATH=...
         ```

         - 使用 clang-cl 编译时，应当先调用 vcvars64.bat 设置 msvc 所需的环境变量（clang-cl 是模仿 msvc 编译器的）。这个文件在可以在 VS 的安装目录里找到。默认的位置可能是`C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat`
         - 应当把 Qt、vcpkg、vulkan、rust 的安装目录输入到 `CMAKE_PREFIX_PATH` 中。
         - vulkan 或者 rust 的安装目录如果已经加入环境变量 `PATH` ，则无需加入 `CMAKE_PREFIX_PATH` 。如果有多个路径，用半角分号`;`连接。

      - Linux

         ```bash
         cmake -S . -B ./build -G Ninja -DCMAKE_C_COMPILER=clang-20 -DCMAKE_CXX_COMPILER=clang++-20 -DMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=install  -DSlopeCraft_GPU_API=Vulkan -DCMAKE_PREFIX_PATH=...
         ```
         - 如果启用 Vulkan，应当把 vulkan 的安装目录输入到 `CMAKE_PREFIX_PATH` 中。
         - 如果系统默认的 Clang 版本高于20，则无需指定 Clang 版本。
         - Ubuntu24 从 apt 下载的 vulkan SDK 版本低于1.3，SlopeCraft **无法**使用它们，会遭遇很多编译错误。

3. 构建所有目标

      ```bash
      cmake --build ./build --parallel
      ```

4. 安装

      运行 `cmake --install` 来安装所有二进制文件和方块文件。默认情况下，安装目录是 `${CMAKE_BINARY_DIR}/install`，你可以在那里找到所有可执行文件。

5. 打包
      运行 `cpack -G <包类型>` 可以再次安装并制作软件包。目前支持的软件包类型：`7Z`、`ZIP`、`TXZ`、`DEB`
