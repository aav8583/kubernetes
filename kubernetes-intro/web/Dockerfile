FROM nginx

ARG UID_GID=1001
ARG PORT=8000
ENV UID=${UID_GID}
ENV GID=${UID_GID}
ENV PORT=${PORT}

COPY wrapper.sh /
COPY html /app
COPY default.conf /etc/nginx/conf.d
COPY nginx.conf /etc/nginx

#add group grnode add user usnode with UID 1001
RUN groupadd --gid $UID grnode \
  && useradd --uid $UID --gid grnode --shell /bin/bash --create-home usnode

## add permissions
RUN chown -R usnode:grnode /app && chmod -R 755 /app && \
        chown -R usnode:grnode /var/cache/nginx && \
        chown -R usnode:grnode /var/log/nginx && \
        chown -R usnode:grnode /etc/nginx/conf.d
RUN touch /var/run/nginx.pid && \
        chown -R usnode:grnode /var/run/nginx.pid

USER $UID
EXPOSE $PORT

CMD ["nginx","-g","daemon off;"]
#CMD ["./wrapper.sh"]