# Specify the architecture at build time: armv7hf
# Should be used for getting API and toolchain images
# Currently, only armv7hf is supported
ARG http_proxy
ARG https_proxy
ARG ARCH=armv7hf
ARG VERSION=3.2

FROM axisecp/acap-sdk:${VERSION}-${ARCH}-ubuntu20.04 as sdk

# Add a sources.list file that contains the armhf repositories
COPY sources.list /etc/apt
RUN apt-get update

ENV DEBIAN_FRONTEND=noninteractive
ARG OPENCV_VERSION=4.5.1
ARG TARGET_ROOT=/target-root
ENV ARCHDIR=arm-linux-gnueabihf

# Get crosscompilation toolchain
RUN dpkg --add-architecture armhf
RUN apt-get update && apt-get install -yf --no-install-recommends \
        ca-certificates \
        crossbuild-essential-armhf \
        cmake \
        curl \
        pkg-config \
 && update-ca-certificates \
 && apt-get clean

ENV PKG_CONFIG_LIBDIR=/usr/share/pkgconfig:/usr/lib/$ARCHDIR/pkgconfig:/usr/lib/$ARCHDIR/pkgconfig

WORKDIR /workspace

# Download OpenCV
RUN curl -fsSL https://github.com/opencv/opencv/archive/$OPENCV_VERSION.tar.gz | tar -xz

WORKDIR /workspace/opencv-$OPENCV_VERSION/build

# Configure OpenCV
# Platform specific optimizations in the form of NEON and VFPV3 are enabled
RUN cmake -D CMAKE_TOOLCHAIN_FILE=../platforms/linux/arm-gnueabi.toolchain.cmake \
          -D CMAKE_BUILD_TYPE=RELEASE \
          -D CMAKE_INSTALL_PREFIX=$TARGET_ROOT/usr \
          -D WITH_OPENEXR=OFF \
          -D ENABLE_NEON=ON \
          -D CPU_BASELINE=NEON,VFPV3 \
          -D ENABLE_VFPV3=ON \
          -D WITH_GTK=OFF \
          -D WITH_V4L=OFF \
          -D WITH_FFMPEG=OFF \
          -D WITH_GSTREAMER=OFF \
          -D WITH_GSTREAMER_0_10=OFF \
          -D BUILD_LIST=core,imgproc,video \
          -D BUILD_EXAMPLES=OFF \
          -D BUILD_OPENCV_APPS=OFF \
          -D BUILD_DOCS=OFF \
          -D BUILD_JPEG=ON \
          -D BUILD_PNG=OFF \
          -D WITH_JASPER=OFF \
          -D BUILD_PROTOBUF=OFF \
          -D OPENCV_GENERATE_PKGCONFIG=ON \
          ..

# Build openCV libraries and other tools
RUN make -j 2 install
RUN ln -sf $TARGET_ROOT/usr/include/opencv4/opencv2 $TARGET_ROOT/usr/include/opencv2

# Build the application
ARG PKG_ARCH=cortexa9hf-neon-poky-linux-gnueabi
RUN cp -r /target-root/* /opt/axis/acapsdk/sysroots/cortexa9hf-neon-poky-linux-gnueabi/
COPY app/ /build/src/
WORKDIR /build/src
RUN cp /opt/axis/acapsdk/sysroots/x86_64-pokysdk-linux/usr/bin/create-package.sh .
RUN . /opt/axis/acapsdk/environment-setup* && create-package.sh
