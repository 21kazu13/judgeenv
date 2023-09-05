FROM ubuntu:22.10

# https://matsuand.github.io/docs.docker.jp.onthefly/engine/install/ubuntu/
RUN apt update -y \
    && apt install -y \
        ca-certificates \
        curl \
        gnupg \
        lsb-release \
        sudo \
    && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt update \
    && apt install -y docker-ce docker-ce-cli containerd.io \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
