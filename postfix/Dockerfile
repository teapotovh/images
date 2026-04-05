ARG DEBIAN_VERSION="13"

FROM debian:${DEBIAN_VERSION}-slim
ARG S6_OVERLAY_VERSION="3.2.0.3"

# Install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    busybox-syslogd \
    postfix \
    postfix-ldap \
    sasl2-bin \
    libsasl2-modules-ldap \
    xz-utils \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN usermod -aG sasl postfix
RUN mkdir -p /var/run/saslauthd && chown root:sasl /var/run/saslauthd && chmod 710 /var/run/saslauthd

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

EXPOSE 25 587 465
VOLUME /var/run/saslauthd
ENTRYPOINT ["/init"]
