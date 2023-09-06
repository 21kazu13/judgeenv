# Declear common argument among multi stages
# https://qiita.com/ktooi/items/9c12802deb52eccf0858
ARG CERT_PATH=/etc/ssl/certs/ca-certificates.crt
ARG CERT_DIR=/etc/ssl/certs/

###############################################################
### Create and prepare self signed certificate for netscope ###
###############################################################
# Install first, then copy and use /etc/ssl/certs/ca-certificates.crt into other stages
# https://stakiran.hatenablog.com/entry/2018/06/12/213727
# http://www.maibun.org/~nt/technicalnote/centos8/ssl4.html
# https://qiita.com/kusanoiskuzuno/items/cf35b65a3c42c11be67c
# https://qiita.com/hi-naoya/items/d325a57d24a39e4d03e8
FROM ubuntu:22.10 as cert
COPY dotfiles/* /usr/share/ca-certificates/extra/
RUN apt update && \
    apt-get install -y ca-certificates
RUN if [ -e /usr/share/ca-certificates/extra/nscacert.pem ]; then \
        # update-ca-certificates will fail without this.
        # http://www.maibun.org/~nt/technicalnote/centos8/ssl4.html
        echo 'extra/nscacert.pem' >> /etc/ca-certificates.conf ; \
    fi
RUN update-ca-certificates

FROM ubuntu:22.10

ARG CERT_PATH
ARG CERT_DIR

# Be careful the order of installation for ca-certificates and COPY from cert.
RUN apt update -y \
    && apt install -y --no-install-recommends \
        ca-certificates
COPY --from=cert $CERT_PATH $CERT_PATH
ENV CERT_PATH=${CERT_PATH}
ENV SSL_CERT_FILE=${CERT_PATH}
ENV CERT_DIR=${CERT_DIR}
ENV SSL_CERT_DIR=${CERT_DIR}
# https://help.zscaler.com/ja/zia/adding-custom-certificate-application-specific-trust-store
ENV REQUESTS_CA_BUNDLE=${CERT_PATH}
# https://help.zscaler.com/ja/zia/adding-custom-certificate-application-specific-trust-store
ENV NODE_EXTRA_CA_CERTS=${CERT_PATH}

# https://matsuand.github.io/docs.docker.jp.onthefly/engine/install/ubuntu/
RUN apt install -y --no-install-recommends \
        curl \
        gnupg \
        lsb-release \
        sudo \
    && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && apt update \
    && apt install -y docker-ce docker-ce-cli --no-install-recommends \
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
