FROM node:24-alpine3.21

WORKDIR /usr/src/app

ENV NODE_ENV=production
ENV DOCKER_BUILDKIT=1

COPY package*.json ./

RUN --mount=type=cache,target=/usr/src/app.npm \
    npm set cache /usr/src/app/.npm && \
    npm ci --only=production

USER node

COPY --chown=node:node ./src .

EXPOSE 3000

CMD ["node", "index.js"]