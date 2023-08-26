# c-lightning-REST
FROM node:18-bullseye as rest
WORKDIR /rest-plugin
RUN git clone https://github.com/Ride-the-Lightning/c-lightning-REST.git . && \
    yarn

# build watchtower-client in builder
FROM rust:bullseye as teos
WORKDIR /rust-teos
RUN rustup component add rustfmt
RUN git clone https://github.com/talaia-labs/rust-teos.git . && \
    cargo install --locked --path watchtower-plugin

# compile clboss with fresh debian image
FROM debian:bullseye-slim as clboss
RUN apt update && \
    apt install -qq -y --no-install-recommends \
        autoconf-archive \
        automake \
        libtool \
        dnsutils \
        libev-dev \
        libcurl4-gnutls-dev \
        pkg-config \
        libsqlite3-dev \
        build-essential \
        ca-certificates \
        curl \
        git \
        wget
RUN git clone https://github.com/ZmnSCPxj/clboss && cd clboss && \
    autoreconf -i && \
    ./configure && \
    make -j$( [ ${MAKE_NPROC} -gt 0 ] && echo ${MAKE_NPROC} || nproc) && \
    make DESTDIR=/tmp/clboss_install install

# final image 
FROM elementsproject/lightningd as final

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        dnsutils \
        libev-dev \
        libcurl4-gnutls-dev \
        curl \
        libsqlite3-dev 

## install node for cl-rest into final-image
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash -s
RUN apt-get install -y nodejs git

RUN apt-get auto-clean && \
    rm -rf /var/lib/apt/lists/*

## copy plugins
COPY --from=rest /rest-plugin /rest-plugin
COPY --from=teos /usr/local/cargo/bin/watchtower-client /teos/watchtower-client
COPY --from=clboss /tmp/clboss_install/usr/local/bin/clboss /clboss/clboss
RUN chmod +x /clboss

# 3001 & 4001 clrest
EXPOSE 3001 4001
