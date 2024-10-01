FROM eclipse-temurin:17-jdk-jammy
 
WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./

# Set the executable permissions for mvnw
RUN chmod +x mvnw

RUN ./mvnw dependency:resolve

COPY src ./src


CMD ["./mvnw", "spring-boot:run"]
