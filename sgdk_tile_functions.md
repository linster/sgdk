This page will (At some point) cover all the tile functions in sgdk

## VDP\_loadTileData ##
VDP\_loadTileData is a function to load a tile usually from a c array.
It is defined as:
```
VDP_loadTileData(const u32 *data, u16 index, u16 num, u8 use_dma);
```
Code example:
This code will display a 16 color reduced version of this image:<br>
<img src='http://imageshack.us/m/864/6267/p1010667fixed.png' />
<br>
To compile you will need to download P1010667_pal.h and P1010667_tile.h<br>
<a href='http://www.mediafire.com/?aw8ygzkv4pceaqt'>http://www.mediafire.com/?aw8ygzkv4pceaqt</a>
<pre><code>#include "genesis.h"<br>
#include "P1010667_pal.h"<br>
#include "P1010667_tile.h"<br>
int main()<br>
{<br>
    //set up the screen<br>
    VDP_setScreenWidth320();<br>
    VDP_setHInterrupt(0);<br>
    VDP_setHilightShadow(0);<br>
    //Now display the picture<br>
    	//load a pallete in the cram<br>
	// arg0 the pallete line<br>
	// arg1 the pallete data<br>
<br>
    VDP_setPalette(PAL0, (const u16 *)P1010667_pal);<br>
<br>
        // load tiles in VRAM<br>
        //  arg0 = tiles data<br>
        //  arg1 = The first tile that is gonna be loaded<br>
        //  arg2 = number of tiles to load<br>
        //  arg3 = use DMA (1) or not (0)<br>
    VDP_loadTileData( (const u32 *)P1010667_tile, 1, 1120, 1);<br>
    //Fill the screen with the proper tiles by incremeting by one each tile<br>
    /*arg0 are we drawing on plane a or plane b defined as APLAN and BPLAN*/<br>
    //arg1 start tile<br>
    //arg2 x<br>
    //arg3 y<br>
    //arg4 width<br>
    //arg5 height<br>
    VDP_fillTileMapRectInc(APLAN, 1, 0, 0, 40, 28);<br>
<br>
    while (1)<br>
    {<br>
     VDP_waitVSync();<br>
    }<br>
}<br>
</code></pre>
Once you have compiled the example you should get something like this:<br>
<img src='http://img813.imageshack.us/img813/8813/kegafusionpicture.png' />
<h2>VDP_drawText</h2>
Sgdk supports drawing text using the VDP_drawText function and it is very easy to use however because c puts a null terminator at the end of every char array you must have an odd number of characters so it is recommended that you use a character counter like the one found here: <a href='http://www.javascriptkit.com/script/script2/charcount.shtml'>http://www.javascriptkit.com/script/script2/charcount.shtml</a>
the reason for this is because the code needed to be even aligned<br>
so I am going to modify my last example to include text<br>
since VDP_drawText draws on the a plane I will change it so that the tiles are draw on the b plane so change<br>
<pre><code>VDP_fillTileMapRectInc(APLAN, 1, 0, 0, 40, 28);<br>
</code></pre>
to<br>
<pre><code>VDP_fillTileMapRectInc(BPLAN, 1, 0, 0, 40, 28);<br>
</code></pre>
and before<br>
<pre><code>VDP_fillTileMapRectInc(BPLAN, 1, 0, 0, 40, 28);<br>
</code></pre>
add<br>
<pre><code>VDP_drawText("the text ",0,0);<br>
</code></pre>
notice how I used a space after "the text" this is because "the text" has an even amount of characters<br>
once you build it you should get something like this<br>
<img src='http://s7.postimage.org/94wn40o61/the_text.png' />
the text displayed but it was hard to see so how do we change that brings us to the next part of the tutorial<br>
<h2>VDP_setTextPalette</h2>
so right now we have 4 palette lines to chose from (0-3) so which one will show up on a blue background the best well here are our current choices:<br>
<img src='http://s18.postimage.org/euxvh8lc7/color_choices.png' />
the text will be draw using the last color in the row so I think that the bright green will show up best and that is on row 2 (counting from zero) so before<br>
<pre><code>VDP_drawText("the text ",0,0);<br>
</code></pre>
I will add<br>
<pre><code>VDP_setTextPalette(2);<br>
</code></pre>
I ended up with<br>
<img src='http://s14.postimage.org/hueff7qhr/green_text.png' />