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