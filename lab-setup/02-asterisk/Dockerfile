FROM ubuntu:24.04

RUN apt update && apt upgrade -y && apt install -y asterisk

COPY modules.conf /etc/asterisk/modules.conf
COPY pjsip.conf /etc/asterisk/pjsip.conf
COPY extensions.conf /etc/asterisk/extensions.conf

CMD ["asterisk", "-f"]
