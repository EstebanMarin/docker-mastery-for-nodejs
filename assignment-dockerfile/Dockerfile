# fill in the blanks to dockerize this node app
FROM node:12.16.1-alpine3.10

EXPOSE 3000

WORKDIR /node

RUN apk add --no-cache tini

RUN mkdir app && chown -R node:node .

WORKDIR /node/app

USER node

COPY package*.json ./

RUN npm install && npm cache clean --force

COPY --chown=node:node . .

ENTRYPOINT [ "/sbin/tini", "--"]

CMD [ "node", "app.js" ]
