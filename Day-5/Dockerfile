FROM ubuntu:latest
LABEL name:"raviiran"
RUN apt update && apt install -y jq net-tools wget curl unzip\
    && apt install -y nginx iputils-ping
CMD ["nginx","-g","daemon off;"]
