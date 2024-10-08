# Copyright (c) 2024 Hemi Labs, Inc.
# Use of this source code is governed by the MIT License,
# which can be found in the LICENSE file.

# Build stage
FROM golang:1.22.6-alpine3.20@sha256:1a478681b671001b7f029f94b5016aed984a23ad99c707f6a0ab6563860ae2f3 AS builder

ARG GO_LDFLAGS

# Add ca-certificates, timezone data, make and git
RUN apk --no-cache add --update ca-certificates tzdata make git

# Create non-root user
RUN addgroup --gid 65532 popmd && \
    adduser --disabled-password --gecos "" \
        --home "/etc/popmd/" --shell "/sbin/nologin" \
        -G popmd --uid 65532 popmd

WORKDIR /build/
COPY . .

RUN make deps
RUN GOOS=$(go env GOOS) GOARCH=$(go env GOARCH) CGO_ENABLED=0 GOGC=off make GO_LDFLAGS="$GO_LDFLAGS" popmd

# Run stage
FROM scratch

# Build metadata
ARG VERSION
ARG VCS_REF
ARG BUILD_DATE
LABEL org.opencontainers.image.created=$BUILD_DATE \
      org.opencontainers.image.authors="Hemi Labs" \
      org.opencontainers.image.url="https://github.com/hemilabs/heminetwork" \
      org.opencontainers.image.source="https://github.com/hemilabs/heminetwork" \
      org.opencontainers.image.version=$VERSION \
      org.opencontainers.image.revision=$VCS_REF \
      org.opencontainers.image.vendor="Hemi Labs" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.title="PoP Miner" \
      org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name="PoP Miner" \
      org.label-schema.url="https://github.com/hemilabs/heminetwork" \
      org.label-schema.vcs-url="https://github.com/hemilabs/heminetwork" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vendor="Hemi Labs" \
      org.label-schema.version=$VERSION \
      org.label-schema.schema-version="1.0"

# Copy files
COPY --from=builder /etc/group /etc/group
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=builder /build/bin/popmd /usr/local/bin/popmd

# Environment variables
ENV POPM_LOG_LEVEL=""
ENV POPM_BTC_PRIVKEY=""
ENV POPM_BFG_URL=""
ENV POPM_BTC_CHAIN_NAME=""
ENV POPM_PROMETHEUS_ADDRESS=""
ENV POPM_PPROF_ADDRESS=""
ENV POPM_REMINE_THRESHOLD=""
ENV POPM_STATIC_FEE=""

USER popmd:popmd
WORKDIR /etc/popmd/
ENTRYPOINT ["/usr/local/bin/popmd"]
