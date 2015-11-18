FROM ubuntu

RUN apt-get install -y nginx

RUN sed -i 's/Welcome to nginx!/Welcome to nginx! running on docker from an image!/g' /usr/share/nginx/html/index.html

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80

ENTRYPOINT  nginx -g 'daemon off;'
