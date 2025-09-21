# FROM ubuntu

# RUN apt-get update
# RUN apt-get install -y curl
# RUN curl -sL https://deb.nodesource.com/setup_18.x | bash -
# RUN apt-get upgrade -y
# RUN apt-get install -y nodejs

# COPY package.json package.json
# COPY package-lock.json package-lock.json
# COPY main.js main.js

# RUN npm install

# ENTRYPOINT [ "node", "main.js" ]


# professional way

# FROM node:18-alpine
# WORKDIR /app
# COPY package*.json ./
# RUN npm install
# COPY . .
# CMD [ "node", "main.js" ]   



# staging and production
FROM node:18-alpine as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build   
FROM node:18-alpine as production-stage
WORKDIR /app
COPY --from=build-stage /app .
EXPOSE 3000
CMD [ "node", "main.js" ]   
# CMD [ "npm", "start" ]