# syntax=docker/dockerfile:1.7

ARG DEBIAN_IMAGE=debian:bookworm-slim

FROM ${DEBIAN_IMAGE} AS download

ARG TARGETARCH
ARG MOQ_RELAY_VERSION=0.12.12
ARG MOQ_RELAY_AMD64_SHA256=be41116503ba9bd50dd003466766d0992a08d8ae0b331ff04e1403e024e2950b
ARG MOQ_RELAY_ARM64_SHA256=b56608e0e6848d2e8578c375936378e46bdf80018508bf19f7a2cf77573ed162

RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends ca-certificates curl; \
  rm -rf /var/lib/apt/lists/*; \
  case "${TARGETARCH}" in \
    amd64) release_arch="x86_64"; checksum="${MOQ_RELAY_AMD64_SHA256}" ;; \
    arm64) release_arch="aarch64"; checksum="${MOQ_RELAY_ARM64_SHA256}" ;; \
    *) echo "Unsupported TARGETARCH: ${TARGETARCH}" >&2; exit 1 ;; \
  esac; \
  url="https://github.com/moq-dev/moq/releases/download/moq-relay-v${MOQ_RELAY_VERSION}/moq-relay-v${MOQ_RELAY_VERSION}-${release_arch}-unknown-linux-gnu"; \
  curl -fsSL "${url}" -o /usr/local/bin/moq-relay; \
  echo "${checksum}  /usr/local/bin/moq-relay" | sha256sum -c -; \
  chmod 0755 /usr/local/bin/moq-relay; \
  test -x /usr/local/bin/moq-relay

FROM ${DEBIAN_IMAGE}

ARG MOQ_RELAY_VERSION=0.12.12

LABEL org.opencontainers.image.title="moq-relay"
LABEL org.opencontainers.image.description="moq-relay packaged from a pinned upstream GitHub release binary"
LABEL org.opencontainers.image.source="https://github.com/moq-dev/moq"
LABEL org.opencontainers.image.version="${MOQ_RELAY_VERSION}"
LABEL org.opencontainers.image.url="https://github.com/moq-dev/moq/releases/tag/moq-relay-v${MOQ_RELAY_VERSION}"

RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends ca-certificates; \
  rm -rf /var/lib/apt/lists/*

COPY --from=download /usr/local/bin/moq-relay /usr/local/bin/moq-relay

ENTRYPOINT ["/usr/local/bin/moq-relay"]
