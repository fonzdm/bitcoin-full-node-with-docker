# If you change this please also update GO_VERSION in Makefile (then run
# `make lint` to see where else it needs to be updated as well).
FROM golang:1.23.9-alpine AS builder
ARG LND_VERSION

# Force Go to use the cgo based DNS resolver. This is required to ensure DNS
# queries required to connect to linked containers succeed.
ENV GODEBUG netdns=cgo

# Install dependencies.
RUN apk add --no-cache --update alpine-sdk \
    bash \
    git \
    make 

# Copy in the local repository to build from.
WORKDIR /

RUN git clone -b v$LND_VERSION https://github.com/lightningnetwork/lnd.git \
    &&  cd lnd \
    &&  make \
    &&  make install-all tags="signrpc walletrpc chainrpc invoicesrpc peersrpc kvdb_sqlite"

# Start a new, final image to reduce size.
FROM alpine AS final

ARG LND_USER_ID
ARG GROUP_ID

# Expose lnd ports (server, rpc).
EXPOSE 9735 10009

# Copy the binaries and entrypoint from the builder image.
COPY --from=builder /go/bin/lncli /bin/
COPY --from=builder /go/bin/lnd /bin/
COPY --from=builder /lnd/docker/lnd/start-lnd.sh /bin/

RUN addgroup -g $GROUP_ID lnd
RUN adduser --disabled-password --gecos "" --ingroup lnd --uid "$LND_USER_ID" lnd \
    && mkdir /home/lnd/.lnd \
    && chown lnd:lnd /home/lnd/.lnd


# Add bash.
RUN apk add --no-cache \
    bash pass

RUN chmod +x /bin/start-lnd.sh

USER lnd

CMD /bin/start-lnd.sh
