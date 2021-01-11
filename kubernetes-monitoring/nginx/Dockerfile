FROM nginx:latest

ARG USER=app
ARG UID=1001
ARG GID=1001

COPY nginx.conf /etc/nginx/nginx.conf
#добавление пользователя, группы и создание папки app
RUN useradd ${USER} \
    && usermod -u $UID ${USER} \
    && groupmod -g $GID ${USER} \
    && mkdir -p /app \
    && chown -R ${USER}:${USER} /app

COPY homework.html /app

WORKDIR /app

EXPOSE 8000