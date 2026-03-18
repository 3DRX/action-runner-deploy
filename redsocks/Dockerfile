FROM debian:bookworm-slim

ARG HTTP_PROXY
ARG HTTPS_PROXY
ARG NO_PROXY
ARG http_proxy
ARG https_proxy
ARG no_proxy

RUN apt-get update \
    && apt-get install -y --no-install-recommends redsocks nftables ca-certificates \
    && rm -rf /var/lib/apt/lists/*

COPY redsocks.conf /etc/redsocks/redsocks.conf
COPY docker-redsocks.nft /etc/redsocks/docker-redsocks.nft

CMD ["redsocks", "-c", "/etc/redsocks/redsocks.conf"]
