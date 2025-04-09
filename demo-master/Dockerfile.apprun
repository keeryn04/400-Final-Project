FROM gradle:7.6.1-jdk11

RUN mkdir /home/app
WORKDIR /home/app
COPY . /home/app

CMD ["./gradlew", "apprun"]