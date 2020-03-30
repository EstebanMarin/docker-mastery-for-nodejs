## Stage 1 (production base)
# This gets our prod dependencies installed and out of the way
FROM node:8-slim

EXPOSE 80

WORKDIR /node

# NOTE: these apt dependencies are only needed
# for testing. they shouldn't be in production
RUN apt-get update -qq && apt-get install -qy \ 
  ca-certificates \
  bzip2 \
  curl \
  libfontconfig \
  --no-install-recommends

COPY package*.json ./

RUN npm install \
    && npm cache clean --force

WORKDIR /node/app

ENV PATH /node/app/node_modules/.bin:$PATH

COPY . .

CMD ["node", "server.js"]

