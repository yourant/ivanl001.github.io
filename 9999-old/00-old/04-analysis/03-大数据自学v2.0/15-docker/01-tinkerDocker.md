







```shell
# 进入docker容器中
docker exec -it bash


# 重启docker命令
sudo docker restart tinker-java

# 进入docker
sudo docker exec -it tinker-java /bin/bash

# 关闭docker
sudo docker stop tinker-java
```





```dockerfile
# docker build . -t tinker-java:xxx --build-arg stage=test
#
# Build stage
#
FROM maven:3.6.2-jdk-8-slim AS build
COPY src /home/app/src
COPY pom.xml /home/app
ARG stage
RUN mvn -f /home/app/pom.xml -P ${stage} clean package
#
# Package stage
#
FROM openjdk:8-jre-slim
RUN ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime && \
    echo "America/Los_Angeles" > /etc/timezone
COPY --from=build /home/app/target/azazie-optimizer-java-0.0.1-SNAPSHOT.jar /usr/local/lib/tinker.jar
ENTRYPOINT ["java","-jar","/usr/local/lib/tinker.jar"]

# 如下可行
#FROM java:8
#ADD target/*.jar test.jar
#
#EXPOSE 8888
#ENTRYPOINT ["java", "-jar", "test.jar"]


#FROM openjdk:8-jdk-alpine
#
#VOLUME /tmp
#ARG JAR_FILE
#ADD ${JAR_FILE} app.jar
#
#ENTRYPOINT ["java","-jar","app.jar"]


#FROM openjdk:8-jre
#ENV SPRING_OUTPUT_ANSI_ENABLED=ALWAYS \
#    JAVA_OPTS=""
#
#ADD *.jar app.jar
#
#CMD echo "The application will start " && \
#    java ${JAVA_OPTS} -Djava.security.egd=file:/dev/./urandom -jar /app.jar
#EXPOSE 8080

```

