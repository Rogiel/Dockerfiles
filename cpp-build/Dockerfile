FROM ubuntu:disco
RUN apt-get update && apt-get install -y \
    curl \
    git \
    clang \
    libc++-dev \
    libc++abi-dev \
    g++ \
    gdb \
    ninja-build \
    ccache
RUN curl https://github.com/Kitware/CMake/releases/download/v3.16.0-rc1/cmake-3.16.0-rc1-Linux-x86_64.sh -L > cmake.sh && \
	sh ./cmake.sh --skip-license --prefix=/usr/local && \
	rm cmake.sh
