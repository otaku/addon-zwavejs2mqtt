ARG BUILD_FROM=ghcr.io/hassio-addons/base/amd64:9.1.3
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG ZWAVEJS_VERSION=7cb3648b2275ce2135dc437978b3b6544658fde3
ARG ZWAVEJS2MQTT_VERSION=v2.4.1
ARG YARN_NETWORK_TIMEOUT=30000

# Setup base
# hadolint ignore=DL3003
RUN \
    apk add --no-cache --virtual .build-dependencies \
        build-base=0.5-r2 \
        linux-headers=5.7.8-r0 \
        npm=14.16.0-r0 \
        python3-dev=3.8.8-r0 \
        yarn=1.22.10-r0 \
        jq=1.6-r1 \
    \
    && apk add --no-cache \
        eudev=3.2.9-r3 \
        libusb=1.0.24-r1 \
        nginx=1.18.0-r13 \
        nodejs=14.16.0-r0 \
    \
    && curl -J -L -o /tmp/zwavejs.tar.gz \
        "https://github.com/zwave-js/node-zwave-js/archive/${ZWAVEJS_VERSION}.tar.gz" \
    && mkdir -p /work/zwavejs \
    && tar zxvf \
        /tmp/zwavejs.tar.gz \
        --strip 1 -C /work/zwavejs

    RUN cd /work/zwavejs \
    && yarn install --network-timeout=${YARN_NETWORK_TIMEOUT} \
    && yarn run build:full \
    && yarn install --production --frozen-lockfile \
    && for i in config core serial shared; do \
         cd packages/$i \
         && yarn version --no-git-tag-version --new-version $(yarn versions --json | jq -r '[.data."@zwave-js/'${i}'"]'[0])-${ZWAVEJS_VERSION:0:7} \
         && yarn link \
         && cd ../..; \
       done \
    && cd packages/shared && yarn add fs-extra && cd ../.. \    
    && cd packages/zwave-js \
    && echo "packages/zwave-js" \
    && echo yarn version --no-git-tag-version --new-version $(yarn versions --json | jq -r '[.data."zwave-js"]'[0])-${ZWAVEJS_VERSION:0:7} \
    && yarn version --no-git-tag-version --new-version $(yarn versions --json | jq -r '[.data."zwave-js"]'[0])-${ZWAVEJS_VERSION:0:7} \
    && yarn link \
    \
    && curl -J -L -o /tmp/zwavejs2mqtt.tar.gz \
        "https://github.com/zwave-js/zwavejs2mqtt/archive/${ZWAVEJS2MQTT_VERSION}.tar.gz" \
    && tar zxvf \
        /tmp/zwavejs2mqtt.tar.gz \
        --strip 1 -C /opt \
    \
    && cd /opt \
    && yarn install --network-timeout=${YARN_NETWORK_TIMEOUT} \
    && yarn run build \
    && yarn install --production --frozen-lockfile \
    && yarn version -s --no-git-tag-version --new-version $(yarn versions --json | jq -r .data.zwavejs2mqtt)+${ZWAVEJS2MQTT_VERSION:0:7} \
    && yarn link zwave-js @zwave-js/core @zwave-js/config @zwave-js/serial @zwave-js/shared \
    \
    && apk del --no-cache --purge .build-dependencies \
    \
    && rm -f -r \
        /etc/nginx \
        /opt/build \
        /opt/index.html \
        /opt/package-lock.json \
        /opt/package.sh \
        /opt/src \
        /opt/static \
        /opt/stylesheets \
        /opt/.[!.]* \
        /opt/*.md \
        /opt/nodemon.json \
        /opt/kustomization.yaml \
        /opt/kubernetes \
        /opt/docker \
        /opt/docs \
        /root/.cache \
        /root/.config \
        /root/.npmrc \
        /root/.node-gyp \
        /root/.npm \
        /tmp/.[!.]* \
        /tmp/* \
        /usr/lib/node_modules \
        /usr/local/share/.cache

# Copy root filesystem
COPY rootfs /

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
