# Frequently Asked Questions

!!! warning

    Before submitting an issue, please make sure that the bug has been fixed in the latest version!

1. Can I use it on Bedrock Edition/Netease version?

      - I have never played these versions, so I don't know if it can be used, and I don't know how to use it.
      - **Theoretically**, as long as the version is Java and Litematica mod is installed successfully, it can be used.
      - There is no Litematica mod for Bedrock Edition, so, **in theory**, it can't be used.

2. I want to build in a server, but the server doesn't have the **Litematica mod** installed.

      - First of all, in the interface and all documents of this software, we only needs the **Litematica mod** developed by Masa. Any other mods with the word "projection" or similar functions are not what we need.
      - Litematica mod is a **client mod**, **server installation is not needed**. Only the client needs to install Litematica and its dependency mod Malilib, and then the projection mod can be used normally. So don't ask this kind of stupid question anymore.
      - [Litematica Mod - CurseForge](https://www.curseforge.com/minecraft/mc-mods/litematica)
      - [Litematica Mod - MC Mod](https://www.mcmod.cn/class/2261.html)

3. Where is SlopeCraft.exe/SlopeCraft.app/SlopeCraft Executable File?

      - DO! NOT! DOWNLOAD THE SOURCE CODE!!!
      - How to download the application from GitHub: On the repo page, there is a release section on the right side of the page. Click it and you can see the download links.

4. How to make a non-square pixel arts?

      - Why do people think that SlopeCraft can't make non-square pixel arts?
      - SlopeCraft will **never** scale your image, but reproduce each pixel as it is. **You can make any size of pixel arts, just handle it normally**.

5. Why is the picture almost gray after adjusting the color in SlopeCraft? / Why is the color not ideal / ...

      - There is some color that are set by Mojang, and there are some colors that don't have a similar map color, such as light blue, gray blue, light pink, light purple, etc. You can only find colors that don't differ too much.
      - A dithering algorithm has been implemented in v3.5, which can solve this problem.

6. Why can't I get the map item with the `/give` command?

      - In 1.12, use `/give @p filled_map 1 i` to get the map with the serial number i
      - In 1.13+, use `/give @p filled_map{map:i}` to get the map with the serial number i

7. Can I scale down the map?

      - It is not recommended to scale down the map, because the resolution of the map is still 128*128 pixels, no matter whether it is scaled down or not, and the quality will only be reduced. The best way to increase the resolution is to use multiple maps without scaling.
