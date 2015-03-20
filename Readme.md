# Building out-of-tree (recommended)

## Linux

Clone the repo and create a build directory

```shell
git clone https://github.com/kodi-game/game.mupen64plus.git
cd game.mupen64plus
mkdir build
cd build
```

Generate a build environment with config for debugging

```shell
cmake -DADDONS_TO_BUILD=game.mupen64plus \
      -DADDON_SRC_PREFIX=$HOME/workspace \
      -DCMAKE_BUILD_TYPE=Debug \
      -DCMAKE_INSTALL_PREFIX=$HOME/workspace/xbmc/addons \
      -DPACKAGE_ZIP=1 \
      $HOME/workspace/xbmc/project/cmake/addons
```

If you are developing in Eclipse, you can create a "makefile project with existing code" using `game.mupen64plus/` as the existing code location. To build, enter Properties -> "C/C++ Build" and change the build command to `make -C build`.

It is also possible to generate Eclipse project files with cmake

```shell
cmake -G"Eclipse CDT4 - Unix Makefiles" \
      -D_ECLIPSE_VERSION=4.4 \
      -DADDONS_TO_BUILD=game.mupen64plus \
      -DADDON_SRC_PREFIX=$HOME/workspace \
      -DCMAKE_BUILD_TYPE=Debug \
      -DCMAKE_INSTALL_PREFIX=$HOME/workspace/xbmc/addons \
      -DPACKAGE_ZIP=1 \
      $HOME/workspace/xbmc/project/cmake/addons
```

The compiled .so will be placed at

```
build/game.mupen64plus-prefix/src/game.mupen64plus-build/mupen64plus/src/mupen64plus/mupen64plus_libretro.so
```

Create a symlink from `xbmc/addons/game.mupen64plus/game.mupen64plus.so` to this file so that Kodi will alway use the most recent build.

## Windows

First, download and install [CMake](http://www.cmake.org/download/) and [MinGW](http://www.mingw.org/). Add the MinGW `bin` folder to your path (e.g. `C:\MinGW\bin`).

Run the script from [PR 6658](https://github.com/xbmc/xbmc/pull/6658) to create Visual Studio project files

```
tools\windows\prepare-binary-addons-dev.bat
```

The generated solution can be found at

```
project\cmake\addons\build\kodi-addons.sln
```

Currently, the build system corrupts `addons/game.mupen64plus/game.mupen64plus.dll`. You can find the correct .dll here

```
project\cmake\addons\build\game.mupen64plus-prefix\src\game.mupen64plus-build\mupen64plus\src\mupen64plus\mupen64plus_libretro.dll
```

Copy this to `addons/game.mupen64plus/` and rename to `game.mupen64plus.dll` to match the DLL name in [addon.xml](https://github.com/kodi-game/game.mupen64plus/blob/master/game.mupen64plus/addon.xml).

# Building in-tree (cross-compiling)

Kodi's build system will fetch the add-on from the GitHub URL and git hash specified in [game.mupen64plus.txt](https://github.com/garbear/xbmc/blob/retroplayer-15alpha2/project/cmake/addons/addons/game.mupen64plus/game.mupen64plus.txt).

## Windows

Remember, CMake and an external MinGW are required.

```shell
cd tools\buildsteps\win32
make-addons.bat game.mupen64plus
```

See above for the location of the correct .dll.

## OSX

Per [README.osx](https://github.com/garbear/xbmc/blob/retroplayer-15alpha2/docs/README.osx), enter the `tools/depends` directory and make the add-on:

```shell
cd tools/depends
make -C target/binary-addons ADDONS="game.mupen64plus"
```
Currently, the build system corrupts `addons/game.mupen64plus/game.mupen64plus.dylib`. You can find the correct .dylib here

```
tools/depends/target/binary-addons/macosx10.10_x86_64-target/game.mupen64plus-prefix/src/game.mupen64plus-build/mupen64plus/src/mupen64plus/mupen64plus_libretro.dylib
```

The solution I've found is to symlink to the correct .dylib

```shell
cd addons/game.mupen64plus/
rm game.mupen64plus.*
ln -s ../../tools/depends/target/binary-addons/macosx10.10_x86_64-target/game.mupen64plus-prefix/src/game.mupen64plus-build/mupen64plus/src/mupen64plus/mupen64plus_libretro.dylib   game.mupen64plus.dylib
```

Unfortunately the symlink gets overwritten when the add-on is rebuilt.

## Cleaning build directory

Run the following to clean the build directory. Note, this will clean all add-ons, not just game.mupen64plus.

```shell
make -C target/binary-addons clean
```
