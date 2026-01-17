ARG ALPINE_VERSION="3.23"

FROM alpine:${ALPINE_VERSION}
ARG S6_OVERLAY_VERSION="3.2.0.3"

# Install spamassassin
RUN apk add --no-cache \
    spamassassin \
    spamassassin-client \
    gnupg \
    perl-net-dns \
    perl-dbd-pg \
    perl-mail-spf \
    curl \
    xz

# Install s6-overlay
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz /tmp
RUN tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-x86_64.tar.xz /tmp
RUN tar -C / -Jxpf /tmp/s6-overlay-x86_64.tar.xz

# Copy services
COPY services/ /etc/s6-overlay/s6-rc.d/

# Make the user bundle depend on spamd and cron
RUN mkdir -p /etc/s6-overlay/s6-rc.d/user/contents.d && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/spamd && \
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/cron

# Have sa-update run every day at 4AM
RUN echo "0 4 * * * sa-update && s6-svc -h /run/service/spamd" > /etc/crontabs/root

EXPOSE 783
ENTRYPOINT ["/init"]
