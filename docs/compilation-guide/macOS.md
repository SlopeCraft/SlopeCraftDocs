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

1. clang 编译器

    ```bash
    brew install llvm
    ```

    clang 和 clang++ 将被安装在 `/opt/homebrew/opt/llvm/bin`

    在完成后，你可以使用 `/opt/homebrew/opt/llvm/bin/clang --version` 来检查 clang 编译器是否已经安装成功。如果一切正常，你应该会看到与下面类似的输出：
    ```bash
    Homebrew clang version 17.0.4
    Target: arm64-apple-darwin22.6.0
    Thread model: posix
    InstalledDir: /opt/homebrew/opt/llvm/bin
    ```

2. Ninja 和 CMake

    SlopeCraft 使用 CMake 构建，推荐用 Ninja 作为 CMake 的生成器，因此我们需要安装 CMake 和 Ninja：

    ```bash
    brew install cmake ninja
    ```

    使用 `ninja --version` 来检查 Ninja 是否已经安装成功。如果一切正常，你应该会看到与下面类似的输出：

    ```bash
    1.11.1
    ```

3. zlib & libpng & libzip

    ```bash
    brew install zlib libpng libzip
    ```

    在安装完成后，如果以下路径均存在，则表示安装成功：

    - `/usr/local/Cellar/zlib`
    - `/usr/local/Cellar/libpng`
    - `/usr/local/Cellar/libzip`

4. Qt

    ```bash
    brew install qt
    ```

    在安装完成后，你可以通过检查 `/usr/local/Cellar/qt` 路径来检查 Qt 是否已经安装成功。如果一切正常，你应该会看到一个命名类似于 `6.y.z_1` 的文件夹（例如 `6.4.3_1`）。你只需要确保该文件夹的第一个数字是 `6`，且 `y` 大于等于 `4` 即可。

    你也可以使用静态链接的 Qt，但这通常需要自行编译。编译 Qt 是个比较复杂的步骤，以下给出一个样板代码：
    ```bash
    git clone https://github.com/qt/qtbase.git --recursive
    cd qtbase 
    git checkout v6.6.0
    cd ..

    git clone https://github.com/qt/qttools.git --recursive
    cd qttools
    git checkout v6.6.0
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
    以上代码可以在任何目录中执行（只要没有权限问题），静态库的 Qt6.6.0 将被安装在 当前目录的 install 子目录中。

### 编译 SlopeCraft

1. 克隆 SlopeCraft 仓库并进入仓库目录

    ```bash
    git clone https://github.com/SlopeCraft/SlopeCraft.git && cd SlopeCraft
    ```

2. 配置 CMake

    使用以下命令配置 CMake：

    ```bash
    cmake -S . -B ./build -G "Ninja" -DCMAKE_C_COMPILER=/opt/homebrew/opt/llvm/bin/clang -DCMAKE_CXX_COMPILER=/opt/homebrew/opt/llvm/bin/clang++ -DCMAKE_INSTALL_PREFIX=./build/install -DCMAKE_BUILD_TYPE=Debug
    ```


    在上面的命令中，我们指定了几个参数：

    - `-DCMAKE_C_COMPILER` 指定 C 编译器，可以为`/opt/homebrew/opt/llvm/bin/clang`或者`clang`，前者是标准 LLVM 中的 clang 编译器，后者是 MacOS 上默认的 AppleClang 编译器。
    - `-DCMAKE_CXX_COMPILER` 指定 C++ 编译器路径，可以为`/opt/homebrew/opt/llvm/bin/clang++`或者`clang++`，前者是标准 LLVM 中的 clang++ 编译器，后者是 MacOS 上默认的 AppleClang 编译器。请注意，无论使用哪个编译器，C 编译器和 C++ 编译器都应当是同一套，请不要混合使用不同来源的编译器。
    - `-DCMAKE_INSTALL_PREFIX` 指定安装路径，本指南中我们将安装路径设置为 `./build/install`，你可以根据自己的需要进行修改
    - `-DCMAKE_BUILD_TYPE` 指定编译类型，如 Debug 或 Release

    **如果想使用静态编译的 Qt，需要手动指定`-DCMAKE_PREFIX_PATH`为 Qt 的安装目录，否则 CMake 会找不到 Qt，或者找到另一个 Qt。**

    除此以外，你还可以通过 `SlopeCraft_vectorize` 指定是否使用向量化加速。如果为`ON`，在 intel 芯片上将使用 AVX 和 AVX2 加速，在 apple silicon 芯片上将使用 Neon 加速。**向量化加速默认开启。**

    虽然你可以通过`SlopeCraft_GPU_API`和指定显卡加速，但目前 SlopeCraft 还只支持 OpenCL 的显卡加速手段，而它在 mac 上不能完善使用（OpenCL 在 macOS 下的支持有一些问题），所以非常不建议修改这个选项，目前默认关闭显卡加速，打开也没有作用。


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

5. ~~部署 Qt 可执行文件 **\[可选步骤\]**~~
   
   这一步已经被自动化，在安装时就会自动执行这一步，无需手动操作。

6. 修补 `VisualCraft.app` **\[临时\]**

    这一步操作将在未来合并到安装中。

    在安装时我们调用`macdeployqt` 为可执行文件部署动态库（部署到 `*.app/Contents/Frameworks` 文件夹），但这一功能并不完美，它只部署可执行文件直接链接的动态库，但不保证这些动态库能连接到它们依赖的动态库。VisualCraft 因为使用 libzip 遇上了这个问题。

    libzip 依赖 zlib、libbz2、libzstd 和 liblzma，其中前两个是 macOS 内置的，但后两个不是。`macdeployqt` 一般能够部署 `libzip.5.dylib`，但**不一定**能部署 `liblzma.5.dylib`和`libzstd.1.dylib`。而且即便后两个库被部署了，`libzip.5.dylib`中引用它们的路径指向一个错误的相对路径，需要用`install_name_tool`修正。

    首先，切换到 `install` 目录，然后进入`VisualCraft.app`内部：

    ```bash
    cd install
    cd VisualCraft.app/Contents/Frameworks
    ls
    ```

    上面的 ls 将列出`VisualCraft.app/Contents/Frameworks`中的动态库文件，里面一定含有`libzip.5.dylib`和 libpng、libomp 的动态库。请检查里面有没有`liblzma.5.dylib`和`libzstd.1.dylib`，如果没有，使用 `cp` 命令拷贝它们：
    ```bash
    cp /opt/homebrew/lib/libzstd.1.dylib .
    cp /opt/homebrew/lib/liblzma.5.dylib .
    ```

    如果使用`otool -L libzip.5.dylib`查看`libzip.5.dylib`链接的动态库，将得到如下输出：
    ```bash
    libzip.5.dylib:
        @executable_path/../Frameworks/libzip.5.dylib (compatibility version 5.0.0, current version 5.5.0)
        /usr/lib/libbz2.1.0.dylib (compatibility version 1.0.0, current version 1.0.8)
        @loader_path/../../../../opt/xz/lib/liblzma.5.dylib (compatibility version 10.0.0, current version 10.4.0)
        @loader_path/../../../../opt/zstd/lib/libzstd.1.dylib (compatibility version 1.0.0, current version 1.5.5)
        /usr/lib/libz.1.dylib (compatibility version 1.0.0, current version 1.2.11)
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.100.3)
    ```
    可以发现指向`liblzma.5.dylib`和`libzstd.1.dylib`的相对路径是不存在的，而指向`libz.1.dylib`、`libbz2.1.0.dylib`和`libSystem.B.dylib`的绝对路径一定存在（只要操作系统完整）。
    
    使用`install_name_tool`修正这两个链接：
    ```bash
    install_name_tool libzip.5.dylib -change @loader_path/../../../../opt/xz/lib/liblzma.5.dylib @loader_path/liblzma.5.dylib
    install_name_tool libzip.5.dylib -change @loader_path/../../../../opt/zstd/lib/libzstd.1.dylib @loader_path/libzstd.1.dylib
    ```

    此时 libzip.5.dylib 将能够正确连接到 .app 包内的`liblzma.5.dylib`和`libzstd.1.dylib`。但这一步破坏了 .app 包的签名，需要重新签名：

    ```bash
    cd ../../.. # 回退到 install 文件夹
    codesign --force --deep --sign=- VisualCraft.app
    ```

### 清除已有编译文件

如果你希望完全重新编译，你可以通过使用一下命令删除已有的编译文件：

```bash
rm -rf 3rdParty binaries build
```
