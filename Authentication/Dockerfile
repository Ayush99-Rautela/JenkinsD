FROM eclipse-temurin:17-jdk-alpine

WORKDIR /app

COPY target/*.jar auth.jar

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "auth.jar"]
