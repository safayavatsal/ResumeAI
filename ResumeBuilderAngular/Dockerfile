FROM node:18 AS build
WORKDIR /app
COPY . .
#DEPENDENCY
RUN npm install -g @angular/cli
RUN npm install -g angular-http-server
RUN npm install --force
# DIST FOLDER 
RUN npm run build
#FOR NGINX DEPLOYMENT
FROM nginx:alpine
COPY --from=build /app/dist/resumebuilder /usr/share/nginx/html
EXPOSE 80
CMD ["nginx","-g","daemon off;"]



