# nftables-exporter + static nft (it runs `nft -j -t list ruleset`), in scratch.
# Baseline x86-64 only (router is an N3700, no AVX2).

ARG NFTABLES_VERSION=1.1.6

FROM docker.io/library/golang:alpine AS exporter
ARG REPO=https://github.com/ConnorsApps/nftables-exporter.git
# Full SHA. Use 48a36b2... (unmerged feat/terse-mode): master lacks `nft -t`.
ARG REF
RUN test -n "${REF}" && apk add --no-cache git
RUN git init -q /src && git -C /src fetch -q --depth 1 "${REPO}" "${REF}" && git -C /src checkout -q FETCH_HEAD
WORKDIR /src
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -tags netgo \
      -ldflags "-s -w -X github.com/metal-stack/v.Version=${REF} -X github.com/metal-stack/v.GitSHA1=${REF}" \
      -o /nftables-exporter .

FROM docker.io/library/alpine:3 AS nft
ARG NFTABLES_VERSION
RUN apk add --no-cache build-base bash curl xz linux-headers \
      libmnl-dev libmnl-static libnftnl-dev jansson-dev jansson-static
RUN curl -fsSL "https://netfilter.org/projects/nftables/files/nftables-${NFTABLES_VERSION}.tar.xz" \
      | tar -xJ -C /tmp
WORKDIR /tmp/nftables-${NFTABLES_VERSION}
RUN CONFIG_SHELL=/bin/bash bash ./configure --disable-shared --enable-static --without-cli --with-mini-gmp --with-json \
      --disable-man-doc --disable-python --disable-debug \
 && make -j"$(nproc)" LDFLAGS=-all-static \
 && strip src/nft && cp src/nft /nft

FROM scratch
COPY --from=exporter /nftables-exporter /nftables-exporter
COPY --from=nft /nft /sbin/nft
EXPOSE 9635
ENTRYPOINT ["/nftables-exporter"]
