# Start with Jenkins base image
FROM jenkins/jenkins:alpine

USER root

RUN apk add --no-cache docker openrc

FROM gradle:7.6.1-jdk11 as gradle-build

RUN mkdir /home/app
WORKDIR /home/app
COPY . /home/app

CMD ["./gradlew", "apprun"]

FROM jenkins/jenkins:alpine

USER root
RUN apk add --no-cache docker openrc
