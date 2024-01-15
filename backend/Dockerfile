FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

LABEL authors="lorenzo.grandi"

ENV MONGODB_USERNAME=root
ENV MONGODB_PASSWORD=secret

EXPOSE 8080

#node app.js carica il codice e starta l'applicazione, ma non è utile per live code reloading
#per questo uso nodemon e lo script definito nel package.json per lanciare l'applicazione
# CMD ["node","app.js"]
CMD ["npm","start"]