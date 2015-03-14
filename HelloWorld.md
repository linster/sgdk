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