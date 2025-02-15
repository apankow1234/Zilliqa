# Copyright (C) 2019 Zilliqa
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <https://www.gnu.org/licenses/>.
#

ARG SCILLA_VERSION=v0.13.3

# Common dependencies of the builder and runner stages.
FROM ubuntu:22.04 AS base
# Format guideline: one package per line and keep them alphabetically sorted
RUN apt-get update \
    && apt-get install -y software-properties-common \
    && apt-get update && apt-get install -y --no-install-recommends \
    autoconf \
    bison \
    build-essential \
    ca-certificates \
    ccache \
    clang-format \
    clang-tidy \
    cron \
    curl \
    dnsutils \
    gawk \
    gdb \
    git \
    htop \
    iproute2 \
    lcov \
    pigz \
    libcurl4-openssl-dev \
    libgmp-dev \
    libsecp256k1-dev \
    libssl-dev \
    libtool \
    libxml2-utils \
    logrotate \
    net-tools \
    ninja-build \
    nload \
    ocl-icd-opencl-dev \
    openssh-client \
    pkg-config \
    rsync \
    rsyslog \
    tar \
    trickle \
    unzip \
    vim \
    wget \
    zip \
    && apt-get remove -y cmake python2 python3 && apt-get autoremove -y

FROM base AS builder
ARG EXTRA_BUILD_PARAMS

# Cargo
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs -o install_script.sh && \
    sh install_script.sh -y
ENV PATH="/root/.cargo/bin:${PATH}"

RUN cargo install --git https://github.com/rrw-zilliqa/sccache --branch rrw_compile_fix sccache --locked
#RUN cargo install sccache
ENV SCCACHE_CACHE_SIZE="1G"
ENV SCCACHE_DIR="/sccache"
ENV RUSTC_WRAPPER="/root/.cargo/bin/sccache"

# CMake 3.24
ARG CMAKE_VERSION=3.24.2
RUN wget https://github.com/Kitware/CMake/releases/download/v${CMAKE_VERSION}/cmake-${CMAKE_VERSION}-Linux-x86_64.sh \
    && echo "739d372726cb23129d57a539ce1432453448816e345e1545f6127296926b6754 cmake-${CMAKE_VERSION}-Linux-x86_64.sh" | sha256sum -c \ 
    && mkdir -p "${HOME}"/.local \
    && bash ./cmake-${CMAKE_VERSION}-Linux-x86_64.sh --skip-license --prefix="${HOME}"/.local/ \
    && "${HOME}"/.local/bin/cmake --version \
    && rm cmake-${CMAKE_VERSION}-Linux-x86_64.sh
ENV PATH="/root/.local/bin:${PATH}"

# Setup ccache
RUN ln -s "$(which ccache)" /usr/local/bin/gcc \
  && ln -s "$(which ccache)" /usr/local/bin/g++ \
  && ln -s "$(which ccache)" /usr/local/bin/cc \
  && ln -s "$(which ccache)" /usr/local/bin/c++

# This tag must be equivalent to the hash specified by "builtin-baseline" in vcpkg.json
ARG VCPKG_COMMIT_OR_TAG=2022.09.27
ARG VCPKG_ROOT=/vcpkg

RUN ccache -p && ccache -z

# If COMMIT_OR_TAG is a branch name or a tag, clone a shallow copy which is
# faster; if this fails, just clone the full repo and checkout the commit.
RUN git clone https://github.com/microsoft/vcpkg ${VCPKG_ROOT} \
  && git -C ${VCPKG_ROOT} checkout ${VCPKG_COMMIT_OR_TAG} \
  && ${VCPKG_ROOT}/bootstrap-vcpkg.sh

WORKDIR /zilliqa/build

COPY vcpkg-registry/ vcpkg-registry
COPY vcpkg-configuration.json .
COPY vcpkg.json .

# python3 is unfortunately needed for upb which has a problem finding the
# vcpkg compiled python.
RUN apt-get update
RUN apt-get install -y python3-dev
RUN --mount=type=cache,target=/root/.cache/vcpkg/ ${VCPKG_ROOT}/vcpkg install --triplet=x64-linux-dynamic
RUN apt-get remove -y python3-dev && apt-get autoremove -y

ENV INSTALL_DIR=/usr/local
ENV CCACHE_DIR="/ccache"

WORKDIR /zilliqa

COPY src/libPersistence/*.proto src/libPersistence/
COPY src/libUtils/*.proto src/libUtils/
COPY evm-ds/ evm-ds
RUN --mount=type=cache,target=/sccache/ cd evm-ds && PATH=/zilliqa/build/vcpkg_installed/x64-linux-dynamic/tools/protobuf:$PATH cargo build --release --package evm-ds && sccache --show-stats

RUN ls -l /zilliqa/evm-ds/target

ARG VCPKG_PYTHON_PATH=/zilliqa/build/vcpkg_installed/x64-linux-dynamic/tools/python3
ARG VCPKG_PYTHON3=${VCPKG_PYTHON_PATH}/python3.10
ARG PYTHON3_ENV_PATH=/usr/local
RUN ${VCPKG_PYTHON3} -m ensurepip \
  && ${VCPKG_PYTHON3} -m pip install --upgrade pip \
  && mkdir -p ${PYTHON3_ENV_PATH}/bin \
  && mkdir -p ${PYTHON3_ENV_PATH}/lib \
  && cp ${VCPKG_PYTHON_PATH}/* ${PYTHON3_ENV_PATH}/bin/ \
  && cp /zilliqa/build/vcpkg_installed/x64-linux-dynamic/lib/libffi.so /usr/local/lib  \
  && cp /zilliqa/build/vcpkg_installed/x64-linux-dynamic/lib/libssl.so* /usr/local/lib  \
  && cp /zilliqa/build/vcpkg_installed/x64-linux-dynamic/lib/libcrypto.so* /usr/local/lib  \
  && ldd ${VCPKG_PYTHON3} | grep vcpkg_installed | gawk '{print $3}' | xargs -I{} cp {} /usr/local/lib \
  && cp -R /zilliqa/build/vcpkg_installed/x64-linux-dynamic/lib/python3.10 ${PYTHON3_ENV_PATH}/lib/

RUN update-alternatives --install /usr/bin/python python ${PYTHON3_ENV_PATH}/bin/python3.10 10 \ 
  && update-alternatives --install /usr/bin/python3 python3 ${PYTHON3_ENV_PATH}/bin/python3.10 10 # set python3 as default instead python3

COPY docker/requirements3.txt ./
RUN python3 -m pip install --upgrade pip setuptools wheel \
  && python3 -m pip install --no-cache-dir -r requirements3.txt

RUN echo '#!/bin/bash\npython3 -m pip "$@"' > /usr/bin/pip3 \
  && chmod a+x /usr/bin/pip3 \
  && rm -f /usr/bin/pip \
  && ln -s /usr/bin/pip3 /usr/bin/pip

COPY vcpkg-registry/ vcpkg-registry
COPY vcpkg-configuration.json .
COPY vcpkg.json .
COPY src/ src
COPY cmake/ cmake
COPY daemon/ daemon
COPY scripts/ scripts
COPY constants.xml .
COPY constants_local.xml .
COPY build.sh .
COPY CMakeLists.txt .
COPY tests/ tests
COPY isolated-server-accounts.json .

# Build & install Zilliqa
RUN --mount=type=cache,target=/ccache/ \
    --mount=type=cache,target=/root/.cache/vcpkg/ \
    CCACHE_BASEDIR="/zilliqa" ./build.sh ninja ${EXTRA_BUILD_PARAMS} && cmake --build "/zilliqa/build" --target install

# Make sure all DLLs / shared libraries are installed
RUN ldd /usr/local/bin/zilliqa | grep vcpkg_installed | gawk '{print $3}' | xargs -I{} cp {} /usr/local/lib
RUN echo "built files:" && ls -lh /zilliqa/build && echo "installed libs:" && ls -lh /usr/local/lib

# strip all exes
RUN strip /usr/local/bin/zilliqad \
   /usr/local/bin/genkeypair \
   /usr/local/bin/signmultisig \
   /usr/local/bin/verifymultisig \
   /usr/local/bin/getpub \
   /usr/local/bin/getaddr \
   /usr/local/bin/genaccounts \
   /usr/local/bin/sendcmd \
   /usr/local/bin/gentxn \
   /usr/local/bin/validateDB \
   /usr/local/bin/genTxnBodiesFromS3 \
   /usr/local/bin/getnetworkhistory \
   /usr/local/bin/isolatedServer \
   /usr/local/bin/getrewardhistory \
   /usr/local/bin/buildTxBlockHashesToNums \
#    /usr/local/bin/zilliqa \
   /usr/local/lib/libSchnorr.so \
   /usr/local/lib/libCryptoUtils.so \
   /usr/local/lib/libNAT.so \
   /usr/local/lib/libCommon.so \
   /usr/local/lib/libTrie.so

FROM zilliqa/scilla:${SCILLA_VERSION} AS scilla

# run a copy -L to unfold the symlinkes, and strip all exes
RUN mkdir -p /scilla/0/bin2/ && cp -L /scilla/0/bin/* /scilla/0/bin2/ && strip /scilla/0/bin2/*

FROM builder AS test_runner
RUN mkdir -p \
    /scilla/0/bin /scilla/0/src/stdlib /scilla/0/vcpkg_installed/x64-linux-dynamic \
    && ln -s /usr/local/lib /scilla/0/vcpkg_installed/x64-linux-dynamic
# pour in scilla binaries
COPY --from=scilla  /scilla/0/bin2                /scilla/0/bin
# pour in scilla tests
COPY --from=scilla  /scilla/0/tests                /scilla/0/tests
# pour in scilla conntract stdlibs
COPY --from=scilla  /scilla/0/src/stdlib          /scilla/0/src/stdlib
# pour in zilliqa scripts
COPY --from=builder /zilliqa/scripts         /zilliqa/scripts
# pour in zilliqa binaries and dynamic libs
WORKDIR /zilliqa


FROM base

# make dirs for scilla and zilliqa
RUN mkdir -p \
    /scilla/0/bin /scilla/0/src/stdlib \
    /zilliqa/scripts

# pour in scilla binaries
COPY --from=scilla  /scilla/0/bin2                /scilla/0/bin
# pour in scilla conntract stdlibs
COPY --from=scilla  /scilla/0/src/stdlib          /scilla/0/src/stdlib
# pour in zilliqa scripts
COPY --from=builder /zilliqa/scripts         /zilliqa/scripts
# pour in zilliqa binaries and dynamic libs
COPY --from=builder /usr/local/bin/*     /usr/local/bin/
COPY --from=builder /usr/local/lib/*.so* /usr/local/lib/
COPY --from=builder /usr/local/lib/python3.10 /usr/local/lib/python3.10
COPY --from=builder /zilliqa/evm-ds/target/release/evm-ds /usr/local/bin/
COPY --from=builder /zilliqa/evm-ds/log4rs.yml /usr/local/etc/

ADD https://github.com/krallin/tini/releases/latest/download/tini /tini

# The above COPY doesn't preserve symlinks which causes gcc/g++/cc/c++
# to hang as stand-alone binaries on Ubuntu 22.04.
RUN rm -f /usr/local/bin/gcc \
      /usr/local/bin/g++ \
      /usr/local/bin/cc \
      /usr/local/bin/c++ \
    && ln -s "$(which ccache)" /usr/local/bin/gcc \
    && ln -s "$(which ccache)" /usr/local/bin/g++ \
    && ln -s "$(which ccache)" /usr/local/bin/cc \
    && ln -s "$(which ccache)" /usr/local/bin/c++

ARG PYTHON3_ENV_PATH=/usr/local
RUN update-alternatives --install /usr/bin/python python ${PYTHON3_ENV_PATH}/bin/python3.10 10 \
  && update-alternatives --install /usr/bin/python3 python3 ${PYTHON3_ENV_PATH}/bin/python3.10 10 # set python3 as default instead python3

RUN echo '#!/bin/bash\npython3 -m pip "$@"' > /usr/bin/pip3 \
  && chmod a+x /usr/bin/pip3 \
  && rm -f /usr/bin/pip \
  && ln -s /usr/bin/pip3 /usr/bin/pip

ENV LD_LIBRARY_PATH=/usr/local/lib

ENTRYPOINT ["/bin/bash"]
