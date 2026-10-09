
FROM eclipse-temurin:21-jdk

WORKDIR /app

COPY lib/ ./lib/
COPY src/ ./src/

RUN mkdir -p out \
    && javac -cp "lib/*" -d out $(find src -name "*.java")

CMD ["sh", "-c", "java -cp 'out:lib/*' server.CauseConnectServer"]