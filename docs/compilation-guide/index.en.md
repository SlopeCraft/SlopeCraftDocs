# Compilation Guide

Welcome to compilation guide of SlopeCraft. This guide will help you compile SlopeCraft. Please read this general guide, and then read the detailed steps on your OS.

!!! question "Questions"
    If you have any difficulty when building SlopeCraft, do not hesitate to draw us an issue to our [Github repository](https://github.com/SlopeCraft/SlopeCraft/issues).

## Supported compilers

Only gcc is fully supported. MSVC and clang may not pass the compilation. Your compiler must supports C++ 20 standard, so please use gcc12 or later(gcc13). Uptil now, SlopeCraft is compiled with gcc 12.2.0.

!!! warning "Warning"
    Only gcc is fully supported. MSVC and clang may not pass the compilation. The clang compiler provided by Apple Xcode CommandLineTools **is not supported**.

## Library dependents

1. **Qt v6.2.4 or later** for gui
     - If your Qt kit is compiled and configured by yourself, make sure you have `QtBase` and `QtNetwork` installed.
     - Qt must be compiled with gcc.
2. **Eigen v3.4.0 or later** for linear algebra
      - Can be downloaded automatically.
3. [**HeuristicFlow v1.6.2.1**](https://github.com/ToKiNoBug/HeuristicFlow.git) for mordern optimization algorithms
      - If the repo is not public, draw me an issue to get it.
      - Can be downloaded automatically.
4. **zlib v1.12.11 or later** for gzip compression
      - Usually it is installed with compiler so you do not need to compile it.
5. **Nlohmann json**
      - Can be downloaded automatically.
6. **libpng**
      - You must install it manually, and add the installation prefix to `CMAKE_PREFXI_PATH`.
7. **libzip**
      - You must install it manually, and add the installation prefix to `CMAKE_PREFXI_PATH`.
8. **fmtlib**
      - Can be download and compiled automatically.
9. **cli11**
    - Can be downloaded automatically.
10 **magic_enum**
    - Can be downloaded automatically.


## Projects in SlopeCraft

|    Project name    | Binary type | Dependents(external)                                                             | Description                                              |
| :----------------: | :---------: | :------------------------------------------------------------------------------- | :------------------------------------------------------- |
|   `imageCutter`    | Executable  | Qt base                                                                          | A unnecessary image preprocesser                         |
|    `MapViewer`     | Executable  | Qt base, Eigen, zlib                                                             | A tool to browse Minecraft map data files                |
| `BlockListManager` | Static lib  | Qt base                                                                          | A class to manage blocks that are avaliable for map arts |
|   `GAConverter`    | Static lib  | Eigen, HeuristicFlow                                                             | A converter based on genetic algorithm                   |
|   `SlopeCraftL`    | Shared lib  | Eigen, zlib HeuristicFlow                                                        | The kernel of SlopeCraft                                 |
|  `SlopeCraftMain`  | Executable  | Qt base, Eigen                                                                   | The executable of SlopeCraft                             |
|   `VisualCraftL`   | Shared lib  | libpng, libzip, Eigen, zlib, nlohmann json, maigc_enum, fmtlib, OpenCL(optional) | The kernel of VisualCraft                                |
|   `VisualCraft`    | Executable  | Qt base,  nlohmann json, magic_enum                                              | The gui executable of VisualCraft                        |
|       `vccl`       | Executable  | Qt base, magic_enum, fmtlib, cli11                                               | The CLI executable of VisualCraft                        |

## CMake arguments for SlopeCraft

You may need to pass other parameters to cmake. The cmake script of SlopeCraft recevies following parameters:

   |             Parameter              |  Type  |        Default value        | Description                                                                    |
   | :--------------------------------: | :----: | :-------------------------: | :----------------------------------------------------------------------------- |
   |        `CMAKE_PREFIX_PATH`         |  PATH  |             ""              | Tell cmake where to find Qt, zlib, libpng, libzip and GPU api sdk(like OpenCL) |
   |       `CMAKE_INSTALL_PREFIX`       |  PATH  | ${CMAKE_BINARY_DIR}/install | Where to install SlopeCraft.                                                   |
   |        `SlopeCraft_GPU_API`        | STRING |          "OpenCL"           | API used to compute. Valid values : OpenCL, None. Metal may be supported.      |
   |    `SlopeCraft_update_ts_files`    |  BOOL  |            false            | Whether to update language files before build                                  |
   | `SlopeCraft_update_ts_no_obsolete` |  BOOL  |            false            | Remove obsolete translations from ts files.                                    |
   |         `SlopeCraft_gprof`         |  BOOL  |            false            | Profile with gprof.                                                            |

## CMake generater for SlopeCraft

|    Generator    |      OS       | Description                                                                                                                                                                                                                                                                                        |
| :-------------: | :-----------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|      Ninja      | macOS & Linux | N/A                                                                                                                                                                                                                                                                                                |
| MinGW Makefiles |    Windows    | `windres.exe` can not process spaces in including directories correctly, which will cause compilation errors. **This is a bug in GNU bintuils, and Ninja have no way to avoid this error, but mingw makefiles does.** On other platforms, `windres` is no longer used, so the bug is also avoided. |

## Compilation steps

Now you have learned necessary informations about SlopeCraft. Click the following links to see compilation steps on different OS.

- [Windows/Linux](Windows-Linux.en.md)
- [macOS](macOS.en.md)
