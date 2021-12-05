FROM alpine:3.15.0 as builder

ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

RUN set -xe; \
    apk add --no-cache \
    linux-headers \
    alpine-sdk \
    ca-certificates \
    cmake \
    git \
    libssl3 \
    tcl \
    openssl-dev \
    zlib-dev \
    ;

# belabox patched srt
#
ARG BELABOX_SRT_VERSION=master
RUN set -xe; \
    mkdir -p /build; \
    git clone https://github.com/BELABOX/srt.git /build/srt; \
    cd /build/srt; \
    git checkout $BELABOX_SRT_VERSION; \
    ./configure --prefix=/usr/local; \
    make -j4; \
    make install;

# belabox srtla
#
ARG SRTLA_VERSION=main
RUN set -xe; \
    mkdir -p /build; \
    git clone https://github.com/BELABOX/srtla.git /build/srtla; \
    cd /build/srtla; \
    git checkout $SRTLA_VERSION; \
    make -j4;

RUN cp /build/srtla/srtla_rec /build/srtla/srtla_send /usr/local/bin


FROM alpine:3.15.0

ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
COPY --from=builder /usr/local/bin /usr/local/bin

ENV SRTLA_INGRESS_PORT=5000
ENV SLS_INGRESS_HOST=localhost
ENV SLS_INGRESS_PORT=8282

EXPOSE 5000/udp