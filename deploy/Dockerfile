#  ======================== Stage 1 ========================
FROM node:24-alpine3.21 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

# TODO: Add a base_path feature
ENV BASE_PATH=/ 

RUN npm run build

# ======================== Stage 2 ========================
FROM nginx:1.29.1-alpine

ARG USERNAME=appuser
ARG USER_UID=4000
ARG USER_GID=$USER_UID

# Create the user
RUN addgroup -g $USER_GID $USERNAME \
    && adduser -u $USER_UID -G $USERNAME -h /home/$USERNAME -D $USERNAME

COPY --from=builder /app/build /user/share/nginx/html
COPY docker/deploy/nginx.conf /etc/nginx/conf.d/default.conf

RUN chown -R $USERNAME:$USERNAME /var/cache/nginx && \
    chown -R $USERNAME:$USERNAME /var/log/nginx && \
    chown -R $USERNAME:$USERNAME /etc/nginx/conf.d
RUN touch /var/run/nginx.pid && \
    chown -R $USERNAME:$USERNAME /var/run/nginx.pid

USER $USERNAME

ENTRYPOINT ["nginx", "-g", "daemon off;"]