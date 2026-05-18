FROM node:14-alpine

WORKDIR /app

ENV HOST=0.0.0.0
ENV CHOKIDAR_USEPOLLING=true

COPY package*.json ./

RUN npm install --legacy-peer-deps
RUN npm install chokidar@3.5.3 --legacy-peer-deps

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
