FROM ubuntu:22.10

WORKDIR /tmp
# Timezone Setting for Debian GNU/Linux 10
# Ref: https://qiita.com/pyon_kiti_jp/items/e9235f9f49430bb845bb
ARG TZ=Asia/Tokyo
RUN ln -s -f /usr/share/zoneinfo/$TZ /etc/localtime 
RUN echo $TZ | tee /etc/timezone >/dev/null
ENV TZ=$TZ

# 依存関係のインストール。 参考:https://devguide.python.org/getting-started/setup-building/#build-dependencies
RUN env DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true \
    && apt update \
    && apt install -y \
        build-essential gdb lcov pkg-config \
        libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \
        libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \
        lzma lzma-dev tk-dev uuid-dev zlib1g-dev git \
        libgmp-dev libmpc-dev libmpfr-dev \
        wget time vim sudo \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Python 3.11.4をソースからインストール
RUN wget https://www.python.org/ftp/python/3.11.4/Python-3.11.4.tar.xz -O Python-3.11.4.tar.xz \
    && tar xf Python-3.11.4.tar.xz
WORKDIR /tmp/Python-3.11.4

# 追加したいconfigureのオプションがあればご自由にどうぞ
RUN ./configure --enable-optimizations \
    && make \
    # 元から入っているpythonとバッティングしないように/usr/local/bin/python3.11にインストール
    && make altinstall

WORKDIR /tmp

# setuptoolsを更新しないとcppyyのインストールでエラーが出る場合がある
ARG TARGETPLATFORM
RUN TORCHVERSION=$( \
        case ${TARGETPLATFORM} in \
            linux/amd64 ) echo "1.13.1";; \
            linux/arm64 ) echo "2.0.0";; \
        esac \
    ) \
    && python3.11 -m pip install --no-cache-dir\
        numpy==1.24.1 \
        scipy==1.10.1 \
        networkx==3.0 \
        sympy==1.11.1 \
        sortedcontainers==2.4.0  \
        more-itertools==9.0.0 \
        shapely==2.0.0 \
        bitarray==2.6.2 \
        PuLP==2.7.0 \
        mpmath==1.2.1 \
        pandas==1.5.2 \
        z3-solver==4.12.1.0 \
        scikit-learn==1.2.0 \
        ortools==9.5.2237 \
        torch==$TORCHVERSION \
        polars==0.15.15 \
        lightgbm==3.3.1 \
        gmpy2==2.1.5 \
        numba==0.57.0 \
        git+https://github.com/not522/ac-library-python \
        online-judge-tools==11.5.1 \
    && python3.11 -m pip install -U setuptools==66.0.0 \
    && python3.11 -m pip install cppyy==2.4.1

# Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
# https://unit42.paloaltonetworks.jp/non-root-containers-kubernetes-cve-2019-11245-care/
# https://qiita.com/syoyo/items/6fa6597b7a6625000e33
ARG USERNAME=oj
ARG USERPASSWD=oj
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    && echo $USERNAME:$USERPASSWD | chpasswd \
    # Add sudo support for non-root user
    && echo $USERNAME ALL=\(ALL\) ALL >> /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

USER nobody
WORKDIR /py

CMD ["python3.11"]
