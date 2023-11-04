FROM ubuntu:20.04
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends apache2 \
    && rm -rf /var/lib/apt/lists/*
EXPOSE $port
ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]