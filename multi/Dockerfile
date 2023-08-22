FROM python:3.11.4 as builder-python

WORKDIR /tmp
COPY req_python.txt .

RUN python -m pip install -U setuptools==66.0.0 wheel \
    && python -m pip wheel -w /tmp/wheels -r req_python.txt

FROM pypy:3.10-7.3.12 as builder-pypy
# RUN apt update \
# # git : ac-library-pythonのインストールなどに使う
# # wget, bzip2 : バイナリのDLと展開に必要
#     && apt install -y --no-install-recommends \
#         git wget bzip2 \
#         gcc g++ gfortran libopenblas-dev liblapack-dev pkg-config libgeos-dev \
#         # python3 and build-essential is required for cppyy installation
#         build-essential python3

RUN apt update \
    && apt install -y --no-install-recommends \
        # ../../meson.build:63:0: ERROR: Unknown compiler(s): [['gfortran'], ['flang'], ['nvfortran'], ['pgfortran'], ['ifort'], ['ifx'], ['g95']]
        gfortran \
        # ../../scipy/meson.build:134:7: ERROR: Dependency "OpenBLAS" not found, tried pkgconfig
        # https://qiita.com/itotomball/items/3ec9eddbe0f6cbab66ad
        libopenblas-dev \
        # src/geos.h:15:10: fatal error: geos_c.h: No such file or directory
        # https://techoverflow.net/2021/08/13/6348/
        libgeos-dev

WORKDIR /tmp
COPY req_pypy.txt .
RUN pypy -m pip install -U setuptools==66.0.0 wheel \
    && pypy -m pip wheel -w /tmp/wheels -r req_pypy.txt

# root@72f80cdd8f7a:/opt/pypy# ls -lt
# total 32
# drwxr-xr-x 1 root root  4096 Aug 16 10:32 bin
# drwxr-xr-x 1 root root  4096 Jun 15 13:08 lib
# -rw-r--r-- 1 root root 13360 Jun 15 13:08 LICENSE
# -rw-r--r-- 1 root root  1143 Jun 15 13:08 README.rst
# drwxr-xr-x 3 root root  4096 Jun 15 12:39 include
# root@72f80cdd8f7a:/opt/pypy/bin# ls -lt
# total 73428
# -rwxr-xr-x 1 root root      224 Aug 16 10:32 pip
# -rwxr-xr-x 1 root root      224 Aug 16 10:32 pip3
# -rwxr-xr-x 1 root root      224 Aug 16 10:32 pip3.10
# -rwxr-xr-x 1 root root      211 Aug 16 10:32 wheel
# -rwxrwxr-x 1 root root 59789776 Jun 15 13:08 libpypy3.10-c.so
# -rw-r--r-- 1 root root 15353848 Jun 15 13:08 libpypy3.10-c.so.debug
# lrwxrwxrwx 1 root root        8 Jun 15 13:08 pypy -> pypy3.10
# lrwxrwxrwx 1 root root        8 Jun 15 13:08 pypy3 -> pypy3.10
# -rwxr-xr-x 1 root root    14504 Jun 15 13:08 pypy3.10
# -rw-r--r-- 1 root root     4856 Jun 15 13:08 pypy3.10.debug
# lrwxrwxrwx 1 root root        8 Jun 15 13:08 python -> pypy3.10
# lrwxrwxrwx 1 root root        8 Jun 15 13:08 python3 -> pypy3.10
# lrwxrwxrwx 1 root root        8 Jun 15 13:08 python3.10 -> pypy3.10

FROM python:3.11.4-slim as runner

COPY --from=builder-pypy /opt/pypy /usr/local/pypy
RUN ln -s /usr/local/pypy/bin/pypy3 /usr/local/bin/pypy3 \
    && ln -s /usr/local/pypy/bin/pypy /usr/local/bin/pypy 

# Pypy installation
# ref: https://zenn.dev/ytdrep/articles/d65c26201042eb
# https://downloads.python.org/pypy/
# WORKDIR /
# ARG TARGETPLATFORM
# RUN TARGETPYPY=$( \
#         case ${TARGETPLATFORM} in \
#             linux/amd64 ) echo "linux64";; \
#             linux/arm64 ) echo "aarch64";; \
#         esac \
#     ) && \
#     wget https://downloads.python.org/pypy/pypy3.10-v7.3.12-${TARGETPYPY}.tar.bz2 --no-check-certificate \
#     && tar -xvf pypy3.10-v7.3.12-${TARGETPYPY}.tar.bz2 -C /usr/local \
#     && ln -s /usr/local/pypy3.10-v7.3.12-${TARGETPYPY}/bin/pypy3 /usr/local/bin/pypy3 \
#     && ln -s /usr/local/pypy3.10-v7.3.12-${TARGETPYPY}/bin/pypy /usr/local/bin/pypy \
#     && wget https://bootstrap.pypa.io/get-pip.py \
#     && pypy3 get-pip.py --break-system-packages 
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

# WORKDIR /tmp/wheels
# COPY req_pypy.txt .
# # https://www.unixcloudfusion.in/2022/02/solved-invalid-argument-typebind-for.html
# RUN --mount=type=bind,from=builder-cmn,source=/tmp/wheels,target=/tmp/wheels \
#     pypy -m pip install -U setuptools wheel \
#     && python -m pip wheel -w /tmp/wheels -r req_pypy.txt

# FROM python:3.11.4-slim as runner
# ENV PATH=/root/.local/bin:$PATH
# # COPY --from=builder /tmp/wheels /tmp/wheels
RUN --mount=type=bind,from=builder-python,source=/tmp/wheels,target=/tmp/wheels/python \
    --mount=type=bind,from=builder-pypy,source=/tmp/wheels,target=/tmp/wheels/pypy \
    python -m pip install -U setuptools==66.0.0 wheel --no-cache \
    && python -m pip install --no-index --no-deps /tmp/wheels/python/*.whl --no-cache \
    pypy -m pip install -U setuptools==66.0.0 wheel --no-cache \
    && pypy -m pip install --no-index --no-deps /tmp/wheels/pypy/*.whl --no-cache \
    # && rm -rf /tmp/wheels

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

