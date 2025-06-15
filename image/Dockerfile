# Use osixia/light-baseimage
# sources: https://github.com/osixia/docker-light-baseimage
FROM osixia/light-baseimage:alpine-0.1.6-dev

# Keepalived version
#ARG KEEPALIVED_VERSION=2.0.20
ARG KEEPALIVED_VERSION=2.3.3
# keepalived-exporter version
ARG EXPORTER_VERSION="1.7.0"

# automatic args from buildx
ARG TARGETPLATFORM
ARG BUILDPLATFORM
ARG TARGETARCH
ARG BUILDARCH
ARG TARGETOS
ARG BUILDOS

RUN echo ">>> i am running on $BUILDPLATFORM [os: $BUILDOS, arch: $BUILDARCH], building for $TARGETPLATFORM [os: $BUILDOS, arch: $BUILDARCH]"


# Download, build and install Keepalived
RUN apk --no-cache add \
    autoconf \
    curl \
    gcc \
    ipset \
    ipset-dev \
    iptables \
    iptables-dev \
    libnfnetlink \
    libnfnetlink-dev \
    libnl3 \
    libnl3-dev \
    make \
    musl-dev \
    openssl \
    openssl-dev \
    && curl -o keepalived.tar.gz -SL http://keepalived.org/software/keepalived-${KEEPALIVED_VERSION}.tar.gz \
    && mkdir -p /container/keepalived-sources \
    && tar -xzf keepalived.tar.gz --strip 1 -C /container/keepalived-sources \
    && cd /container/keepalived-sources \
    && ./configure --disable-dynamic-linking --enable-json \
    && make && make install \
    && cd - && mkdir -p /etc/keepalived \
    && rm -f keepalived.tar.gz \
    && rm -rf /container/keepalived-sources \
    && apk --no-cache del \
    autoconf \
    gcc \
    ipset-dev \
    iptables-dev \
    libnfnetlink-dev \
    libnl3-dev \
    make \
    musl-dev \
    openssl-dev

RUN apk --no-cache add wget ca-certificates && \
    case ${TARGETPLATFORM} in \
        "linux/amd64") EXPORTER_ARCH="amd64" ;; \
        "linux/arm64") EXPORTER_ARCH="arm64" ;; \
        "linux/arm/v7") EXPORTER_ARCH="armv7" ;; \
        *) echo "Unsupported TARGETPLATFORM: ${TARGETPLATFORM}"; EXPORTER_ARCH="UNKNOWN"; exit 1 ;; \
    esac && \
    echo "Downloading keepalived-exporter version ${EXPORTER_VERSION} for architecture ${EXPORTER_ARCH}..." && \
    wget "https://github.com/mehdy/keepalived-exporter/releases/download/v${EXPORTER_VERSION}/keepalived-exporter_${EXPORTER_VERSION}_linux_${EXPORTER_ARCH}.tar.gz" -O /tmp/exporter.tar.gz && \
    tar -xzvf /tmp/exporter.tar.gz -C /usr/local/bin/ && \
    rm /tmp/exporter.tar.gz && \
    chmod +x /usr/local/bin/keepalived-exporter && \
    apk del wget ca-certificates

# Add service directory to /container/service
ADD service /container/service

# Use baseimage install-service script
# https://github.com/osixia/docker-light-baseimage/blob/stable/image/tool/install-service
RUN /container/tool/install-service

# Add default env variables
ADD environment /container/environment/99-default

# Expose keepalived-exporter port
EXPOSE 9650

# Startup script (adjust as needed, especially the keepalived-exporter config path)
###COPY start.sh /usr/local/bin/start.sh
###RUN chmod +x /usr/local/bin/start.sh

###ENTRYPOINT ["/usr/local/bin/start.sh"]
