FROM docker.n8n.io/n8nio/n8n

USER root

RUN apk add --no-cache \
    ffmpeg \
    python3 \
    py3-pip \
    curl \
    && pip3 install --break-system-packages yt-dlp

USER node
