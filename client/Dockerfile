FROM node:18-alpine
WORKDIR /frontend-docker-example/
EXPOSE 3000

COPY package*.json /frontend-docker-example/
RUN npm install

COPY public/ /frontend-docker-example/public
COPY src/  /frontend-docker-example/src

CMD [ "npm", "start" ]