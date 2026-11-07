# Compilation Guide

Welcome to the compilation guide of SlopeCraft. This guide will help you build SlopeCraft. Please read this general guide first, and then read the guide for your platform to complete the build.

!!! question "Questions"
    If you meet any problem when building SlopeCraft, please draw us an issue on our [GitHub repository](https://github.com/SlopeCraft/SlopeCraft/issues).

## Compilers

Currently both gcc12+ and clang15+ can be used to build SlopeCraft (exception: the AppleClang shipped with macOS makes cmake fail to find openmp). Therefore the Windows release is built with clang (together with the MSVC standard library), while the releases for the other operating systems are built with gcc. In theory msvc can build this project as well.

## Dependencies

1. **Qt v6.4.0 or later** for the GUI
    - If your Qt is built and configured by yourself, make sure you have installed `QtBase`, `QtNetwork` and `QtTools`.
    - If you want to build Qt yourself, it is best to build SlopeCraft and Qt with the same compiler.
2. **Eigen v3.4.0 or later** for linear algebra
     - Can be downloaded automatically.
3. [**HeuristicFlow v1.6.2.1**](https://github.com/ToKiNoBug/HeuristicFlow) for modern optimization algorithms
     - Can be downloaded automatically.
4. **zlib v1.12.11 or later** for gzip compression
     - Usually it is installed together with the compiler, so you do not need to build it.
5. **Nlohmann json**
     - Can be downloaded automatically.
6. **libpng**
     - You must install it, and add its installation path to `CMAKE_PREFIX_PATH`.
7. **libzip**
     - You must install it, and add its installation path to `CMAKE_PREFIX_PATH`.
8. **fmtlib**
     - Can be downloaded and built automatically.
9. **cli11**
     - Can be downloaded automatically.
10. **magic_enum**
     - Can be downloaded automatically.

## Projects in SlopeCraft

|       Name        | Binary type | Dependencies (external)                                                          | Description                                        |
| :---------------: | :---------: | :------------------------------------------------------------------------------- | :------------------------------------------------- |
|   `imageCutter`   | Executable  | Qt base                                                                          | A simple image preprocessing program               |
|    `MapViewer`    | Executable  | Qt base, Eigen, zlib                                                             | A tool to browse Minecraft map data files          |
| `BlockListManager` | Static lib | Qt base                                                                          | A class to manage blocks that are available for map arts |
|   `GAConverter`   | Static lib  | Eigen, HeuristicFlow                                                             | A converter based on genetic algorithm             |
|   `SlopeCraftL`   | Shared lib  | Eigen, zlib HeuristicFlow                                                        | The kernel of SlopeCraft                           |
|  `SlopeCraftMain` | Executable  | Qt base, Eigen                                                                   | The executable of SlopeCraft                       |
|   `VisualCraftL`  | Shared lib  | libpng, libzip, Eigen, zlib, nlohmann json, magic_enum, fmtlib, OpenCL(optional) | The kernel of VisualCraft                          |
|   `VisualCraft`   | Executable  | Qt base, nlohmann json, magic_enum                                               | The GUI of VisualCraft                             |
|       `vccl`      | Executable  | Qt base, magic_enum, fmtlib, cli11                                               | The command line of VisualCraft                    |

## CMake arguments for SlopeCraft

|               Parameter                |  Type  |                  Default value                   | Description                                                                            |
| :------------------------------------: | :----: | :----------------------------------------------: | :------------------------------------------------------------------------------------- |
|          `CMAKE_PREFIX_PATH`           |  PATH  |                       ""                         | Tell cmake where to find Qt, zlib, libpng, libzip and GPU api sdk (like OpenCL)         |
|         `CMAKE_INSTALL_PREFIX`         |  PATH  |           ${CMAKE_BINARY_DIR}/install            | Where to install SlopeCraft.                                                           |
|          `SlopeCraft_GPU_API`          | STRING |                    "OpenCL"                      | The GPU API used for computation. Valid values: OpenCL, None. Metal may be supported.  |
|         `SlopeCraft_vectorize`         |  BOOL  |                      true                        | Enable the AVX and AVX2 instruction sets, which speeds up computation, but cannot be used on older computers and virtual machines |
|      `SlopeCraft_update_ts_files`      |  BOOL  |                      false                       | Whether to update the ts files before building.                                        |
|   `SlopeCraft_update_ts_no_obsolete`   |  BOOL  |                      false                       | Remove obsolete translations from the ts files.                                        |
|           `SlopeCraft_gprof`           |  BOOL  |                      false                       | Profile performance with gprof.                                                        |
| `SlopeCraft_windeployqt_flags_install` | STRING |              "--no-translations"                 | Arguments passed to windeployqt when installing (Windows only)                         |
|  `SlopeCraft_windeployqt_flags_build`  | STRING | "${SlopeCraft_windeployqt_flags_install};--force" | Arguments passed to windeployqt after building (used for debugging only, Windows only) |

## CMake generators for SlopeCraft

|    Generator    |         OS         | Description                                                                                                                                                                                                                                                                                        |
| :-------------: | :----------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|      Ninja      | macOS & Linux & Windows | Compiles relatively fast                                                                                                                                                                                                                                                                       |
| MinGW Makefiles |      Windows       | Because of a bug in GNU binutils, `windres.exe` cannot handle directories containing spaces correctly, which causes compilation errors, and Ninja has no way to avoid this error, so Windows must use MinGW Makefiles. On other platforms `windres` is no longer used, so Ninja works on macOS and Linux. |

## Compilation steps

Now you have learned the information you need to know before building SlopeCraft. Next, click the link of your operating system to see the detailed compilation steps.

- [Windows/Linux](Windows-Linux.en.md)
- [macOS](macOS.en.md)
