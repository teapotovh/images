ARG ALPINE_VERSION="3.23"

FROM alpine:${ALPINE_VERSION} as builder

# TODO: we should not need to do this. We should directly use what's in ARG
# but it won't work inside the ADD rule.
ENV ALPINE_VERSION="3.23"
ENV CYRUS_SASL_VERSION=2.1.28

RUN apk add --no-cache \
    autoconf \
    automake \
    curl \
    db-dev \
    g++ \
    gcc \
    gzip \
    heimdal-dev \
    libtool \
    make \
    openldap-dev \
    openssl-dev \
    tar \
    cyrus-sasl \
    libldap \
    patch

ADD https://github.com/cyrusimap/cyrus-sasl/releases/download/cyrus-sasl-${CYRUS_SASL_VERSION}/cyrus-sasl-${CYRUS_SASL_VERSION}.tar.gz /tmp

WORKDIR /tmp/cyrus-sasl
RUN tar -xzf /tmp/cyrus-sasl-${CYRUS_SASL_VERSION}.tar.gz --strip=1 -C /tmp/cyrus-sasl

# NOTE: These URLs are blocked from github runners (they answer with code 418)
# ADD https://gitlab.alpinelinux.org/alpine/aports/-/raw/${ALPINE_VERSION}-stable/main/cyrus-sasl/gcc15.patch /tmp/cyrus-sasl
# ADD https://gitlab.alpinelinux.org/alpine/aports/-/raw/${ALPINE_VERSION}-stable/main/cyrus-sasl/sasl2-m4.patch /tmp/cyrus-sasl
# ADD https://gitlab.alpinelinux.org/alpine/aports/-/raw/${ALPINE_VERSION}-stable/main/cyrus-sasl/time-h-gcc14.patch /tmp/cyrus-sasl

ADD gcc15.patch /tmp/cyrus-sasl
ADD sasl2-m4.patch /tmp/cyrus-sasl
ADD time-h-gcc14.patch /tmp/cyrus-sasl

RUN patch -p1 < gcc15.patch && \
    patch -p1 < sasl2-m4.patch && \
    patch -p1 < time-h-gcc14.patch

RUN aclocal --install && autoreconf -vfi -Wno-portability

RUN mkdir /dest
RUN ./configure \
        --prefix=/usr \
        --sysconfdir=/etc/sasl2 \
        --localstatedir=/var \
        --disable-anon \
        --enable-cram \
        --enable-digest \
        --enable-ldapdb \
        --enable-login \
        --enable-ntlm \
        --disable-otp \
        --enable-plain \
        --with-gss_impl=heimdal \
        --with-devrandom=/dev/urandom \
        --with-ldap=/usr \
        --with-saslauthd=/var/run/saslauthd \
        --mandir=/usr/share/man
RUN make && make DESTDIR="/dest" install

FROM alpine:${ALPINE_VERSION}
ARG S6_OVERLAY_VERSION="3.2.0.3"

# Install spamassassin
RUN apk add --no-cache \
    postfix \
    postfix-ldap \
    lmdb \
    heimdal

COPY --from=builder /dest/usr /usr

# Install s6-overlay
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz /tmp
RUN tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-x86_64.tar.xz /tmp
RUN tar -C / -Jxpf /tmp/s6-overlay-x86_64.tar.xz

# Copy services
COPY services/ /etc/s6-overlay/s6-rc.d/

# Make the user bundle depend on spamd and cron
RUN mkdir -p /etc/s6-overlay/s6-rc.d/user/contents.d && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/syslogd && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/saslauthd && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/postfix

EXPOSE 25
EXPOSE 587
EXPOSE 465
VOLUME /var/run/saslauthd
ENTRYPOINT ["/init"]
