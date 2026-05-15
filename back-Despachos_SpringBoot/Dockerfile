# Stage 1: Build (Compilación usando Maven y Java 17)
FROM maven:3.8.5-openjdk-17 AS build
WORKDIR /app

# Copiamos el archivo pom.xml que está dentro de la subcarpeta
COPY back-Despachos_SpringBoot/pom.xml .

# Copiamos el código fuente que está dentro de la subcarpeta
COPY back-Despachos_SpringBoot/src ./src

# Compilamos el proyecto saltándonos los tests para acelerar el pipeline
RUN mvn clean package -DskipTests

# Stage 2: Runtime (Imagen final ligera para ejecución)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Creamos un usuario de sistema sin privilegios por seguridad (Buenas prácticas)
RUN addgroup --system spring && adduser --system spring --ingroup spring
USER spring

# Copiamos el archivo JAR generado en la etapa anterior
COPY --from=build /app/target/*.jar app.jar

# Exponemos el puerto interno del contenedor (8081 según tu application.properties)
EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]