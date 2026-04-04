ARG ALPINE_VERSION="3.23"

FROM alpine:${ALPINE_VERSION}
ARG S6_OVERLAY_VERSION="3.2.0.3"

# Install spamassassin
RUN apk add --no-cache \
    postfix \
    postfix-ldap

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
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/postfix

EXPOSE 25
EXPOSE 587
EXPOSE 465
ENTRYPOINT ["/init"]
