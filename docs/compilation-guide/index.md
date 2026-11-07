# 编译指南

欢迎阅读 SlopeCraft 的编译指南。这个指南将帮助你编译 SlopeCraft，请先阅读本通用指南，然后阅读具体平台的相关指南完成编译。

!!! question "问题"
    如果你在编译 SlopeCraft 时遇到了问题，请前往 [GitHub 仓库](https://github.com/SlopeCraft/SlopeCraft/issues) 向我们提交 issue。

## 编译器

目前，clang 20+ 可以用于编译 SlopeCraft，理论上任何支持C++23的环境都可以编译。目前Windows和Linux都采用LLVM Clang编译，macOS上采用Apple Clang编译。

## 依赖

1. **Qt v6.5.0 或更高版本**用于 GUI
    - 如果你的 Qt 是自己编译和配置的，请确保你安装了 `QtBase` 、 `QtNetwork` 和 `QtTools`
    - 推荐用Qt 6.12 LTS
2. **Eigen v3.4.0 或更高版本**用于线性代数
     - 可以自动下载。
3. [**magic_enum**](https://github.com/SlopeCraft/mc_schem)用于原理图库
     - 可以自动下载
     - 依赖rust工具链
4. [**gapp**](https://github.com/KRM7/gapp)用于遗传算法
     - 可以自动下载
5. [**xsimd**](https://github.com/xtensor-stack/xsimd)用于向量化
     - 可以自动下载
5. [**libnbt++**](ttps://github.com/PrismLauncher/libnbtplusplus)用于NBT库
     - 可以自动下载
6. [**Nlohmann json**](https://github.com/nlohmann/json) [**cli11**](https://github.com/CLIUtils/CLI11) [**magic_enum**](https://github.com/Neargye/magic_enum)
     - 可以自动下载
7. **libpng** **libzip** **zlib** **boost**
     - 你必须安装它，并将安装路径添加到 `CMAKE_PREFIX_PATH`
8. **7z**用于制作压缩包
     - 你必须安装它，但是无需 `CMAKE_PREFIX_PATH`

## SlopeCraft 中的子项目

|        名称        | 二进制类型 | 描述                                  |
| :----------------: | :--------: | :------------------------------------ |
|   `imageCutter`    | 可执行文件 | 一个简单的图像预处理程序              |
|    `MapViewer`     | 可执行文件 | 一个浏览 Minecraft 地图数据文件的工具 |
| `BlockListManager` |   静态库   | 管理地图画可用方块的类                |
|   `SlopeCraftL`    |   动态库   | SlopeCraft 的内核                     |
|    `SlopeCraft`    | 可执行文件 | SlopeCraft 的可执行程序               |
|       `sccl`       | 可执行文件 | SlopeCraft 的命令行工具               |
|   `VisualCraftL`   |   动态库   | VisualCraft 的内核                    |
|   `VisualCraft`    | 可执行文件 | VisualCraft 的 GUI 界面               |
|       `vccl`       | 可执行文件 | VisualCraft 的命令行工具              |

## SlopeCraft 接受的 CMake 参数

|                  参数                  |  类型  |                      默认值                       | 说明                                                                           |
| :------------------------------------: | :----: | :-----------------------------------------------: | :----------------------------------------------------------------------------- |
|          `CMAKE_PREFIX_PATH`           |  PATH  |                        ""                         | 告诉 cmake 在哪里可以找到 Qt、zlib、libpng、libzip 和 GPU api sdk（如 Vulkan） |
|         `CMAKE_INSTALL_PREFIX`         |  PATH  |            ${CMAKE_BINARY_DIR}/install            | 在哪里安装 SlopeCraft。                                                        |
|        `SlopeCraft_no_download`        |  BOOL  |                        OFF                        | 禁用Configure阶段的任何下载功能，支持可复现的编译                              |
|          `SlopeCraft_GPU_API`          | STRING |                       None                        | 用于计算的显卡 API。有效值 : Vulkan, None。                                    |
|         `SlopeCraft_vectorize`         |  BOOL  |                       true                        | 启用 向量化指令集，这可以提高运行速度，但在较老的电脑和虚拟机中将不能使用      |
|      `SlopeCraft_update_ts_files`      |  BOOL  |                       false                       | 是否在构建前更新 ts 文件。                                                     |
|   `SlopeCraft_update_ts_no_obsolete`   |  BOOL  |                       false                       | 从 ts 文件中删除过时的翻译。                                                   |
|           `SlopeCraft_gprof`           |  BOOL  |                       false                       | 用 gprof 分析性能。                                                            |
| `SlopeCraft_windeployqt_flags_install` | STRING |                "--no-translations"                | 安装时传递给 windeployqt 的参数（仅 windows 有效）                             |
|  `SlopeCraft_windeployqt_flags_build`  | STRING | "${SlopeCraft_windeployqt_flags_install};--force" | 在编译之后，传递给 windeployqt 的参数（仅用于调试，仅 windows 有效）           |

## SlopeCraft 使用的 CMake 生成器

| 生成器 |        操作系统         | 说明         |
| :----: | :---------------------: | :----------- |
| Ninja  | macOS & Linux & Windows | 编译的比较快 |

## 编译步骤

现在，你已经了解了在编译 SlopeCraft 之前所需要知道的一些信息，接下来，点击下方你的操纵系统的链接，查看对应操纵系统的具体编译步骤。

- [Windows/Linux](Windows-Linux.md)
- [macOS](macOS.md)
