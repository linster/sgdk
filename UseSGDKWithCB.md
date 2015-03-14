**Here's how to use the devkit with Code::Blocks IDE**

  * Define "GDK" environment variable to your installation path in unix path format (example D:/apps/gendev).

  * Define "GDK\_WIN" which still point to your installation path but in windows format (exemple D:\apps\gendev).

  * Download Code::Blocks and install it ( http://www.codeblocks.org/ )

  * Launch Code::Blocks and go to the Setting --> Compiler and debugger menu

![http://images4.hiboox.com/images/3312/d49cb27f03f5a9df1337362a0a8ff535.jpg](http://images4.hiboox.com/images/3312/d49cb27f03f5a9df1337362a0a8ff535.jpg)


  * Create a new compiler configuration by doing a copy of the basic GNU GCC Compiler. Name it as you want ("Sega Genesis Compiler" here).

![http://images4.hiboox.com/images/3312/949cf50241179bee327fcae850f2d6f6.jpg](http://images4.hiboox.com/images/3312/949cf50241179bee327fcae850f2d6f6.jpg)


  * "Toolchain executables" tab, enter the mini dev kit path (as you set in GDK\_WIN) in the "Compiler's installation directory". Unfortunately it does not accept variable name so you have to enter it to its own. Then set the executable filename as on the picture :

![http://images4.hiboox.com/images/3312/a0417e09abf34ae159950dea7e745810.jpg](http://images4.hiboox.com/images/3312/a0417e09abf34ae159950dea7e745810.jpg)


**At this point we already finished the compiler configuration :)**

**Now here's how to do your own project and compile it :**

  * Do a new project

![http://images4.hiboox.com/images/3312/11fbbaea4627f2a380aa8c99611a1a8b.jpg](http://images4.hiboox.com/images/3312/11fbbaea4627f2a380aa8c99611a1a8b.jpg)


  * Choose a empty project type and click on "Go" button

![http://images4.hiboox.com/images/3312/a1f6dacd14386bbdae09186f982731a1.jpg](http://images4.hiboox.com/images/3312/a1f6dacd14386bbdae09186f982731a1.jpg)


  * Click on Next...

![http://images4.hiboox.com/images/3312/c6d3a8aff6015cb505cfec59a949c01f.jpg](http://images4.hiboox.com/images/3312/c6d3a8aff6015cb505cfec59a949c01f.jpg)


  * Choose a name and a directory for your project, others box are automatically filled but you can modify them if you want, then click on Next.

![http://images4.hiboox.com/images/3312/bb175fa7708a1a5d71df9c062aa1b3ff.jpg](http://images4.hiboox.com/images/3312/bb175fa7708a1a5d71df9c062aa1b3ff.jpg)


  * Choose the genesis compiler you just set up ("Sega Genesis Compiler" here). Uncheck the "Debug" configuration which is useless here and rename the "Release" configuration to "default" as this is the only used here. Change the outputs directory to "out\" then click on finish.

![http://images4.hiboox.com/images/3312/6e1bb70d33c98fb2839286428d56d101.jpg](http://images4.hiboox.com/images/3312/6e1bb70d33c98fb2839286428d56d101.jpg)


  * Open the contextual menu on the project and choose "Properties..."

![http://images4.hiboox.com/images/3312/4cefa2f431f8c5d508eac39cdf57854c.jpg](http://images4.hiboox.com/images/3312/4cefa2f431f8c5d508eac39cdf57854c.jpg)


  * Use the provided makefile.gen file in the devkit as the project makefile. Don't forget to check the "This is a custom Makefile" checkbox. Then click to the "Project's build options" button.

![http://images4.hiboox.com/images/3312/d7ec7a2db131390f472b75c7ccdfb8ea.jpg](http://images4.hiboox.com/images/3312/d7ec7a2db131390f472b75c7ccdfb8ea.jpg)


  * Select the "default" configuration in left column, check if the Selected compiler is the good one (Sega Genesis Compiler here) and go to the last tab "Make". Then modify the make commands as here :

![http://images4.hiboox.com/images/3312/ccdb685ad1d8f19183694988f67f408c.jpg](http://images4.hiboox.com/images/3312/ccdb685ad1d8f19183694988f67f408c.jpg)


  * Validate your changes and now you can add files to your project, your files should be localized in your project directory as following :
```
sources files (C, S) : root directory or "src" directory
includes files (H, INC) : root directory or "include" directory
resources files (S, ASM, TFC, TFD, PCM, RAW, WAV, BIN, BMP, RC) : root directory or "res" directory
```

![http://images4.hiboox.com/images/3312/21bbc58aa42ccfa344fff974c733fa21.jpg](http://images4.hiboox.com/images/3312/21bbc58aa42ccfa344fff974c733fa21.jpg)


  * Compile your project in the "Build" menu, "Build" command or press Ctrl+F9 keys. If all is correctly setup you should obtain a rom.bin file in the out directory of your project directory :)