FROM python:3.13-slim-trixie@sha256:cf50a4f6ac4f026458b08d5222023d700ac806410dff5e9ca8728fa44f2b850f AS builder

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    VIRTUAL_ENV=/opt/venv

RUN apt-get update && apt-get install -y --no-install-recommends g++ git

COPY --from=ghcr.io/astral-sh/uv:0.8.15@sha256:a5727064a0de127bdb7c9d3c1383f3a9ac307d9f2d8a391edc7896c54289ced0 /uv /uvx /bin/
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --extra cpu --no-dev --no-editable --no-install-project --compile-bytecode --no-progress --active --link-mode copy

FROM python:3.13-slim-trixie@sha256:cf50a4f6ac4f026458b08d5222023d700ac806410dff5e9ca8728fa44f2b850f AS mimalloc
RUN echo "deb http://deb.debian.org/debian trixie-backports main" > /etc/apt/sources.list.d/backports.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends libmimalloc3/trixie-backports && \
    cp /usr/lib/*-linux-gnu/libmimalloc.so.3 /libmimalloc.so

FROM python:3.13-slim-trixie@sha256:cf50a4f6ac4f026458b08d5222023d700ac806410dff5e9ca8728fa44f2b850f AS prod

ENV MACHINE_LEARNING_MODEL_ARENA=false

COPY --from=mimalloc /libmimalloc.so /usr/lib/libmimalloc.so
RUN apt-get update && \
    apt-get install -y --no-install-recommends tini ccache libgomp1 && \
    apt-get autoremove -yqq && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    echo /usr/lib/libmimalloc.so > /etc/ld.so.preload

WORKDIR /usr/src
ENV TRANSFORMERS_CACHE=/cache \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/opt/venv/bin:$PATH" \
    PYTHONPATH=/usr/src \
    VIRTUAL_ENV=/opt/venv \
    MACHINE_LEARNING_CACHE_FOLDER=/cache

# prevent core dumps
RUN echo "hard core 0" >> /etc/security/limits.conf && \
    echo "fs.suid_dumpable 0" >> /etc/sysctl.conf && \
    echo 'ulimit -S -c 0 > /dev/null 2>&1' >> /etc/profile

COPY --from=builder /opt/venv /opt/venv
COPY scripts/healthcheck.py .
COPY immich_ml immich_ml

ARG BUILD_ID
ARG BUILD_IMAGE
ARG BUILD_SOURCE_REF
ARG BUILD_SOURCE_COMMIT

ENV IMMICH_BUILD=${BUILD_ID}
ENV IMMICH_BUILD_URL=https://github.com/arsgab/immich/actions/runs/${BUILD_ID}
ENV IMMICH_BUILD_IMAGE=${BUILD_IMAGE}
ENV IMMICH_BUILD_IMAGE_URL=https://github.com/arsgab/immich/pkgs/container/immich-machine-learning
ENV IMMICH_REPOSITORY=arsgab/immich
ENV IMMICH_REPOSITORY_URL=https://github.com/arsgab/immich
ENV IMMICH_SOURCE_REF=${BUILD_SOURCE_REF}
ENV IMMICH_SOURCE_COMMIT=${BUILD_SOURCE_COMMIT}
ENV IMMICH_SOURCE_URL=https://github.com/arsgab/immich/commit/${BUILD_SOURCE_COMMIT}

# store huggingface in /cache subfolder to enable rootless deployment with a single volume
ENV HF_HOME=/cache/hf-cache

ENTRYPOINT ["tini", "--"]
CMD ["python", "-m", "immich_ml"]

HEALTHCHECK CMD python3 healthcheck.py
