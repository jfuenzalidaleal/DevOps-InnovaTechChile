# Etapa 1: Construcción
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Etapa 2: Servidor Nginx para producción
FROM nginx:alpine
# Copiamos los archivos compilados de React
COPY --from=build /app/dist /usr/share/nginx/html

# ¡ESTA LÍNEA ES CRUCIAL! Asegura que pise el archivo de Nginx por defecto
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]