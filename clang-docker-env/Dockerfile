FROM ubuntu

ARG UID=1000
RUN useradd -m -u ${UID} -s /bin/bash builder

RUN DEBIAN_FRONTEND="noninteractive" apt-get update && apt-get -y install tzdata wget gpg lsb-release wget software-properties-common

RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | gpg --dearmor - | tee /usr/share/keyrings/kitware-archive-keyring.gpg >/dev/null && \
    echo 'deb [signed-by=/usr/share/keyrings/kitware-archive-keyring.gpg] https://apt.kitware.com/ubuntu/ focal main' | tee /etc/apt/sources.list.d/kitware.list >/dev/null

RUN bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)" && \
    ln -s /usr/bin/clang++-14 /usr/bin/c++ && \
    ln -s /usr/bin/clang-14 /usr/bin/cc

RUN apt-get update &&  \
    apt-get install --no-install-recommends -y rsync tar git zip curl unzip pkg-config gdb cmake ninja-build && \
    apt-get clean

# ENV CC='/usr/bin/clang'
# ENV CXX='/usr/bin/clang++'

RUN git clone https://github.com/microsoft/vcpkg.git && \
    cd vcpkg && ./bootstrap-vcpkg.sh && \
    ./vcpkg install gtest benchmark && \
    chown -R builder /vcpkg

USER builder
WORKDIR /app

