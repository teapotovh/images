FROM debian:13-slim AS build

RUN apt-get update && apt-get install -y --no-install-recommends iptables

FROM gcr.io/distroless/static-debian13

COPY --from=build /usr/sbin/ip6tables /usr/sbin/ip6tables
COPY --from=build /usr/sbin/iptables /usr/sbin/iptables

COPY --from=build /lib/x86_64-linux-gnu/libxtables.so.12 /lib/x86_64-linux-gnu/
COPY --from=build /lib/x86_64-linux-gnu/libmnl.so.0 /lib/x86_64-linux-gnu/
COPY --from=build /lib/x86_64-linux-gnu/libnftnl.so.11 /lib/x86_64-linux-gnu/
COPY --from=build /lib/x86_64-linux-gnu/libc.so.6 /lib/x86_64-linux-gnu/
COPY --from=build /lib64/ld-linux-x86-64.so.2 /lib64/
COPY --from=build /usr/lib/x86_64-linux-gnu/xtables/ /usr/lib/x86_64-linux-gnu/xtables/

ENV PATH="/usr/sbin:/usr/bin:/sbin:/bin"
