FROM alpine@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c as builder
ARG GIT_TAG=v4.41-9782-beta
RUN mkdir /usr/local/src && apk add binutils --no-cache\
  build-base \
  readline-dev \
  openssl-dev \
  ncurses-dev \
  git \
  cmake \
  zlib-dev \
  libsodium-dev \
  gnu-libiconv 

ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so
WORKDIR /usr/local/src
RUN git clone -b ${GIT_TAG} https://github.com/SoftEtherVPN/SoftEtherVPN_Stable.git
ENV USE_MUSL=YES
RUN cd SoftEtherVPN_Stable &&\
  git submodule init &&\
  git submodule update &&\
  ./configure &&\
  make
RUN ls /usr/local/src/SoftEtherVPN_Stable/bin/vpnclient

FROM alpine@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c

RUN apk add --no-cache readline \
  openssl \
  libsodium \
  gnu-libiconv\
  iptables\
  dhclient

ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so
ENV LD_LIBRARY_PATH /root
WORKDIR /usr/local/bin
VOLUME /mnt
RUN ln -s /mnt/vpn_client.config vpn_client.config && \
  mkdir /mnt/backup.vpn_client.config &&\
  ln -s /mnt/backup.vpn_client.config backup.vpn_client.config &&\
  ln -s /mnt/lang.config lang.config
COPY --from=builder /usr/local/src/SoftEtherVPN_Stable/bin/vpnclient/vpnclient /usr/local/src/SoftEtherVPN_Stable/bin/vpncmd/vpncmd /usr/local/src/SoftEtherVPN_Stable/bin/vpnclient/hamcore.se2 ./
COPY ./interfaces /etc/network/interfaces
COPY --chmod=777 ./nic.sh /usr/local/bin/nic.sh

COPY --chmod=777 docker-entrypoint.sh /tmp

ENV VPN_DOMAIN=vpn.example.com

USER root
ENTRYPOINT ["/tmp/docker-entrypoint.sh"]
