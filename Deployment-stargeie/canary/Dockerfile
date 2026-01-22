FROM nginx:alpine

WORKDIR /usr/share/nginx/html

# RUN rm -f index.html && echo '<h1>V1</h1>' > index.html
RUN rm -f index.html && echo '<h1>V2</h1>' > index.html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

# docker build --no-cache -t harishnshetty/rolling:v1 .
# docker run -d --name nginx-v1 -p 80:80 harishnshetty/rolling:v1

# docker build --no-cache -t harishnshetty/rolling:v2 .
# docker run -d --name nginx-v2 -p 80:80 harishnshetty/rolling:v2