FROM node:16-alpine

ENV VUE_APP_API_URL=http://localhost:8080

RUN mkdir /app

COPY . /app

WORKDIR /app

RUN yarn install

CMD ["yarn", "serve"]
