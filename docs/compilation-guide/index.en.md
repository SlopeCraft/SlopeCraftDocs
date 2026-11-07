# Compilation Guide

Welcome to the compilation guide of SlopeCraft. This guide will help you build SlopeCraft. Please read this general guide first, and then read the guide for your platform to complete the build.

!!! question "Questions"
    If you meet any problem when building SlopeCraft, please draw us an issue on our [GitHub repository](https://github.com/SlopeCraft/SlopeCraft/issues).

## Compilers

Currently clang 20+ can be used to build SlopeCraft; in theory any environment that supports C++23 can build it. Windows and Linux are currently built with LLVM Clang, while macOS is built with Apple Clang.

## Dependencies

1. **Qt v6.5.0 or later** for the GUI
    - If your Qt is built and configured by yourself, make sure you have installed `QtBase`, `QtNetwork` and `QtTools`
    - Qt 6.12 LTS is recommended
2. **Eigen v3.4.0 or later** for linear algebra
     - Can be downloaded automatically.
3. [**mc_schem**](https://github.com/SlopeCraft/mc_schem) for the schematic library
     - Can be downloaded automatically
     - Requires the rust toolchain
4. [**gapp**](https://github.com/KRM7/gapp) for the genetic algorithm
     - Can be downloaded automatically
5. [**xsimd**](https://github.com/xtensor-stack/xsimd) for vectorization
     - Can be downloaded automatically
5. [**libnbt++**](ttps://github.com/PrismLauncher/libnbtplusplus) for the NBT library
     - Can be downloaded automatically
6. [**Nlohmann json**](https://github.com/nlohmann/json) [**cli11**](https://github.com/CLIUtils/CLI11) [**magic_enum**](https://github.com/Neargye/magic_enum)
     - Can be downloaded automatically
7. **libpng** **libzip** **zlib** **boost**
     - You must install it, and add the installation path to `CMAKE_PREFIX_PATH`
8. **7z** for creating archives
     - You must install it, but `CMAKE_PREFIX_PATH` is not needed

## Projects in SlopeCraft

|        Name        | Binary type | Description                                              |
| :----------------: | :---------: | :------------------------------------------------------- |
|   `imageCutter`    | Executable  | A simple image preprocessing program                     |
|    `MapViewer`     | Executable  | A tool to browse Minecraft map data files                |
| `BlockListManager` | Static lib  | A class to manage blocks that are available for map arts |
|   `SlopeCraftL`    | Shared lib  | The kernel of SlopeCraft                                 |
|    `SlopeCraft`    | Executable  | The executable of SlopeCraft                             |
|       `sccl`       | Executable  | The command line tool of SlopeCraft                      |
|   `VisualCraftL`   | Shared lib  | The kernel of VisualCraft                                |
|   `VisualCraft`    | Executable  | The GUI of VisualCraft                                   |
|       `vccl`       | Executable  | The command line tool of VisualCraft                     |

## CMake arguments for SlopeCraft

|               Parameter                |  Type  |                   Default value                   | Description                                                                                                                     |
| :------------------------------------: | :----: | :-----------------------------------------------: | :------------------------------------------------------------------------------------------------------------------------------ |
|          `CMAKE_PREFIX_PATH`           |  PATH  |                        ""                         | Tell cmake where to find Qt, zlib, libpng, libzip and GPU api sdk (like Vulkan)                                                 |
|         `CMAKE_INSTALL_PREFIX`         |  PATH  |            ${CMAKE_BINARY_DIR}/install            | Where to install SlopeCraft.                                                                                                    |
|        `SlopeCraft_no_download`        |  BOOL  |                        OFF                        | Disable every download during the configure stage, which allows reproducible builds                                             |
|          `SlopeCraft_GPU_API`          | STRING |                       None                        | The GPU API used for computation. Valid values: Vulkan, None.                                                                   |
|         `SlopeCraft_vectorize`         |  BOOL  |                       true                        | Enable the vectorized instruction sets, which speeds up computation, but cannot be used on older computers and virtual machines |
|      `SlopeCraft_update_ts_files`      |  BOOL  |                       false                       | Whether to update the ts files before building.                                                                                 |
|   `SlopeCraft_update_ts_no_obsolete`   |  BOOL  |                       false                       | Remove obsolete translations from the ts files.                                                                                 |
|           `SlopeCraft_gprof`           |  BOOL  |                       false                       | Profile performance with gprof.                                                                                                 |
| `SlopeCraft_windeployqt_flags_install` | STRING |                "--no-translations"                | Arguments passed to windeployqt when installing (Windows only)                                                                  |
|  `SlopeCraft_windeployqt_flags_build`  | STRING | "${SlopeCraft_windeployqt_flags_install};--force" | Arguments passed to windeployqt after building (used for debugging only, Windows only)                                          |

## CMake generators for SlopeCraft

| Generator |           OS            | Description              |
| :-------: | :---------------------: | :----------------------- |
|   Ninja   | macOS & Linux & Windows | Compiles relatively fast |

## Compilation steps

Now you have learned the information you need to know before building SlopeCraft. Next, click the link of your operating system to see the detailed compilation steps.

- [Windows/Linux](Windows-Linux.en.md)
- [macOS](macOS.en.md)
