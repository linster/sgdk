## Input ##

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

SGDK also supports Multipad, Justifier and Mouse, wiki about how use these adapters will come in future...