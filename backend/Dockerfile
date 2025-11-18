FROM node:18-alpine AS build

WORKDIR /app

COPY . .

EXPOSE 4000

RUN npm install

COPY package*.json ./

CMD [ "node", "server.js" ]