# VisualCraft 教程

## 简介

VisualCraft 是一款全新的 Minecraft 像素画生成器，由 MC 玩家 TokiNoBug 开发，是 SlopeCraft 的子项目。与其他类似的软件不同，VisualCraft 旨在跟进最新的 MC 版本 (1.12~最新版)、支持最多的 MC 特性，提供最强大的功能。

目前 VisualCraft 能够解析许多第三方资源包，也允许自定义增加加新方块。与传统的思路不同，VisualCraft 以方块模型的方式来解析资源包，尽量贴近 Minecraft 的方式，因此支持各种自定义的方块模型。

在导出方面，VisualCraft 支持 Litematica mod 的投影 (**\*.litematic**)、WorldEdit 原理图 (**\*.shem**)(仅 1.13+可用)、原版结构方块文件 (**\*.nbt**)、平面示意图 (**\*.png**) 等方式。

VisualCraft 支持用各种透明方块互相叠加，产生更多的颜色。软件最多支持不超过 65534 种颜色，受此限制，像素画的层数不超过 3 层。

由于颜色数量很多，VisualCraft 使用了显卡加速。目前支持的 API 有 OpenCL，但也能只用 CPU 计算。
