FROM node:12.16.1-alpine3.10 as prod
ENV NODE_ENV=production
EXPOSE 3000
WORKDIR /node
RUN apk add --no-cache tini
RUN mkdir app && chown -R node:node .
WORKDIR /node/app
USER node
COPY package*.json ./
# ensure you install just prod dependencies
RUN npm install --only=prod \
  && npm cache clean --force
COPY --chown=node:node . .
ENTRYPOINT [ "/sbin/tini", "--"]
# You could run a npm run test here to ensure it passes all test before building the image
# RUN npm run test
# not that easey as I need to install dev dependencies
CMD [ "node", "./bin/www" ]

# Development
FROM prod as dev
ENV NODE_ENV=development
RUN npm install --only=development
CMD ["npx", "nodemon",  "./bin/www", "--inspect=0.0.0.0:9229"]


# Test
FROM dev as test
ENV NODE_ENV=development
CMD ["npm", "test"]
#  LocalWords:  www tini
