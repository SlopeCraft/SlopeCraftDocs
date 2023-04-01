# Principal of Map Pixel Art

This article will introduce the working principle of map pixel art in form of 3D map, 2D map and map data file. SlopeCraft was developed based on these mechanisms. By studying and using these mechanisms, people can make the map display custom images.

!!! tip
    `Map` mentionned in this article refers to the map item in Minecraft rather than the saving file.

## Why are we using maps?

We can seperate Minecraft pixel art into two types: visual and map. Visual map pixel art is built to be viewed directly. It can be placed horizontally (on the floor or on the ceiling) or vertically (on the wall). It is straightforward and simple, but it is often very large and not very convenient to view. However, maps can be placed in item frames and displayed on the floor or walls. Multiple maps can even be placed together to form a large image. This is the advantage of maps.

In visual pixel art, blocks shows the color of their texture directly. Each block is different. However, this is not the case for map pixel art. The color shown on the map might be slightly different from the color of the texture, and many blocks have the same color, such as snow and white concrete. In addition, for most blocks, the color shown on the map have some minor but noticeable differences from the color on the placed block. Many visual pixel art looks bad on the map because of this. So the first thing to do is to understand the mechanism of Minecraft maps.

## How does the map show blocks?

**The resolution of each map is 128 × 128 pixels, regardless of the scale.** The area shown by the map is determined by the `scale`, which is an integer ranging from 0 to 4 (including 0 and 4). The area shown by the map is a square, with a length and width of $128\times 2^{Scale}$. When `scale` is 0, the map corresponds to a 128 × 128 range of blocks, where each pixel represents a block; otherwise, each pixel represents $2^{Scale}$ blocks. Maps with `scale` greater than 0 are meaningless for map pixel art, so I won't discuss them anymore, but I will tell you why they are meaningless.

To make the map easier to put together, each map will automatically align with a grid. The starting coordinate of the map (the northwest corner) is $(128\times 2^{Scale}-64,y,128\times 2^{Scale}-64)$.

The table below shows the relationship between the coordinate axis and the four directions.

| Direction | Map Direction | Coordinate Axis Direction |
| :-------: | :-----------: | :-----------------------: |
|   North   |      Up       |            z-             |
|   South   |     Down      |            z+             |
|   West    |     Left      |            x-             |
|   East    |     Right     |            x+             |

1. How does Minecraft store map data?

    In Minecraft Java Edition, the content of the map is not stored in each map item, but in the **data** folder. Their file names are in the form of **map_i.dat**, where `i` is a natural number under 2,147,483,647 (32767 in 1.12). `i` is the map number, and each map item in the game only stores the corresponding map number.

    For example, if a player holds a map with number 114, when the save is loaded, Minecraft will find the map data file named *`map_114.dat`* and load it. Then its content will be displayed on the map. Therefore, the essence of the map is not the item, but the map data file.

    Nearly all the data of the map is stored in the map data file, including but not limited to each pixel. The most important thing about the map is how the map data is stored.

2. Map pixels and MapColor

    Each pixel of the map is an 8-bit unsigned integer, ranging from 0 to 255. Since the map data file is in compressed NBT format, the map pixels are stored in a byte array containing 16384 elements (although the byte in NBT format is a signed integer, it doesn't matter to treat it as an unsigned integer). This 128 × 128 pixel matrix is stored in row prioritized order.

    This determines that each pixel may have at most 256 colors, and each value corresponds to a different color. **This 8-bit unsigned integer is the map color.** We can say that the map is an 8-bit index image with a fixed color table.

3. BaseColor and Shadow

    As mentioned above, the map color is an 8-bit unsigned integer. The high 6 bits form an 6-bit unsigned integer, which is the base color (BaseColor), and the remaining 2 bits form another 2-bit unsigned integer, which is the the shadow. The relationship between the map color, the base color and the shadow is:

    $$
    MapColor = 4 \times BaseColor + Shadow
    $$

    The base color depends on the block type. There may be up to 64 base colors in the game. As of the latest version (1.17 when this article was written), 62 base colors have been used in the game; in 1.16, 59 base colors were used, and each base color had a corresponding color (RGB); from 1.12 to 1.15, 52 base colors were used. Unused base colors have no corresponding color, and they are meaningless to the map.

    On Minecraft wiki, there is a table of base colors and blocks: [Map Item Format](https://minecraft.fandom.com/wiki/Map_item_format).

    But the base color is not the color of the map color. The R, G, and B components of the base color must be multiplied by a factor, and then divided by 255 (rounded down) to get the color of the map color. This factor is determined by the shadow value.

    | Shadow |  Factor  |
    | :----: | :------: |
    |   0    |   180    |
    |   1    |   220    |
    |   2    |   255    |
    |   3    |   135    |

    As you can see, the map color with shadow value `2` is the same as the base color, the map color with shadow value `1` is slightly darker, the map color with shadow value `0` is darker, and the map color with shadow value `3` is the darkest.

    What is the meaning of the shadow? The shadow represents the relative height of a block compared to the surrounding blocks. If the coordinates of block A are $(x,y_A,z)$, and the coordinates of the block B northbound of A are $(x,y_B,z-1)$, then the shadow value of A is

    $$
    Shadow(A)=\left\{
    \begin{aligned}
        0\quad,&\quad \text{if } y_A<y_B \\
        1\quad,&\quad \text{if } y_A=y_B \\
        2\quad,&\quad \text{if } y_A>y_B
    \end{aligned}
    \right.
    $$

    In all the base colors, water (`12`) is the most special. **The shadow value of water is not related to the relative height, but the water depth.** When the water depth is 1, the shadow value is `2`; when the water depth is 6, the shadow value is `1`; when the water depth is equal to or greater than 11, the shadow value is `0`.

    If a block is higher than the block to the north, it will be brighter; (if the water is shallower, it will be brighter), **which allows the map to show the ups and downs of the terrain**.

    **To be noted, the shadow is a 2-bit unsigned integer, which can have 4 values, but only the first 3 are used. The remaining shadow value 3 can work normally on the map, but it is impossible to get in the original survival.** I can't figure out what Mojang was thinking.

4. `0` is a special base color

    Base color `0` represents **air** or **unexplored**, which is completely different from all other base colors. The color corresponding to base color `0` is fully transparent, and only the background color of the map item (or the texture of the item frame) can be seen. Many transparent blocks belong to base color `0`, such as glass, nether portal, torch, etc. It is very strange that redstone lamp also have a base color `0`.

5. Why is it meaningless to scale the map?

    According to the map mechanism, scaling the map cannot increase the resolution of the map, nor can it bring more colors. So building a larger map by scaling the picture is purely a waste of blocks and space.

## How does the map pixel art work?

1. 3D map mechanism

    In 3D map pixel arts, blocks are placed in a specific position to form a 3D structure. Each block has two functions:

    - Show the base color of itself
    - Determine the shadow of the block southbound of itself

    As you can see, the height of each block is not random, but determined by the map color.

2. 2D map mechanism

    If you want the map to be only composed of base colors with shadow value `1` (or water with shadow value `2`), you will get a 2D map pixel art. Each block has the same height.

3. Map data file mechanism

    For both 2D and 3D map pixel arts, you will have to build the generated structure in the game first, and then use the map to record it. This is indeed the only way to build a map pixel art in the vanilla survival mode. But if you don;t care if it is vanilla, you can also replace the map data file directly. The map pixel art in form of data files can be put directly into the saving folder, or replace the existing map data file with the generated one, and then get the corresponding map item with the command. **This is the only way to use the third shadow.**

## What is height compression?

Height compression is a new technology that can reduce the maximum height of 3D map pixel arts. Since large map pixel arts can very easily exceed the height limit, compression is very meaningful. Currently, there are two ways of compression: **lossless compression** and **lossy compression**.

1. Lossless compression

    When lossless compression is used, the display effect of the map is completely unchanged. It tries to sink some parts of the map.

    During the compression, SlopeCraft processes each column of the map independently.

    $$
    \displaylines{
        \Delta H_i=\left\{
            \begin{aligned}
                Shadow_i-1\quad &,\quad \text{if }BaseColor_i\neq 0,12 \\
                0 \quad &,\quad \text{else}
            \end{aligned}
        \right.
        \newline
        H_i=\sum_{j=0}^i \Delta H_j
        \newline
        maxHeight=\max H
    }
    $$

    In the formula above, $\Delta H$ is the height difference between each block and the block northbound, and $H$ is the height of each block. This formula is slightly different from the actual source code, but the basic principle is the same.

    The formula above limits the height difference to `-1`, `0` or `1`, but this restriction is not necessary. For example, a block with shadow `2` only requires its height difference with the block northbound to be greater than `0`, but not necessarily `1`. This provides an opportunity for lossless compression.

    Lossless compression also does special processing for water and air blocks, because their shadow values are not related to the height difference. Although this makes the code implementation much more difficult, the compression effect will be better.

    **The lossless compression cannot promise to compress the height to 256 or less. Sometimes a column of the map may even be monotonic increasing/decreasing, which is not compressible.**

2. Lossy compression

     Apparently, lossless compression cannot solve the problem completely. To compress the total height of the map to any value, it is also acceptable to slightly adjust the map color of some pixels. The lossy compression algorithm will slightly adjust the map color of some pixels to compress the total height, and also ensure that the color difference is as small as possible.

     **Lossy compression can compress the map to any height.** Because it is implemented with genetic algorithm, the performance of lossy compression is somewhat random and relatively slow.

    Although the total height can be theoretically compressed to any value, **I don't recommend that you set the maximum allowed height to a value less than `14`**, otherwise the genetic algorithm needs a very looooong time to compress - it may even fail to compress.

    And it is worth mentioning that the above two methods are parallel, you can use both methods at the same time, or only use one. If you have enabled lossy compression, I recommend that you use the lossless compression at the same time, which can reduce the damage to the quality caused by lossy compression.

## Glass Bridge in 3D Map Pixel Art

If you take any horizontal slice of a 3D map pixel art, you will find that there are many isolated blocks on the slice. This structure is really difficult to build in vanilla survival, even with the help of schematica mod. **But if we connect these isolated blocks together, it will be easier to build.**

The glass bridge is this "connection". Using Prim algorithm can connect all blocks in a layer with the fewest glass, and this process is called bridge building. Usually, not every layer needs to be bridged, otherwise a lot of glass will be wasted. By default, the interval is 4 layers, which means a bridge will be built every 5 layers.

Considering the time complexity of Prim algorithm is $O(n^3)$, it will take some time. But compared to the time spent on building, it is actually saving time.
