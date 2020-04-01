## Stage 1 (production base)
# This gets our prod dependencies installed and out of the way
FROM node:8-slim as base

LABEL org.opencontainers.image.authors=estebanmarin002@gmail.com
LABEL org.opencontainers.image.title="Node.js Ultimate Dockerfile"
LABEL org.opencontainers.image.licenses=MIT
LABEL com.estebanmarin.nodeversion=$NODE_VERSION
# set our node environment, either development or production
# defaults to production, compose overrides this to development on build and run
ARG NODE_ENV=production
ENV NODE_ENV $NODE_ENV

# default to port 80 for node, and 9229 and 9230 (tests) for debug
ARG PORT=8080
ENV PORT $PORT
EXPOSE $PORT

# you'll likely want the latest npm, regardless of node version, for speed and fixes
# but pin this version for the best stability
RUN npm i npm@latest -g

#non-root user and setting node user
RUN mkdir /opt/node && chown node:node /opt/node
WORKDIR /opt/node

COPY package*.json ./

# we use npm ci here so only the package-lock.json file is used
RUN npm config list \
  && npm ci --only=prod\
  && npm cache clean --force


ENV PATH /opt/node/node_modules/.bin:$PATH
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]
CMD ["node", "server.js"]

## Stage 2 (development)
# we don't COPY in this stage because for dev you'll bind-mount anyway
# this saves time when building locally for dev via docker-compose

FROM base as dev

ENV NODE_ENV=development
# ensure the path? need to double check
ENV PATH=/opt/node/node_modules/.bin:$PATH

EXPOSE $PORT 9229 9230

WORKDIR /opt/node

RUN npm install --only=development

WORKDIR /opt/node/app

CMD ["nodemon", "./server.js", "--inspect=0.0.0.0:9229"]

## Stage 3 (copy in source)
# This gets our source code into builder for use in next two stages
# It gets its own stage so we don't have to copy twice
# this stage starts from the first one and skips the last two
FROM base as source

WORKDIR /opt/node/app

COPY . .

## Stage 4 (testing)
# use this in automated CI
# it has prod and dev npm dependencies
# In 18.09 or older builder, this will always run
# In BuildKit, this will be skipped by default 

FROM source as test

# # NOTE: these apt dependencies are only needed
# # for testing. they shouldn't be in production
# USER root

RUN apt-get update -qq && apt-get install -qy \
  ca-certificates \
  bzip2 \
  curl \
  libfontconfig \
  --no-install-recommends

ENV NODE_ENV=development
ENV PATH=/opt/node/node_modules/.bin:$PATH

# this copies all dependencies (prod+dev)
COPY --from=dev /opt/node/node_modules /opt/node/node_modules

# # run unit tests as part of build
# RUN npm test
RUN npm audit

# # run integration testing with docker-compose later
# CMD ["npm", "run", "int-test"] 

## Stage 5 (default, production)
FROM test as pre-prod
RUN rm -rf ./tests && rm -rf ./node_modules

## Stage 6 (default, production)
# this will run by default if you don't include a target
# it has prod-only dependencies
# In BuildKit, this is skipped for dev and test stages
FROM source as prod
# COPY --from=pre-prod /node/app /node/app
HEALTHCHECK CMD curl http://127.0.0.1/ || exit 1
USER node
