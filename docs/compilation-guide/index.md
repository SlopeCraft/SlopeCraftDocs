# 编译指南

欢迎阅读 SlopeCraft 的编译指南。这个指南将帮助你编译 SlopeCraft，请先阅读本通用指南，然后阅读具体平台的相关指南完成编译。

!!! question "问题"
    如果你在编译 SlopeCraft 时遇到了问题，请前往 [GitHub 仓库](https://github.com/SlopeCraft/SlopeCraft/issues) 向我们提交 issue。

## 编译器

目前，gcc12+ 和 clang15+ 都可以用于编译 SlopeCraft（例外：MacOS 自带的 AppleClang 会出现 cmake 找不到 openmp 的问题）。因此，Windows 上的发行版采用 clang 编译（搭配 MSVC 标准库），其他操作系统上的发行版采用 gcc 编译。理论上 msvc 也能编译本项目。

## 依赖

1. **Qt v6.4.0 或更高版本**用于 GUI
    - 如果你的 Qt 是自己编译和配置的，请确保你安装了 `QtBase` 、 `QtNetwork` 和 `QtTools`。
    - 如果你希望自行编译 Qt，最好使用相同的编译器编译 SlopeCraft 和 Qt。
2. **Eigen v3.4.0 或更高版本**用于线性代数
     - 可以自动下载。
3. 用于现代优化算法的 [**HeuristicFlow v1.6.2.1**](https://github.com/ToKiNoBug/HeuristicFlow)。
     - 可以自动下载。
4. **zlib v1.12.11 或更高版本**用于 gzip 压缩
     - 通常它是和编译器一起安装的，所以你不需要去编译它。
5. **Nlohmann json**
     - 可以自动下载。
6. **libpng**
     - 你必须安装它，并将安装路径添加到 `CMAKE_PREFIX_PATH`。
7. **libzip**
     - 你必须安装它，并将安装路径添加到 `CMAKE_PREFIX_PATH`。
8. **fmtlib**
     - 可以下载并自动编译。
9. **cli11**
     - 可以自动下载。
10. **magic_enum**
     - 可以自动下载。

## SlopeCraft 中的子项目

|        名称        | 二进制类型 | 依赖项 (外部依赖库)                                                            | 描述                                    |
| :----------------: | :--------: | :----------------------------------------------------------------------------- | :-------------------------------------- |
|   `imageCutter`    | 可执行文件 | Qt base                                                                        | 一个简单的图像预处理程序。              |
|    `MapViewer`     | 可执行文件 | Qt base，Eigen，zlib                                                           | 一个浏览 Minecraft 地图数据文件的工具。 |
| `BlockListManager` |   静态库   | Qt base                                                                        | 管理地图画可用方块的类                  |
|   `GAConverter`    |   静态库   | Eigen, HeuristicFlow                                                           | 一个基于遗传算法的转换器                |
|   `SlopeCraftL`    |   动态库   | Eigen, zlib HeuristicFlow                                                      | SlopeCraft 的内核                       |
|  `SlopeCraftMain`  | 可执行文件 | Qt base, Eigen                                                                 | SlopeCraft 的可执行程序                 |
|   `VisualCraftL`   |   动态库   | libpng, libzip, Eigen, zlib, nlohmann json, maigc_enum, fmtlib, OpenCL（可选） | VisualCraft 的内核                      |
|   `VisualCraft`    | 可执行文件 | Qt base, nlohmann json, magic_enum                                             | VisualCraft 的 GUI 界面                 |
|       `vccl`       | 可执行文件 | Qt base, magic_enum, fmtlib, cli11                                             | VisualCraft 的命令行                    |

## SlopeCraft 接受的 CMake 参数

|                  参数                  |  类型  |                      默认值                       | 说明                                                                              |
| :------------------------------------: | :----: | :-----------------------------------------------: | :-------------------------------------------------------------------------------- |
|          `CMAKE_PREFIX_PATH`           |  PATH  |                        ""                         | 告诉 cmake 在哪里可以找到 Qt、zlib、libpng、libzip 和 GPU api sdk（如 OpenCL）    |
|         `CMAKE_INSTALL_PREFIX`         |  PATH  |            ${CMAKE_BINARY_DIR}/install            | 在哪里安装 SlopeCraft。                                                           |
|          `SlopeCraft_GPU_API`          | STRING |                     "OpenCL"                      | 用于计算的显卡 API。有效值 : OpenCL, None. 可能支持 Metal。                       |
|         `SlopeCraft_vectorize`         |  BOOL  |                       true                        | 启用 AVX 和 AVX2 指令集，这可以提高允运行速度，但在较老的电脑和虚拟机中将不能使用 |
|      `SlopeCraft_update_ts_files`      |  BOOL  |                       false                       | 是否在构建前更新 ts 文件。                                                        |
|   `SlopeCraft_update_ts_no_obsolete`   |  BOOL  |                       false                       | 从 ts 文件中删除过时的翻译。                                                      |
|           `SlopeCraft_gprof`           |  BOOL  |                       false                       | 用 gprof 分析性能。                                                               |
| `SlopeCraft_windeployqt_flags_install` | STRING |                "--no-translations"                | 安装时传递给 windeployqt 的参数（仅 windows 有效）                                |
|  `SlopeCraft_windeployqt_flags_build`  | STRING | "${SlopeCraft_windeployqt_flags_install};--force" | 在编译之后，传递给 windeployqt 的参数（仅用于调试，仅 windows 有效）              |

## SlopeCraft 使用的 CMake 生成器

|     生成器      |        操作系统         | 说明                                                                                                                                                                                                                          |
| :-------------: | :---------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|      Ninja      | macOS & Linux & Windows | 编译的比较快                                                                                                                                                                                                                  |
| MinGW Makefiles |         Windows         | 由于 GNU bintuils 中的一个错误，`windres.exe` 不能正确处理包含空格的目录，而 Ninja 无法避免这个错误，所以 Windows 上需要使用 MinGW Makefiles。但在其他平台上，`windres` 不再被使用，所以 Ninja 可以在 macOS 和 Linux 上工作。 |

## 编译步骤

现在，你已经了解了在编译 SlopeCraft 之前所需要知道的一些信息，接下来，点击下方你的操纵系统的链接，查看对应操纵系统的具体编译步骤。

- [Windows/Linux](Windows-Linux.md)
- [macOS](macOS.md)
