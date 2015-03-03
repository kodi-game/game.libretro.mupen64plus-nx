
To build game add-ons with Kodi:

```shell
KODI_PREFIX=$HOME/kodi

git clone -b retroplayer-15alpha1 https://github.com/garbear/xbmc.git xbmc
cd xbmc
./bootstrap
./configure
make -j8
make install DESTDIR=$KODI_PREFIX
make -C tools/depends/target/xbmc-game-addons/ PREFIX=$KODI_PREFIX
cd ..
```

To build just Mupen64 add-on, after doing the above:

```shell
KODI_PREFIX=$HOME/kodi
SOURCE_DIR=`pwd`/source/game.mupen64plus
BUILD_DIR=`pwd`/build/game.mupen64plus

mkdir -p $SOURCE_DIR $BUILD_DIR
git clone https://github.com/garbear/game.mupen64plus.git $SOURCE_DIR
cd $BUILD_DIR
cmake $SOURCE_DIR -DCMAKE_BUILD_TYPE=Debug -DCMAKE_PREFIX_PATH=$KODI_PREFIX -DCMAKE_INSTALL_PREFIX=$KODI_PREFIX

# or, for Eclipse 4.4 project files:

cmake $SOURCE_DIR -G"Eclipse CDT4 - Unix Makefiles" -D_ECLIPSE_VERSION=4.4 -DCMAKE_BUILD_TYPE=Debug -DCMAKE_PREFIX_PATH=$KODI_PREFIX -DCMAKE_INSTALL_PREFIX=$KODI_PREFIX

# and then

make
```

