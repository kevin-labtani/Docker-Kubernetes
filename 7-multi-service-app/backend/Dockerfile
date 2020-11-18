FROM node:14-alpine

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

ENV NONGODB_USERNAME root
ENV MONGODB_PASSWORD 123456

CMD ["npm", "start"]