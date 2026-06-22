# syntax=docker/dockerfile:1.7

ARG GIT_IMAGE=alpine/git:latest
ARG NIX_IMAGE=nixos/nix:latest

FROM ${GIT_IMAGE} AS source
ARG MOQ_REPO=https://github.com/moq-dev/moq.git
ARG MOQ_REF=main

WORKDIR /src
RUN git init . \
  && git remote add origin "${MOQ_REPO}" \
  && git fetch --depth 1 origin "${MOQ_REF}" \
  && git checkout --detach FETCH_HEAD \
  && git submodule update --init --recursive \
  && git rev-parse HEAD > /tmp/moq-source-commit

FROM ${NIX_IMAGE} AS builder
ARG MOQ_PACKAGE=moq-relay
ENV NIX_CONFIG="experimental-features = nix-command flakes"

WORKDIR /build
RUN mkdir -p /output/store
COPY --from=source /src/ .
COPY --from=source /tmp/moq-source-commit /output/source-commit

RUN --mount=type=cache,target=/root/.cache \
  nix build ".#${MOQ_PACKAGE}" --out-link result \
  && cp -r $(nix-store -qR result) /output/store \
  && cp -r "$(readlink -f result)" /output/result \
  && rm -rf "/output/store/$(basename "$(readlink -f result)")" \
  && printf '#!/bin/sh\nexec /bin/%s "$@"\n' "${MOQ_PACKAGE}" > /output/entry.sh \
  && chmod +x /output/entry.sh

FROM ${NIX_IMAGE}
ARG MOQ_REPO=https://github.com/moq-dev/moq.git
ARG MOQ_REF=main
ARG MOQ_PACKAGE=moq-relay

LABEL org.opencontainers.image.title="moq-relay"
LABEL org.opencontainers.image.description="moq-relay built from the upstream moq repository"
LABEL org.opencontainers.image.source="${MOQ_REPO}"
LABEL org.opencontainers.image.revision="${MOQ_REF}"

COPY --from=builder /output/entry.sh /bin/entry.sh
COPY --from=builder /output/store /nix/store
COPY --from=builder /output/result/bin/* /bin/
COPY --from=builder /output/source-commit /usr/share/moq/source-commit

RUN test -x "/bin/${MOQ_PACKAGE}" \
  && test -s /usr/share/moq/source-commit

ENTRYPOINT ["/bin/entry.sh"]
