FROM node:20-alpine

WORKDIR /app

COPY package.json ./
RUN npm install --production

EXPOSE 4444

CMD ["node", "node_modules/y-webrtc/bin/server.js"]
