# 常见问题

!!! warning "注意"
    提交 issue 反馈 bug 之前，请确认最新版本是否已经修复了该 bug！

1. 基岩版/网易版能不能用？

      - 我从来不玩这些版本，没有研究过，不知道能不能用，不知道怎么用。
      - **理论上**无论是国际版 java 还是网易 java，只要是 Java 版且成功安装了 Litematica mod，那么就可以使用。
      - 基岩版目前没有 Litematica mod，所以**理论上**不可以使用。

2. 想在服务器里造，但服务端没装**投影 mod**

      - 首先强调，本软件的界面及一切文档中，投影 mod 一律指 **Litematica mod**，开发者为 Masa。其他任何含“投影”字样或类似功能的 mod，都不是本软件需要的。
      - Litematica mod 是**客户端 mod**，**不需要服务端安装**它。只需要客户端安装了 Litematica 和它依赖的前置 mod Malilib，即可正常使用投影 mod。所以不要再问这种蠢问题了。
      - [Litematica Mod - CurseForge](https://www.curseforge.com/minecraft/mc-mods/litematica)
      - [Litematica Mod - MC 百科](https://www.mcmod.cn/class/2261.html)

3. SlopeCraft.exe/SlopeCraft.app/Slopecraft 可执行文件 是在文件里的哪个地方？

      - 不！要！下！载！源！代！码！！！！！！！
      - Github 下载教程：在 repo 的主界面 (main)，在网页右边有一个 release 的地方，点击之后就能看到用户可以使用的下载方式了

4. SlopeCraft/VisualCraft启动时，报“应用程序无法正常启动（0xc0000142）”的错误
   
      - 这通常是因为你的运行环境不支持 AVX2 指令集。你可以在 Release 里下载标有`noAVX2`字样的压缩包，这是完全不使用 AVX 加速的版本，可以在更旧的硬件下运行。
      - 可能的原因：
        1. 在虚拟机里启动。虚拟机常常不支持 AVX 和 AVX2
        2. 电脑的 CPU 型号比较老旧。Intel 的酷睿 CPU 从 4 代（2013 年）开始支持 AVX2，2013 年以前的 CPU 一定不支持 AVX2。

5. 是否支持 windows7？
   
      - 不再支持，做不到了。而且微软已经淘汰 win7。

6. 该怎么制作非方形的地图画？

      - 为什么有人会认为 SlopeCraft 不能制作非方形的呢？
      - SlopeCraft**从来不会缩放你的图片**，而是忠实的还原每一个像素。**你可以制作任何尺寸的地图画，照常处理就行**。

7. SlopeCraft 点了调整颜色之后图片几乎是灰的？/为什么颜色不理想/……

      - 有哪几种颜色是 Mojang 钦定的，有一些颜色就是没有特别相似的地图色，比如浅蓝、灰蓝、浅粉、浅紫，只能寻找差别不是太大的颜色。
      - v3.5 已经实现了抖动/仿色算法，它可以解决这个问题。

8. `/give` 命令得不到地图物品怎么办？

      - 1.12 使用 `/give @p filled_map 1 i`，获得序号为 i 的地图
      - 1.13+使用 `/give @p filled_map{map:i}`，获得序号为 i 的地图

9.  可不可以把地图缩小？

      - 不建议，因为无论缩小与否，地图的分辨率仍然是 128*128 像素，画质只会降低。提高分辨率的最好方式还是用多张无缩放的地图组合。

10. 导出的投影文件不能正常读取？

      - 最好不要在**投影区域名称**中写中文，否则可能会因为汉字编码格式问题而导致乱码。
