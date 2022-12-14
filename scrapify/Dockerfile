### base image ###
ARG PYTHON_VERSION=3.11.0

FROM python:$PYTHON_VERSION AS base

#ARG ENV
ARG POETRY_HOME="/opt/poetry"
ARG POETRY_VERSION=1.3.1
ARG PYSETUP_PATH="/opt/pysetup"

#ENV ENV=$ENV
ENV PYTHONUNBUFFERED=1
ENV PYTHONFAULTHANDLER=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONHASHSEED=random
#ENV PIP_DISABLE_PIP_VERSION_CHECK=1
#ENV PIP_NO_CACHE_DIR=1
#ENV PIP_DEFAULT_TIMEOUT=100
ENV POETRY_HOME=$POETRY_HOME
ENV POETRY_VERSION=$POETRY_VERSION
ENV POETRY_NO_ANSI=1
ENV POETRY_NO_INTERACTION=1
ENV POETRY_VIRTUALENVS_IN_PROJECT=true
ENV PYSETUP_PATH=$PYSETUP_PATH
ENV VENV_PATH="$PYSETUP_PATH/.venv"

# prepend poetry and venv to path
ENV PATH="$POETRY_HOME/bin:$VENV_PATH/bin:$PATH"

### builder image ###
FROM base AS builder

ARG ENV

ENV ENV=$ENV

# install required packages
#RUN apt-get update
#RUN apt-get install --no-install-recommends -y curl build-essential

# install poetry - respects $POETRY_VERSION & $POETRY_HOME
RUN curl -sSL https://install.python-poetry.org | python3 -

# copy project requirement files here to ensure they will be cached.
WORKDIR $PYSETUP_PATH
COPY pyproject.toml ./

# install runtime deps - uses $POETRY_VIRTUALENVS_IN_PROJECT internally
RUN poetry install $(test $ENV != prod && echo "--with dev,tests,docs -vvv") --sync


### local image ###
FROM base AS local

ARG ENV

ENV ENV=$ENV

COPY --from=builder $PYSETUP_PATH $PYSETUP_PATH
COPY /src /src

CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "80"]
#CMD [ "python3", "./src/main.py" ]

### dev image ###
FROM base AS dev

ARG ENV

ENV ENV=$ENV

COPY --from=builder $PYSETUP_PATH $PYSETUP_PATH
COPY /src /src

CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "80"]
