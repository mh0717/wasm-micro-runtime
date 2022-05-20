# Copyright (C) 2019 Intel Corporation.  All rights reserved.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

FROM gcc:9.3.0 AS BASE

## set work directory
WORKDIR /root/
COPY resource /root/

##  - download cmake with wget and set up
RUN wget https://github.com/Kitware/CMake/releases/download/v3.23.1/cmake-3.23.1-linux-aarch64.tar.gz \
    && tar -zxvf cmake-3.23.1-linux-aarch64.tar.gz \
    && rm -f cmake-3.23.1-linux-aarch64.tar.gz \
    && mv cmake-3.23.1-linux-aarch64 /opt/cmake \
    && ln -s /opt/cmake/bin/cmake /bin/cmake \
    && apt-get install make \
    && cmake --help

##  -clone wamr-repo and build iwasm
RUN git clone -b main --depth=1 https://github.com/bytecodealliance/wasm-micro-runtime.git \
    && cd /root/wasm-micro-runtime/product-mini/platforms/linux \
    && mkdir build && cd build \
    && cmake .. -DWAMR_BUILD_DEBUG_INTERP=1 -DWAMR_BUILD_AOT=0 -DWAMR_BUILD_MULTI_MODULE=1 -DWAMR_BUILD_SHARED_MEMORY=1 -DWAMR_BUILD_BULK_MEMORY=1 -DWAMR_BUILD_THREAD_MGR=1 -DWAMR_BUILD_LIB_PTHREAD=1 -DWAMR_BUILD_SIMD=0 -DCMAKE_C_FLAGS="-DAPP_THREAD_STACK_SIZE_MIN=131072 -DAPP_THREAD_STACK_SIZE_DEFAULT=131072" && make \
    && cp /root/wasm-micro-runtime/product-mini/platforms/linux/build/iwasm /root/iwasm \
    && rm -fr /root/wasm-micro-runtime

FROM ubuntu:20.04
# COPY files from BASE image
COPY --from=BASE /root/iwasm /root
COPY --from=BASE /root/debug.sh /root
COPY --from=BASE /root/run.sh /root

RUN chmod +x /root/run.sh

WORKDIR /root/