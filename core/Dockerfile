FROM ubuntu:18.04

RUN apt-get update && apt-get install inotify-tools -y && apt-get install openssl -y

WORKDIR /tagion/

COPY . .

ENV PATH="/tagion/utils/dart:/tagion/utils/hibon:/tagion/utils/tagion:${PATH}"

WORKDIR /workspace