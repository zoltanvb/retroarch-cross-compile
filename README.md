# retroarch-cross-compile

Container image configurations to enable cross-compilation of retroarch and libretro cores, where host is a generic x86-64 machine, and target is something else. Currently, armhf target is available.

# armhf
Image that can compile Retroarch and libretro cores for ARM hardfloat platforms. Tested with armv7.

## Building the image
`cd armhf  
docker build .`  
Note that build process will take a while (up to one hour), as it will include building a complete toolchain with crosstools-ng.

## Using the image to build retroarch / cores
Clone the repository to build (RetroArch, individual core, or even libretro-super), then:  
`sudo docker run --rm -it -v "<cloned dir>:/build" <image name>`  
`cd /build`  
Anything that you put under /build, will be preserved after you exit the container, others will be permanently lost.
### Building RetroArch
`./configure --host=arm-linux-gnueabihf`  
`make -j <CPU count of your builder machine>`  
Note that there is no sense in doing "make install" inside the container. You will have the binary, but you can't execute it on your x86_64 host. But you can transfer the resulting binary to your ARM system, and run it there.

### Building cores with libretro-super
`export JOBS=<CPU count of your builder machine>`  
`export platform=linux-armv7-neon`  
`./libretro-fetch.sh <core name>`  
`./libretro-build.sh <core name>`  
Note that this will produce a binary .so compiled with `platform=unix-armv7-hardfloat-neon`. Compiled library will be copied to `dist/unix`.

### Building individual cores (or anything else)
There are two sets of environment variables set up in the image. The default is the toolchain installed from Ubuntu:
- `CC=/usr/bin/arm-linux-gnueabihf-gcc`
- `AR=/usr/bin/arm-linux-gnueabihf-gcc-ar`
- `CXX=/usr/bin/arm-linux-gnueabihf-g++`
- `PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig/`  

The other one is the toolchain built by crosstools-ng, extended with libs of the default toolchain:
- `CXX17="/opt/x-tools/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-g++ -idirafter /usr/include -L/usr/lib/arm-linux-gnueabihf/"`
- `CC17="/opt/x-tools/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc -idirafter /usr/include -L/usr/lib/arm-linux-gnueabihf/"`
- `AR17=/opt/x-tools/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc-ar`  

It is highly dependent on the build system, whether cross compilation will be possible, and if so, what platform / arch / target values need to be used. Libretro cores usually honor values of CC/CXX, so to use the newer compiler, try `CC=$(CC17) CXX=$(CXX17) make platform=unix-armv7`. But there is no guarantee that build system will recognize this target correctly, you need to read the makefile.

## More details
The reason why the build is based on such an old Ubuntu release is to avoid any library dependency problem when using the compiled images. The libretro base image can probably be substituted with a clean build from xenial as well, but it speeds up the build process.

The image has the Linaro gcc5 by default, and gcc9.4 compiled as extra. Hardfloat setting is coming from compiler (both compilers). Architecture armv7 is in gcc5 default. The gcc9 setup tries to mimic the environment used for gcc5, to avoid any compatibility problems. Usage of neon is not hardcoded anywhere, except for the target name in libretro-super.

## Caveats
- `gcc` is present, but it produces x86_64 code, if this happens, makefile has redefined CC/CXX. Use `readelf -h` on the produced binary to check.
hardfloat is coming from compiler (both compilers)
- compiling for armv6 is theoretically supported but needs to be tested (see https://stackoverflow.com/questions/35132319/build-for-armv6-with-gnueabihf/51201725#51201725)
- `cmake` is present but wasn't tested in detail
- image does not contain any particular HW library like `libbrcmEGL` for Raspberry Pi. so it will not be able to link against that. This means a severe performance hit for any core that uses OpenGL (or Retroarch itself).
- image is for Unix/Linux target. Does not contain any Android or iOS tools.
