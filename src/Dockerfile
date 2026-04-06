FROM eclipse-temurin:17
WORKDIR /src

RUN apt-get update && apt-get install -y wget

RUN wget https://repo1.maven.org/maven2/com/graphhopper/graphhopper-web/8.0/graphhopper-web-8.0.jar \
    -O graphhopper.jar

COPY . .

EXPOSE 8989
CMD ["java","-Xmx1400m","-XX:+UseSerialGC","-jar","graphhopper.jar","server","config.yml"]
