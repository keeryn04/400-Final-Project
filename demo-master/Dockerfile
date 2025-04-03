# Start with Jenkins base image
FROM jenkins/jenkins:alpine

# Switch to root user to install additional packages
USER root

# Install Docker and OpenRC
RUN apk add --no-cache docker openrc

# Now, switch to the Gradle base image to build your Gradle project
FROM gradle:7.6.1-jdk11 as gradle-build

# Set up application directory and work directory
RUN mkdir /home/app
WORKDIR /home/app
COPY . /home/app

# Run Gradle build task (adjust if needed)
CMD ["./gradlew", "apprun"]

# Now, we can optionally copy the built artifact into the Jenkins container
FROM jenkins/jenkins:alpine

USER root
RUN apk add --no-cache docker openrc
