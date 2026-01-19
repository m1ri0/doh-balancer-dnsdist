# Official python image
FROM python:3.14.2-bookworm

# Comands to run on the container
RUN pip install dnspython httpx[http2] "fastapi[standard]"

COPY main.py /