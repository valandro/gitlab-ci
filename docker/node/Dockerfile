FROM node:10.15-slim
# Create app directory
WORKDIR /usr/src/app
# Install app dependencies
COPY . ./
RUN npm install
# Bundle app source
CMD ["npm", "run", "prod"]