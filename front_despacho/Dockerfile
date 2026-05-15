# Stage 1: Build - Usamos Node para compilar la aplicación React
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

  # Stage 2: Production - Servimos los archivos estáticos usando Nginx
FROM nginx:alpine
  # Copiamos la carpeta de salida de Vite (dist) al directorio público de Nginx
COPY --from=build /app/dist /usr/share/nginx/html

  # Copiamos una configuración personalizada para resolver las rutas de React Router
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]