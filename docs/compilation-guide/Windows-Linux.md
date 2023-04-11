# Windows 与 Linux 编译指南

本文介绍了如何在 Windows 与 Linux 上编译 SlopeCraft。

## 编译步骤

### 安装依赖

#### Windows:

手动安装，或者自己编译

#### Linux (Ubuntu):
```bash
#安装libzip， libpng， Eigen3
sudo apt install libzip-dev zipcmp ziptool zipmerge libpng-dev libeigen3-dev

# 安装 Qt6.2.4
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
         cmake -S . -B ./build -G "MinGW Makefiles" -DCMAKE_CXX_COMPILER=gcc
         ```
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

## 注意事项

1. 如果你在**linux**上构建 VisualCraftL，可能会出现一些围绕 libzip 的链接问题。为了解决这个问题，有 2 种可能的方法。

    1. 使用 libzip 的**静态库，而不是动态库**。但是这个 libzip.a 应该用参数 **`-fPIC`** 编译，因为 `libVisualCraftL.so` 这个共享库会链接到它。
        - 如果你的 libzip.a 链接了其他压缩库（例如 lzma），链接可能会失败。这是因为 libzip 链接到它的依赖项时是 PRIVATE 链接，不管 libzip 是不是静态库。我猜这是 libzip 的一个设计错误。所以使用静态的 libzip 是不可取的。
    2. 使用动态的 libzip，并将 `libzip.so` 和指向它的符号链接复制到 `CMAKE_CURRENT_BINARY_DIR`。这就是我现在的方案。