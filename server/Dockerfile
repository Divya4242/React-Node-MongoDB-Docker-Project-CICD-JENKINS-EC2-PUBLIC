FROM node:18-alpine as builder
WORKDIR /backend-docker-examplee/
EXPOSE 5000

COPY package*.json /backend-docker-example/
RUN npm install
ENV FRONTEND_URL=http://localhost:4000
COPY app.js  /backend-docker-examplee/app.js
CMD [ "npm", "start" ]

#Production Stage
# FROM node:18-alpine
# WORKDIR /backend-docker-example/
# COPY --from=builder /backend-docker-example/ .
# EXPOSE 5000
# CMD [ "npm", "start" ]
