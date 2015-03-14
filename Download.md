## Download ##

Get the last version: [SGDK 1.12](https://www.dropbox.com/s/2cnlpcoiowq6178/sgdk112.7z?dl=0)

**Old versions**
  * [SGDK 1.11](https://www.dropbox.com/s/fk51tsowolmjwaf/sgdk111.7z?dl=0)
  * [SGDK 1.10](https://www.dropbox.com/s/ng7ga5reyay51p6/sgdk110.7z?dl=0)
  * [SGDK 1.01](https://www.dropbox.com/s/059zcf44176r9z9/sgdk101.7z?dl=0)
  * [SGDK 1.00](https://www.dropbox.com/s/o056nj6zgxa6k4j/sgdk100.7z?dl=0)
  * [SGDK 0.96d](https://www.dropbox.com/s/a4oqp7i7hizivsd/sgdk096d.7z?dl=0)


---


## Changelog 1.12 ##

**COMPILER**
  * XGMTool
    * minors changes and fixes.

**LIBRARY**
  * SPRITE
    * added `VDP_resetSpritesDirect()` method.
  * SOUND
    * minor fix to XGM driver (PCM in music was wrong in some case).



## Changelog 1.11 ##

**LIBRARY**
  * CONTROLLER
    * fixed small issue in joystick code (Chilly Willy).
  * SOUND
    * added Z80 CPU load information in XGM driver (experimental).



## Changelog 1.10 ##

**COMPILER**
  * XGMTool
    * removed DAC enabled command (automatically handle by the XGM driver).
    * added DAC enabled state (XGM driver uses it when no PCM are playing).
    * now uses the VGM 1.60 _stream id_ information to allow multi PCM channel for XGM conversion (note that each channel has its own priority as VGM music does not contains PCM priority information).

**LIBRARY**
  * VDP
    * `VDP_fade(..)` method now automatically disables interrupts if needed.
  * SPRITE
    * minor optimization in the Sprite Engine to quickly discard disabled sprites (visibility forced to off).
  * SOUND
    * added automatic DAC enabled control (XGM driver).
    * fixed issue with music pause operation on real hardware (XGM driver).
    * minor tweak to reduce a bit the size of the Z80 drivers.
    * removed _Z80\_DRIVER\_4PCM_ which is useless (use _Z80\_DRIVER\_4PCM\_ENV_ driver instead).
  * CONTROLLER
    * added Sega Phaser support (Chilly Willy).
  * SYSTEM
    * disable library debug info.
  * SAMPLE
    * updated _Joy_ sample to add Phaser test (Chilly Willy).
    * updated _Sound_ sample to remove _Z80\_DRIVER\_4PCM_ test and add an example of the MVS driver PCM SFX.



## Changelog 1.01 ##

**COMPILER**
  * fixed bugs in XGMTool.
  * removed linear interpolation when converting WAV file.
  * others minors changes.



## Changelog 1.00 ##

A big addition for SGDK 1.00, the new XGM sound driver !
You can find more informations about it in the [XGM driver specifications](https://drive.google.com/file/d/0BztTfEamf-d9MjVsOGN1MXEwdk0/view?usp=sharing).

**COMPILER**
  * Rescomp
    * added support to XGM resource in rescomp.
  * added xgmtool to convert VGM into XGM and compile XGM file.
  * removed Genitile tool sources.
  * updated wavtoraw to support sample interpolation.

**LIBRARY**
  * VDP
    * added `VDP_setScanMode(u16 mode)` method to change the interlaced mode.
    * added `VDP_interruptFade()` method to interrupt async palette fading.
  * SPRITE
    * fixed a bug with `SPR_init(..)` method when using same definition, sometime timer could be not reseted and then animation is not working anymore.
    * fixed declaration of `VDP_setSprite(..)` / `VDP_setSpriteDirect(..)` / `VDP_setSpritePosition(..)` methods.
  * SOUND
    * Z80 memory is cleared before loading a custom driver.
    * added Z80\_read(..) and Z80\_write(..) methods for simple Z80 RAM read/write operations.
    * added XGM driver methods:
      * `u8 SND_isPlaying_XGM();`
      * `void SND_startPlay_XGM(const u8 *song);`
      * `void SND_stopPlay_XGM();`
      * `void SND_resumePlay_XGM();`
      * `u8 SND_isPlayingPCM_XGM(const u16 channel_mask);`
      * `void SND_setPCM_XGM(const u8 id, const u8 *sample, const u32 len);`
      * `void SND_setPCMFast_XGM(const u8 id, const u8 *sample, const u32 len);`
      * `void SND_startPlayPCM_XGM(const u8 id, const u8 priority, const u16 channel);`
      * `void SND_stopPlayPCM_XGM(const u16 channel);`
  * MATH
    * minor fix in min/max defines.
  * CONTROLLER
    * fixed declaration of `JOY_readJoypadX/Y(..)` methods (they should return s16 and not u16)
  * SYSTEM
    * sega.s and rom\_head.c files are now copied into the _src/boot_ project folder so they can easily be customized per project.
  * SAMPLE
    * modified sound sample to add XGM driver example.
  * some fixes in the doxygen documentation.
  * others changes and improvements.



## Changelog 0.96d ##

**LIBRARY**
  * removed direct VRam Map data unpacking as it was buggy.



## Changelog 0.96c ##

**COMPILER**
  * Rescomp
    * fixed some issues on compression.
    * now accept string to define which compression to use (AUTO, APLIB, RLE) in resource definition.

**LIBRARY**
  * minors changes to Doxygen documentation.



## Changelog 0.96 ##

**COMPILER**
  * Rescomp
    * improved BMP image support.
    * byte data are now output in word format to avoid the GCC bug (compilation with `-g` flag fails when byte data is encountered).
    * fixed issue on empty sprite animation detection.
    * fixed path separator issue on old windows system.
    * fixed compilation issues on unix system.
  * Makefile
    * Added `release` and `debug` targets to makefile (default target is `release`, `debug` target allow you to use GDB interactive debugger through emulator supporting it).
  * added `appack` tool sources code (compatibility for linux system).
  * some cleanup in `bintos` tool (removed the useless `-sizealign` command done by `sizebnd` tool).

**LIBRARY**
  * VDP
    * fixed palette fading methods where the last frame colors weren't always correct.
    * `VDP_drawImage(..)` and `VDP_drawBitmap(..)` now use dynamic VRAM tile index so they does not erase anymore the previous drawn image. `curTileInd` variable which contains the VRAM tile index where next tile will be uploaded is public.
  * TILE
    * fixed some bugs in the tile cache engine.
    * removed the`MEM_free(..)` call from the VInt callback (tile cache engine). We don't need anymore to disable interrupts at each memory allocation operation for safety ;)
  * SPRITE
    * fixed some bugs in the sprite engine.
    * added `SPR_setAlwaysVisible(..)` and `SPR_setNeverVisible(..)` to force (not) visibility on sprite (sprite engine)
  * BITMAP
    * minor performance improvement on the `BMP_drawPolygon(..)` method.
  * SOUND
    * fixed YM2612 write methods (can have issue on Megadrive 2 system).
  * DMA
    * fixed VRam Copy DMA.
  * MATH
    * fixed 2D projection calculation in `M3D_project_xxx(..)` methods. Now the camera distance is correctly taken in account for the final projection (adding a minor impact on performance).
  * SYSTEM
    * fixed soft reset issues.
  * TIMER
    * fixed a minor issue with `getTime(..)` method.
    * `waitSubTick(..)` is now more accurate when called from V-Interrupt code.
  * SAMPLE
    * minors changes and improvements on the Sonic Sprite sample.
    * minors changes to 3D cube flat sample.
  * DEBUG
    * added some KDebug log methods (`KLog, KLog_Uxx, KLog_Sxx...`)
  * MISC
    * added `strcmp(..)` method.
    * others changes and improvements.