FROM node:18 as builder-acc
ENV NODE_OPTIONS=--openssl-legacy-provider
WORKDIR /tmp
RUN git clone https://github.com/Tatamo/atcoder-cli.git
WORKDIR /tmp/atcoder-cli
RUN npm install \
    && npm install -g