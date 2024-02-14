FROM node:18-alpine
WORKDIR /backend-docker-example/
EXPOSE 5000

COPY package*.json /backend-docker-example/
RUN npm install

COPY app.js  /backend-docker-example/app.js

CMD [ "npm", "start" ]