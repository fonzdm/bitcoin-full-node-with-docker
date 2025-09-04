FROM debian:stable-slim AS build
ARG BITCOIN_VERSION
RUN apt-get update \
  && apt-get install -y \
    build-essential \
    cmake \
    curl \
    git \
    gpg \
    ninja-build \
    pkg-config \
  && cd /tmp \
  && curl \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/achow101.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/fanquake.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/glozow.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/guggero.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/hebasto.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/theStack.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/vertiond.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/willyko.gpg \
  && gpg --import *.gpg \
  && git clone -b v${BITCOIN_VERSION} https://github.com/bitcoin/bitcoin.git \
  && cd bitcoin \
  && git verify-tag v${BITCOIN_VERSION} \
  \
  # Build depends
  && cd depends \
  && make HOST=$(gcc -dumpmachine) NO_BDB=1 NO_QT=1 NO_QR=1 NO_SQLITE=1 NO_NATPMP=1 NO_UPNP=1 NO_USDT=1 NO_WALLET=1 NO_HARDEN=1 \
  \
  # Build Bitcoin Core using CMake
  && cd .. \
  && mkdir build \
  && cd build \
  && cmake .. \
    -GNinja \
    -DCMAKE_TOOLCHAIN_FILE=../depends/$(gcc -dumpmachine)/share/toolchain.cmake \
    -DCMAKE_INSTALL_PREFIX=../depends/$(gcc -dumpmachine) \
    -DBUILD_BITCOIN_CLI=ON \
    -DBUILD_BITCOIN_QT=OFF \
    -DBUILD_BITCOIN_UTILS=OFF \
    -DBUILD_TESTS=OFF \
    -DBUILD_BENCH=OFF \
    -DENABLE_MAN=OFF \
    -DENABLE_EXTERNAL_SIGNER=OFF \
    -DENABLE_ZMQ=ON \
    -DENABLE_WALLET=OFF \
    -DENABLE_USDT=OFF \
    -DENABLE_UPNP=OFF \
    -DENABLE_NATPMP=OFF \
    -DENABLE_REDUCE_EXPORTS=ON \
  && ninja src/bitcoind src/bitcoin-cli \
  && strip src/bitcoind src/bitcoin-cli \
  && apt-get remove -y build-essential cmake curl git gpg ninja-build pkg-config


FROM debian:stable-slim
ARG BITCOIN_USER_ID
ARG GROUP_ID
COPY --from=build /tmp/bitcoin/build/src/bitcoind /usr/local/bin/bitcoind
COPY --from=build /tmp/bitcoin/build/src/bitcoin-cli /usr/local/bin/bitcoin-cli
RUN addgroup --gid ${GROUP_ID} bitcoin \
  && adduser -u ${BITCOIN_USER_ID} --disabled-password --gecos "" --ingroup bitcoin bitcoin \
  && mkdir /home/bitcoin/.bitcoin \
  && chown bitcoin:bitcoin /home/bitcoin/.bitcoin

USER bitcoin

CMD ["bitcoind"]
