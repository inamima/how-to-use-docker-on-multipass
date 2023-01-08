FROM ubuntu:22.04

# Install dependencies
RUN apt update && \
 apt -y install nginx

COPY ./index.html /var/www/html

EXPOSE 80

ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
