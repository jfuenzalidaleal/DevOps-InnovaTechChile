# Stage 1: Build
FROM maven:3.8.5-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

RUN addgroup --system spring && adduser --system spring --ingroup spring
USER spring

COPY --from=build /app/target/*.jar app.jar

# Exponemos el puerto interno (Spring por defecto usa el 8081 internamente)
EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]