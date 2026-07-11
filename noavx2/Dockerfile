ARG BASE_IMAGE=ghcr.io/tphakala/birdnet-go:dev
FROM ${BASE_IMAGE}

ARG TFLITE_ASSET_URL=https://github.com/spawnegit/tflite-no-avx2/releases/download/V5.0/libtensorflowlite_c.so

USER root

RUN apt-get update -q && apt-get install -q -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*

RUN echo "Fetching no-AVX2 TFLite library from ${TFLITE_ASSET_URL}" \
    && rm -f /usr/lib/libtensorflowlite_c.so* \
    && curl -fsSL "${TFLITE_ASSET_URL}" -o /usr/lib/libtensorflowlite_c.so \
    && ldconfig \
    && echo "Library replaced:" \
    && ls -la /usr/lib/libtensorflowlite_c.so*
