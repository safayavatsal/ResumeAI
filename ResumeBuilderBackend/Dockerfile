FROM node:18 AS build
WORKDIR /app
COPY . . 
#DEPENDENCY
RUN npm install
RUN npm install -g typescript
#dist
RUN tsc --build

EXPOSE 4292
CMD ["npm","run","start"]