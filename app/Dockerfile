# Official python image
FROM python:3.14.2-bookworm

WORKDIR /code

COPY . /code

# Comands to run on the container
RUN pip install -r /code/requirements.txt

CMD [ "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "8", "--limit-concurrency", "2000" ]
# CMD ["uvicorn", "main:app", "--workers", "8", "--worker-connections", "2000", "--loop", "uvloop", "--http", "h11", "--backlog", "2048", "--limit-concurrency", "1000", "--no-access-log"]
# CMD ["gunicorn", "main:app", "-w", "8", "-k", "uvicorn.workers.UvicornWorker", "--bind", "0.0.0.0:8000", "--worker-connections", "1000", "--max-requests", "1000", "--max-requests-jitter", "100", "--keep-alive", "5"]