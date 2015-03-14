## Table of Content ##
  1. **[Introduction](#Introduction.md)**
  1. **[Hello World !](#Hello_World.md)**
  1. **[Read the joypad](#Input.md)**
  1. **[Tiles](#Tiles.md)**
  1. **[Sprites](#Sprites.md)**
  1. _more_
  1. **[History and changes](#History.md)**


---


## Introduction ##
These several basic tutorials aim to give you the basis to start developing on the Sega Genesis / Megadrive using SGDK.

Written in C language (ASM is out of scope), you'll be able to fully understand them if you already have basic knowledge on how the Genny works.

So, before you start reading the tutorials, be sure to check and read these others documents:
  * [Genesis Technical Overview](http://emudocs.org/Genesis/sega2f.htm) by Sega
  * [Sega Genesis VDP documentation](http://cgfm2.emuviews.com/txt/genvdp.txt) by Charles MacDonald
  * [Sega Genesis hardware notes](http://cgfm2.emuviews.com/txt/gen-hw.txt) by Charles MacDonald
  * Sega Genesis I/O Chip and Peripherals by Charles MacDonald (_dead link_)


---


## Hello World ##

Whenever you use [Code::Block](UseSGDKWithCB.md), [Eclipse](UseSGDKWithEclipse.md), [Notepad](HowToUseSGDK.md) or any other souce code editor, you need your `main` function.

```
int main( )
{
	return (0);
}
```


This function is your starting point and is automatically called on start up.

Before that, SGDK initializes several things for you  :
  * set default values for each [VDP register](http://info.sonicretro.org/SCHG:VDP_Documentation/General/Registers)
  * clean Video RAM
  * load 4 default palettes : grey, red, green and blue
  * load a default font for your text print need
  * init input handling
  * reset sound and music
  * load a default sound driver ("2 channels PCM sample" driver in SGDK 0.7)

so, for basic stuff and testing, everything is ready to use!
It's so easy to call VDP, input or sprites stuff right on your main function....if you don't forget to include the `genesis.h` header!


For now, let's see how to print the infamous _**Hello World**_.

Printing on screen is a VDP task so you need to call `VDP_drawText(<your_text>, <x>, <y>)`.

Remember, x and y are in tile unit, not pixel unit with 1 tile = 8 pixels.

Ex:
```
#include <genesis.h>

int main()
{
	VDP_drawText("Hello World!", 10, 13);
	return (0);
}
```

This code is correct but not Genesis friendly : do you think we can return (and stop) a program ?

Video games repeatly update the TV screen, and it's up to you to handle things before or while a refresh.

So a more Genny Hello World is more like this :
```
#include <genesis.h>

int main()
{
	VDP_drawText("Hello Genny World!", 10, 13);

	while(1)
	{
		//read input
		//move sprite
		//update score
		//draw current screen (logo, start screen, settings, game, gameover, credits...)
	}
	return (0);
}
```

This code is correct, even if `return(0)` will never be reached.

The code inside the while loop will execute in full speed...great! but do you think the Genesis will handle so much ?

The TV screen is updated 50 (PAL) or 60 (NTSC) times per second...you have to be sync with this.

An usefull way is to wait for screen update, or vertical synchronisation.

SGDK is here to help you with the `VDP_waitVSync` func.

So, this time, you could write your first Genesis fully compliant demo.

Ex:
```
#include <genesis.h>

int main()
{
	VDP_drawText("Hello Genny World!", 10, 13);

	while(1)
	{
		//read input
		//move sprite
		//update score
		//draw current screen (logo, start screen, settings, game, gameover, credits...)

		//wait for screen refresh
		VDP_waitVSync();
	}
	return (0);
}
```
Congratulations! You're now ready to make a little more!

Download : [Sample Hello World project](http://sgdk.googlecode.com/svn/wiki/files/tut1_HelloWorld.zip)


---


## Input ##

### Basic ###
You'll now learn how to handle input from controllers

The first thing to do is to call `JOY_init()`, this will reset the handler and read informations about the connected controllers.

SGDK uses a call back function to handle input.

This way, you don't need to poll each loop the data port but to wait for you callback function to be ... calle everytime the input change (button A released, left pad pressed, ...).

Using `JOY_setEventHandler`, you had to register this callback function to be called.

It's very important to define your callback function with this signature : `void myJoyHandler( u16 joy, u16 changed, u16 state)`.

It's also good practice to always use SGDK defined constants.

Ex:
```
void myJoyHandler( u16 joy, u16 changed, u16 state)
{
	if (joy == JOY_1)
	{
		if (state & BUTTON_START)
		{
			//player 1 press START button
		}
		else if (changed & BUTTON_START)
		{
			//player 1 released START button	
		}
	}	
}

int main( )
{
	JOY_init();
	JOY_setEventHandler( &myJoyHandler );

	while(1)
	{
		VDP_waitVSync();
	}
	return 0;
}
```

Download : [Basic input project](http://sgdk.googlecode.com/svn/wiki/files/tut2_Input.zip)

<a href='Hidden comment: 
=== Multipad ===

This will work for up to 4 players using a [http://www.segaretro.org/4_Way_Play EA4Way] and up to 8 players using 2 [http://www.segaretro.org/Team_Player Sega TeamPlayer].

If you plan to support multiplayers, remember to use the correct joy IDs : JOY_1/JOY_1A, JOY_1B, JOY_1C, JOY_1D, JOY2/JOY_2A, JOY_2B, JOY_2C, JOY_2D.

Of course, you must be able to detect if there is a !TeamPlayer or a EAWay connected.

There are actually 2 ways to know this :
# throught the port (PORT_1 or PORT_2) using JOY_isMultipad( port ).
```
This will return 0 if it"s not a multiplayer mode, JOY_SUPPORT_EA4WAY for an EA4Way and JOY_SUPPORT_TEAMPLAY for... a Sega TeamPlay
```
# throught the joypad itself using JOY_getJoyID( joy )
```
This will return the ID of the joy.
If (ID & JOY_SUPPORT_EA4WAY), the joypad is connected to a EA4Way
If (ID & JOY_SUPPORT_TEAMPLAY), the joypad is connected to a SegaTeam play
```


Ex:
```
void myJoyHandler( u16 joy, u16 changed, u16 state)
{
	if (joy == JOY_1B) //2nd player on multitap
	{
		if (state & BUTTON_START)
		{
			//player 3 press START button
		}
		else if (changed & BUTTON_START)
		{
			//player 3 released START button	
		}
	}	
}

int main( )
{
	JOY_init();
	JOY_setEventHandler( &myJoyHandler );

	while(1)
	{
		VDP_waitVSync();
	}
	return 0;
}
```


JOY_getJoyID gives you ever more information about the current joypad.

Actually, SGDK is able to give you the ID of
* 3 buttons joypad (ID & *JOY_SUPPORT_3BTN*)
* 6 buttons joypad (ID & *JOY_SUPPORT_6BTN*)
* Sega !TeamPlayer (ID & *JOY_SUPPORT_TEAMPLAY*)
* EA 4Way or !TeamPlayer in Extra mode (ID & *JOY_SUPPORT_EA4WAY*)
* Sega Mouse (ID & *JOY_SUPPORT_MOUSE*)
* Konami Justifier (ID & *JOY_SUPPORT_JUSTIFIER*)

The informations are defined on JOY_init() and after at least one refresh.

Ex:
```
int main( )
{
	JOY_init();
	VDP_waitVSync();
	
	if (JOY_getJoyID(JOY_1) & JOY_SUPPORT_JUSTIFIER)
	{
		VDP_drawText("JUSTIFIER NOT YET SUPPORTED", 10, 10);
	}

	while(1)
	{
		VDP_waitVSync();
	}
	return 0;
}
```

Yes, SGDK doesn"t handle Justifier yet, we"re working on it but it handles mouse!

=== Mouse ===

Mouse support is almost the same as joypad but you had to register another function to handle mouse move with JOY_setMouseEventHandler.

Similary, the signature of your callback function must be void myMouseHandler( u16 joy, s8 xDelta, s8 yDelta, u16 buttonState).

Ex:
```
s16 mouseX=0, mouseY=0;
void myMouseHandler( u16 joy, s8 xDelta, s8 yDelta, u16 buttonState)
{
	char str[4];
	u8 mouseState;

	//move only if left pressed
	if ( buttonState & MOUSE_LEFT )
	{
		mouseX+= xDelta;
		if (mouseX<0)	mouseX = 0;

		mouseY+= yDelta;
		if (mouseY<0)	mouseY = 0;
	}
}
int main( )
{
	JOY_init();
	JOY_setEventHandler( &myJoyHandler ); //mouse button pressed is handled like a joypad, mainly to handle start
	JOY_setMouseEventHandler( &myMouseHandler );

	while(1)
	{
		VDP_waitVSync();
	}
	return 0;
}
```
'></a>
#### Misc ####

Some others usefull functions are available:
  * `JOY_update()` refreshes joypad status (called every screen refresh so don't call it yourself unless you really need it)
  * `JOY_readJoypad( joy )` gives you the joy1 status whenever you need it
  * `JOY_waitPressBtn()` waits for any buttons to be pressed (not directions)
  * `JOY_waitPress(joy, BUTTON_A | BUTTON_UP)` waits for wanted buttons to be pressed on specific joypad



_Note for advanced developer :_

if you ever made you're own VBlankCallback, `JOY_update()` is always called whatever you do.


---


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

Several tools were made by the communauty ([Genitile](http://www.pascalorama.com/article.php?news=28&cat=21), [B2T](http://gendev.spritesmind.net/page-b2t.html), [GenRes](http://gendev.spritesmind.net/page-genres.html), [Mega-Happy-Sprite](http://uberleethackerforce.deepgeek.us/prods.html)
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
	VDP_setTileMap(APLAN, 1, 5, 5);
	
	// ... code
	
	while(1)
	{
		// ... code
		VDP_waitVSync();
	}
	
```

The tile is so drawn on screen using some fade of grey...but what about flipping and pal ?

The 2nd argument of `VDP_setTileMap` func could be more than only the tile index.

It is, in fact, the tile properties : pal index, priority, vflipping, hflipping and tile index.

So, to write it flipped green on B plane, you could write

```
	// ... code
	
	//PAL2 = green pal
	// 0 = low priority
	// 1 = vflip
	// 0 = no hflip
	// 1 = tile 1
	VDP_setTileMap(BPLAN, TILE_ATTR_FULL(PAL2, 0, 1, 0, 1), 6, 5);
	
	// ... code
```


It's now time to talk about the priority flag.

This flag tell which plane is behind the others (even more complex if you add sprites !).

A powerful feature of the Genny is to let the developper define the planes order at the tile level.

It means a tile on A plane could be in front of a tile on B plane at (x,y) and inverse the priority at (n,m).

You could easily see the power of this when scrolling planes at different speed for example.

```
	// ... code
	
	// the same tile is drawn on the 2 plane but with different pal and priority...which one is in front ?
	VDP_setTileMap(APLAN, TILE_ATTR_FULL(PAL1, 1, 0, 0, 1), 7, 7);
	VDP_setTileMap(BPLAN, TILE_ATTR_FULL(PAL2, 0, 0, 0, 1), 7, 7);
	VDP_setTileMap(APLAN, TILE_ATTR_FULL(PAL1, 0, 0, 0, 1), 8, 7);
	VDP_setTileMap(BPLAN, TILE_ATTR_FULL(PAL2, 1, 0, 0, 1), 8, 7);
	
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
  1. the bitmap sould be a 16 colors, 4 bits/pixel BMP... be careful about this, some image editor save 256 colors BMP with only the first 16 colors filled

SGDK is able to handle several raw files if they are copied in a `res/` subfolder.

**WARNING** : you need at least SGDK v0.7b since there is a bug on pal support in previous versions

_Don't be afraid by the warning messages, it's a little bug on the extractor_

If you copy your bitmap on this folder, SGDK will produce a `name_of_bmp_file.h` and a `name_of_bmp_file.o`

`name_of_bmp_file.o` is the object form of your bitmap.

`name_of_bmp_file.h` let you access the bitmap raw data.

so, including `name_of_bmp_file.h`, you could load the bitmap on VRAM this way

```
#include "moon.h"

int main( )
{
	// get the image width (in pixel) ==> should be 8pix aligned
	u16 w = moon[0];
	// get the image height (in pixel)  ==> should be 8px aligned
	u16 h = moon[1];

	// get the palette at moon[2 to 17]
	VDP_setPalette(PAL1, &moon[2]);

	
	// load bitmap data at moon[18....] in VRAM
	// w/8 = width in tiles we want to load
	// h/8 = height in tile we want to load
	// w/8 = width in tiles of the bitamp
	// the 3rd arg is needed because you could load only a part of the bitmap if you want but SGDK needs the width as reference
	VDP_loadBMPTileData((u32*) &moon[18], 1, w / 8, h / 8, w/8 );


	while(1)
	{
		VDP_waitVSync();
	}
	return 0;
}
```

For this moon, you have 8x8 tiles loaded.

You could draw them by hand using `VDP_setTileMap` but it's boring and, as usual when it's a boring process, SGDK has a magical function for it : `VDP_fillTileMapRectInc`.

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


---


## Sprites ##

If you read documents linked on [first part](#Introduction.md), you should know the main difference between tiles for sprites and tiles for planes :
  * the plane draws the tiles from left to right THEN top to bottom (ie row order)
  * the sprite draws the tiles from top to bottom THEN left to right (ie column order)

It's why we can't use the same methods to draw sprites than to draw plane tiles.

The good news is that it is the way to DRAW which is different, not the way to load it.

Planes and Sprites uses tiles loaded on the VRAM.

So yes, you could use tiles for planes, sprites or the two at the same time !

### Basic ###

Some other data you should know about sprites
  * the sprite "plane" is, unlike the 2 others playfield, not scrollable and fixed size (512x512 in common case)
  * it means only a part of the sprite plane is visible, this part starts at (128,128)
  * any sprite outside this display area is so not visible
  * sprite position "rolls" : ex (300, 812) is the same as (300, 300)
  * you can control the order to draw the sprites using a _link_ property
  * max 80 sprites could be defined in PAL mode
  * max 20 sprites could be draw on a same line in PAL mode
  * sprites are made of 1x1 to 4x4 tiles (so **big** sprites are in fact multi sprites)

So, the steps to draw a sprite on screen are
  1. load the tiles on VRAM
  1. load its pal (if not already loaded)
  1. define it
  1. (define the other ones)
  1. ask for drawing

Like before, we will first define the tiles in pure C Array.

We want a 2x2 sprite so we need 4 tiles :

```
const u32 spriteTiles[4*8]=
{
		0x00001111, //Tile Top Left
		0x00001111,
		// ...

		0x11112222, //Tile Bottom Left
		0x11112222,
		// ...

		0x11110000, //Tile Top Right
		0x11110000,
		// ...

		0x22221111, //Tile Bottom Right
		0x22221111,
		// ...

};

```

This sprite has the same shape we used in tile basic tutorial.

Now, just follow the steps

```
	// ... code
	
	//load the tiles on VRAM
	VDP_loadTileData( (const u32 *)spriteTiles, 1, 4, 0); 
	
	//load its pal (if not already loaded)
	//we'll use one of the pre-loaded pal for now
	
	//define it
	// arg0 : sprite idx (from 0 to 79)
	// arg1 : x
	// arg2 : y
	// arg3 : size (from 1x1 to 4x4 tiles)
	// arg4 : tile(s) attributes
	// arg5 : link property (more on this later)
	VDP_setSprite(0, 40, 40, SPRITE_SIZE(2,2), TILE_ATTR_FULL(PAL0,1,0,0,1), 0);
 
	// ask for draw
	VDP_updateSprites()
	
	// ... code
	
	while(1)
	{
		// ... code
		VDP_waitVSync();
	}
	
	// ... code
```

Important things to note
  * you could define as much sprites (80 max) as you need before ask for drawing
  * using SGDK, sprite position is based on the display area not the sprite plane, which means x=0 & y=0 mean (0,0) on screen and (128,128) on sprite plane
  * SPRITE\_SIZE is needed to pass the usefull value (0000b for 1x1, 0101b for  2x2, etc...)
  * TILE\_ATTR\_FULL is the same macro you use with tiles

A sprite is mainly used for moving object, so you will mostly update the x,y of the sprites.

Using the `VDP_setSprite` is making thing difficult : you have to re-set the sprite each times.

A useful way is to use a structure to maintain the sprite properties, change what need to be changed and update sprites.

This could be done using `VDP_setSpriteP` and a `_spritedef` type object.

```
	// ... code
	
	_spritedef mySprite;
	
	mySprite.posx = 0;
	mySprite.posy = 0;
	mySprite.size = SPRITE_SIZE(2,2);
	mySprite.tile_attr = TILE_ATTR_FULL(PAL0,1,0,0,TILE1);
	mySprite.link  = 0;
	VDP_setSpriteP(0, &mySprite);
	
	// ... code
```

and, to make it move,

```

	// ... code
	while(1)
	{
		mySprite.posx++;
		mySprite.posy++;
		VDP_setSpriteP(0, &mySprite);

		VDP_updateSprites();

		VDP_waitVSync();
	}

```

Of course, you could use `VDP_setSprite` or `VDP_setSpriteP`, it doesn't matter. SDGK gives you `VDP_setSpriteP` to make it easiest.

For now, we didn't talk about _link_ property yet and, unless you understand its use, you won't be able to draw more than 1 sprite.

The _link_ property contains the index of the next sprite to draw (with last one getting back to sprite 0).

This means that, using our first test, you will only draw sprite 0 (since its link property is 0).

```
mySprite.link  = 0;
```


To draw 2 sprites (sprite 0 then sprite 1), you should use

```
	// ... code
	
	mySprite.link  = 1; //link to next sprite
	VDP_setSpriteP(0, &mySprite);

	// ... code
	mySprite2.link  = 0; //mySprite2 is the last one so loop back to sprite 0
	VDP_setSpriteP(1, &mySprite2);
	
	// ... code
```

The _link_ property is so also used to define the order to draw sprites : the first one will be below the next one.

It so lets you control which and when a sprite could be drawn.

You could easily define a sprite but not drawn it like this

```
	// ... code
	
	mySprite.link  = 2; //link to sprite index 2
	VDP_setSpriteP(0, &mySprite);

	// ... code
	mySprite2.link  = 2; // 
	VDP_setSpriteP(1, &mySprite2);
	
	// ... code
	mySprite3.link  = 0; // back to sprite 0 so sprite 1 won't be drawn
	VDP_setSpriteP(2, &mySprite3);
	
	// ... code
```

Of course, you can't avoid sprite 0.

**BEWARE**: a misuse of the _link_ property is the main reason of bugs. 3 things to check:
  1. each one of you sprite link to another one
  1. there is no cycling link (for ex : 1->2, 2->3, 3->1 ...)
  1. the last sprite links back to sprite 0

The full use of linking is useful only on very special case (like the dragon boss on Space Harrier for ex).

So, for now, you should avoid problems and always link to next sprite and get back to sprite 0.

Download : [Basic sprites project](http://sgdk.googlecode.com/svn/wiki/files/tut4_SpritesBasic.zip)


---

### GenRes's SPRITE support ###

Unlike tiles for plane, SGDK doesn't come with a native sprite 16 colors bitmap support but you could use _**GenRes**_.

We will talk about the `SPRITE` mode this time.

As you know, GenRes relies on a resource declaration file where each line defines the convert mode, the output name, the file and some parameters.

With `SPRITE`, it could be done this way :

```
	SPRITE output_var_name "directory/file.bmp" <sprite_width> <sprite_height>
	; always finish with a blank line or comment
```

If you write it down in a `resource.rc`, SGDK will call GenRes to compile `directory/file.bmp` to a linked `resource.o`.

For version up to 0.7d, GenRes output format is undefined in SGDK, so you have to write your own format structure like this :

```
	struct genresSprites
	{
			u16 *pal; 		//pointer to pal data
			u32 **sprites;		//pointer to sprites data
			u16 count;		//nb sprites
			u16 width;		//width of each sprite in pixels (not tiles!)
			u16 height;		//height of each sprite in pixels (not tiles!)
			u16 size; 		//since we use width/height in pixel, useful info on sprite size
							//TODO : size is not SGDK compliant, you need to use size>>8
							//		will be fixed in coming release
	};
```

You could then access the data using your `output_var_name`.
```
	extern struct genresSprites output_var_name;
```


That should ring a bell if you read this tutorial is the `sprites` property.

It's not `u32 *sprites` but `u32 **sprites`.

In fact, it's because GenRes converts sprites sheet, not unique sprite (even if a sprites sheet with a unique sprite is valid).

What is a sprites sheet ? a sheet with several sprites in correct position.

If you have 3 sprites of 16x32, the first one will be at (0,0), the 2nd at (16,0) and the 3rd at (32,0) or (0,32).

![http://sgdk.googlecode.com/svn/wiki/pictures/tutSprite_sonic.png](http://sgdk.googlecode.com/svn/wiki/pictures/tutSprite_sonic.png)

This one is valid. The black grid is Photoshop's one, it should not be drawn on the bitmap.

So, with this sheet, using `sonic.sprites[6]` will give you the data of sprite 6 : Red Sonic.

This data is standard tile data. You could so load it using `VDP_loadTileData` with a valid 3rd argument (number of tiles).

```
	// .... code
	
	// each sprite is height/8 *  width/8 (since it's pixel size, not tile size)
	u16 nbTiles = (sonic.height>>3) * (sonic.width>>3);
	
	VDP_loadTileData( sonic.sprites[0], 1, nbTiles, 0);

	// load in PAL1
	VDP_setPalette(PAL1, sonic.pal);

	VDP_resetSprites();
	VDP_setSprite(0, 0, 0, sonic.size>>8, TILE_ATTR_FULL(PAL1,1,0,0,1), 0);
	VDP_updateSprites();
	// .... code

```

**Beware** : there is actual a problem the `size` property. It should be resolved in a future release. Just use `size>>8` right now.

Now that the sprite tile is loaded, you could use it anyway you want.

Genny supports horizontal and vertical flipping (but no zoom or rotation).

```
	// .... code
	
	// flipped sonic
	VDP_setSprite(0, 0, 0, sonic.size>>8, TILE_ATTR_FULL(PAL1,1,0,1,1), 0);

	// .... code
```

But Sonic isn't Sonic if he doesn't run. So how to we make him run ?

You know how to move it (using `_spritedef`) but you miss something : sprite animation.

There are 2 (basic) ways to make sprite animation
  1. load each frame and define the sprite each refresh with the correct first tile.
  1. define a sprite and load each refresh the frame data onto first tile.

The first one need a lot of VRAM.

The second one make a lot of loading and, so, could slow down.

Let's try the way I (KanedaFr) use the most : frame update, the seconde one.

The steps to follow are simply
  * define the sprite (using `VDP_setSprite`)
  * each refresh, load frame tile data (using `VDP_loadTileData`)

So make Sonic runs! You noticed you need the tile data of 3 sprites : sprite 1, 2 & 3.


```
	// .... code

	u8 frame = 0;
	
	// define the sprite (using a _spritedef to easily make Sonic move later)
	mySprite.posx = 40;
	mySprite.posy = 40;
	mySprite.size = sonic.size>>8;
	mySprite.tile_attr = TILE_ATTR_FULL(PAL1,1,0,0,1);
	mySprite.link  = 0;
	VDP_setSpriteP(0, &mySprite);

	// .... code
	while(1)
	{
		// we still use nbTiles since ALL sprite are of the size in a sprite sheet
		VDP_loadTileData( sonic.sprites[frame + 1], 1, nbTiles, 0);
		frame++; // next frame
		frame%=3; // we only need 3 frames, so roll back


		//make it move!
		mySprite.posx+=10;
		VDP_setSpriteP(0, &mySprite);
		
		//flush
		VDP_updateSprites();
		
		// .... code
		
		VDP_waitVSync();
	}
```

Easy no ?

Download : [Sprites animation using GenRes](http://sgdk.googlecode.com/svn/wiki/files/tut4_SpritesGenRes.zip)

---


### Misc ###

One useful way to test your sprite engine is throught [Gens KMod](http://gendev.spritesmind.net/page-gensK.html).

You could explore the sprite list and trace any issue.

![http://sgdk.googlecode.com/svn/wiki/pictures/tutSprite_Kmod.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/tutSprite_Kmod.jpg)


---


## History ##

23 February 2001 - Sprites
  * Sprites with GenRes
  * Sprites animation

23 February 2001 - Sprites
  * Basic sprites
  * added missing info in Basic tiles

21 February 2011 - Tiles Part
  * Tiles with GenRes

31 January 2011 - Tiles part
  * Basic tiles
  * Tiles from bitmap

26 January 2011 - Initial release
  * Introduction
  * Hello world sample
  * Basic input