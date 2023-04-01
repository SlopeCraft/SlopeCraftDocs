# Windows 与 Linux 编译指南

本文介绍了如何在 Windows 与 Linux 上编译 SlopeCraft。

## 编译步骤

### 安装依赖

在你的电脑上安装 Qt、libzip 和 libpng

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

   运行 `cmake --install` 来安装所有二进制文件和方块文件。默认情况下，安装目录是 `${CMAKE_BINARY_DIR}/install`，你可以在那里找到所有可执行文件。可能还有一个文件夹叫 `please_delete_this_folder`，把它删除。

5. 部署 Qt

   你可能需要为可执行文件部署 Qt 动态库。对于 Windows，在命令行中运行 `windeployqt SlopeCraft.exe`。在其他平台上也有类似的部署命令，比如 Linux 上的 `linuxdeployqt`。

## 注意事项

1. 如果你在**linux**上构建 VisualCraftL，可能会出现一些围绕 libzip 的链接问题。为了解决这个问题，有 2 种可能的方法。

    1. 使用 libzip 的**静态库，而不是动态库**。但是这个 libzip.a 应该用参数 **`-fPIC`** 编译，因为 `libVisualCraftL.so` 这个共享库会链接到它。
        - 如果你的 libzip.a 链接了其他压缩库（例如 lzma），链接可能会失败。这是因为 libzip 链接到它的依赖项时是 PRIVATE 链接，不管 libzip 是不是静态库。我猜这是 libzip 的一个设计错误。所以使用静态的 libzip 是不可取的。
    2. 使用动态的 libzip，并将 `libzip.so` 和指向它的符号链接复制到 `CMAKE_CURRENT_BINARY_DIR`。这就是我现在的方案。

2. 如果你用 OpenCL 作为 GPU API 构建 SlopeCraft，**你的第一次编译预计会遇到链接失败**。这是因为 OpenCL 的 C 源代码是作为资源文件嵌入的（不是 Windows .rc），所以引入了一个基于 cmake 的第三方 [资源文件实现（ResourceCreator.cmake）](https://github.com/isRyven/ResourceCreator.cmake.git)。它于 Qt 有点犯冲，因为 "ResourceCreator.cmake" 在 cmake 构建时生成了 C 代码（**__rsc_ColorManip_cl_rc.c**），然而 Qt 在 cmake 配置时要求所有源文件都存在。现在，C 源文件被 **touch** 生成了，但一旦文件存在，正确的代码就不会生成。目前还没有找到一个完美的解决方案，所以请通过以下步骤解决这个问题。

    1. 构建，链接器将报错，报错信息可能是 *undefined reference to `ColorManip_cl_rc_length` or `ColorManip_cl_rc`*。
    2. 删除 `${CMAKE_BINARY_DIR}/utilities/GPUWrapper/OpenCL/__rsc_ColorManip_cl_rc.c`。你删除的是一个由 touch 生成的空文件，构建时将会生成正确的文件。
    3. 再次构建。**不要再运行 cmake configure** ！此时的编译应该会成功。
