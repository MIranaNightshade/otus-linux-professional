FROM alpine:3.24
RUN apk add --no-cache nginx
COPY index.html cat.png /usr/share/nginx/html/
COPY default.conf /etc/nginx/http.d/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
