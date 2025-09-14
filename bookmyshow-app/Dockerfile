FROM node:latest

# Set working directory
WORKDIR /app
COPY package.json package-lock.json ./

# Force install a compatible PostCSS version to fix the issue
RUN npm install postcss@8.4.21 postcss-safe-parser@6.0.0 --legacy-peer-deps

#To Install dependencies
RUN npm i chokidar
RUN npm install

# To copy the entire project
COPY . .

EXPOSE 3000

#To set environment variable to prevent OpenSSL errors
ENV NODE_OPTIONS=--openssl-legacy-provider
ENV PORT=3000

# To start the application
CMD ["npm", "start"]