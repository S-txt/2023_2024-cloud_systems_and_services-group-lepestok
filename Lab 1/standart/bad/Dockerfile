FROM ubuntu:latest
RUN apt-get update && apt-get install -y apache2
ARG port=80
EXPOSE $port
CMD ["apache2ctl", "-D", "FOREGROUND"]
