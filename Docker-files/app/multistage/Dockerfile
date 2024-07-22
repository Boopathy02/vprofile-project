FROM openjdk:11 AS BUILD_IMAGE
RUN apt update && apt install maven git -y
RUN git clone -b cicd-jenkins https://github.com/Boopathy02/vprofile-project.git
WORKDIR vprofile-project/
RUN git checkout cicd-jenkins 
RUN mvn -s ./vprofile-project/settings.xml install -X

FROM tomcat:9-jre11

RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=BUILD_IMAGE vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
