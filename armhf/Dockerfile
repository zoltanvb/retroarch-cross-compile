FROM ubuntu:xenial AS ctngbuild
ARG CTNG_UID=1000
ARG CTNG_GID=1000
RUN groupadd -g $CTNG_GID ctng
RUN useradd -d /home/ctng -m -g $CTNG_GID -u $CTNG_UID -s /bin/bash ctng
RUN apt-get update
RUN apt-get install -y gcc g++ gperf bison flex texinfo help2man make libncurses5-dev \
    python3-dev autoconf automake libtool libtool-bin gawk wget bzip2 xz-utils unzip \
    patch libstdc++6 rsync meson ninja-build
RUN mkdir /opt/ctng && chmod 777 /opt/ctng
RUN mkdir /opt/x-tools && chmod 777 /opt/x-tools
RUN echo 'export PATH=/opt/ctng/bin:$PATH' >> /etc/profile
USER ctng
RUN wget -O /tmp/crosstool.bz2 http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-1.25.0.tar.bz2
RUN cd /home/ctng && tar xvf /tmp/crosstool.bz2
RUN rm /tmp/crosstool.bz2
RUN cd /home/ctng/crosstool-ng-1.25.0 && ./configure --prefix=/opt/ctng
RUN cd /home/ctng/crosstool-ng-1.25.0 && make
RUN cd /home/ctng/crosstool-ng-1.25.0 && make install

RUN echo 'CT_CONFIG_VERSION="4"' >> /tmp/defconfig && \
    echo 'CT_PREFIX_DIR="/opt/x-tools/${CT_HOST:+HOST-${CT_HOST}/}${CT_TARGET}"' >> /tmp/defconfig && \
    echo 'CT_ARCH_ARM=y' >> /tmp/defconfig && \
    echo 'CT_OMIT_TARGET_VENDOR=y' >> /tmp/defconfig && \
    echo 'CT_ARCH_FLOAT_HW=y' >> /tmp/defconfig && \
    echo 'CT_KERNEL_LINUX=y' >> /tmp/defconfig && \
    echo 'CT_LINUX_V_4_4=y' >> /tmp/defconfig && \
    echo 'CT_BINUTILS_V_2_32=y' >> /tmp/defconfig && \
    echo 'CT_GLIBC_V_2_23=y' >> /tmp/defconfig && \
    echo 'CT_GCC_V_9=y' >> /tmp/defconfig && \
    echo 'CT_CC_LANG_CXX=y' >> /tmp/defconfig && \
    echo 'CT_CC_GCC_LIBGOMP=y' >> /tmp/defconfig
RUN cd /tmp && /opt/ctng/bin/ct-ng defconfig
RUN echo 'CT_ZLIB_MIRRORS="http://downloads.sourceforge.net/project/libpng/zlib/${CT_ZLIB_VERSION} https://www.zlib.net/ https://www.zlib.net/fossils"' >> /tmp/.config
RUN cd /tmp && /opt/ctng/bin/ct-ng build


FROM git.libretro.com:5050/libretro-infrastructure/libretro-build-amd64-ubuntu:xenial-gcc9

ENV DEBIAN_FRONTEND="noninteractive"

ARG uid
ARG branch=master
ENV branch=$branch

ENV PACKAGES \
    gcc-5-arm-linux-gnueabihf \
    g++-5-arm-linux-gnueabihf \
    pkgconf \
    gcc-multilib-arm-linux-gnueabihf \
    g++-multilib-arm-linux-gnueabihf \
    pkg-config-arm-linux-gnueabihf \
    crossbuild-essential-armhf

RUN set -eux; \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C6BF758A33A3A276 6AF7F09730B3F0A4; \
    apt-get update; \
    apt-get -y install $PACKAGES; \
    chown -R developer:developer /developer

ENV ARM_PACKAGES \
    libaio-dev:armhf \
    libasound2-dev:armhf \
    libbz2-dev:armhf \
    libcaca-dev:armhf \
    libdrm-dev:armhf \
    libffi-dev:armhf \
    libflac-dev:armhf \
    libfreetype6-dev:armhf \
    libfribidi-dev:armhf \
    libgbm-dev:armhf \
    libgdbm-dev:armhf \
    libglm-dev:armhf \
    libglu1-mesa-dev:armhf \
    libltdl-dev:armhf \
    liblua5.3-dev:armhf \
    liblzma-dev:armhf \
    libmbedtls-dev:armhf \
    libminiupnpc-dev:armhf \
    libncurses5-dev:armhf \
    libosmesa6-dev:armhf \
    libreadline-dev:armhf \
    libsixel-dev:armhf \
    libslang2-dev:armhf \
    libsystemd-dev:armhf \
    libtool:armhf \
    libtool-bin:armhf \
    libusb-1.0-0-dev:armhf \
    libv4l-dev:armhf \
    libvulkan-dev:armhf \
    libwayland-dev:armhf \
    libx11-dev:armhf \
    libx11-xcb-dev:armhf \
    libxcb-shm0-dev:armhf \
    libxkbcommon-dev:armhf \
    libxml2-dev:armhf \
    mesa-common-dev:armhf \
    uuid-dev:armhf \
    x11proto-xext-dev:armhf \
    zlib1g-dev:armhf \
    libfontconfig1:armhf \
    alsa-utils:armhf \
    libmpv-dev:armhf \
    libroar-dev:armhf \
    libssl-dev:armhf \
    libjack-jackd2-dev:armhf \
    libopenal-dev:armhf \
    libsdl2-dev:armhf \
    qtbase5-dev:armhf \
    libavutil-dev:armhf \
    libass-dev:armhf \
    libavformat-dev:armhf \
    libswscale-dev:armhf \
    libpcap0.8-dev:armhf

RUN set -eux; \
    dpkg --add-architecture armhf; \
    rm -rf /etc/apt/sources.list; \
    echo "deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports xenial main restricted" | tee -a /etc/apt/sources.list && \
    echo "deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports xenial-updates main restricted" | tee -a /etc/apt/sources.list && \
    echo "deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports xenial universe" | tee -a /etc/apt/sources.list && \
    echo "deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports xenial-updates universe" | tee -a /etc/apt/sources.list && \
    echo "deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports xenial multiverse" | tee -a /etc/apt/sources.list && \
    echo "deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports xenial-updates multiverse" | tee -a /etc/apt/sources.list && \
    echo "deb [arch=armhf] http://ports.ubuntu.com/ubuntu-ports xenial-backports main restricted universe multiverse"  | tee -a /etc/apt/sources.list && \
    apt-get update; \
    apt-get install -y $ARM_PACKAGES; \
    apt-get -y autoremove; \
    apt-get -y clean; \
    mkdir -p /opt/x-tools; \
    rm -rf /var/lib/apt/lists/*

ENV HOME=/developer
ENV CC=/usr/bin/arm-linux-gnueabihf-gcc
ENV AR=/usr/bin/arm-linux-gnueabihf-gcc-ar
ENV CXX=/usr/bin/arm-linux-gnueabihf-g++
ENV PKG_CONFIG_PATH=/usr/lib/arm-linux-gnueabihf/pkgconfig/
ENV CXX11=/usr/bin/arm-linux-gnueabihf-g++
ENV CXX17="/opt/x-tools/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-g++ -idirafter /usr/include -L/usr/lib/arm-linux-gnueabihf/"
ENV CC17="/opt/x-tools/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc -idirafter /usr/include -L/usr/lib/arm-linux-gnueabihf/"
ENV AR17=/opt/x-tools/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc-ar
ENV AS17=/opt/x-tools/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc-as

USER root
COPY --from=ctngbuild /opt/x-tools/arm-linux-gnueabihf /opt/x-tools/arm-linux-gnueabihf
WORKDIR /developer
VOLUME /developer

CMD /bin/bash
