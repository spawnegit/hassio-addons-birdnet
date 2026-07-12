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
# Wrap ffmpeg to strip -allowed_media_types audio, which some RTSP camera
# servers (e.g. Tapo C510W) fail to negotiate correctly, causing a
# connected-but-zero-bytes stream. birdnet-go hardcodes this flag when
# launching ffmpeg for audio-only RTSP capture, so we intercept and drop
# it transparently before invoking the real binary.
RUN mv /usr/bin/ffmpeg /usr/bin/ffmpeg-real \
    && printf '#!/bin/sh\nargs=""\nskip_next=0\nfor arg in "$@"; do\n    if [ "$skip_next" = "1" ]; then\n        skip_next=0\n        continue\n    fi\n    if [ "$arg" = "-allowed_media_types" ]; then\n        skip_next=1\n        continue\n    fi\n    args="$args \\"$arg\\""\ndone\neval exec /usr/bin/ffmpeg-real $args\n' > /usr/bin/ffmpeg \
    && chmod +x /usr/bin/ffmpeg
