ARG BASE_IMAGE=ghcr.io/tphakala/birdnet-go:dev
FROM ${BASE_IMAGE}

ARG TFLITE_ASSET_URL=https://github.com/spawnegit/tflite-no-avx2/releases/download/V5.0/libtensorflowlite_c.so

USER root

RUN apt-get update -q && apt-get install -q -y --no-install-recommends curl gcc libc6-dev \
    && rm -rf /var/lib/apt/lists/*

RUN echo "Fetching no-AVX2 TFLite library from ${TFLITE_ASSET_URL}" \
    && rm -f /usr/lib/libtensorflowlite_c.so* \
    && curl -fsSL "${TFLITE_ASSET_URL}" -o /usr/lib/libtensorflowlite_c.so \
    && ldconfig \
    && ls -la /usr/lib/libtensorflowlite_c.so*

# Create stub for XNNPACK symbols that the birdnet-go binary expects
# but our no-XNNPACK TFLite library doesn't provide
RUN echo '#include <stddef.h>' > /tmp/xnnpack_stub.c \
    && echo 'typedef struct { int dummy; } TfLiteXNNPackDelegateOptions;' >> /tmp/xnnpack_stub.c \
    && echo 'TfLiteXNNPackDelegateOptions TfLiteXNNPackDelegateOptionsDefault() { TfLiteXNNPackDelegateOptions o = {0}; return o; }' >> /tmp/xnnpack_stub.c \
    && echo 'void* TfLiteXNNPackDelegateCreate(TfLiteXNNPackDelegateOptions* o) { return NULL; }' >> /tmp/xnnpack_stub.c \
    && echo 'void TfLiteXNNPackDelegateDelete(void* d) {}' >> /tmp/xnnpack_stub.c \
    && gcc -shared -o /usr/lib/libxnnpack_stub.so /tmp/xnnpack_stub.c \
    && echo "/usr/lib/libxnnpack_stub.so" >> /etc/ld.so.preload \
    && ldconfig \
    && rm /tmp/xnnpack_stub.c
