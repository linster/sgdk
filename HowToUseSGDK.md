
```
%GDK_WIN%\bin\make -f %GDK_WIN%\makelib.gen
```
  1. When the library is compiled you should obtain the following files
```
%GDK%/lib/libmd.a
```
  1. Before compiling your own project you first need to respect the following constraint in your project source tree:
    * sources files (C, S, ASM, S80): root directory or _src_ directory
    * includes files (H, INC): root directory or _inc_ directory
    * primary resource files (C, S, RC, RES): root directory or _res_ directory
    * others resources files (TFC, TFD, PCM, RAW, WAV, BIN, BMP, PNG): can be wherever you want while you are referring them correctly in your resource definition files
  1. Then to compile your project you need to use the following command directly from your project directory :
```
%GDK_WIN%\bin\make -f %GDK_WIN%\makefile.gen
```


Normally you should obtain a rom.bin file in the out directory.

Then test it with an emulator (or directly on the hardware if you are a lucky owner of a flash cart).