FROM python:3.5-slim AS builder
WORKDIR /app

COPY requirements.txt .
RUN pip install  --prefix=/install -r requirements.txt

FROM python:3.5-slim
RUN groupadd --gid 10001 appgroup \
    && useradd --uid 10001 --gid 10001 --create-home --shell /usr/sbin/nologin appuser
WORKDIR /app
COPY --from=builder /install /usr/local
COPY . .
EXPOSE 8000
USER appuser
