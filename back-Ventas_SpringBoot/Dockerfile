# Stage 1: Build
FROM maven:3.8.5-openjdk-17 AS build
WORKDIR /app

# El comodín **/ busca el archivo sin importar las subcarpetas internas
COPY **/pom.xml ./
COPY **/src ./src

RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
RUN addgroup -S spring && adduser -S spring -G spring
USER spring
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8082
ENTRYPOINT ["java", "-jar", "app.jar"]