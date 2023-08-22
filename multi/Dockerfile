FROM python:3.11.4 as builder-cmn

WORKDIR /tmp
# RUN env DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true \
#     && apt update \
#     && apt install -y --no-install-recommends \
#         build-essential gdb lcov pkg-config \
#         libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \
#         libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \
#         lzma lzma-dev tk-dev uuid-dev zlib1g-dev git \
#         libgmp-dev libmpc-dev libmpfr-dev 

COPY req_common.txt .

RUN python -m pip install -U setuptools wheel \
    && python -m pip wheel -w /tmp/wheels -r req_common.txt

FROM ubuntu:22.10 as builder-pypy
RUN apt update \
# git : ac-library-pythonのインストールなどに使う
# wget, bzip2 : バイナリのDLと展開に必要
    && apt install -y --no-install-recommends \
        git wget bzip2 \
        gcc g++ gfortran libopenblas-dev liblapack-dev pkg-config libgeos-dev \
        # python3 and build-essential is required for cppyy installation
        build-essential python3 

# Pypy installation
# ref: https://zenn.dev/ytdrep/articles/d65c26201042eb
# https://downloads.python.org/pypy/
WORKDIR /
ARG TARGETPLATFORM
RUN TARGETPYPY=$( \
        case ${TARGETPLATFORM} in \
            linux/amd64 ) echo "linux64";; \
            linux/arm64 ) echo "aarch64";; \
        esac \
    ) && \
    wget https://downloads.python.org/pypy/pypy3.10-v7.3.12-${TARGETPYPY}.tar.bz2 --no-check-certificate \
    && tar -xvf pypy3.10-v7.3.12-${TARGETPYPY}.tar.bz2 -C /usr/local \
    && ln -s /usr/local/pypy3.10-v7.3.12-${TARGETPYPY}/bin/pypy3 /usr/local/bin/pypy3 \
    && ln -s /usr/local/pypy3.10-v7.3.12-${TARGETPYPY}/bin/pypy /usr/local/bin/pypy \
    && wget https://bootstrap.pypa.io/get-pip.py \
    && pypy3 get-pip.py --break-system-packages 
# root@2fe522bd0ffe:/usr/local/pypy3.10-v7.3.12-linux64# pwd
# /usr/local/pypy3.10-v7.3.12-linux64
# root@2fe522bd0ffe:/usr/local/pypy3.10-v7.3.12-linux64#  ls -lt
# total 32
# drwxr-xr-x 2 root root  4096 Aug 22 06:56 bin
# drwxr-xr-x 5 root root  4096 Jun 15 13:08 lib
# -rw-r--r-- 1 root root 13360 Jun 15 13:08 LICENSE
# -rw-r--r-- 1 root root  1143 Jun 15 13:08 README.rst
# drwxr-xr-x 3 root root  4096 Jun 15 12:39 include
# 
# root@a3b11e08d027:/usr/local/bin# pwd
# /usr/local/bin
# root@a3b11e08d027:/usr/local/bin# ls -lt
# total 0
# lrwxrwxrwx 1 root root 44 Aug 22 07:14 pypy -> /usr/local/pypy3.10-v7.3.12-linux64/bin/pypy
# lrwxrwxrwx 1 root root 45 Aug 22 07:14 pypy3 -> /usr/local/pypy3.10-v7.3.12-linux64/bin/pypy3

WORKDIR /tmp/wheels
COPY req_pypy.txt .
RUN --mount=type=bind,from=builder-cmn,source=/tmp/wheels,target=/tmp/wheels \
    pypy -m pip install -U setuptools wheel \
    && python -m pip wheel -w /tmp/wheels -r req_pypy.txt

# FROM python:3.11.4-slim as runner
# ENV PATH=/root/.local/bin:$PATH
# # COPY --from=builder /tmp/wheels /tmp/wheels
# RUN --mount=type=bind,from=builder,source=/tmp/wheels,target=/tmp/wheels\
#     python -m pip install -U setuptools wheel --no-cache \
#     && python -m pip install --no-index --no-deps /tmp/wheels/*.whl --no-cache
#     # && rm -rf /tmp/wheels

# RUN apt update \
#     && apt install -y --no-install-recommends \
#         gcc \
#         g++ \
#         clang \
#     && apt-get autoremove -y \
#     && apt-get clean -y \
#     && rm -rf /var/lib/apt/lists/*


# # python -c 'import cppyy, numpy, scipy, networkx, sympy, sortedcontainers, more_itertools, shapely, bitarray, pulp, mpmath, pandas, z3, sklearn, ortools, torch, polars, lightgpm, gmpy2, numba, atcoder'
# # oj --help

