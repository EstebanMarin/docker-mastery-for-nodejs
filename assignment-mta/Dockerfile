FROM node:8-jessie
# FROM node:8-jessie as prod
ENV NODE_ENV=production
WORKDIR /node
# Linux version does not uses Tini
# RUN apk add --no-cache tini
RUN apt-get update && apt-get install -y graphicsmagick
RUN mkdir app && chown -R node:node .
WORKDIR /node/app
USER node
RUN mkdir in \
  && mkdir out
COPY package*.json ./
# ensure you install just prod dependencies
RUN npm install --only=prod \
  && npm cache clean --force
COPY --chown=node:node . .
# directly to node as there is no tini
# ENTRYPOINT [ "/sbin/tini", "--"]
# You could run a npm run test here to ensure it passes all test before building the image
# RUN npm run test
# not that easey as I need to install dev dependencies
CMD [ "node", "./index.js" ]
