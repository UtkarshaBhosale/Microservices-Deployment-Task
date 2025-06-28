FROM node:latest

WORKDIR /app

COPY package*.json ./
COPY . .

RUN npm install

EXPOSE 3002

CMD ["node", "app.js"]