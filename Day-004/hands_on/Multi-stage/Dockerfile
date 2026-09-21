FROM eclipse-temurin:17-jdk-alpine AS builder

WORKDIR /app

COPY src/Main.java .

RUN javac Main.java


FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

COPY --from=builder /app/Main.java .

CMD ["java", "Main"]

