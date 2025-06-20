FROM nginx

RUN apt update && apt install vim curl -y

WORKDIR /usr/share/nginx/html

RUN rm index.html

COPY index.html .

EXPOSE 80

CMD [ "nginx", "-g", "daemon off;" ]

