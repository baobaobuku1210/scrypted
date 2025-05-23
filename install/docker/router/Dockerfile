ARG BASE="noble-lite"
FROM ghcr.io/koush/scrypted-common:${BASE}

# tools
RUN apt -y update && apt -y install nano net-tools dnsutils dnsmasq ifupdown vlan bridge-utils netplan.io nftables
RUN rm -f /etc/systemd/system/multi-user.target.wants/dnsmasq.service
RUN rm -f /etc/systemd/system/sysinit.target.wants/systemd-resolved.service

# go + caddy
RUN apt -y install golang-go
RUN apt install -y debian-keyring debian-archive-keyring apt-transport-https
RUN curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-xcaddy-archive-keyring.gpg
RUN curl -1sLf 'https://dl.cloudsmith.io/public/caddy/xcaddy/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-xcaddy.list
RUN apt -y update
RUN apt -y install xcaddy
RUN xcaddy build --with github.com/caddy-dns/cloudflare --output /usr/local/bin/caddy

# nftables
COPY scrypted-nftables.service /etc/systemd/system
RUN systemctl enable scrypted-nftables
RUN bash -c 'echo include \"/etc/nftables.d/*.conf\"\; >> /etc/nftables.conf'
RUN mkdir -p /etc/nftables.d
COPY 01-scrypted.conf /etc/nftables.d

WORKDIR /
# cache bust
ADD "https://www.random.org/cgi-bin/randbyte?nbytes=10&format=h" skipcache
ARG SCRYPTED_INSTALL_VERSION="latest"
RUN test -n "$SCRYPTED_INSTALL_VERSION"
RUN npx -y scrypted@latest install-server ${SCRYPTED_INSTALL_VERSION}

COPY scrypted.service /etc/systemd/system/scrypted.service
RUN ln -s /etc/systemd/system/scrypted.service /etc/systemd/system/multi-user.target.wants/scrypted.service

WORKDIR /
CMD ["/sbin/init"]