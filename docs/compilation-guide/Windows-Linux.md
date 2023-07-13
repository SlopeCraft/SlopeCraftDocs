# Windows 与 Linux 编译指南

本文介绍了如何在 Windows 与 Linux 上编译 SlopeCraft。

## 编译步骤

### 安装依赖

#### Windows:

手动安装，或者自己编译

或者使用 vcpkg：
```bash
# 使用 msvc 或 clang-cl 编译
vcpkg install --triplet=x64-windows zlib libzip eigen3 libpng

# 使用 gcc 编译
vcpkg install --triplet=x64-mingw-dynamic zlib libzip eigen3 libpng
```

#### Linux (Ubuntu):
```bash
#安装libzip， libpng， Eigen3
sudo apt install libzip-dev zipcmp ziptool zipmerge libpng-dev libeigen3-dev

# 安装 Qt6.2.4，ubuntu。其他Linux发行版同理
sudo apt install libqt6widgets6 libqt6gui6 libqt6network6 libqt6concurrent6 qt6-base-dev qt6-tools-dev-tools qt6-tools-dev qt6-l10n-tools
sudo apt install x11-utils libxcb-xinerama0 libxv1 libgl-dev
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
         # 使用 clang-cl 编译
         .../vcvars64.bat
         cmake -S . -B ./build -G Ninja -DCMAKE_CXX_COMPILER=clang-cl -DCMAKE_PREFIX_PATH=...
         # 使用 gcc 编译
         cmake -S . -B ./build -G "MinGW Makefiles" -DCMAKE_CXX_COMPILER=gcc -DCMAKE_PREFIX_PATH=...
         ```
         使用 clang-cl 编译时，应当先调用 vcvars64.bat 设置 msvc 所需的环境变量（clang-cl 是模仿 msvc 编译器的）。这个文件在可以在 VS 的安装目录里找到。默认的位置可能是*C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat*
         
         应当把 Qt、zlib、libpng 和 libzip 的安装目录输入到 CMAKE_PREFIX_PATH 中。如果有多个路径，用半角分号`;`连接。
      - Linux

         ```bash
         cmake -S . -B ./build -G "Ninja" -DCMAKE_CXX_COMPILER=gcc
         ```

3. 构建所有目标

      ```bash
      cmake --build . --parallel
      ```

4. 安装，并删除不必要的文件

      运行 `cmake --install` 来安装所有二进制文件和方块文件。默认情况下，安装目录是 `${CMAKE_BINARY_DIR}/install`，你可以在那里找到所有可执行文件。

5. 部署 Qt

      你可能需要为可执行文件部署 Qt 动态库。对于 Windows，在命令行中运行 `windeployqt SlopeCraft.exe`。Linux 平台上不需要部署，用户应当通过包管理器安装 Qt、libpng 和 libzip。

**现在安装时已经可以自动执行 `windeployqt` 或 `macdeployqt`，因此这一步可以省略。**