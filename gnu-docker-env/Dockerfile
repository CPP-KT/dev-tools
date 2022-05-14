FROM ubuntu:20.04

ARG UID=1000
RUN useradd -m -u ${UID} -s /bin/bash builder

RUN DEBIAN_FRONTEND="noninteractive" apt-get update && \
    apt-get -y install tzdata wget gpg lsb-release wget software-properties-common

RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null && \
    echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ focal main' | tee /etc/apt/sources.list.d/kitware.list >/dev/null && \
    add-apt-repository ppa:ubuntu-toolchain-r/test

RUN apt-get update &&  \
    apt-get install --no-install-recommends -y rsync tar git zip curl unzip pkg-config gdb cmake ninja-build gcc-11 g++-11 binutils linux-tools-common linux-tools-generic && \
    apt-get clean && \
    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110 \
        --slave /usr/bin/g++ g++ /usr/bin/g++-11 \
        --slave /usr/bin/gcov gcov /usr/bin/gcov-11 \
        --slave /usr/bin/gcc-ar gcc-ar /usr/bin/gcc-ar-11 \
        --slave /usr/bin/gcc-ranlib gcc-ranlib /usr/bin/gcc-ranlib-11 \
        --slave /usr/bin/cpp cpp /usr/bin/cpp-11

RUN ln -fs /usr/lib/linux-tools/*/perf /usr/bin/perf

RUN ln -s /usr/bin/g++ /usr/bin/c++ && \
    ln -s /usr/bin/gcc /usr/bin/cc

RUN git clone https://github.com/microsoft/vcpkg.git && \
    cd vcpkg && ./bootstrap-vcpkg.sh && \
    ./vcpkg install gtest benchmark && \
    chown -R builder /vcpkg

USER builder
WORKDIR /app

