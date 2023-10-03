# macOS 编译指南

本文介绍了如何在 macOS 上编译 SlopeCraft。

!!! tip "提示"
    本文内容会尽量确保大部分用户都可以顺利理解其内容，所以可能会包含一些较为基础的内容（例如如何安装 GCC），如果你发现你已经熟悉了你正在看的内容，你可以酌情跳过这部分内容。

## 前置步骤

本文内容涉及使用终端命令行进行操作，我们建议各位使用 iTerm 2 作为终端模拟器并在开始之前熟悉命令行基本操作。

## 编译步骤

### 安装依赖

首先，请确保你已经安装了 [Homebrew](https://brew.sh/)（我们将会使用 Homebrew 安装依赖）。如果你还没有安装 Homebrew，请在终端中输入以下命令：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

然后，我们需要安装一些 SlopeCraft 编译脚本不会自动安装的依赖：

1. GCC 编译器

    ```bash
    brew install GCC
    ```

    在完成后，你可以使用 `gcc-12 --version` 来检查 gcc 编译器是否已经安装成功。如果一切正常，你应该会看到与下面类似的输出：

    ```bash
    COLLECT_GCC=gcc-12
    COLLECT_LTO_WRAPPER=/usr/local/Cellar/gcc/12.2.0/bin/../libexec/gcc/x86_64-apple-darwin22/12/lto-wrapper
    Target: x86_64-apple-darwin22
    Configured with: ../configure --prefix=/usr/local/opt/gcc --libdir=/usr/local/opt/gcc/lib/gcc/current --disable-nls --enable-checking=release --with-gcc-major-version-only --enable-languages=c,c++,objc,obj-c++,fortran --program-suffix=-12 --with-gmp=/usr/local/opt/gmp --with-mpfr=/usr/local/opt/mpfr --with-mpc=/usr/local/opt/libmpc --with-isl=/usr/local/opt/isl --with-zstd=/usr/local/opt/zstd --with-pkgversion='Homebrew GCC 12.2.0' --with-bugurl=https://github.com/Homebrew/homebrew-core/issues --with-system-zlib --build=x86_64-apple-darwin22 --with-sysroot=/Library/Developer/CommandLineTools/SDKs/MacOSX13.sdk
    Thread model: posix
    Supported LTO compression algorithms: zlib zstd
    gcc version 12.2.0 (Homebrew GCC 12.2.0)
    ```

    如果你注意到了，在输出中的第二行中，`COLLECT_LTO_WRAPPER=` 后面有一串路径，我们只需要注意该路径的最前面一部分，即 `/usr/local/Cellar/gcc/12.2.0/bin`（根据 brew 安装的 GCC 版本不同，该路径可能会有所不同）。请记住该路径，这个是我们的 GCC 编译器路径。

2. Ninja

    我们需要使用 Ninja 作为 CMake 的生成器，因此我们需要安装 Ninja：

    ```bash
    brew install ninja
    ```

    使用 `ninja --version` 来检查 Ninja 是否已经安装成功。如果一切正常，你应该会看到与下面类似的输出：

    ```bash
    1.11.1
    ```

3. Qt

    ```bash
    brew install qt
    ```

    在安装完成后，你可以通过检查 `/usr/local/Cellar/qt` 路径来检查 Qt 是否已经安装成功。如果一切正常，你应该会看到一个命名类似于 `6.y.z_1` 的文件夹（例如 `6.4.3_1`）。你只需要确保该文件夹的第一个数字是 `6`，且 `y` 大于等于 `4` 即可。

4. zlib & libpng & libzip

    ```bash
    brew install zlib libpng libzip
    ```

    在安装完成后，如果以下路径均存在，则表示安装成功：

    - `/usr/local/Cellar/zlib`
    - `/usr/local/Cellar/libpng`
    - `/usr/local/Cellar/libzip`

### 编译 SlopeCraft

1. 克隆 SlopeCraft 仓库并进入仓库目录

    ```bash
    git clone https://github.com/SlopeCraft/SlopeCraft.git && cd SlopeCraft
    ```

2. 配置 CMake

    使用以下命令配置 CMake：

    ```bash
    cmake -S . -B ./build -G "Ninja" -DCMAKE_C_COMPILER=/usr/local/Cellar/gcc/12.2.0/bin/gcc-12 -DCMAKE_CXX_COMPILER=/usr/local/Cellar/gcc/12.2.0/bin/g++-12 -DCMAKE_INSTALL_PREFIX=./build/install
    ```


    在上面的命令中，我们指定了几个参数：

    - `-DCMAKE_C_COMPILER` 指定 C 编译器路径，你可能需要将 `=` 后面的路径参数替换为你自己的路径。C 编译器为我们在安装 GCC 时找到的路径（在本文中为 `/usr/local/Cellar/gcc/12.2.0/bin`）后面加上 `/gcc-12`
    - `-DCMAKE_CXX_COMPILER` 指定 C++ 编译器路径，你可能需要将 `=` 后面的路径参数替换为你自己的路径。C++ 编译器为我们在安装 GCC 时找到的路径（在本文中为 `/usr/local/Cellar/gcc/12.2.0/bin`）后面加上 `/g++-12`
    - `-DCMAKE_INSTALL_PREFIX` 指定安装路径，本指南中我们将安装路径设置为 `./build/install`，你可以根据自己的需要进行修改

    虽然你可以通过`SlopeCraft_GPU_API`和`SlopeCraft_vectorize`指定显卡加速和向量化，但目前 SlopeCraft 还只支持 OpenCL 和 AVX 的加速手段，而它们在 mac 上都不能完善使用（OpenCL 在 macOS 下的支持有一些问题，M1 和 M2 芯片的 MacOS 设备不能翻译 AVX 指令），所以非常不建议修改这两个选项，目前默认关闭显卡加速和向量化，打开也没有作用。


3. 切换到 build 目录并编译 SlopeCraft

    使用以下命令切换到 `build` 目录并编译 SlopeCraft：

    ```bash
    cd build
    cmake --build . --parallel
    ```

4. 安装

    使用以下命令安装 SlopeCraft：

    ```bash
    cmake --install .
    ```

5. 部署 Qt 可执行文件 **\[可选步骤\]**

    首先，切换到 `install` 目录：

    ```bash
    cd install
    ```

    你应该可以找到 `SlopeCraft.app`、`VisualCraft.app`、 `MapViewer.app` 和 `imageCutter.app` 三个可执行 app 文件。我们需要进行 Qt 部署以使这些可执行文件能够正常运行。使用以下命令进行 Qt 部署：

    ```bash
    macdeployqt *.app
    ```

    双击 `SlopeCraft.app` 或者其他的可执行 app 文件，如果一切正常启动，那么恭喜你，你已经成功地在 macOS 上编译了 SlopeCraft！你也可以将 app 文件拖动到应用程序文件夹中完成安装。

    **这一步不是必需的，如果 macdeployqt 后程序反而不能运行（通常是 brew 提供的 Qt 有 bug），那么建议你删除 install 文件夹，重新执行上一步（安装），然后跳过这一步。这样将直接使用 brew 安装的 Qt 的 dylib，其他电脑如果使用这样的发行版，也需要安装 brew 和 Qt。**

### 清除已有编译文件

如果你希望完全重新编译，你可以通过使用一下命令删除已有的编译文件：

```bash
rm -rf 3rdParty binaries build
```
