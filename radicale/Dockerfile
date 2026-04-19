# This file is intended to be used apart from the containing source code tree.

FROM python:3-alpine AS builder

# Version of Radicale (e.g. v3)
ARG VERSION=v3.7.1

# Optional dependencies (e.g. bcrypt or ldap)
ARG DEPENDENCIES=bcrypt,ldap

RUN apk add --no-cache --virtual gcc libffi-dev musl-dev && python -m venv /app/venv

RUN /app/venv/bin/pip install --no-cache-dir "Radicale[${DEPENDENCIES}] @ https://github.com/Kozea/Radicale/archive/${VERSION}.tar.gz"
RUN /app/venv/bin/pip install --no-cache-dir "Radicale-SQL @ https://github.com/koalyorg/radicale-sql/archive/main.tar.gz"


FROM python:3-alpine

WORKDIR /app

RUN addgroup -g 1000 radicale \
    && adduser radicale --home /var/lib/radicale --system --uid 1000 --disabled-password -G radicale \
    && apk add --no-cache ca-certificates openssl curl git

COPY --chown=radicale:radicale --from=builder /app/venv /app

# Persistent storage for data
VOLUME /var/lib/radicale
# Run Radicale
ENTRYPOINT [ "/app/bin/python", "/app/bin/radicale"]
CMD ["--hosts", "0.0.0.0:5232,[::]:5232"]

USER radicale
