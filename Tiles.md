## Tiles ##

If you read documents linked on [first part](#Introduction.md), you should know how to write more than 'Hello World' on screen :
  * the Genny redraws 2 planes on refresh (+ a third one for the sprites)
  * each plane is filled with 8x8 pixel tile
  * each plane could be 32x32 to 128x128 tiles in memory (only up to 40x28 is visible on screen)
  * each plane is filled from left to right and top to bottom
  * each tile could be used several times in any plane, with the same pal or not
  * each tile could be used with any of the 4 pal available
  * each tile could be drawn flipped, w/o more memory
  * a very important amout of tiles could be loaded using DMA

### Basic ###
Some other data you should know :
  * one tile is made of 32bytes : 4byte per line
  * each pixel of the tile is so a 4bit value which is the color index, from 0x0 to 0xF
  * the first tile (tile 0) on VRam will be used to fill the background
  * SGDK initialize enought space on VRAM for 1310 tiles (+ 96 for the font)
  * a tile is also called a pattern or a char
  * a tile on screen isn't removed on refresh (ie no need to draw it each refresh!)
  * a pal is 16 colors wide

So, the steps to draw a tile on screen are the following
  1. load the tile on VRAM (we don't bother about DMA yet)
  1. load its pal (if not already loaded)
  1. plot it on the selected plane, at (x,y) with the wanted pal

But first, we need a tile !

Rescomp and others tools made by the community ([Genitile](http://www.pascalorama.com/article.php?news=28&cat=21), [B2T](http://gendev.spritesmind.net/page-b2t.html), [GenRes](http://gendev.spritesmind.net/page-genres.html), [Mega-Happy-Sprite](http://uberleethackerforce.deepgeek.us/prods.html)
...) let you convert a 16 color bitmap in Genny's tile and pal.

But, to learn, we'll first make it the hard way : pure C Array !

```
const u32 tile[8]=
{
		0x00111100,
		0x01144110,
		0x11244211,
		0x11244211,
		0x11222211,
		0x11222211,
		0x01122110,
		0x00111100
};
```

Our tile is only made of color 0, 1, 2 & 4.

Now, just follow the steps

```
	// ... code
	
	//we load our unique tile data at position 1 on VRAM
	VDP_loadTileData( (const u32 *)tile, 1, 1, 0); 
	
	//we'll use one of the pre-loaded pal for now
	
	//write our tile 1 on plane A at (5,5) with pal 0
	VDP_setTileMapXY(APLAN, 1, 5, 5);
	
	// ... code
	
	while(1)
	{
		// ... code
		VDP_waitVSync();
	}
	
```

The tile is so drawn on screen using some fade of grey...but what about flipping and pal ?

The 2nd argument of `VDP_setTileMapXY` func could be more than only the tile index.

It is, in fact, the tile properties : pal index, priority, vflipping, hflipping and tile index.

So, to write it flipped green on B plane, you could write

```
	// ... code
	
	//PAL2 = green pal
	// 0 = low priority
	// 1 = vflip
	// 0 = no hflip
	// 1 = tile 1
	VDP_setTileMapXY(BPLAN, TILE_ATTR_FULL(PAL2, 0, 1, 0, 1), 6, 5);
	
	// ... code
```


It's now time to talk about the priority flag.

This flag tell which plane is behind the others (even more complex if you add sprites !).

A powerful feature of the Genny is to let the developer define the planes order at the tile level.

It means a tile on A plane could be in front of a tile on B plane at (x,y) and inverse the priority at (n,m).

You could easily see the power of this when scrolling planes at different speed for example.

```
	// ... code
	
	// the same tile is drawn on the 2 plane but with different pal and priority...which one is in front ?
	VDP_setTileMapXY(APLAN, TILE_ATTR_FULL(PAL1, 1, 0, 0, 1), 7, 7);
	VDP_setTileMapXY(BPLAN, TILE_ATTR_FULL(PAL2, 0, 0, 0, 1), 7, 7);
	VDP_setTileMapXY(APLAN, TILE_ATTR_FULL(PAL1, 0, 0, 0, 1), 8, 7);
	VDP_setTileMapXY(BPLAN, TILE_ATTR_FULL(PAL2, 1, 0, 0, 1), 8, 7);
	
	// ... code
```

Here, at (7,7) plane A is in front of plane B but, at (8,7), plane B is in front of plane A.


But, what will happen if the planes get the same priority ?

The default order is kept : plane A is in front of plane B.


### Fill ###

To fill a screen, you have to draw each tile one by one !

Very boring but, hopefully, SGDK came to the rescue with `VDP_fillTileMapRect`.

```
	// ... code
	
	// fill a 8x8 square of blue tile at (12,12)
	VDP_fillTileMapRect(BPLAN, TILE_ATTR_FULL(PAL3, 0, 0, 0, TILE1), 12, 12, 8, 8);
	
	// ... code
```


Download : [Basic tiles project](http://sgdk.googlecode.com/svn/wiki/files/tut3_TilesBasic.zip)


### Multi tile ###

It will be hard to make a game using one and unique tile.

We'll try now to draw this moon on screen

![http://sgdk.googlecode.com/svn/wiki/pictures/tutTile-blue-moon.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/tutTile-blue-moon.jpg)

2 important things to check before to continue
  1. the bitmap size should be 8 pixel aligned (ie : 64x32 is good where 67x31 isn't)
  1. the bitmap should be 4bpp or 8bpp and respect megadrive color constraints (16 colors per tile and no more than 64 colors at max)...

SGDK is able to handle many type of resources file (PNG file are also accepted), you just need to define them in a .res file and rescomp will compile them (see rescomp.txt to see how to define your resources) and generate a 'name\_of\_res\_file.h` and a `name\_of\_res\_file.o`

`name_of_res_file.o` is the object form of your compiled resources.

`name_of_res_file.h` contains the resource declarations and let you access, for instance, your bitmap image as a Bitmap data structure.

so, including `name_of_bmp_file.h`, you could load the bitmap on VRAM this way

```
#include "moon.h"

int main( )
{
	// get the image width (in pixel) ==> should be 8pix aligned
	u16 w = moon.w;
	// get the image height (in pixel)  ==> should be 8px aligned
	u16 h = moon.h;

	// get the palette data of moon
	VDP_setPalette(PAL1, moon.palette->data);

	
	// load bitmap data of moon in VRAM
	// w/8 = width in tiles we want to load
	// h/8 = height in tile we want to load
	// w/8 = width in tiles of the bitamp
	// the 3rd arg is needed because you could load only a part of the bitmap if you want but SGDK needs the width as reference
	VDP_loadBMPTileData(moon.image, 1, w / 8, h / 8, w/8 );


	while(1)
	{
		VDP_waitVSync();
	}
	return 0;
}
```

For this moon, you have 8x8 tiles loaded.

You could draw them by hand using `VDP_setTileMapXY` but it's boring and, as usual when it's a boring process, SGDK has a magical function for it : `VDP_fillTileMapRectInc`.

This function draw FROM the tile index in argument to the last needed to fill the rect wanted.

```
	// draw the moon at (12,12)
	VDP_fillTileMapRectInc(BPLAN, TILE_ATTR_FULL(PAL1, 0, 0, 0, 1), 12, 12, w / 8, h / 8);
```


Download : [Bitmap tiles project](http://sgdk.googlecode.com/svn/wiki/files/tut3_TilesBitmap.zip)

### GenRes's BITMAP support ###

SGDK comes with another way to load tiles: using _**GenRes**_.

GenRes is a tool which, actually, converts bitmap, animation & map for Genny development.

For this part of the tutorial, we will only talk about the `BITMAP` mode.

GenRes relies on a resource declaration file where each line defines the convert mode, the output name, the file and some parameters.

In your case, you will use the `BITMAP` convert mode this way :

```
	BITMAP output_var_name "directory/file.bmp" 0
	; always finish with a blank line or comment
```

If you write it down in a `resource.rc`, SGDK will call GenRes to compile `directory/file.bmp` to a linked `resource.o`.

For version up to 0.7d, GenRes output format is undefined in SGDK, so you have to write your own format structure like this :

```
	struct genresTiles
	{
			u16 *pal; 		//pointer to pal data
			u32 *tiles;		//pointer to tiles data
			u16 width;		//width in tiles
			u16 height;		//height in tiles
			u16 compressedSize; //0
	};
```

Unlike SGDK's native bitmap support, it doesn't generate a header file but you could access the data using your `output_var_name`.
```
	extern struct genresTiles output_var_name;
```


`VDP_loadBMPTileData` is dedicated to BMP. Since GenRes converts a bitmap to tiles data, you need to use another VDP function, dedicated to tiles : `VDP_loadTileData`.

```
	// .... code
	
	// load tiles in VRAM
	//  arg0 = tiles data
	//  arg1 = index for first destination tile
	//  arg2 = number of tiles to load
	//  arg3 = use DMA (1) or not (0)
	VDP_loadTileData(moon.tiles, TILE1, moon.width*moon.height, 0);

	// .... code

```

For pal loading and drawing, you still need `VDP_setPalette` and `VDP_fillTileMapRectInc`.

And so, you could draw the moon this way :

```
	// .... code

	VDP_setPalette(PAL1, moon.pal);
	VDP_loadTileData(moon.tiles, TILE1, moon.width*moon.height, 0);
	VDP_fillTileMapRectInc(BPLAN, TILE_ATTR_FULL(PAL1, 0, 0, 0, TILE1), 12, 12, moon.width, moon.height);

	// .... code
```


To the question _"SGDK native bitmap support or GenRes ?"_, it's only a matter of personal taste and not the goal of this tutorial.


Download : [Genres tiles project](http://sgdk.googlecode.com/svn/wiki/files/tut3_TilesGenRes.zip)


### Compression ###

If you use a large amount of tiles, we strongly suggest you to compress them (using RLE, huffman, ...).

SGDK doesn't include (yet?) a method to load compressed bitmap data so it's up to you to make your own (de)compression algorithm.

GenRes BITMAP could use a RLE-based compression. _more details soon_

Check if the tool you use could export compressed data too.

### Misc ###

One useful way to test your tile engine is throught [Gens KMod](http://gendev.spritesmind.net/page-gensK.html).

You could explore the VRAM and trace any issue.

![http://sgdk.googlecode.com/svn/wiki/pictures/tutTile_Kmod.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/tutTile_Kmod.jpg)