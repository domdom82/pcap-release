# Dockerfile for image cf-routing.common.repositories.cloud.sap/pcap-release-testflight
FROM bosh/integration:main

ARG GINKGO_VERSION=latest
ARG GOLANGCILINT_VERSION=latest
RUN apt-get update && apt-get install -y libpcap-dev && rm -rf /var/lib/apt/lists/*

RUN rm -rf /usr/local/go
COPY --from=golang:1.20 /usr/local/go/ /usr/local/go/
ENV GOPATH=/go PATH=${PATH}:/go/bin

RUN go install "github.com/onsi/ginkgo/v2/ginkgo@${GINKGO_VERSION}" \
 && go install "github.com/golangci/golangci-lint/cmd/golangci-lint@${GOLANGCILINT_VERSION}"
