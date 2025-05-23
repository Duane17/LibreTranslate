FROM python:3.11.11-slim-bullseye AS builder

WORKDIR /app

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update -qq \
  && apt-get -qqq install --no-install-recommends -y pkg-config gcc g++ \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN python -mvenv venv && ./venv/bin/pip install --no-cache-dir --upgrade pip

COPY . .

RUN ./venv/bin/pip install Babel==2.12.1 \
  && ./venv/bin/python scripts/compile_locales.py \
  && ./venv/bin/pip install torch==2.2.0 --extra-index-url https://download.pytorch.org/whl/cpu \
  && ./venv/bin/pip install "numpy<2" \
  && ./venv/bin/pip install . \
  && ./venv/bin/pip cache purge

FROM python:3.11.11-slim-bullseye

RUN addgroup --system --gid 1032 libretranslate && adduser --system --uid 1032 libretranslate \
  && mkdir -p /home/libretranslate/.local && chown -R libretranslate:libretranslate /home/libretranslate/.local

USER libretranslate

COPY --from=builder --chown=1032:1032 /app /app
COPY --from=builder --chown=1032:1032 /app/venv/bin/ltmanage /usr/bin/

WORKDIR /app

RUN ./venv/bin/python scripts/install_models.py --load_only_lang_codes en,fr

EXPOSE 10000
ENTRYPOINT ["./venv/bin/libretranslate", "--host", "0.0.0.0", "--port", "10000"]
