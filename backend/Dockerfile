FROM python:3.5-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt

FROM python:3.5-slim
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1
RUN groupadd --gid 10001 appgroup \
    && useradd --uid 10001 --gid 10001 --create-home --shell /usr/sbin/nologin appuser
WORKDIR /app
COPY --from=builder /install /usr/local
COPY . .
RUN chmod +x /app/entrypoint.sh
EXPOSE 8000
USER appuser
ENTRYPOINT ["/app/entrypoint.sh"]
CMD [ "gunicorn", "conduit.wsgi:application", "--bind", "0.0.0.0:8000" ]
