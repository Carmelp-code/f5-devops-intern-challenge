# Proxy image inherited from the previous team.

FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf

RUN mkdir -p /var/cache/nginx && \
    chown -R nginx:nginx /var/cache/nginx

USER nginx 

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
