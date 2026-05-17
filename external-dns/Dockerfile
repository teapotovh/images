# This file is intended to be used apart from the containing source code tree.

FROM golang:1.26-alpine AS builder

WORKDIR /

RUN apk add --no-cache git make bash

RUN git clone --depth 1 https://github.com/teapotovh/external-dns

RUN cd external-dns && make build.amd64

FROM gcr.io/distroless/static-debian13

COPY --from=builder /external-dns/build/external-dns /bin

ENTRYPOINT ["/bin/external-dns"]
