FROM ubuntu:22.10

WORKDIR /tmp
# Timezone Setting for Debian GNU/Linux 10
# Ref: https://qiita.com/pyon_kiti_jp/items/e9235f9f49430bb845bb
ARG TZ=Asia/Tokyo
RUN ln -s -f /usr/share/zoneinfo/$TZ /etc/localtime 
RUN echo $TZ | tee /etc/timezone >/dev/null
ENV TZ=$TZ

RUN apt update \
# git : ac-library-pythonのインストールなどに使う
# wget, bzip2 : バイナリのDLと展開に必要
    && apt install -y --no-install-recommends \
        git wget bzip2 \
        gcc g++ gfortran libopenblas-dev liblapack-dev pkg-config libgeos-dev \
        # python3 and build-essential is required for cppyy installation
        build-essential python3 \
        # oj required these three
        #   Building lxml version 4.9.3.
        #   Building without Cython.
        #   Error: Please make sure the libxml2 and libxslt development packages are installed.
        #   [end of output]
        libxml2-dev libxslt-dev zlib1g-dev \
        # GNU time is required memory calculation in oj
        vim time sudo \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

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
    wget https://downloads.python.org/pypy/pypy3.10-v7.3.12-${TARGETPYPY}.tar.bz2 \
    && tar -xvf pypy3.10-v7.3.12-${TARGETPYPY}.tar.bz2 -C /usr/local \
    && ln -s /usr/local/pypy3.10-v7.3.12-${TARGETPYPY}/bin/pypy3 /usr/local/bin/pypy3 \
    && ln -s /usr/local/pypy3.10-v7.3.12-${TARGETPYPY}/bin/pypy /usr/local/bin/pypy \
    && wget https://bootstrap.pypa.io/get-pip.py \
    && pypy3 get-pip.py --break-system-packages 

RUN pypy -m pip install -U setuptools wheel \
    pypy -m pip wheel -w /tmp/wheels\
    numpy==1.24.1 \
    scipy==1.10.1 \
    networkx==3.0 \
    sympy==1.11.1 \
    sortedcontainers==2.4.0 \
    more-itertools==9.0.0 \
    shapely==2.0.0 \
    bitarray==2.6.2 \
    PuLP==2.7.0 \
    mpmath==1.2.1 \
    pandas==1.5.2 \
    z3-solver==4.12.1.0 \
    scikit-learn==1.3.0 \
    typing-extensions==4.4.0 \
    cppyy==2.4.1 \
    git+https://github.com/not522/ac-library-python \
    online-judge-tools==11.5.1

ENV PATH=$PATH:/usr/local/pypy3.10-v7.3.12-aarch64/bin

RUN pypy3 -m pip install --break-system-packages --no-cache \
    numpy==1.24.1 \
    scipy==1.10.1 \
    networkx==3.0 \
    sympy==1.11.1 \
    sortedcontainers==2.4.0 \
    more-itertools==9.0.0 \
    shapely==2.0.0 \
    bitarray==2.6.2 \
    PuLP==2.7.0 \
    mpmath==1.2.1 \
    pandas==1.5.2 \
    z3-solver==4.12.1.0 \
    scikit-learn==1.3.0 \
    typing-extensions==4.4.0 \
    cppyy==2.4.1 \
    git+https://github.com/not522/ac-library-python \
    online-judge-tools==11.5.1

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

CMD ["pypy3"]
