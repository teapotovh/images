ARG ALPINE_VERSION="3.23"

FROM alpine:${ALPINE_VERSION} AS builder

RUN apk add --no-cache \
    build-base \
    autoconf \
    automake \
    git \
    libmilter-dev \
    spamassassin \
    spamassassin-client

WORKDIR /build

RUN mkdir /dest
RUN git clone https://github.com/andybalholm/spamass-milter.git . && \
    ./autogen.sh && \
    ./configure --prefix=/dest && \
    make && \
    make install


FROM alpine:${ALPINE_VERSION}
ARG S6_OVERLAY_VERSION="3.2.0.3"

# Install spamassassin
RUN apk add --no-cache \
    libmilter \
    spamassassin \
    spamassassin-client \
    gnupg \
    perl-net-dns \
    perl-dbd-pg \
    perl-mail-spf \
    curl \
    xz \
    libstdc++ \
    libgcc

COPY --from=builder /dest /usr

RUN sed -i '/Plugin::\(AWL\|AntiVirus\)/s/^#//' /etc/mail/spamassassin/v310.pre

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
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/spamd && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/spamass-milter && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/cron

# Have sa-update run every day at 4AM
RUN echo "0 4 * * * sa-update && s6-svc -h /run/service/spamd" > /etc/crontabs/root

EXPOSE 783
EXPOSE 10017
ENTRYPOINT ["/init"]
