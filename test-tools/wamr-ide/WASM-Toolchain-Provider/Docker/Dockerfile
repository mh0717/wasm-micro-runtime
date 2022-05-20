# Copyright (C) 2019 Intel Corporation.  All rights reserved.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

##  Build docker image that consists of gcc, cmake, wasi-sdk
FROM gcc:9.3.0 AS BASE

## set work directory
WORKDIR /root/

COPY resource /root/

##  set compilation environment for wamrc
#    - cmake
#    - wasi-sdk
#    - wamr-sdk

##  - download cmake with wget and set up
# RUN wget https://github.com/Kitware/CMake/releases/download/v3.23.1/cmake-3.23.1-macos-universal.tar.gz \
#     && tar -zxvf cmake-3.23.1-macos-universal.tar.gz \
#     && rm -f cmake-3.23.1-macos-universal.tar.gz \
#     && mv cmake-3.23.1-macos-universal /opt/cmake \
#     && ln -s /opt/cmake/bin/cmake /bin/cmake \
#     && apt-get install make

RUN wget https://github.com/Kitware/CMake/releases/download/v3.23.1/cmake-3.23.1-linux-aarch64.tar.gz \
    && tar -zxvf cmake-3.23.1-linux-aarch64.tar.gz \
    && rm -f cmake-3.23.1-linux-aarch64.tar.gz \
    && mv cmake-3.23.1-linux-aarch64 /opt/cmake \
    && ln -s /opt/cmake/bin/cmake /bin/cmake \
    && apt-get install make \
    && cmake --help

##  - download wasi-sdk with wget and set up to /opt/wasi-sdk
# RUN wget https://github.com/WebAssembly/wasi-sdk/releases/download/wasi-sdk-15/wasi-sdk-15.0-macos.tar.gz \
#     && tar -zxvf wasi-sdk-15.0-macos.tar.gz \
#     && rm -f wasi-sdk-15.0-macos.tar.gz
RUN wget https://github.com/mh0717/myimages/releases/download/v0.3/wasi-sdk.tar.gz \
    && tar -zxvf wasi-sdk.tar.gz \
    && rm -f wasi-sdk.tar.gz

RUN git clone -b main --depth=1 https://github.com/bytecodealliance/wasm-micro-runtime.git \
    && cd /root/wasm-micro-runtime/wamr-compiler \
    && ./build_llvm.sh --arch AArch64 --project clang --platform darwin \
    #  - build wamrc
    && cd /root/wasm-micro-runtime/wamr-compiler \
    && mkdir build \
    && cd build \
    && cmake -DWAMR_BUILD_TARGET=AArch64 .. \
    && make \
    #  - copy the wamrc to /root
    && cp /root/wasm-micro-runtime/wamr-compiler/build/wamrc /root/wamrc \
    && mkdir -p /opt/wamr-sdk/app \
    && cp -r /root/wasm-micro-runtime/wamr-sdk/app/libc-builtin-sysroot /opt/wamr-sdk/app/ \
    #  - remove the wamr repo to save the size
    && rm -fr /root/wasm-micro-runtime

## STAGE 2
FROM ubuntu:20.04
RUN mkdir -p /opt/wasi-sdk \
 && mkdir -p /opt/cmake \
 && mkdir -p /opt/wamr-sdk/app

# COPY files from BASE image
COPY --from=BASE /opt/cmake/ /opt/cmake/
COPY --from=BASE /opt/wamr-sdk/app/ /opt/wamr-sdk/app/
COPY --from=BASE /root/wasi-sdk/ /opt/wasi-sdk/
COPY --from=BASE /root/wamrc /root
COPY --from=BASE /root/build_wasm.sh /root

RUN chmod +x /root/build_wasm.sh

RUN ln -s /opt/cmake/bin/cmake /usr/bin/cmake \
 && ln -s /root/wamrc /usr/bin/wamrc
RUN apt-get update && apt-get install make

WORKDIR /root