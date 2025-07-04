FROM docker.io/python:3.12.9-slim-bookworm AS python

FROM python AS python-build-stage

ARG BUILD_ENVIRONMENT=production

RUN apt-get update && apt-get install --no-install-recommends -y \
  build-essential \
  libpq-dev


COPY ./requirements .

RUN pip wheel --wheel-dir /usr/src/app/wheels  \
  -r ${BUILD_ENVIRONMENT}.txt


FROM python AS python-run-stage

ARG BUILD_ENVIRONMENT=production
ARG APP_HOME=/app

ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV BUILD_ENV=${BUILD_ENVIRONMENT}

WORKDIR ${APP_HOME}

RUN addgroup --system django \
  && adduser --system --ingroup django django


RUN apt-get update && apt-get install --no-install-recommends -y \
  libpq-dev \
  gettext \
  wait-for-it \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/*


COPY --from=python-build-stage /usr/src/app/wheels  /wheels/

RUN pip install --no-cache-dir --no-index --find-links=/wheels/ /wheels/* \
  && rm -rf /wheels/


COPY --chown=django:django ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r$//g' /entrypoint
RUN chmod +x /entrypoint


COPY --chown=django:django ./compose/production/django/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start


COPY --chown=django:django . ${APP_HOME}

RUN chown -R django:django ${APP_HOME}

USER django

ENTRYPOINT ["/entrypoint"]
