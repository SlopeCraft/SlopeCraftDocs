# 欢迎阅读SlopeCraft文档

![SlopeCraft](_static/image/SlopeCraft.png)

请访问 [mkdocs.org](https://www.mkdocs.org) 查看完整文档。

## 关于SlopeCraft
SlopeCraft是一个Minecraft像素画/地图画生成器，包含如下几个工具：

|    名称     | 介绍                                                                           |
| :---------: | :----------------------------------------------------------------------------- |
| SlopeCraft  | 生成用于Minecraft地图的生成器，包括但不限于立体地图和，**不适合玩家直接观看**  |
| VisualCraft | 生成普通像素画的生成器，**适合玩家直接观看**，支持俯视、侧视和仰视视角的像素画 |
| imageCutter | 缩放、切割图像用的辅助工具                                                     |
|  MapViewer  | 浏览Minecraft地图数据文件的辅助工具                                            |
|    vccl     | 与VisualCraft功能相同的命令行工具                                              |


## 为什么要有SlopeCraft？

很多玩家在建造地图画之后，会习惯于用地图记录地图画，然后把地图挂在墙上。但这时候细心一点的人就会发现地图呈现的颜色与直接看到的差异很大。SlopeCraft就是为了克服这种差异而诞生的，也因此，SlopeCraft生成的地图画在设计之初就不是给玩家直接观看的。

一个常见的误区是，SlopeCraft允许生成**平板地图画**，但平板地图画与VisualCraft的任务是完全不同的。平板地图画在设计之初就不是给玩家直接观看的，而只是立体地图画的一个亚种，它只是二维的立体地图画。

## 为什么要有VisualCraft？

传统的像素画生成器确实已经很多了，看起来VisualCraft似乎不是很必要。但我实现了几个新颖的技术：

1. 将多个半透明方块叠加在不透明发方块上，组成上万种不同的新颜色，极大提升像素画转化的效果。
2. 根据生物群系正确计算树叶和草的颜色
3. 通过解析方块模型，支持第三方材质包自定义的方块模型

为了实现这些功能，我开发了VisualCraft。

## 命令

* `mkdocs new [dir-name]` - 创建一个新的项目。
* `mkdocs serve` - 启动一个开发服务器。
* `mkdocs build` - 构建静态网站文件。
* `mkdocs -h` - 获取命令帮助。

## 项目文件结构

    mkdocs.yml    # 配置文件。
    docs/
        index.md  # 文档首页。
        ...       # 其他 markdown 页面，图片和其他文件。
