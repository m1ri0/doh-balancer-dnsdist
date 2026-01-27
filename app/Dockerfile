# Official python image
FROM python:3.14.2-bookworm

WORKDIR /code

COPY . /code

# Comands to run on the container
RUN pip install -r /code/requirements.txt

CMD [ "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "8", "--limit-concurrency", "2000" ]