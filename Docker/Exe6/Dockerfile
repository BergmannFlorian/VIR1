FROM ubuntu:latest
COPY ./heartbeat.sh /
WORKDIR /
ENV HEARTBEATSTEP=2
RUN chmod +x ./heartbeat.sh
CMD ./heartbeat.sh HELLO