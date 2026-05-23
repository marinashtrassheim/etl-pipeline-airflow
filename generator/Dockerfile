# Event generator: writes JSON files to /data/events (shared volume in docker-compose).
FROM python:3.11-slim

ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Folders watched by Airflow FileSensor (raw / bad / processed)
RUN mkdir -p /data/events/raw /data/events/bad /data/events/processed \
    && chmod -R 777 /data/events

COPY generator.py .

CMD ["python", "generator.py"]
