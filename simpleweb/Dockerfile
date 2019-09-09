FROM node:alpine

# makes future commands relative to this folder
WORKDIR /usr/app

# Install some dependencies
COPY ./package.json ./
RUN npm install

# default command
CMD ["npm", "start"]