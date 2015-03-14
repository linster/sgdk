**Here's how to use the devkit with Eclipse CDT IDE**

## Setup your files ##
Download Eclipse CDT at http://www.eclipse.org/cdt/downloads.php

  * Create a directory for Genesis development only, **without** space or special chars
  * Unzip EclipseCDT and SGDK there
  * Rename SGDK's one as "_sdk_"
  * Create an other directory called "_projects_".

In the end, you have
```
<parent folder>
	->eclipse folder
	->sdk folder
	->projects folder
```


Don't forget to add SGDK's bin folder in your Windows PATH

For XP, see tutorial at http://roy.vanegas.org/teaching/c/modifying_windows_path_for_gcc.html

For 7, open control panel > system, select advanded system parameters and jump to step 4 of above tutorial

## Setup Eclipse ##

**IMPORTANT : Be sure to use slash "/" to define any folder else make will fail**

Launch Eclipse and select _<parent folder>/projects_ as workspace folder
```
 each new project will be added to this folder by Eclipse 
```

Go to the workbench and select menu _Window > Preferences_ to setup Eclipse

In _General > Workspace_, select _**Save automatically before build**_ and unselect _**Build automatically**_

In _C/C++ > Build > Build Variables_, add a new variable :
  * Variable name : GDK
  * Type : Directory
  * Value : (browse to <parent folder>/sdk)

```
 this will define $GDK needed by sdk/makefile.gen
```

![http://sgdk.googlecode.com/svn/wiki/pictures/eclipse01.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/eclipse01.jpg)

In _C/C++ > Build > Environnement_, add a new variable
  * Name : GDK
  * Value : (_full path to  <parent folder>/sdk_)

```
 this will define a var used on Eclipse IDE
```

![http://sgdk.googlecode.com/svn/wiki/pictures/eclipse02.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/eclipse02.jpg)

In _C/C++ > New CDT Project Wizard > Makefile Project_
  * unselect _**Use default build command**_ in _Builder Settings_ tab
  * set, as _Build command_,  `${GDK}\bin\make -f ${GDK}/makefile.gen`

```
 this will define  sdk/makefile.gen as the default makefile of any project
```

![http://sgdk.googlecode.com/svn/wiki/pictures/eclipse03.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/eclipse03.jpg)

## Setup Project ##
You could now create a new project (_File > New > C Project_).

Right-click on it and select _Properties_ to setup the project itself.

In _C/C++ General > Paths and Symbols_, add a new directory in _Includes_ tab
  * Directory : `${GDK}/include`
  * Add to all configurations
  * Add to all languages

![http://sgdk.googlecode.com/svn/wiki/pictures/eclipse05.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/eclipse05.jpg)

Click _Apply_ and rebuild the index

![http://sgdk.googlecode.com/svn/wiki/pictures/eclipse06.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/eclipse06.jpg)
```
this will make available the headers on <parent folder>/sdk/include to your project
```


On _Make Target_ view, create a new target for your project.
  * give it a name
  * unselect _**Same as the target name**_
  * empty _Make target_ field
![http://sgdk.googlecode.com/svn/wiki/pictures/eclipse07.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/eclipse07.jpg)
```
 double click on this target to generate a out/rom.bin according your project's file, sdk/makefile.gen and sgdk library
```

## Setup Gens ##
If you want to launch the rom on Gens from Eclipse
  * Right click on the generated out/rom.bin.
  * Select _Open With > Others..._
  * Browse for _gens.exe_ on _External programs_ tab
![http://sgdk.googlecode.com/svn/wiki/pictures/eclipse08.jpg](http://sgdk.googlecode.com/svn/wiki/pictures/eclipse08.jpg)

## Common error ##
If you have an error on build like **`main() not found`**, be sure to click _**Apply**_ on project properties's _C/C++ General > Paths and Symbols_.

Another source of problem : be sure to un-mark _Project>Build automatically_

And be sure to use slash "/" !!