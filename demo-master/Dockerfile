FROM gradle:8-jdk11

RUN mkdir /home/app
WORKDIR /home/app
COPY . /home/app

EXPOSE 8080

CMD ["./gradlew", "apprun"]