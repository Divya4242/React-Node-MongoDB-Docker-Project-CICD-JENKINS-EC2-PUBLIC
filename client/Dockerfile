FROM node:18-alpine as builder
WORKDIR /frontend-docker-example/
EXPOSE 3000

COPY package*.json /frontend-docker-example/
RUN npm install
ENV REACT_APP_API_URL=http://localhost:6000/add
COPY public/ /frontend-docker-example/public
COPY src/  /frontend-docker-example/src
CMD ["npm", "start"]
# RUN npm run build

# FROM nginx:alpine
# COPY --from=builder /frontend-docker-example/build /usr/share/nginx/html
# EXPOSE 80
# CMD ["nginx", "-g", "daemon off;"]
