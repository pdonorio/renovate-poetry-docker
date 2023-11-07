# syntax=docker/dockerfile:1.4

FROM python:3.10-bullseye as base

ENV PLAYWRIGHT_BROWSERS_PATH=/browsers

WORKDIR /code

FROM base as venv

ENV POETRY_VERSION=1.7.0

RUN pip3 install poetry==${POETRY_VERSION}

ENV POETRY_HOME=/usr/local
ENV POETRY_VIRTUALENVS_IN_PROJECT=true

COPY --link pyproject.toml ./
# COPY --link poetry.lock pyproject.toml ./

RUN --mount=type=cache,target=/code/.venv \
  poetry install --only main --sync \
  && cp -ar /code/.venv /code/.venv-copy

FROM base as browsers

COPY --link --from=venv /code/.venv-copy/ /code/.venv/

# TODO: install only a browser specified
# e.g. using ARG from Jenkins parameter
# by using .venv/bin/playwright install <browser>
RUN --mount=type=cache,target=$PLAYWRIGHT_BROWSERS_PATH \
  .venv/bin/playwright install \
  && cp -ar $PLAYWRIGHT_BROWSERS_PATH ${PLAYWRIGHT_BROWSERS_PATH}-copy

FROM base as with-deps

RUN apt-get update && apt-get install -y \
  libnss3 \
  libnspr4 \
  libatk1.0-0 \
  libatk-bridge2.0-0 \
  libcups2 \
  libdrm2 \
  libdbus-1-3 \
  libxkbcommon0 \
  libxcomposite1 \
  libxdamage1 \
  libxfixes3 \
  libxrandr2 \
  libgbm1 \
  libasound2 \
  libatspi2.0-0 \
  libwayland-client0 \
  && rm -rf /var/lib/apt/lists/*

FROM with-deps as runtime

COPY --link --from=venv /code/.venv-copy/ /code/.venv/
COPY --link --from=browsers ${PLAYWRIGHT_BROWSERS_PATH}-copy/ $PLAYWRIGHT_BROWSERS_PATH/

USER root
RUN chmod -R 777 $PLAYWRIGHT_BROWSERS_PATH \
  && chmod -R 777 /code \
  && useradd -ms /bin/bash my-user \
  && echo ". ~/.bashrc\n. /my/.venv/bin/activate" >> /home/my-user/.venvrc

USER my-user
FROM runtime as runner

USER root
RUN .venv/bin/playwright install-deps
USER my-user
# CMD /usr/local/bin/run_tests.sh
