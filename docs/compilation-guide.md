# 编译指南

## 支持的编译器

只有 gcc 是完全支持的，MSVC 和 clang 可能无法通过编译。你的编译器必须支持 C++20 标准，所以请使用 gcc12 或更高版本（gcc13）。到现在为止，SlopeCraft 是用 gcc12.2.0 编译的。

## 依赖的库

1. **Qt v6.4.0 或更高版本**用于 gui
     - 如果你的 Qt 是自己编译和配置的，请确保你安装了`QtBase`和`QtNetwork`。
     - Qt 必须用 gcc 进行编译。
2. **Eigen v3.4.0 或更高版本**用于线性代数
      - 可以自动下载。
3. 用于现代优化算法的 [**HeuristicFlow v1.6.2.1**](https://github.com/ToKiNoBug/HeuristicFlow)。
      - 可以自动下载。
4. **zlib v1.12.11 或更高版本**用于 gzip 压缩
      - 通常它是和编译器一起安装的，所以你不需要去编译它。
5. **Nlohmann json**
      - 可以自动下载。
6. **libpng**
      - 你必须编译和安装它，并将安装路径添加到`CMAKE_PREFIX_PATH`。
7. **libzip**
      - 你必须编译和安装它，并将安装路径添加到`CMAKE_PREFIX_PATH`。
8. **fmtlib**
      - 可以下载并自动编译。
9. **cli11**
    - 可以自动下载。
10. **magic_enum**
    - 可以自动下载。

## SlopeCraft 中的子项目

|        名称        | 二进制类型 | 依赖项 (外部依赖库)                                                            | 描述                                    |
| :----------------: | :--------: | :----------------------------------------------------------------------------- | :-------------------------------------- |
|   `imageCutter`    | 可执行文件 | Qt base                                                                        | 一个不必要的图像预处理程序。            |
|    `MapViewer`     | 可执行文件 | Qt base，Eigen，zlib                                                           | 一个浏览 Minecraft 地图数据文件的工具。 |
| `BlockListManager` |   静态库   | Qt base                                                                        | 管理地图画可用方块的类                  |
|   `GAConverter`    |   静态库   | Eigen, HeuristicFlow                                                           | 一个基于遗传算法的转换器                |
|   `SlopeCraftL`    |   动态库   | Eigen, zlib HeuristicFlow                                                      | SlopeCraft 的内核                       |
|  `SlopeCraftMain`  | 可执行文件 | Qt base, Eigen                                                                 | SlopeCraft 的可执行程序                 |
|   `VisualCraftL`   |   动态库   | libpng, libzip, Eigen, zlib, nlohmann json, maigc_enum, fmtlib, OpenCL（可选） | VisualCraft 的内核                      |
|   `VisualCraft`    | 可执行文件 | Qt base, nlohmann json, magic_enum                                             | VisualCraft 的 gui 界面                 |
|       `vccl`       | 可执行文件 | Qt base, magic_enum, fmtlib, cli11                                             | VisualCraft 的命令行                    |

注意，当 SlopeCraftL 链接 GAConverter 时，链接模式是**public**，因为 GAConverter 使用 SlopeCraftL 中实现的全局只读变量。

## 步骤

### 1. 在你的电脑上安装 Qt、libzip 和 libpng

### 2. clone SlopeCraft 代码仓库

```bash
git clone https://github.com/SlopeCraft/SlopeCraft.git
```

### 3. Configure

运行 cmake 配置这个项目。命令行示例：

```bash
cmake -S . -B ./build -G "MinGW Makefiles" -DCMAKE_CXX_COMPILER=gcc 
```

你可以使用其他 Generator，如 Ninja，但在 windows 上可能无法工作，因为`windres.exe`不能正确处理包括目录中的空格，这将导致编译错误。**这是 GNU bintuils 的一个错误，Ninja 没有办法避免这个错误，但 mingw makefiles 可以。**在其他平台上，`windres`不再被使用，所以这个错误也被避免了。

你可能需要向 cmake 传递其他参数。SlopeCraft 的 cmake 脚本会接收以下参数。

|                参数                |  类型  |           默认值            | 说明                                                                           |
| :--------------------------------: | :----: | :-------------------------: | :----------------------------------------------------------------------------- |
|        `CMAKE_PREFIX_PATH`         |  PATH  |             ""              | 告诉 cmake 在哪里可以找到 Qt、zlib、libpng、libzip 和 GPU api sdk（如 OpenCL） |
|       `CMAKE_INSTALL_PREFIX`       |  PATH  | ${CMAKE_BINARY_DIR}/install | 在哪里安装 SlopeCraft。                                                        |
|        `SlopeCraft_GPU_API`        | STRING |          "OpenCL"           | 用于计算的 API。有效值 : OpenCL, None. 可能支持 Metal。                        |
|    `SlopeCraft_update_ts_files`    |  BOOL  |            false            | 是否在构建前更新 ts 文件。                                                     |
| `SlopeCraft_update_ts_no_obsolete` |  BOOL  |            false            | 从 ts 文件中删除过时的翻译。                                                   |
|         `SlopeCraft_gprof`         |  BOOL  |            false            | 用 gprof 分析性能。                                                            |

### 4. 构建所有目标

```bash
cmake --build . --parallel
```

### 5. 安装，并删除不必要的文件

运行 `cmake --install` 来安装所有二进制文件和方块文件。默认情况下，安装目录是 `${CMAKE_BINARY_DIR}/install`，你可以在那里找到所有可执行文件。可能还有一个文件夹叫`please_delete_this_folder`，把它删除。

### 6. 部署 Qt

你可能需要为可执行文件部署 Qt 动态库。对于 Windows，在命令行中运行`windeployqt SlopeCraft.exe`。在其他平台上也有类似的部署命令，比如 MacOS 上的`macdeployqt`。

## 注意事项

1. 如果你遇到任何问题，请给我画一个新的问题。
2. 如果你在**linux**上构建 VisualCraftL，可能会出现一些围绕 libzip 的链接问题。为了解决这个问题，有 2 种可能的方法。
   1. 使用 libzip 的**静态库，而不是动态库**。但是这个 libzip.a 应该用参数 **`-fPIC`** 编译，因为 `libVisualCraftL.so`这个共享库会链接到它。
      - 如果你的 libzip.a 链接了其他压缩库（例如 lzma），链接可能会失败。这是因为 libzip 链接到它的依赖项时是 PRIVATE 链接，不管 libzip 是不是静态库。我猜这是 libzip 的一个设计错误。所以使用静态的 libzip 是不可取的。
   2. 使用动态的 libzip，并将 `libzip.so` 和指向它的符号链接复制到 `CMAKE_CURRENT_BINARY_DIR`。这就是我现在的方案。
3. 如果你用 OpenCL 作为 GPU API 构建 SlopeCraft，**你的第一次编译预计会遇到链接失败**。这是因为 OpenCL 的 C 源代码是作为资源文件嵌入的（不是 Windows .rc），所以引入了一个基于 cmake 的第三方[资源文件实现（ResourceCreator.cmake）](https://github.com/isRyven/ResourceCreator.cmake.git)。它于 qt 有点犯冲，因为 "ResourceCreator.cmake"在 cmake 构建时生成了 C 代码（**__rsc_ColorManip_cl_rc.c**），然而 Qt 在 cmake 配置时要求所有源文件都存在。现在，C 源文件被**touch**生成了，但一旦文件存在，正确的代码就不会生成。目前还没有找到一个完美的解决方案，所以请通过以下步骤解决这个问题。
    1. 构建，链接器将报错，报错信息可能是*undefined reference to`ColorManip_cl_rc_length` or `ColorManip_cl_rc`*。
    2. 删除`${CMAKE_BINARY_DIR}/utilities/GPUWrapper/OpenCL/__rsc_ColorManip_cl_rc.c`。你删除的是一个由 touch 生成的空文件，构建时将会生成正确的文件。
    3. 再次构建。**不要再运行 cmake configure！**此时的编译应该会成功。
