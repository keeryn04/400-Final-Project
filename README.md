# ENSF 400 Final Project
## The Team:
Cody Casselman, Evan Mann, Sam Haque, Keeryn Johnson

## How to Run:
1. Clone the repo
2. Change your working directory to `Demo-Master`
3. Pull the Docker image from Docker Hub using: `docker pull evanmann/ensf400-final-project:apprun`
4. Run the Docker image with: `docker run -it -p 8080:8080 evanmann/ensf400-final-project:apprun`

## Inside the Jenkins Dashboard:
1. Make a Multi-Pipeline
2. define as 'Pipeline script from SCM'
3. set repository URL as: https://github.com/SamHawq/400-Final-Project
4. set branch to main (NOT master)
5. Go to Manage Jenkins
6. Select Tools
7. scroll to gradle installations
8. add a gradle installation and name it 'gradlew'
9. select gradle version 7.6.1
10. find jdk installations
11. add an installation named 'jdk11'
12. check install automatically
13. select Extract *.zip/tar.gz
14. fill download URL for binary archive with 'https://github.com/adoptium/temurin11-binaries/releases/download/jdk-11.0.22+7/OpenJDK11U-jdk_x64_linux_hotspot_11.0.22_7.tar.gz'
15. press apply
16. press save
17. Go to Manage Jenkins again
18. Go to plugins
19. make sure to add and install Docker API Plugin, Docker Commons Plugin, Docker Pipeline, Docker pluging

any more plugins???

## Error checking
1. if you see cannot find gradlew, go back to your tools and make sure you apply and save
