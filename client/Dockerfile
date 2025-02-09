FROM ubuntu:latest

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y curl dnsutils siege && \
    rm -rf /var/lib/apt/lists/*

COPY urls.txt /data/urls.txt

CMD ["sleep", "infinity"]
