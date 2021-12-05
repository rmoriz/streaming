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

# srt-live-server
# Notes
# - adjusted LD_LIBRARY_PATH to include the patched srt lib
# - SRTLA patch applied from https://github.com/b3ck/sls-b3ck-edit/commit/c8ba19289a583d964dc5e54c746e2b24499226f5
# - upstream patch for logging on arm
COPY patches/sls-SRTLA.patch \
    patches/sls-version.patch \
    patches/480f73dd17320666944d3864863382ba63694046.patch /tmp/

ARG SRT_LIVE_SERVER_VERSION=master
RUN set -xe; \
    mkdir -p /build; \
    git clone https://github.com/IRLDeck/srt-live-server.git /build/srt-live-server; \
    cd /build/srt-live-server; \
    git checkout $SRT_LIVE_SERVER_VERSION; \
    patch -p1 < /tmp/sls-SRTLA.patch; \
    patch -p1 < /tmp/480f73dd17320666944d3864863382ba63694046.patch; \
    LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH make -j4; \
    cp bin/* /usr/local/bin;


FROM alpine:3.15.0
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

RUN set -xe; \
    apk add --no-cache \
    libstdc++ \
    libgcc \
    ;

COPY --from=builder /usr/local/lib /usr/local/lib
COPY --from=builder /usr/local/include /usr/local/include
COPY --from=builder /usr/local/bin /usr/local/bin

EXPOSE 8181/tcp 8282/udp

ENTRYPOINT ["/usr/local/bin/sls"]
CMD ["-c", "/etc/sls/sls.conf"]