# macOS 编译指南

本文介绍了如何在 macOS 上编译 SlopeCraft。

!!! tip "提示"
    本文内容会尽量确保大部分用户都可以顺利理解其内容，所以可能会包含一些较为基础的内容（例如如何安装 Clang），如果你发现你已经熟悉了你正在看的内容，你可以酌情跳过这部分内容。
    由于核心开发者ToKiNoBug没有macOS设备，本文主要参照 SlopeCraft 在github actions中成功跑通的编译脚本编写，并不是绝对唯一的指南。如果你熟稔macOS软件开发，欢迎按照自己喜欢的方式编译。

## 前置步骤

本文内容涉及使用终端命令行进行操作，我们建议各位使用 iTerm 2 作为终端模拟器并在开始之前熟悉命令行基本操作。

## 编译步骤

### 安装依赖

首先，请确保你已经安装了 [Homebrew](https://brew.sh/)（我们将会使用 Homebrew 安装依赖）。如果你还没有安装 Homebrew，请在终端中输入以下命令：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

然后，我们需要安装一些 SlopeCraft 编译脚本不会自动安装的依赖：

1. Apple clang编译器

    macOS 上默认的 C/C++ 编译器是随 Xcode 命令行工具（Command Line Tools）一起分发的 Apple clang，它没有单独的安装包，安装命令行工具即可。如果你还没有安装，请在终端中执行：

    ```bash
    xcode-select --install
    ```

    随后会弹出一个窗口，点击“安装”，等待下载和安装完成即可。如果你已经安装了完整的 Xcode，那么命令行工具会随 Xcode 一并安装，这一步可以跳过；如果系统提示 `xcode-select: error: command line tools are already installed`，也说明它已经安装好了。

    建议执行以下命令检查编译器的版本，版本最好不低于21，否则很难保证它完整的支持c++23。理论上可以用homebrew的LLVM clang，但可能也会撞在openMP上。如果你熟悉macOS环境不妨试试。
    ```bash
    /usr/bin/clang++ --version
    ```

    如果一切正常，你应该会看到与下面类似的输出（具体的版本号会随 Xcode / 命令行工具的版本而变化）：
    ```bash
    Apple clang version 21.0.0 (clang-2100.1.1.101)
    Target: x86_64-apple-darwin25.6.0
    Thread model: posix
    InstalledDir: /Applications/Xcode_26.6.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
    ```
    值得注意的是，Apple clang不自带OpenMP，所以第4步必须安装`libomp`。

2. Ninja 和 CMake

    SlopeCraft 使用 CMake 构建，推荐用 Ninja 作为 CMake 的生成器，因此我们需要安装 CMake 和 Ninja：

    ```bash
    brew install cmake ninja
    ```

3. Rust工具链
   
   SlopeCraft采用的原理图库mc_schem由rust编写，需要安装rust工具链。我们推荐使用 rustup 来安装和管理 rust，请在终端中执行：

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

   这个脚本会下载 rustup 并列出几个安装选项，直接按回车选择默认的 `1) Proceed with standard installation` 即可。安装完成后，rustup 会把 `cargo`、`rustc`、`rustup` 等命令安装到 `~/.cargo/bin`，并把该目录写入 shell 的 `PATH` 环境变量，**因此你需要重新打开一个终端（或者执行 `source "$HOME/.cargo/env"`）才能使用这些命令。**

   然后执行以下命令，确认工具链已经安装成功，并确保默认使用的是 stable 工具链：

   ```bash
   rustup default stable
   rustc --version
   cargo --version
   ```

   如果一切正常，`rustc` 和 `cargo` 都会输出各自的版本号。如果你更希望用 Homebrew 管理，也可以执行 `brew install rustup` 安装 rustup，再运行 `rustup-init`，效果是一样的。

4. 第三方依赖

    ```bash
    brew install libzip libpng xz bzip2 boost magic_enum xsimd eigen sevenzip libomp
    ```

5. Qt

    安装动态版本的Qt6：
    ```bash
    brew install qt
    ```

    你也可以使用静态链接的 Qt，但这通常需要自行编译。编译 Qt 是个比较复杂的步骤，以下给出一个样板代码：
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
    以上代码可以在任何目录中执行（只要没有权限问题），静态库的 Qt6.12.0 将被安装在 当前目录的 install 子目录中。

### 编译 SlopeCraft

1. 克隆 SlopeCraft 仓库并进入仓库目录

    ```bash
    git clone https://github.com/SlopeCraft/SlopeCraft.git && cd SlopeCraft
    ```

2. 配置 CMake

    使用以下命令配置 CMake：

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


    在上面的命令中，我们指定了几个参数：

    - `CMAKE_C_COMPILER` 指定 C 编译器，应当为`/usr/bin/clang`，目前优先适配Apple clang
    - `CMAKE_CXX_COMPILER` 指定 C++ 编译器路径，应当为`/usr/bin/clang++`。理论上两个编译器可以改用LLVM Clang，但总之必须一致，绝不能混用
    - `CMAKE_INSTALL_PREFIX` 指定安装路径，本指南中我们将安装路径设置为 `./build/install`，你可以根据自己的需要进行修改
    - `CMAKE_BUILD_TYPE` 指定编译类型，如 Debug 或 Release
    - `CMAKE_OSX_DEPLOYMENT_TARGET` 指定最低的 macOS 版本，建议：
      - 对于 Apple Silicon 设备，设置为 `13.3` 
      - 对于 Intel 设备，设置为 `15.0` 
    - `CMAKE_PREFIX_PATH` 指定 CMake 找软件包的路径。可以有多个路径，采用`;`分隔。

    **如果想使用静态编译的 Qt，需要把Qt安装目录加入到 `CMAKE_PREFIX_PATH`，否则 CMake 会找不到 Qt，或者找到另一个 Qt。**

    除此以外，你还可以通过 `SlopeCraft_vectorize` 指定是否使用向量化加速。如果为`ON`，在 intel 芯片上将使用 AVX 和 AVX2 加速，在 apple silicon 芯片上将使用 Neon 加速。**向量化加速默认开启。**

    虽然你可以通过`SlopeCraft_GPU_API`和指定显卡加速，但目前 SlopeCraft 还只支持 Vulkan 的显卡加速手段，而它在 mac 上没有官方支持（没错苹果就是这么唯我独尊，能不兼容就绝不兼容），目前没有试过社区实现的vulkan支持。


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

### 清除已有编译文件

如果你希望完全重新编译，你可以通过使用一下命令删除已有的编译文件：

```bash
rm -rf 3rdParty binaries build
```
