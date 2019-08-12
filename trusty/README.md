## OSMesa

Followed [this guide](https://ubuntuforums.org/showthread.php?t=910717), verified correct directory structure and contents by looking at [this deb](https://github.com/mmatl/travis_debs/blob/master/xenial/mesa_18.3.3-0.deb) built for 16.04.

Built and installed from source using following lines (in Dockerfile form):

    # Copy and rename an apt lib file so that apt-add-repository works (cleaner way would be to symlink it
    # but Dockerfiles don't seem to like symlinks)
    RUN cp /usr/lib/python3/dist-packages/apt_pkg.cpython-34m-x86_64-linux-gnu.so /usr/lib/python3/dist-packages/apt_pkg.cpython-36m-x86_64-linux-gnu.so

    # Add new apt repositories and then apt-add some OSMesa deps
    RUN apt-add-repository "deb http://apt.llvm.org/trusty/ llvm-toolchain-trusty-6.0 main"
    RUN add-apt-repository -y ppa:ubuntu-toolchain-r/test
    RUN apt-get update && apt-get install --yes llvm-6.0 freeglut3 freeglut3-dev pkg-config

    # Download OSMesa, then build and install it
    RUN curl -o mesa-18.3.3.tar.gz ftp://ftp.freedesktop.org/pub/mesa/mesa-18.3.3.tar.gz
    RUN tar xfv mesa-18.3.3.tar.gz
    WORKDIR ./mesa-18.3.3
    RUN ./configure --prefix=/usr/local                           \
                --enable-opengl --disable-gles1 --disable-gles2   \
                --disable-va --disable-xvmc --disable-vdpau       \
                --enable-shared-glapi                             \
                --disable-texture-float                           \
                --enable-gallium-llvm --enable-llvm-shared-libs   \
                --with-gallium-drivers=swrast,swr                 \oyth
                --disable-dri --with-dri-drivers=                 \
                --disable-egl --with-egl-platforms= --disable-gbm \
                --disable-glx                                     \
                --disable-osmesa --enable-gallium-osmesa          \
                ac_cv_path_LLVM_CONFIG=llvm-config-6.0
    RUN make -j8
    RUN make install
