# Creating multi-stage build for production
# This reduces the file size of the docker image for prod
# Also adds ability to change base uri for api

# BASE
ARG NODE_VERSION=18

FROM node:${NODE_VERSION}-alpine as build
RUN apk update && apk add --no-cache build-base gcc autoconf automake zlib-dev libpng-dev vips-dev git > /dev/null 2>&1
ENV NODE_ENV=production

ARG STRAPI_VERSION=latest
RUN yarn global add @strapi/strapi@${STRAPI_VERSION}

RUN mkdir -p /build/app && chown 1000:1000 -R /build/app

WORKDIR /build
COPY package.json yarn.lock ./
RUN yarn global add node-gyp
RUN NODE_TLS_REJECT_UNAUTHORIZED=0 yarn config set network-timeout 600000 -g && yarn install --production
ENV PATH /build/node_modules/.bin:$PATH
WORKDIR /build/app
COPY ./server.js ./config/server.js
RUN ls /build/app/config
RUN yarn build

# Creating final production image
FROM node:${NODE_VERSION}-alpine
RUN apk add --no-cache vips-dev
ENV NODE_ENV=production
RUN mkdir -p /srv/app && chown 1000:1000 -R /srv/app
WORKDIR /srv
COPY --from=build /build/node_modules ./node_modules
WORKDIR /srv/app
COPY --from=build /build/app ./
ENV PATH /build/node_modules/.bin:$PATH

RUN chown -R node:node /srv/app
USER node
EXPOSE 1337
CMD ["yarn", "start"]
