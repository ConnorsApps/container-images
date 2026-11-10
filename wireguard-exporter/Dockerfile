# prometheus_wireguard_exporter + wg (it shells out to `wg`), static, in scratch.
# Baseline x86-64 only (router is an N3700, no AVX2).

ARG EXPORTER_VERSION=3.6.6
ARG WG_TOOLS_VERSION=1.0.20250521

FROM docker.io/library/rust:1-alpine AS exporter
ARG EXPORTER_VERSION
RUN apk add --no-cache musl-dev
RUN cargo install prometheus_wireguard_exporter \
      --version "${EXPORTER_VERSION}" --locked --root /out \
 && strip /out/bin/prometheus_wireguard_exporter

FROM docker.io/library/alpine:3 AS wg
ARG WG_TOOLS_VERSION
RUN apk add --no-cache build-base curl xz linux-headers
RUN curl -fsSL "https://git.zx2c4.com/wireguard-tools/snapshot/wireguard-tools-${WG_TOOLS_VERSION}.tar.xz" \
      | tar -xJ -C /tmp \
 && make -C "/tmp/wireguard-tools-${WG_TOOLS_VERSION}/src" wg LDFLAGS=-static \
 && strip "/tmp/wireguard-tools-${WG_TOOLS_VERSION}/src/wg" \
 && cp "/tmp/wireguard-tools-${WG_TOOLS_VERSION}/src/wg" /wg

FROM scratch
COPY --from=exporter /out/bin/prometheus_wireguard_exporter /prometheus_wireguard_exporter
COPY --from=wg /wg /usr/bin/wg
ENV PATH=/usr/bin
EXPOSE 9586
ENTRYPOINT ["/prometheus_wireguard_exporter"]
