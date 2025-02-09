FROM ubuntu:latest

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update
RUN apt-get install -y curl dnsutils siege

RUN apt-get install -y build-essential libssl-dev git unzip && \
    git clone https://github.com/wg/wrk.git && \
    cd wrk && make && cp wrk /usr/local/bin && \
    cd .. && rm -rf wrk

RUN rm -rf /var/lib/apt/lists/*

COPY urls.txt /data/urls.txt

CMD ["sleep", "infinity"]
