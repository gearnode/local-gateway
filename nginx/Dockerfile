FROM alpine:3.8

RUN apk add --no-cache nginx

COPY nginx.conf /etc/nginx/nginx.conf
# COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf

CMD ["nginx", "-g", "daemon off;"]
