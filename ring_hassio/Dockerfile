ARG BUILD_FROM
FROM $BUILD_FROM

ENV LANG C.UTF-8
# s6 aktiv lassen (Standard); NICHT S6_DISABLE_INIT setzen

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
EXPOSE 3000/tcp

RUN apk add --no-cache \
    nodejs \
    npm \
    ffmpeg \
    git && \
    cd / && \
    git clone https://github.com/jeroenterheerdt/ring-hassio && \
    cd ring-hassio/ring_hassio && \
    npm install --unsafe-perm && \
    chmod a+x run.sh && \
    \
    # ----- s6 v3 Service einrichten -----
    mkdir -p /etc/s6-overlay/s6-rc.d/ring \
             /etc/s6-overlay/s6-rc.d/ring/dependencies.d \
             /etc/s6-overlay/s6-rc.d/user/contents.d && \
    printf '%s\n' longrun > /etc/s6-overlay/s6-rc.d/ring/type && \
    # run-Skript: mit Umgebungsvariablen aus /data/options.json starten
    printf '%s\n' '#!/usr/bin/with-contenv bash' \
                  'set -euo pipefail' \
                  'CONFIG_PATH=/data/options.json' \
                  'export RING_REFRESH_TOKEN="$(jq --raw-output ''.ring_refresh_token'' "$CONFIG_PATH")"' \
                  'export CAMERA_NAME="$(jq --raw-output ''.camera_name'' "$CONFIG_PATH")"' \
                  'export RING_PORT="$(jq --raw-output ''.port'' "$CONFIG_PATH")"' \
                  'cd /ring-hassio/ring_hassio' \
                  'exec node livestream.js' \
      > /etc/s6-overlay/s6-rc.d/ring/run && \
    chmod +x /etc/s6-overlay/s6-rc.d/ring/run && \
    # Service in die user-Bootkette aufnehmen
    touch /etc/s6-overlay/s6-rc.d/user/contents.d/ring
