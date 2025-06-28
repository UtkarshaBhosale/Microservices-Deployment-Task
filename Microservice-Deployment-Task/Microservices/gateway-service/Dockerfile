FROM node:latest

WORKDIR /app

COPY package*.json ./
COPY . .

RUN npm install

EXPOSE 3004

CMD ["node", "app.js"]