FROM maven:3.8.3-openjdk-17

WORKDIR /opt

COPY . .

RUN mvn clean package -DskipTests

EXPOSE 8080

CMD ["java", "-jar", "target/student-registration-backend-0.0.1-SNAPSHOT.jar"]