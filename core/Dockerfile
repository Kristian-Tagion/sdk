FROM ubuntu:18.04

RUN apt-get update && apt-get install inotify-tools -y && apt-get install openssl -y

WORKDIR /tagion/

COPY . .

ENV GODEBUG=cgocheck=0
ENV PATH="/tagion/devnet:/tagion/wallet:/tagion/utils/dart:/tagion/utils/hibon:/tagion/utils/tagion:${PATH}"

WORKDIR /workspace/