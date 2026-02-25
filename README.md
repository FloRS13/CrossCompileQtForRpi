# Cross compilation of Qt6.10.2 for RPI
This page shows steps to compile Qt6.10.2 for RPI. 
Hope this page will help those stuck at following official tutorial. 
This tutorial has been done with a RPI 4, RPI OS 2025-12-04 64bit and a VirtualBox VM with Ubunutu 24.04.4 LTS. This also use Qt Creator 18.0.2

This is based on muyepan init commit and https://www.interelectronix.com/qt-68-cross-compilation-raspberry-pi.html

## Prepare RPI
Create a SD card with Raspeberry Pi Imager. Use the Rapberry Pi OS Desktop from 2025-12-04

Before making an upgrade we will put on hold the installation of libqt6* libraries. Tests have shown that these libraries conflict with the Qt Creator installation planned for later. 

Note : (During the upgrade you will be asked (Y/N) whether new packages will be installed. If a new package Qt6 is proposed, abort the installation and add it to the sudo apt-mark hold as listed below)
```
sudo apt update
```
```
sudo apt-mark hold libqt6core6 libqt6dbus6 libqt6gui6 libqt6network6 libqt6opengl6 libqt6openglwidgets6 libqt6widgets6 qt6-gtk-platformtheme qt6-qpa-plugins qt6-translations-l10n
```
```
sudo apt upgrade
```
```
sudo reboot
```
Install necessary packages.
```
sudo apt-get install libboost-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev mesa-common-dev libasound2-dev libpulse-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev  gstreamer1.0-alsa libvpx-dev libsrtp2-dev libsnappy-dev libnss3-dev "^libxcb.*" flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev libatkmm-1.6-dev libxi6 libxcomposite1 libfreetype6-dev libicu-dev libsqlite3-dev libxslt1-dev 
```
```
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libx11-dev freetds-dev libsqlite3-dev libpq-dev libiodbc2-dev firebird-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 libx11-xcb-dev libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-glx0-dev libxi-dev libdrm-dev libxcb-xinerama0 libxcb-xinerama0-dev libatspi2.0-dev libxcursor-dev libxcomposite-dev libxdamage-dev libxss-dev libxtst-dev libpci-dev libcap-dev libxrandr-dev libaudio-dev libxkbcommon-x11-dev gdbserver
```
Activate the Console text console mode for the Rpi
```
sudo raspi-config
```
Go on **System Options** then on **Boot** and select **B1 Console Text console**
```
sudo reboot
```
Make a folder for qt6 installation.
```
sudo mkdir /usr/local/qt6
```
Grant full access to the fold used for the deployment from Qt Creator. 
```
sudo chmod 777 /usr/local/bin
```
Remember versions of gcc(14.2.0), ld(2.44) and ldd(2.41). Source code of the same version should be downloaded to build cross compiler later.

<img width="866" height="519" alt="gcc_ld_ldd_ver" src="https://github.com/user-attachments/assets/d19bae7a-99d7-4dac-a444-442eaaae173e" />

Append following piece of code to the end of ~/.bashrc.
```
sudo nano ~/.bashrc
```
```
export QT_PLUGIN_PATH=/usr/local/qt6/plugins
export QML2_IMPORT_PATH=/usr/local/qt6/qml
unset DISPLAY
export QT_QPA_PLATFORM=eglfs
export QT_QPA_EGLFS_INTEGRATION=eglfs_kms
export QT_QPA_EGLFS_ALWAYS_SET_MODE=1
export QT_QPA_EGLFS_FORCE888=1
export QT_QPA_EGLFS_DEBUG=1
export EGL_PLATFORM=drm
```
Update the changes.
```
source ~/.bashrc
```
## Prepare host
Create a virtual machine for Ubuntu 24.04.4 LTS and then update the system.
```
sudo apt update
```
```
sudo apt upgrade
```
Install necessary packages.
```
sudo apt-get install make build-essential libclang-dev ninja-build gcc git bison python3 gperf pkg-config libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-glx0-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev libatspi2.0-dev libgl1-mesa-dev libglu1-mesa-dev freeglut3-dev build-essential gawk git texinfo bison file wget libssl-dev gdbserver gdb-multiarch libxcb-cursor-dev
```
Append following piece of code to the end of ~/.bashrc.
```
sudo nano ~/.bashrc
```
```
export PATH=/opt/cross-pi-gcc/bin:$PATH
```
Update the changes.
```
source ~/.bashrc
```

### Build the lastest CMake from source
```
cd ~
```
Create a working directory for the following steps
```
mkdir qt-rpi-cc && cd qt-rpi-cc
```
```
git clone https://github.com/Kitware/CMake.git
```
```
cd CMake
```
```
./bootstrap && make -j$(nproc)&& sudo make install
```
<img width="866" height="277" alt="cmake_ver" src="https://github.com/user-attachments/assets/c7e74ee3-ff1e-4771-9cd2-bf5cab4735d1" />

```
cd ~/qt-rpi-cc
```
```
rm -rf CMake
```
### Build gcc as a cross compiler
Download necessary source code. **You should modify the following commands to your needs.**
For the time I make this page, they are:
* gcc 14.2.0(gcc version)
* binutils 2.44 (ld version)
* glibc 2.41 (ldd version)
```
cd ~/qt-rpi-cc/
```
```
mkdir gcc_all && cd gcc_all
```
```
wget https://ftpmirror.gnu.org/binutils/binutils-2.44.tar.bz2
```
```
wget https://ftpmirror.gnu.org/glibc/glibc-2.41.tar.bz2
```
```
wget https://ftpmirror.gnu.org/gcc/gcc-14.2.0/gcc-14.2.0.tar.gz
```
```
git clone --depth=1 https://github.com/raspberrypi/linux
```
```
tar xf binutils-2.44.tar.bz2
```
```
tar xf glibc-2.41.tar.bz2
```
```
tar xf gcc-14.2.0.tar.gz
```
```
rm *.tar.*
```
```
cd gcc-14.2.0
```
```
contrib/download_prerequisites
```
Make a folder for the compiler installation.
```
sudo mkdir -p /opt/cross-pi-gcc
```
```
sudo chown $USER /opt/cross-pi-gcc
```
Copy the kernel headers in the above folder.
```
cd ~/qt-rpi-cc/gcc_all
```
```
cd linux
```
```
KERNEL=kernel7
```
```
make ARCH=arm64 INSTALL_HDR_PATH=/opt/cross-pi-gcc/aarch64-linux-gnu headers_install
```
Build Binutils. **You should modify the following commands to your needs.**
```
cd ~/qt-rpi-cc/gcc_all
```
```
mkdir build-binutils && cd build-binutils
```
```
../binutils-2.44/configure --prefix=/opt/cross-pi-gcc --target=aarch64-linux-gnu --with-arch=armv8 --disable-multilib
```
```
make -j$(nproc)
```
```
make install
```
Edit gcc-14.2.0/libsanitizer/asan/asan_linux.cpp. Add following piece of code.
```
#ifndef PATH_MAX
#define PATH_MAX 4096
#endif
```

Do a partial build of gcc. **You should modify the following commands to your needs.**
```
cd ~/qt-rpi-cc/gcc_all
```
```
mkdir build-gcc && cd build-gcc
```
```
../gcc-14.2.0/configure --prefix=/opt/cross-pi-gcc --target=aarch64-linux-gnu --enable-languages=c,c++ --disable-multilib --disable-libsanitizer
```
```
make -j$(nproc) all-gcc
```
```
make install-gcc
```
Partially build Glibc. **You should modify the following commands to your needs.**
```
cd ~/qt-rpi-cc/gcc_all
```
```
mkdir build-glibc && cd build-glibc
```
```
../glibc-2.41/configure \
  --prefix=/opt/cross-pi-gcc/aarch64-linux-gnu \
  --build=$MACHTYPE \
  --host=aarch64-linux-gnu \
  --target=aarch64-linux-gnu \
  --with-headers=/opt/cross-pi-gcc/aarch64-linux-gnu/include \
  --disable-multilib \
  --disable-werror \
  --disable-mathvec \
  libc_cv_forced_unwind=yes
```
```
make install-bootstrap-headers=yes install-headers
```
```
make -j$(nproc) csu/subdir_lib
```
```
install csu/crt1.o csu/crti.o csu/crtn.o /opt/cross-pi-gcc/aarch64-linux-gnu/lib
```
```
aarch64-linux-gnu-gcc -nostdlib -nostartfiles -shared -x c /dev/null -o /opt/cross-pi-gcc/aarch64-linux-gnu/lib/libc.so
```
```
touch /opt/cross-pi-gcc/aarch64-linux-gnu/include/gnu/stubs.h
```
Back to gcc.
```
cd ~/qt-rpi-cc/gcc_all/build-gcc
```
```
make -j$(nproc) all-target-libgcc
```
```
make install-target-libgcc
```
Finish building glibc.
```
cd ~/qt-rpi-cc/gcc_all/build-glibc
```
```
make -j$(nproc)
```
```
make install
```
Finish building gcc.
```
cd ~/qt-rpi-cc/gcc_all/build-gcc
```
```
make -j$(nproc)
```
```
make install
```
At this point, we have a full cross compiler toolchain with gcc. Folder gcc_all is not need any more. You can delete it.
```
cd ~/qt-rpi-cc
```
```
rm -rf gcc_all
```
## Building Qt6
Make folders for sysroot and qt6.
```
cd ~/qt-rpi-cc
```
```
mkdir rpi-sysroot rpi-sysroot/usr rpi-sysroot/opt
```
```
mkdir qt6 qt6/host qt6/pi qt6/host-build qt6/pi-build qt6/src
```
Download QtBase source code
```
cd ~/qt-rpi-cc/qt6/src
```
```
wget https://download.qt.io/official_releases/qt/6.10/6.10.2/submodules/qtbase-everywhere-src-6.10.2.tar.xz
```
```
tar xf qtbase-everywhere-src-6.10.2.tar.xz
```
### Build Qt6 for host
```
cd $HOME/qt-rpi-cc/qt6/host-build/
```
```
cmake ../src/qtbase-everywhere-src-6.10.2/ -GNinja -DCMAKE_BUILD_TYPE=Release -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX=$HOME/qt-rpi-cc/qt6/host
```
```
cmake --build . --parallel$(nproc)
```
```
cmake --install .
```
Binaries will be in $HOME/qt-rpi-cc/qt6/host
### Build Qt6 for rpi
copy and paste a few folders from rpi using rsync through SSH. **You should modify the following commands to your needs.**
For this example the ip of the board is 192.168.1.149 
```
cd ~
```
```
rsync -avz --rsync-path="sudo rsync" pi@192.168.1.149:/usr/include qt-rpi-cc/rpi-sysroot/usr
```
```
rsync -avz --rsync-path="sudo rsync" pi@192.168.1.149:/lib qt-rpi-cc/rpi-sysroot
```
```
rsync -avz --rsync-path="sudo rsync" pi@192.168.1.149:/usr/lib qt-rpi-cc/rpi-sysroot/usr
```

Create a file named ```toolchain.cmake``` in $HOME/qt-rpi-cc/qt6.
```
cmake_minimum_required(VERSION 3.18)
include_guard(GLOBAL)

# -------------------------
# Target system
# -------------------------
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

# You should change location of sysroot to your needs.
set(TARGET_SYSROOT /home/$ENV{USER}/qt-rpi-cc/rpi-sysroot)
set(TARGET_ARCHITECTURE aarch64-linux-gnu)
set(CMAKE_SYSROOT ${TARGET_SYSROOT})

# -------------------------
# pkg-config setup
# -------------------------
set(ENV{PKG_CONFIG_PATH} $PKG_CONFIG_PATH:${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/pkgconfig)
set(ENV{PKG_CONFIG_LIBDIR} /usr/lib/pkgconfig:/usr/share/pkgconfig/:${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/pkgconfig:${TARGET_SYSROOT}/usr/lib/pkgconfig)
set(ENV{PKG_CONFIG_SYSROOT_DIR} ${CMAKE_SYSROOT})

# -------------------------
# Cross-compilers
# -------------------------
set(CMAKE_C_COMPILER /opt/cross-pi-gcc/bin/${TARGET_ARCHITECTURE}-gcc)
set(CMAKE_CXX_COMPILER /opt/cross-pi-gcc/bin/${TARGET_ARCHITECTURE}-g++)

# -------------------------
# C / C++ flags
# -------------------------
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -isystem=/usr/include -isystem=/usr/local/include -isystem=/usr/include/${TARGET_ARCHITECTURE}")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS}")

# Optional Qt flags
set(QT_COMPILER_FLAGS "-march=armv8-a")
set(QT_COMPILER_FLAGS_RELEASE "-O2 -pipe")
set(QT_LINKER_FLAGS "-Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed -Wl,-rpath-link=${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE} -Wl,-rpath-link=$HOME/qt6/pi/lib")

# -------------------------
# Library / include paths
# -------------------------
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_BUILD_RPATH ${TARGET_SYSROOT})

include(CMakeInitializeConfigs)

function(cmake_initialize_per_config_variable _PREFIX _DOCSTRING)
  if (_PREFIX MATCHES "CMAKE_(C|CXX|ASM)_FLAGS")
    set(CMAKE_${CMAKE_MATCH_1}_FLAGS_INIT "${QT_COMPILER_FLAGS}")
        
    foreach (config DEBUG RELEASE MINSIZEREL RELWITHDEBINFO)
      if (DEFINED QT_COMPILER_FLAGS_${config})
        set(CMAKE_${CMAKE_MATCH_1}_FLAGS_${config}_INIT "${QT_COMPILER_FLAGS_${config}}")
      endif()
    endforeach()
  endif()


  if (_PREFIX MATCHES "CMAKE_(SHARED|MODULE|EXE)_LINKER_FLAGS")
    foreach (config SHARED MODULE EXE)
      set(CMAKE_${config}_LINKER_FLAGS_INIT "${QT_LINKER_FLAGS}")
    endforeach()
  endif()

  _cmake_initialize_per_config_variable(${ARGV})
endfunction()

# -------------------------
# OpenGL / EGL / GBM / XCB libraries
# -------------------------
set(XCB_PATH_VARIABLE ${TARGET_SYSROOT})

set(GL_INC_DIR ${TARGET_SYSROOT}/usr/include)
set(GL_LIB_DIR ${TARGET_SYSROOT}:${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/:${TARGET_SYSROOT}/usr:${TARGET_SYSROOT}/usr/lib)

set(EGL_INCLUDE_DIR ${GL_INC_DIR})
set(EGL_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libEGL.so)

set(OPENGL_INCLUDE_DIR ${GL_INC_DIR})
set(OPENGL_opengl_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libOpenGL.so)

set(GLESv2_INCLUDE_DIR ${GL_INC_DIR})
set(GLIB_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libGLESv2.so)

set(GLESv2_INCLUDE_DIR ${GL_INC_DIR})
set(GLESv2_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libGLESv2.so)

set(gbm_INCLUDE_DIR ${GL_INC_DIR})
set(gbm_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libgbm.so)

set(Libdrm_INCLUDE_DIR ${GL_INC_DIR})
set(Libdrm_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libdrm.so)

set(XCB_XCB_INCLUDE_DIR ${GL_INC_DIR})
set(XCB_XCB_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libxcb.so)

set(DBUS_LIBRARY ${CMAKE_SYSROOT}/usr/lib/aarch64-linux-gnu/libdbus-1.so)


list(APPEND CMAKE_LIBRARY_PATH ${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE})
list(APPEND CMAKE_PREFIX_PATH "/usr/lib/${TARGET_ARCHITECTURE}/cmake")

set(CMAKE_SKIP_RPATH FALSE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "/usr/local/qt6/lib")

```
Fix absolute symbolic links
```
cd ~
```
```
wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
```
```
chmod +x sysroot-relativelinks.py
```
```
python3 sysroot-relativelinks.py rpi-sysroot
```
Compile source code for rpi.
```
cd $HOME/qt-rpi-cc/qt6/pi-build
```
```
cmake ../src/qtbase-everywhere-src-6.10.2/ -GNinja -DCMAKE_BUILD_TYPE=Release -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DQT_HOST_PATH=$HOME/qt-rpi-cc/qt6/host -DCMAKE_STAGING_PREFIX=$HOME/qt-rpi-cc/qt6/pi -DCMAKE_INSTALL_PREFIX=/usr/local/qt6 -DCMAKE_TOOLCHAIN_FILE=$HOME/qt-rpi-cc/qt6/toolchain.cmake -DQT_QMAKE_TARGET_MKSPEC=devices/linux-rasp-pi4-aarch64 -DQT_FEATURE_eglfs=ON -DQT_FEATURE_linuxfb=ON -DQT_FEATURE_xcb=OFF -DQT_FEATURE_opengl=ON -DQT_FEATURE_eglfs_kms=ON -DQT_FEATURE_vulkan=OFF -DQT_FEATURE_eglfs_gbm=ON -DQT_FEATURE_drm=ON -DQT_FEATURE_gbm=ON
```
```
cmake --build . --parallel$(nproc)
```
```
cmake --install .
```
Send the binaries to rpi. **You should modify the following commands to your needs.**
```
rsync -avz --rsync-path="sudo rsync" $HOME/qt-rpi-cc/qt6/pi/* pi@192.168.1.149:/usr/local/qt6
```
## Start with Qt Creator
### Install Qt Creator
```
cd ~/qt-rpi-cc
```
```
mkdir qtcreator && cd qtcreator
```
```
wget https://download.qt.io/official_releases/qtcreator/18.0/18.0.2/qt-creator-opensource-linux-x86_64-18.0.2.run
```
```
chmod +x ./qt-creator-opensource-linux-x86_64-18.0.2.run
```
```
./qt-creator-opensource-linux-x86_64-18.0.2.run
```
Install Qt in a directory in the qt-rpi-cc that we already create
<img width="848" height="626" alt="Screenshot from 2026-02-25 19-12-21" src="https://github.com/user-attachments/assets/268d6020-47c0-4b76-88a9-283fef406a84" />

After install it, open it :)

### Configure Qt Creator
Go to **Edit** and **Preferences**

#### Set up **Compilers**
Click on **Add** and **Custom**
<img width="1324" height="899" alt="Screenshot from 2026-02-25 19-26-11" src="https://github.com/user-attachments/assets/46b25924-0261-4b3f-a868-d81b4da34bac" />

#### Set up **Debuggers**
<img width="1324" height="899" alt="Screenshot from 2026-02-25 19-17-46" src="https://github.com/user-attachments/assets/a87db2e2-7ccf-46a7-bb36-72589de197df" />

#### Set up **Devices**
Click on **Add...**, **Remote Linux Device** and **Start Wizard**
<img width="1324" height="899" alt="Screenshot from 2026-02-25 20-41-59" src="https://github.com/user-attachments/assets/6aa427ab-f14b-472d-af44-ffd1cb5f6e0b" />
<img width="448" height="396" alt="Screenshot from 2026-02-25 20-04-44" src="https://github.com/user-attachments/assets/83699da7-4f40-48b4-9e31-f0cb22283466" />
<img width="669" height="426" alt="Screenshot from 2026-02-25 20-35-11" src="https://github.com/user-attachments/assets/205416fc-b4d2-4dc4-8025-e5d267a1182f" />

For the key deployment click on **Create New Key Pair**
Click on **Generate and Save Key Pair**, and **Deploy Public Key**

<img width="422" height="297" alt="Screenshot from 2026-02-25 20-36-19" src="https://github.com/user-attachments/assets/89de9012-9a28-4575-a02d-e5be3032d5c6" />
<img width="891" height="426" alt="Screenshot from 2026-02-25 20-36-47" src="https://github.com/user-attachments/assets/84bb2653-4efc-4521-9f2c-07e382a06eed" />
<img width="1324" height="899" alt="Screenshot from 2026-02-25 20-37-08" src="https://github.com/user-attachments/assets/85adf4af-d7d0-48e1-9d02-003dd232ad5f" />

Test the device.

<img width="648" height="646" alt="Screenshot from 2026-02-25 20-45-26" src="https://github.com/user-attachments/assets/3cf21901-6adf-4cd2-9a3a-5f4d9ce6bdb3" />


#### Set up **Qt Versions**.
<img width="1324" height="899" alt="Screenshot from 2026-02-25 19-21-08" src="https://github.com/user-attachments/assets/5f1273a2-05ca-4c54-9b74-8e414dac738e" />

#### Set up **Kits**.
<img width="1324" height="899" alt="Screenshot from 2026-02-25 20-47-18" src="https://github.com/user-attachments/assets/029e5a47-2df9-405e-808a-e3db642f8ccf" />

On **CMake Configuration** option, click Change and add follow commands. **You should modify the following commands to your needs.**
```
-DCMAKE_TOOLCHAIN_FILE:UNINITIALIZED=/home/fcha/qt-rpi-cc/qt6/pi/lib/cmake/Qt6/qt.toolchain.cmake
```
<img width="850" height="412" alt="Screenshot from 2026-02-25 20-48-05" src="https://github.com/user-attachments/assets/9490e25d-34d7-48d4-9821-09b601a63a92" />

### Test HelloWorld
Create a new project. Select **Qt Widgets Application**
<img width="908" height="586" alt="Screenshot from 2026-02-25 19-24-08" src="https://github.com/user-attachments/assets/ff38eaab-b43c-457f-9588-2bd748830292" />

Name it **HelloWorld** and put it in a directory in **qt-rpi-cc**
<img width="828" height="566" alt="Screenshot from 2026-02-25 19-24-39" src="https://github.com/user-attachments/assets/f2e3b759-db78-445c-945a-8fbc96c64c1a" />
<img width="828" height="566" alt="Screenshot from 2026-02-25 20-54-32" src="https://github.com/user-attachments/assets/4ef66ccb-ff43-43fc-81b2-aa862dc56414" />
<img width="828" height="566" alt="Screenshot from 2026-02-25 20-55-24" src="https://github.com/user-attachments/assets/5da0b33f-aa51-433e-92bb-71aaedbf81a0" />

In the **CMakeLists.txt**, add this in **target_link_libraries**
```
Qt6::DBus
```
<img width="1686" height="815" alt="Screenshot from 2026-02-25 20-57-43" src="https://github.com/user-attachments/assets/b35974dc-e1d4-4d56-b537-b026ed55d4d3" />


Goto **Projects**
Check the settings in **Run Settings**
<img width="1854" height="1048" alt="Screenshot from 2026-02-25 21-04-46" src="https://github.com/user-attachments/assets/9d24dcdf-ef72-4906-89af-04f6ef59a50a" />

In **Environment** section, click **Details** to expand the environment option. Click **Add**, then on **Variable** paste those variable
```
QT_PLUGIN_PATH=/usr/local/qt6/plugins
QML2_IMPORT_PATH=/usr/local/qt6/qml
unset DISPLAY
QT_QPA_PLATFORM=eglfs
QT_QPA_EGLFS_INTEGRATION=eglfs_kms
QT_QPA_EGLFS_ALWAYS_SET_MODE=1
QT_QPA_EGLFS_FORCE888=1
QT_QPA_EGLFS_DEBUG=1
EGL_PLATFORM=drm
```

Run.
<img width="1854" height="1048" alt="Screenshot from 2026-02-25 21-06-36" src="https://github.com/user-attachments/assets/fda65b9f-30c6-47a3-8c3b-ddd6ee60b787" />

We have HelloWorld running on rpi now. :D 

## Add QML module
Download source code.
```
cd ~/qt-rpi-cc/qt6/src
```
```
wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtshadertools-everywhere-src-6.5.1.tar.xz
```
```
tar xf qtshadertools-everywhere-src-6.5.1.tar.xz
```
```
wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtdeclarative-everywhere-src-6.5.1.tar.xz
```
```
tar xf qtdeclarative-everywhere-src-6.5.1.tar.xz
```
You can check dependencies at ~/qt-rpi-cc/qt6/src/qtdeclarative-everywhere-src-6.5.1/dependencies.yaml and ~/qt-rpi-cc/qt6/src/qtshadertools-everywhere-src-6.5.1/dependencies.yaml
Make sure required modules should be built and installed first. 

Build the modules for host
```
cd ~/qt-rpi-cc/qt6/host-build
```
```
rm -rf *
```
```
$HOME/qt-rpi-cc/qt6/host/bin/qt-configure-module ../src/qtshadertools-everywhere-src-6.5.1
```
```
cmake --build . --parallel 8
```
```
cmake --install .
```
```
rm -rf *
```
```
$HOME/qt-rpi-cc/qt6/host/bin/qt-configure-module ../src/qtdeclarative-everywhere-src-6.5.1
```
```
cmake --build . --parallel 8
```
```
cmake --install .
```
Build the modules for rpi
```
cd ~/qt-rpi-cc/qt6/pi-build
```
```
rm -rf *
```
```
$HOME/qt-rpi-cc/qt6/pi/bin/qt-configure-module ../src/qtshadertools-everywhere-src-6.5.1
```
```
cmake --build . --parallel 8
```
```
cmake --install .
```
```
rm -rf *
```
```
$HOME/qt-rpi-cc/qt6/pi/bin/qt-configure-module ../src/qtdeclarative-everywhere-src-6.5.1
```
```
cmake --build . --parallel 8
```
```
cmake --install .
```
Send the binaries to rpi. **You should modify the following commands to your needs.**
```
rsync -avz --rsync-path="sudo rsync" $HOME/qt-rpi-cc/qt6/pi/* pi@rpi4:/usr/local/qt6
```
## Test HelloWorldQml
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/f67fd349-3537-42f0-8e15-244f138a09d4)
